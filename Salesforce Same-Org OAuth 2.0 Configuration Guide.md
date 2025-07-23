
# Salesforce Same-Org OAuth 2.0 Configuration Guide

**The complete setup of External Client Apps using OAuth 2.0 Client Credentials flow within the same Salesforce org requires specific configuration across three integrated components: External Client App, External Credential, and Named Credential, with proper LWC integration patterns**. This same-org scenario provides secure server-to-server authentication without user interaction, enabling LWC components to access Salesforce APIs through declarative OAuth configuration with automatic token management.

Modern Salesforce External Credentials architecture eliminates the need for custom OAuth implementations by providing built-in token acquisition, refresh, and security management. The Client Credentials flow is particularly suited for same-org scenarios where automated processes need API access without user consent, such as data synchronization, backup operations, or system integrations.

## External Client App configuration essentials

**External Client App Name and API Name** follow standard Salesforce naming conventions with the Name serving as the display name (e.g., "Same Org API Integration") and the API Name auto-generated as the identifier. The **Distribution State should be "Local"** for same-org usage, avoiding unnecessary external distribution complexity.

**OAuth Settings require enabling OAuth** with a callback URL placeholder `https://[your-domain].my.salesforce.com/services/oauth2/callback`. While Client Credentials flow doesn't use callbacks, Salesforce requires this field for OAuth-enabled apps.

**OAuth Scopes selection** should follow the principle of least privilege. Essential scopes include:
- **Manage user data via APIs (api)** - Required for REST API access
- **Perform requests at any time (refresh_token, offline_access)** - For token management
- Additional scopes only as specifically needed for your integration

**Flow Enablement requires "Enable Client Credentials Flow"** to be checked. Other flows (Authorization Code, Device Flow, JWT Bearer, Token Exchange) should remain unchecked unless specifically required, as they add unnecessary complexity and potential security vectors.

**Security settings should include "Require secret for Web Server Flow"** for enhanced security. PKCE extension is not applicable to Client Credentials flow and should remain unchecked. Refresh Token Rotation is also not applicable since Client Credentials flow doesn't use refresh tokens.

**Critical post-creation step**: After saving the External Client App, edit it again to configure the **"Run As" user** in the Client Credentials Flow section. This user's permissions determine all API access for the OAuth flow.

## External Credential configuration specifications

**For same-org OAuth 2.0 Client Credentials, External Credentials handle OAuth directly without needing External Identity Providers**. This modern approach simplifies configuration and reduces potential failure points.

**Authentication Protocol must be OAuth 2.0** with **Authentication Flow Type set to "Client Credentials with Client Secret Flow"**. The Label and Name fields follow standard conventions, with descriptive labels like "Same Org OAuth Client Credentials" and auto-generated API names.

**Scope configuration** typically includes the `api` scope for REST API access. Additional scopes should match those selected in the External Client App configuration. The scope field accepts space-separated values for multiple scopes.

**Identity Provider URL should point to the same-org token endpoint**: `https://[your-domain].my.salesforce.com/services/oauth2/token`. This URL handles token acquisition for the Client Credentials flow.

**"Pass client credentials in request body"** should remain unchecked for standard Salesforce OAuth implementations, as credentials are passed in the Authorization header per OAuth 2.0 standards.

**Additional Status Codes for Token Refresh** should include `401,403` to trigger automatic token refresh attempts when authorization fails.

**Principal configuration** requires:
- **Parameter Name**: Descriptive identifier (e.g., "ClientCredentialsPrincipal")
- **Client ID**: Consumer Key from External Client App
- **Client Secret**: Consumer Secret from External Client App

These credentials are encrypted after entry and require re-entry if modifications are needed.

## Same-org OAuth security considerations

**Client Credentials flow provides enhanced security** over username-password authentication by using time-limited access tokens instead of persistent credentials. All API calls execute in the context of the designated Run As user, making permission management critical for security.

**Best practices for Client ID and Client Secret management** include treating secrets with password-level security, implementing regular rotation schedules, using secure storage solutions like Named Credentials, and maintaining separate credentials for development, staging, and production environments.

