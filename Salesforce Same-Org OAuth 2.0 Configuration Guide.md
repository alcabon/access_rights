# Salesforce Same-Org OAuth 2.0 Configuration Guide

**The complete setup of External Client Apps using OAuth 2.0 Client Credentials flow within the same Salesforce org requires specific configuration across four integrated components: External Identity Provider, External Credential, Named Credential, and proper LWC integration patterns**. This same-org scenario provides secure server-to-server authentication without user interaction, enabling LWC components to access Salesforce APIs through declarative OAuth configuration with automatic token management.

Modern Salesforce External Credentials architecture eliminates the need for custom OAuth implementations by providing built-in token acquisition, refresh, and security management. The Client Credentials flow is particularly suited for same-org scenarios where automated processes need API access without user consent, such as data synchronization, backup operations, or system integrations.

## External Identity Provider configuration essentials

**Label and Name requirements** follow standard Salesforce naming conventions with the Label serving as the display name (e.g., "Salesforce Same Org OAuth") and the Name auto-generated as the API identifier. The Authentication Protocol must be set to OAuth 2.0, with **Authentication Flow Type configured as Client Credentials** for server-to-server same-org scenarios rather than Authorization Code, which requires user interaction.

**Client ID and Client Secret setup** requires the Consumer Key and Consumer Secret from a Connected App configured in the target org. The Connected App must have "Enable Client Credentials Flow" checked and a designated "Run As" user specified with appropriate API permissions. This Run As user context determines all data access for the OAuth flow.

The **"Pass client credentials in request body" setting** should typically remain unchecked for Salesforce-to-Salesforce OAuth, as the platform expects credentials in the Authorization header per OAuth 2.0 standards. Only enable this setting if the target system specifically requires credentials in the request body.

**Endpoint URLs for same-org setup** require My Domain configuration for consistency and security. Use the format `https://[your-domain].my.salesforce.com/services/oauth2/authorize` for the Authorize Endpoint URL and `https://[your-domain].my.salesforce.com/services/oauth2/token` for the Token Endpoint URL. Avoid using generic `login.salesforce.com` or `test.salesforce.com` endpoints, as these can cause cross-domain authentication issues.

**User Info Endpoint URL is not required** for Client Credentials flow since there's no user context involved. Leave this field blank unless your specific integration requires user information. **PKCE Extension should not be enabled** for Client Credentials flow, as PKCE is designed for Authorization Code flows with enhanced security requirements.

## External Credential configuration specifications

**Authentication Protocol must be OAuth 2.0** with **Authentication Flow Type set to "Client Credentials with Client Secret Flow"**. The Label and Name fields follow the same conventions as External Identity Providers, with descriptive labels like "Same Org API Access" and auto-generated API names.

**Scope configuration for same-org access** typically includes the `api` scope for REST API access and `refresh_token` if token refresh capabilities are needed. However, Client Credentials flow doesn't support refresh tokens, so access tokens must be re-requested upon expiration. Additional scopes should follow the principle of least privilege.

The **Identity Provider URL should point to the same-org token endpoint**: `https://[your-domain].my.salesforce.com/services/oauth2/token`. This URL must match the Token Endpoint URL configured in the linked External Identity Provider.

**"Pass client credentials in request body"** setting mirrors the External Identity Provider configuration and should remain unchecked for standard Salesforce OAuth implementations. The **Additional Status Codes for Token Refresh** should include 401 and 403 status codes to trigger automatic token refresh attempts when authorization fails.

**Principal configuration** requires a Parameter Name (descriptive identifier), Client ID (Consumer Key from Connected App), and Client Secret (Consumer Secret). These credentials are encrypted after entry and not visible in the interface, requiring re-entry if modifications are needed.

## Same-org OAuth security and endpoint considerations

**Client Credentials flow provides enhanced security** over username-password authentication by using time-limited access tokens instead of persistent credentials. All API calls execute in the context of the designated Run As user, making permission management critical for security.

**Best practices for Client ID and Client Secret management** include treating secrets with password-level security, implementing regular rotation schedules, using secure storage solutions like Named Credentials or managed packages, and maintaining separate credentials for development, staging, and production environments.

**Endpoint URL formats must use My Domain** for consistency: `https://[your-domain].my.salesforce.com/services/oauth2/token` for token requests. My Domain ensures proper cross-org isolation and prevents authentication bleeding between orgs. The **standard Salesforce OAuth endpoints** support both production (`login.salesforce.com`) and sandbox (`test.salesforce.com`) environments, but My Domain URLs are recommended for same-org scenarios.

**Security considerations specific to same-org scenarios** include creating dedicated integration users with API-Only permissions, using Permission Sets rather than Profiles for granular access control, implementing IP restrictions where applicable, and maintaining comprehensive audit trails through Login History and API Usage monitoring.

**Common security pitfalls** include using overprivileged integration users (use dedicated users with minimal permissions), hardcoding client secrets in source code (use Named Credentials or secure storage), selecting inappropriate OAuth flows (Client Credentials for server-to-server, not user-context operations), and insufficient token management (implement proper expiration handling).

