
 [sfdx-hardis] Generating Permission set PS_trade_Profile from profile trade Profile
 
✔ [sfdx-hardis][command] **sf shane:profile:convert -p "trade Profile" -n "PS_trade_Profile" -e**

```bash
[sfdx-hardis] copying applicationVisibilities to perm set
copying classAccesses to perm set
found no externalDataSourceAccesses on the profile
copying fieldPermissions to perm set
copying objectPermissions to perm set
copying pageAccesses to perm set
copying recordTypeVisibilities to perm set
copying tabVisibilities to perm set
Permissions removed from force-app/main/default/profiles/trade Profile.profile-meta.xml
Permissions added in force-app/main/default/permissionsets/PS_trade_Profile.permissionset-meta.xml
```


Cette conversion transforme un profil en ensemble de permissions (permission set) sans affecter directement les utilisateurs existants, mais il y a quelques points importants à considérer :

**Impact immédiat :**
- Les permissions sont déplacées du profil vers le nouveau permission set `PS_trade_Profile`
- Le profil `trade Profile` est vidé de ses permissions spécifiques
- Les utilisateurs gardent leur profil assigné mais perdent temporairement ces permissions

**Ce qu'il faut faire ensuite :**
1. **Assigner le nouveau permission set** aux utilisateurs qui avaient le profil `trade Profile` pour qu'ils retrouvent leurs permissions
2. **Tester** que tous les accès fonctionnent correctement
3. **Considérer** si le profil doit être remplacé par un profil standard (comme "Standard User") maintenant qu'il est allégé

**Avantages de cette approche :**
- Plus de flexibilité dans la gestion des permissions
- Possibilité d'assigner ces permissions à des utilisateurs avec différents profils
- Meilleure maintenabilité du setup de sécurité

La conversion en elle-même est "safe" mais nécessite une étape supplémentaire d'assignation du permission set pour que les utilisateurs retrouvent leurs accès. C'est une bonne pratique moderne dans Salesforce de privilégier les permission sets aux profils personnalisés.

Oui, absolument ! Il existe plusieurs approches pour automatiser l'affectation des permission sets aux utilisateurs par code :

## 1. **Via Apex (le plus direct)**

```apex
// Récupérer tous les utilisateurs avec le profil "trade Profile"
List<User> usersToUpdate = [SELECT Id, ProfileId FROM User WHERE Profile.Name = 'trade Profile' AND IsActive = true];

// Récupérer l'ID du permission set
PermissionSet ps = [SELECT Id FROM PermissionSet WHERE Name = 'PS_trade_Profile' LIMIT 1];

// Créer les assignments
List<PermissionSetAssignment> assignments = new List<PermissionSetAssignment>();
for(User u : usersToUpdate) {
    assignments.add(new PermissionSetAssignment(
        PermissionSetId = ps.Id,
        AssigneeId = u.Id
    ));
}

// Insérer les assignments
insert assignments;
```

## 2. **Via SFDX/SF CLI** 

```bash
# Exporter les utilisateurs avec le profil
sf data query --query "SELECT Id FROM User WHERE Profile.Name='trade Profile' AND IsActive=true" --result-format csv > users.csv

# Script pour créer les assignments
sf data create --sobject PermissionSetAssignment --file assignments.csv
```

## 3. **Via Data Loader ou Workbench**
Créer un CSV avec les colonnes :
- `PermissionSetId` (ID du permission set)
- `AssigneeId` (ID de l'utilisateur)

## 4. **Script automatisé avec sfdx-hardis**
```bash
# Il est possible que sfdx-hardis ait une commande pour ça
sfdx hardis:user:permset:assign --permset "PS_trade_Profile" --profile "trade Profile"
```

## 5. **Solution complète en Node.js/JavaScript**

```javascript
const { Connection } = require('jsforce');

async function assignPermissionSet() {
    const conn = new Connection({ /* config */ });
    
    // Récupérer les utilisateurs
    const users = await conn.query(`
        SELECT Id FROM User 
        WHERE Profile.Name = 'trade Profile' 
        AND IsActive = true
    `);
    
    // Récupérer le permission set
    const permSet = await conn.query(`
        SELECT Id FROM PermissionSet 
        WHERE Name = 'PS_trade_Profile'
    `);
    
    // Créer les assignments
    const assignments = users.records.map(user => ({
        PermissionSetId: permSet.records[0].Id,
        AssigneeId: user.Id
    }));
    
    // Insérer
    await conn.sobject('PermissionSetAssignment').create(assignments);
}
```

**La solution recommandée** serait d'intégrer cette logique directement dans le processus `sfdx-hardis` ou de créer un script post-conversion qui :
1. Identifie automatiquement les utilisateurs concernés
2. Assigne le nouveau permission set
3. Vérifie que l'assignment a réussi

Cela rendrait la conversion vraiment transparente pour les utilisateurs !