**Endpoint URL formats must use My Domain** for consistency: `https://[your-domain].my.salesforce.com/services/oauth2/token` for token requests. My Domain ensures proper cross-org isolation and prevents authentication bleeding between orgs.

**Security considerations specific to same-org scenarios** include creating dedicated integration users with API-Only permissions, using Permission Sets rather than Profiles for granular access control, implementing IP restrictions where applicable, and maintaining comprehensive audit trails through Login History and API Usage monitoring.

**Common security pitfalls** include using overprivileged integration users (use dedicated users with minimal permissions), hardcoding client secrets in source code (use Named Credentials or secure storage), selecting inappropriate OAuth flows (Client Credentials for server-to-server, not user-context operations), and insufficient token management (implement proper expiration handling).

## Named Credentials integration with LWC implementation

**Named Credentials serve as the bridge** between External Credentials and LWC callouts, providing a declarative way to handle OAuth authentication without custom code. The configuration requires linking the Named Credential to the External Credential and setting the base URL for API calls.

**LWC components cannot make direct API calls** due to security policies that disable API access for Lightning component sessions. Instead, the pattern is: **LWC → Apex Controller → Named Credential → External API**. The Apex controller uses the `callout:Named_Credential_Name` syntax to leverage automatic OAuth token management.

**Critical Apex controller implementation** requires checking authentication status before making callouts and handling scenarios where authentication might fail. The controller should implement retry logic for token refresh scenarios and proper error handling for various HTTP response codes.

**CORS considerations for same-org callouts** require configuring the CORS allowlist with `https://*.my.salesforce.com` as the origin URL pattern. However, CORS only affects browser-based requests and doesn't impact server-side Apex callouts. Named Credentials automatically handle OAuth tokens for server-side requests.

**Permission configuration is mandatory** for External Credentials with principals. Users **must have** "External Credential Principal Access" assigned via Permission Sets and CRUD permissions on the "User External Credential" object. **Without these permissions, the External Credential will not function at all** - authentication will fail with credential access errors even if all other configuration is correct.

## Step-by-step setup guidance

### Step 1: External Client App creation
Navigate to Setup → Apps → External Client App Manager → New External Client App and configure:

**External Client App Manager**
- **External Client App Name**: `Same Org API Integration` (or descriptive name)
- **API Name**: Auto-generated (e.g., `Same_Org_API_Integration`)
- **Contact Email**: Your admin email address
- **Distribution State**: `Local` (for same-org usage)
- Contact Phone: Optional
- Info URL: Optional
- Logo/Icon URLs: Optional (can use sample logos)
- Description: Brief description of the integration purpose

**App Settings**
- **Enable OAuth**: ✅ Checked (required)
- **Callback URL**: `https://[your-domain].my.salesforce.com/services/oauth2/callback` (placeholder for Client Credentials)

**OAuth Scopes** (Select only what you need):
- ✅ **Manage user data via APIs (api)** - Essential for API access
- ✅ **Perform requests at any time (refresh_token, offline_access)** - For token management
- Optional: **Access the identity URL service (id, profile, email, address, phone)** - If user info needed
- Optional: **Full access (full)** - Only if broad access required (not recommended)

**Flow Enablement**
- ✅ **Enable Client Credentials Flow** - Critical for your use case
- ⬜ Enable Authorization Code and Credentials Flow (not needed for server-to-server)
- ⬜ Enable Device Flow (not needed)
- ⬜ Enable JWT Bearer Flow (not needed unless using JWT)
- ⬜ Enable Token Exchange Flow (not needed)

