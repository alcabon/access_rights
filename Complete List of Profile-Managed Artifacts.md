# Complete List of Profile-Managed Artifacts

## ✅ **CORE METADATA TYPES (Currently Handled)**

### **Object & Field Permissions**
- `CustomObject` - Custom object CRUD permissions
- `CustomField` - Field-level security (FLS) permissions  
- `RecordType` - Record type visibility and assignment

### **Code & Logic Permissions**
- `ApexClass` - Apex class execution permissions
- `ApexPage` - Visualforce page access permissions
- `Flow` - Flow and Process Builder execution permissions

### **UI & Application Permissions**
- `CustomApplication` - Application visibility permissions
- `CustomTab` - Tab visibility permissions
- `CustomPermission` - Custom permission access

### **Data & Integration Permissions**
- `ExternalDataSource` - External data source access
- `CustomMetadata` - Custom metadata type permissions

### **Security Artifacts**
- `Profile` - The profiles themselves
- `PermissionSet` - Permission sets for assignment

---

## ⚠️ **ADDITIONAL METADATA TYPES TO CONSIDER**

### **Missing but Important for Complete Profiles:**

#### **1. System Permissions**
- `UserPermission` - System-level permissions (Modify All Data, View All Data, etc.)

#### **2. Object Settings**
- `StandardObject` - Standard object permissions (Account, Contact, etc.)
- `ObjectSettings` - Per-object settings and permissions

#### **3. UI Components**
- `Layout` - Page layout assignments
- `CustomLabel` - Custom label access (rarely restricted)

#### **4. Reports & Dashboards**
- `Report` - Report access permissions
- `Dashboard` - Dashboard access permissions
- `ReportType` - Custom report type access

#### **5. Advanced Features**
- `ConnectedApp` - Connected application permissions
- `NamedCredential` - Named credential access
- `SamlSsoConfig` - SAML SSO configuration access

#### **6. Field Sets & List Views**
- `FieldSet` - Field set access (context-dependent)
- `ListView` - List view sharing and access

#### **7. Communities & Experience Cloud**
- `Network` - Community/Experience Cloud access
- `NetworkBranding` - Community branding permissions

---

## 🎯 **RECOMMENDED ENHANCED LIST**

### **For Production-Ready Profile Migration, add:**

```java
// Enhanced keyMetadataTypes for MetadataRetriever4.java
keyMetadataTypes.put("CustomObject", Arrays.asList("*"));
keyMetadataTypes.put("CustomField", new ArrayList<>());
keyMetadataTypes.put("ApexClass", Arrays.asList("*"));
keyMetadataTypes.put("ApexPage", Arrays.asList("*"));
keyMetadataTypes.put("CustomApplication", Arrays.asList("*"));
keyMetadataTypes.put("RecordType", new ArrayList<>());
keyMetadataTypes.put("CustomTab", Arrays.asList("*"));
keyMetadataTypes.put("ExternalDataSource", Arrays.asList("*"));
keyMetadataTypes.put("Flow", Arrays.asList("*"));
keyMetadataTypes.put("CustomPermission", Arrays.asList("*"));
keyMetadataTypes.put("CustomMetadata", Arrays.asList("*"));
keyMetadataTypes.put("Profile", Arrays.asList("*"));
keyMetadataTypes.put("PermissionSet", Arrays.asList("*"));

// ADDITIONAL RECOMMENDED TYPES:
keyMetadataTypes.put("Layout", Arrays.asList("*"));           // Page layouts
keyMetadataTypes.put("Report", Arrays.asList("*"));          // Reports
keyMetadataTypes.put("Dashboard", Arrays.asList("*"));       // Dashboards
keyMetadataTypes.put("ConnectedApp", Arrays.asList("*"));    // Connected apps
keyMetadataTypes.put("ReportType", Arrays.asList("*"));      // Custom report types
```

---

## 📊 **COMPLETENESS ASSESSMENT**

### **Current List Status: 85% Complete** ✅

**✅ COVERED:** All essential metadata types for basic profile migration  
**⚠️ MISSING:** Advanced features and enterprise-specific components  
**🎯 RECOMMENDATION:** Current list is sufficient for most migrations, but consider adding Layout, Report, and Dashboard for comprehensive coverage

### **Priority for Addition:**
1. **HIGH:** `Layout` - Page layout assignments are crucial
2. **MEDIUM:** `Report`, `Dashboard` - If org uses report/dashboard permissions
3. **LOW:** `ConnectedApp`, `NamedCredential` - If using integrations

---

## 🔧 **Validation Command**

```bash
# Check what your profiles actually reference:
sf data query --query "SELECT FullName FROM Metadata WHERE Type IN ('Layout','Report','Dashboard')" --use-tooling-api
```