## Named Credentials integration with LWC implementation

**Named Credentials serve as the bridge** between External Credentials and LWC callouts, providing a declarative way to handle OAuth authentication without custom code. The configuration requires linking the Named Credential to the External Credential and setting the base URL for API calls.

**LWC components cannot make direct API calls** due to security policies that disable API access for Lightning component sessions. Instead, the pattern is: **LWC → Apex Controller → Named Credential → External API**. The Apex controller uses the `callout:Named_Credential_Name` syntax to leverage automatic OAuth token management.

Critical **Apex controller implementation** requires checking authentication status before making callouts using `ConnectApi.NamedCredentials.getCredential()` and handling scenarios where users need to authenticate. The controller should implement retry logic for token refresh scenarios and proper error handling for various HTTP response codes.

**CORS considerations for same-org callouts** require configuring the CORS allowlist with `https://*.my.salesforce.com` as the origin URL pattern. However, CORS only affects browser-based requests and doesn't impact server-side Apex callouts. Named Credentials automatically handle OAuth tokens for server-side requests.

**Permission configuration is critical** for user access to External Credentials. Users need "External Credential Principal Access" assigned via Permission Sets and CRUD permissions on the "User External Credential" object. Without these permissions, authentication will fail with credential access errors.

## Step-by-step setup guidance

### Step 1: External Client App creation (Connected App)
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

**After saving, configure the Run As user:**
1. Edit the newly created External Client App
2. Navigate to the **Client Credentials Flow** section
3. **Run As**: Select a dedicated integration user with API permissions
4. Save the configuration

**Critical post-creation step**: Note the **Consumer Key** and **Consumer Secret** - these become your Client ID and Client Secret for the External Identity Provider configuration.

### Step 2: External Identity Provider setup
In Setup → Security → Named Credentials → External Auth Identity Providers → New, configure:
- Label: Descriptive name for the provider
- Authentication Protocol: OAuth 2.0
- Authentication Flow Type: Client Credentials
- Authorize Endpoint URL: `https://[your-domain].my.salesforce.com/services/oauth2/authorize`
- Token Endpoint URL: `https://[your-domain].my.salesforce.com/services/oauth2/token`
- Client ID: Consumer Key from Connected App
- Client Secret: Consumer Secret from Connected App
- Leave User Info Endpoint URL blank

### Step 3: External Credential configuration
Create External Credential linking to the Identity Provider:
- Authentication Protocol: OAuth 2.0
- Authentication Flow Type: Client Credentials with Client Secret Flow
- Scope: `api` and additional scopes as needed
- Identity Provider URL: Token endpoint from Step 2
- Configure Principal with Parameter Name, Client ID, and Client Secret

### Step 4: Named Credential setup
Create Named Credential referencing the External Credential:
- URL: `https://[your-domain].my.salesforce.com`
- External Credential: Select the credential from Step 3
- Generate Authorization Header: Enabled
- Configure additional settings as needed

### Step 5: Permission configuration
Create or modify Permission Sets to grant:
- External Credential Principal Access for your External Credential
- User External Credential object permissions (Create, Read, Edit, Delete)
- Assign Permission Sets to users who will use the integration

## Configuration issues and troubleshooting

**The most common configuration error is "no client credentials user enabled"**, which occurs when the Connected App lacks a properly configured Run As user. The solution requires editing the Connected App's policies (not the initial setup) and specifying a user with API permissions in the Client Credentials Flow section.

**Endpoint URL errors** typically manifest as `unsupported_grant_type` or `request not supported on this domain` messages. These errors indicate incorrect endpoint URLs—use My Domain format `https://[your-domain].my.salesforce.com/services/oauth2/token` rather than generic Salesforce login URLs.

**Permission-related failures** often show as "couldn't access the endpoint" errors, indicating users lack access to External Credential Principals or User External Credential object permissions. The resolution involves assigning proper Permission Sets with External Credential access and ensuring users have CRUD permissions on User External Credentials.

**LWC integration problems** frequently involve attempts to use Lightning session IDs for API calls, which fails due to security policies. The error "sessions created by Lightning components aren't enabled for API access" requires using Named Credentials exclusively for API calls from LWC-invoked Apex controllers.

**Token refresh issues** in Client Credentials flow need careful handling since this flow doesn't support refresh tokens. Instead, implement logic to re-request access tokens when they expire, typically by catching 401/403 errors and initiating new token requests.

## Conclusion

Same-org OAuth 2.0 Client Credentials configuration provides a robust, secure foundation for server-to-server integrations within Salesforce environments. The declarative approach through External Credentials eliminates custom OAuth implementations while providing enterprise-grade security through automated token management, encrypted credential storage, and granular permission controls.

Success depends on proper configuration of the integrated components—External Identity Provider, External Credential, Named Credential, and Permission Sets—along with following security best practices for integration users and credential management. The LWC integration pattern enables modern Lightning components to securely access Salesforce APIs through server-side controllers, maintaining security policies while providing seamless user experiences.