**Security Settings**
- ✅ **Require secret for Web Server Flow** (recommended)
- ⬜ Require secret for Refresh Token Flow (Client Credentials doesn't use refresh tokens)
- ⬜ Require Proof Key for Code Exchange (PKCE) (not applicable to Client Credentials)
- ⬜ Enable Refresh Token Rotation (not applicable)
- ⬜ Issue JSON Web Token (JWT)-based access tokens (unless specifically needed)

**After saving, configure the Run As user with Tooling API permissions:**
1. Edit the newly created External Client App
2. Navigate to the **Client Credentials Flow** section
3. **Run As**: Select a dedicated integration user with **Tooling API permissions**:
   - ✅ API Enabled
   - ✅ View All Data (required for most Tooling objects)
   - ✅ Author Apex (for Apex-related Tooling objects)
   - ✅ Customize Application (for component-related Tooling objects)
   - ✅ View Setup and Configuration (for setup metadata)
4. Save the configuration

**Critical post-creation step**: Note the **Consumer Key** and **Consumer Secret** - these become your Client ID and Client Secret for the External Credential configuration.

### Step 2: External Credential configuration (Modern Approach)
**For same-org OAuth 2.0 Client Credentials, you only need External Credentials - not External Identity Providers.**

Navigate to Setup → Security → Named Credentials → External Credentials → New:

**Basic Information:**
- **Label**: `Same Org OAuth Client Credentials`
- **Name**: Auto-generated API name
- **Authentication Protocol**: `OAuth 2.0`
- **Authentication Flow Type**: `Client Credentials with Client Secret Flow`

**Configuration:**
- **Scope**: `api` (and additional scopes as needed)
- **Identity Provider URL**: `https://[your-domain].my.salesforce.com/services/oauth2/token`
- **Pass client credentials in request body**: ⬜ Unchecked (standard OAuth header)

**Callout Options:**
- **Additional Status Codes for Token Refresh**: `401,403`

**Principal Configuration:**
- **Parameter Name**: `ClientCredentialsPrincipal` (descriptive name)
- **Client ID**: Consumer Key from External Client App
- **Client Secret**: Consumer Secret from External Client App

**Note**: External Identity Providers are primarily for Authorization Code flows where users need to authenticate. For Client Credentials (server-to-server), External Credentials handle everything directly.

### Step 3: Named Credential setup
Create Named Credential referencing the External Credential:
- **Label**: `Same Org API Access`
- **Name**: Auto-generated API name
- **URL**: `https://[your-domain].my.salesforce.com`
- **External Credential**: Select the credential from Step 2
- **Generate Authorization Header**: ✅ Enabled
- **Allow Merge Fields in HTTP Header**: Optional based on needs
- **Allow Merge Fields in HTTP Body**: Optional based on needs

### Step 4: Permission configuration (MANDATORY)
**Permission Sets are required for External Credentials to function**. Additionally, **Tooling API objects require specific system permissions** beyond basic External Credential access.

**Basic External Credential Permissions:**
- ✅ **External Credential Principal Access** for your specific External Credential (mandatory)
- ✅ **User External Credential** object permissions:
  - Create: ✅ Required
  - Read: ✅ Required  
  - Edit: ✅ Required
  - Delete: ✅ Required

**Additional Tooling API System Permissions (Required for metadata access):**
For accessing Tooling API objects, the **Run As user** in your External Client App must have these additional permissions:

- ✅ **View All Data** - Required for most Tooling API objects
- ✅ **Author Apex** - Required for ApexClassMember, ApexTriggerMember, MetadataContainer
- ✅ **Customize Application** - Required for ApexComponentMember, ApexPageMember, ContainerAsyncRequest, MetadataContainer
- ✅ **View Setup and Configuration** - Required for OperationLog and other setup metadata

**Specific Tooling API Object Requirements:**
- **ApexClassMember, ApexTriggerMember**: Requires View All Data + Author Apex
- **ApexComponentMember, ApexPageMember, ContainerAsyncRequest**: Requires View All Data + Customize Application  
- **MetadataContainer**: Requires View All Data + (Author Apex OR Customize Application)
- **OperationLog**: Requires View Setup and Configuration
- **Most other Tooling objects**: Require View All Data minimum

**Assignment:**
- **Assign Permission Sets to ALL users** who will use integrations that leverage this External Credential
- **Configure the "Run As" user** in the External Client App with appropriate Tooling API permissions
- **System Administrators** also need these permissions explicitly assigned
- **Without proper permission assignment, the External Credential will not work**

**Verification:**
- Test authentication by having a user with permissions attempt to use the Named Credential
- Test Tooling API access: `GET /services/data/v58.0/tooling/sobjects/ApexClass/` 
- Check Setup → External Credentials → [Your Credential] → Principals to verify user access

## Named Credentials 404 "Not Found" errors (when credential resolves correctly)

**When Named Credentials appear correctly in debug logs but return 404 errors**, the issue is typically with the endpoint construction or target API availability, not the authentication itself. The Named Credential authentication is working, but the final URL being called doesn't exist.

### Primary causes of 404 errors with functional Named Credentials:

**1. API Version mismatch**
The most common cause is API version incompatibility between environments. A user experienced 404 errors when using API version v52.0 in production while the org only supported up to v50.0, and the issue was resolved by changing to the correct API version.

**2. Incorrect endpoint path construction**
Even with proper authentication, the URL path after the Named Credential might be wrong:
- **Wrong**: `callout:My_Credential/services/data/v58.0/` (if the Named Credential URL already includes `/services/data/`)
- **Correct**: `callout:My_Credential/sobjects/Account/` (when base URL is properly configured)

**3. Target API endpoint doesn't exist**
The endpoint `/services/data/v45.0` does not accept PUT requests for creating sObjects, and appending incorrect fragments to Named Credential URLs will produce invalid endpoints leading to 404 errors.

**4. Case sensitivity in Named Credential names**
- **Wrong**: `callout:my_credential` 
- **Correct**: `callout:My_Credential` (exact case match required)

**5. My Domain vs standard URL conflicts**
For same-org scenarios, mixing My Domain and standard Salesforce URLs can cause routing issues:
- Named Credential URL: `https://mydomain.my.salesforce.com`
- Actual request: Trying to access endpoints that don't exist on the specific domain

**6. HTTP method not supported by endpoint**
The target endpoint exists but doesn't support the HTTP method being used (GET, POST, PUT, PATCH, DELETE).

### Troubleshooting steps when Named Credential resolves but returns 404:

**1. Verify the complete constructed URL**
- Add debug statements to log the full endpoint: `System.debug('Full URL: ' + req.getEndpoint());`
- Test the same URL manually in Postman or Workbench

**2. Check API version compatibility**
- Verify the target org supports the API version in your endpoint
- Use `/services/data/` to list available versions

**3. Validate HTTP method support**
- Ensure the endpoint supports your HTTP method
- Check API documentation for supported operations

**4. Test endpoint segments individually**
- Start with base Named Credential URL
- Add path segments incrementally to identify where the 404 occurs

**5. Check for URL encoding issues**
- Ensure special characters in the URL are properly encoded
- Verify query parameters are correctly formatted

### Common 404 patterns in same-org scenarios:

**Tooling API 404s:**
- Using `/services/data/` instead of `/services/data/v58.0/tooling/`
- Accessing Tooling objects without proper permissions (returns 404 instead of 403)

**REST API 404s:**
- Incorrect SObject API names
- Invalid record IDs
- Using wrong API version for the feature being accessed

The key insight is that **Named Credential resolution success in logs only confirms authentication worked** - the 404 indicates the constructed endpoint URL doesn't exist on the target server.

**The most common configuration error is "no client credentials user enabled"**, which occurs when the External Client App lacks a properly configured Run As user. The solution requires editing the External Client App's policies and specifying a user with API permissions in the Client Credentials Flow section.

**Endpoint URL errors** typically manifest as `unsupported_grant_type` or `request not supported on this domain` messages. These errors indicate incorrect endpoint URLs—use My Domain format `https://[your-domain].my.salesforce.com/services/oauth2/token` rather than generic Salesforce login URLs.

**Permission-related failures** are the **most common cause of External Credential authentication failures**. Users without "External Credential Principal Access" or "User External Credential" object permissions will see "couldn't access the endpoint" or "authentication failed" errors. **This is mandatory - not optional**. The resolution requires:

1. Creating a Permission Set with External Credential Principal Access for your specific credential
2. Granting full CRUD permissions on the User External Credential object
3. Assigning the Permission Set to ALL users who will use the integration
4. **Even System Administrators need explicit permission assignment** - admin privileges don't automatically grant External Credential access

**LWC integration problems** frequently involve attempts to use Lightning session IDs for API calls, which fails due to security policies. The error "sessions created by Lightning components aren't enabled for API access" requires using Named Credentials exclusively for API calls from LWC-invoked Apex controllers.

**Token refresh issues** in Client Credentials flow need careful handling since this flow doesn't support refresh tokens. Instead, implement logic to re-request access tokens when they expire, typically by catching 401/403 errors and initiating new token requests.

## LWC Integration Examples

### Example 1: Standard REST API (Account query)

**Apex Controller:**
```apex
@AuraEnabled
public static String getAccounts() {
    try {
        HttpRequest req = new HttpRequest();
        req.setEndpoint('callout:Same_Org_API_Access/services/data/v58.0/sobjects/Account');
        req.setMethod('GET');
        
        Http http = new Http();
        HttpResponse res = http.send(req);
        
        if (res.getStatusCode() == 200) {
            return res.getBody();
        } else {
            throw new AuraHandledException('API call failed: ' + res.getStatusCode() + ' - ' + res.getBody());
        }
    } catch (Exception e) {
        throw new AuraHandledException('Error: ' + e.getMessage());
    }
}
```

### Example 2: Tooling API (InstalledPackageVersion query)

**Apex Controller for Tooling API:**
```apex
@AuraEnabled
public static String getInstalledPackages() {
    try {
        HttpRequest req = new HttpRequest();
        // Note: Tooling API requires /tooling/ in the path
        req.setEndpoint('callout:Same_Org_API_Access/services/data/v58.0/tooling/query/?q=SELECT+Id,SubscriberPackageVersionId,SubscriberPackage.Name,SubscriberPackage.NamespacePrefix,SubscriberPackageVersion.Name,SubscriberPackageVersion.MajorVersion,SubscriberPackageVersion.MinorVersion,SubscriberPackageVersion.PatchVersion,SubscriberPackageVersion.BuildNumber+FROM+InstalledSubscriberPackage');
        req.setMethod('GET');
        
        Http http = new Http();
        HttpResponse res = http.send(req);
        
        System.debug('Tooling API Response Status: ' + res.getStatusCode());
        System.debug('Tooling API Response Body: ' + res.getBody());
        
        if (res.getStatusCode() == 200) {
            return res.getBody();
        } else {
            throw new AuraHandledException('Tooling API call failed: ' + res.getStatusCode() + ' - ' + res.getBody());
        }
    } catch (Exception e) {
        throw new AuraHandledException('Tooling API Error: ' + e.getMessage());
    }
}

@AuraEnabled
public static String getSpecificPackageVersion(String packageName) {
    try {
        HttpRequest req = new HttpRequest();
        // Query for a specific package by name
        String query = 'SELECT Id,SubscriberPackageVersionId,SubscriberPackage.Name,SubscriberPackage.NamespacePrefix,SubscriberPackageVersion.Name,SubscriberPackageVersion.MajorVersion,SubscriberPackageVersion.MinorVersion,SubscriberPackageVersion.PatchVersion,SubscriberPackageVersion.BuildNumber FROM InstalledSubscriberPackage WHERE SubscriberPackage.Name = \'' + String.escapeSingleQuotes(packageName) + '\'';
        String encodedQuery = EncodingUtil.urlEncode(query, 'UTF-8');
        
        req.setEndpoint('callout:Same_Org_API_Access/services/data/v58.0/tooling/query/?q=' + encodedQuery);
        req.setMethod('GET');
        
        Http http = new Http();
        HttpResponse res = http.send(req);
        
        if (res.getStatusCode() == 200) {
            return res.getBody();
        } else {
            throw new AuraHandledException('Package query failed: ' + res.getStatusCode() + ' - ' + res.getBody());
        }
    } catch (Exception e) {
        throw new AuraHandledException('Package query error: ' + e.getMessage());
    }
}
```

**LWC Component JavaScript:**
```javascript
import { LightningElement, track } from 'lwc';
import getAccounts from '@salesforce/apex/MyController.getAccounts';
import getInstalledPackages from '@salesforce/apex/MyController.getInstalledPackages';
import getSpecificPackageVersion from '@salesforce/apex/MyController.getSpecificPackageVersion';

export default class MyComponent extends LightningElement {
    @track accountData;
    @track packageData;
    @track specificPackageData;
    @track isLoading = false;

    async handleAccountCall() {
        this.isLoading = true;
        try {
            const result = await getAccounts();
            this.accountData = JSON.parse(result);
            console.log('Account Data:', this.accountData);
        } catch (error) {
            console.error('Account Error:', error.body.message);
        } finally {
            this.isLoading = false;
        }
    }

    async handlePackageCall() {
        this.isLoading = true;
        try {
            const result = await getInstalledPackages();
            this.packageData = JSON.parse(result);
            console.log('Package Data:', this.packageData);
        } catch (error) {
            console.error('Package Error:', error.body.message);
        } finally {
            this.isLoading = false;
        }
    }

    async handleSpecificPackageCall() {
        this.isLoading = true;
        try {
            const result = await getSpecificPackageVersion({ packageName: 'My Package Name' });
            this.specificPackageData = JSON.parse(result);
            console.log('Specific Package Data:', this.specificPackageData);
        } catch (error) {
            console.error('Specific Package Error:', error.body.message);
        } finally {
            this.isLoading = false;
        }
    }
}
```

**LWC Component HTML:**
```html
<template>
    <lightning-card title="API Integration Examples" icon-name="standard:integration">
        <div class="slds-p-horizontal_medium">
            <div class="slds-grid slds-wrap slds-gutters">
                <div class="slds-col slds-size_1-of-1 slds-medium-size_1-of-3">
                    <lightning-button 
                        label="Get Accounts" 
                        onclick={handleAccountCall}
                        disabled={isLoading}
                        variant="brand">
                    </lightning-button>
                </div>
                <div class="slds-col slds-size_1-of-1 slds-medium-size_1-of-3">
                    <lightning-button 
                        label="Get Installed Packages" 
                        onclick={handlePackageCall}
                        disabled={isLoading}
                        variant="brand">
                    </lightning-button>
                </div>
                <div class="slds-col slds-size_1-of-1 slds-medium-size_1-of-3">
                    <lightning-button 
                        label="Get Specific Package" 
                        onclick={handleSpecificPackageCall}
                        disabled={isLoading}
                        variant="brand">
                    </lightning-button>
                </div>
            </div>

            <template if:true={isLoading}>
                <lightning-spinner alternative-text="Loading..."></lightning-spinner>
            </template>

            <template if:true={accountData}>
                <div class="slds-m-top_medium">
                    <h3>Account Data:</h3>
                    <pre>{accountDataString}</pre>
                </div>
            </template>

            <template if:true={packageData}>
                <div class="slds-m-top_medium">
                    <h3>Installed Packages:</h3>
                    <pre>{packageDataString}</pre>
                </div>
            </template>

            <template if:true={specificPackageData}>
                <div class="slds-m-top_medium">
                    <h3>Specific Package Info:</h3>
                    <pre>{specificPackageDataString}</pre>
                </div>
            </template>
        </div>
    </lightning-card>
</template>
```

### Key Differences for Tooling API:

**1. URL Path Requirements:**
- **Standard API**: `callout:Named_Cred/services/data/v58.0/sobjects/Account`
- **Tooling API**: `callout:Named_Cred/services/data/v58.0/tooling/query/?q=...`

**2. Required Permissions for Run As User:**
- **Standard API**: API Enabled + appropriate object permissions
- **Tooling API**: API Enabled + **View All Data** + **Author Apex** (for some objects) + **Customize Application** (for others)

**3. Query Structure:**
- **Standard API**: REST endpoints (`/sobjects/Account/`)
- **Tooling API**: SOQL queries via `/tooling/query/?q=SELECT...`

**4. Common 404 Causes:**
- Missing `/tooling/` in the path
- Using wrong API version
- Insufficient permissions (returns 404 instead of 403 for some Tooling objects)
- URL encoding issues in SOQL queries

## Conclusion

Same-org OAuth 2.0 Client Credentials configuration provides a robust, secure foundation for server-to-server integrations within Salesforce environments. The modern approach using External Client Apps and External Credentials eliminates complex External Identity Provider configurations while providing enterprise-grade security through automated token management, encrypted credential storage, and granular permission controls.

Success depends on proper configuration of the three core components—External Client App, External Credential, and Named Credential—along with following security best practices for integration users and credential management. The LWC integration pattern enables modern Lightning components to securely access Salesforce APIs through server-side controllers, maintaining security policies while providing seamless user experiences.
