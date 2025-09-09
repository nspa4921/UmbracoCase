public without sharing class NCSelfRegService {
    // Helper to build a new User record for registration
    private static User getPopulatedNewUserRecord(String firstName, String lastName, String email, Id contactId, Id profileId, Id roleId) {
        return new User(
            FirstName = firstName,
            LastName = lastName,
            Email = email,
            UserName = email + '.NovoCare',
            Alias = email.left(8),
            ContactId = contactId,
            CommunityNickname = lastName.left(8) + firstName.left(8) + String.valueOf(System.now().getTime()),
            ProfileId = profileId,
            UserRoleId = roleId,
            TimeZoneSidKey = 'GMT',
            LocaleSidKey = 'en_US',
            LanguageLocaleKey = 'en_US',
            EmailEncodingKey = 'UTF-8'
        );
    }
    @future
    private static void createExperienceCloudUser(String firstName, String lastName, String email, String password, Id contactId) {
        try {
            Id profileId = [SELECT Id FROM Profile WHERE Name = 'NovoCare Community Plus Login User' LIMIT 1].Id;
            Id roleId = [SELECT Id FROM UserRole WHERE Name = 'Community Account Owner' LIMIT 1].Id;
            User u = new User();
            u.Username = email + '.' + System.currentTimeMillis() + '@example.com';
            u.Alias = (firstName.substring(0,1) + lastName.substring(0,1)).toLowerCase();
            u.LastName = lastName != null ? lastName.trim() : '';
            u.Email = email;
            u.EmailEncodingKey = 'UTF-8';
            u.TimeZoneSidKey = 'GMT';
            u.LocaleSidKey = 'en_US';
            u.LanguageLocaleKey = 'en_US';
            u.ProfileId = profileId;
            u.ContactId = contactId;
            u.UserRoleId = roleId;
            insert u;
            System.setPassword(u.Id, password);
        } catch (Exception e) {
            System.debug('Error creating Experience Cloud User: ' + e.getMessage());
        }
    }
    // Returns the Id of a user with a role, found by email. Throws if not found.
    private static Id getPaOwnerId() {
        String ownerEmail = 'qenp+ncjpdev1@novonordisk.com';
        List<User> usersWithRole = [
            SELECT Id FROM User WHERE Email = :ownerEmail AND UserRoleId != null LIMIT 1
        ];
        if (usersWithRole.isEmpty()) {
            throw new AuraHandledException('No user with role found for email: ' + ownerEmail);
        }
        return usersWithRole[0].Id;
    }
    // RegRequest class no longer needed for LWC compatibility
    public class RegResponse {
        @AuraEnabled public Boolean success;
        @AuraEnabled public String message;
        @AuraEnabled public String accountId;
        @AuraEnabled public String userId;
    }

    @AuraEnabled(cacheable=false)
    public static RegResponse register(String firstName, String lastName, String email, String password) {
        RegResponse res = new RegResponse();
        System.debug('NCSelfRegService.register received: ' + firstName + ', ' + lastName + ', ' + email + ', ' + password);
        if (String.isBlank(email) || String.isBlank(password) || String.isBlank(firstName) || String.isBlank(lastName)) {
            res.success = false;
            res.message = 'Missing required fields.';
            return res;
        }
        try {
            // Create Person Account
            Account pa = new Account();
            pa.RecordTypeId = getPersonAccountRecordTypeId();
            pa.FirstName = firstName;
            pa.LastName = lastName;
            pa.PersonEmail = email;
            pa.OwnerId = getPaOwnerId();
            insert pa;
            res.accountId = pa.Id;

            // Get Contact (created by Person Account insert)
            Contact c = [SELECT Id FROM Contact WHERE AccountId = :pa.Id LIMIT 1];

            // Get Profile and Role
            Id profileId = [SELECT Id FROM Profile WHERE Name = 'NovoCare Community Plus Login User' LIMIT 1].Id;
            Id roleId = [SELECT Id FROM UserRole WHERE Name = 'Community Account Owner' LIMIT 1].Id;

            // Build User record
            User newUser = getPopulatedNewUserRecord(firstName, lastName, email, c.Id, profileId, roleId);

            // Use Salesforce API for self-registration
            String regId = System.UserManagement.initSelfRegistration(
                Auth.VerificationMethod.EMAIL, 
                newUser
            );

            res.userId = regId;
            res.success = true;
            res.message = 'Registration successful.';
        } catch (Exception e) {
            res.success = false;
            res.message = 'Error: ' + e.getMessage();
        }
        return res;
    }

    private static Id getPersonAccountRecordTypeId() {
        Map<String, Schema.RecordTypeInfo> byDevName =
            Schema.SObjectType.Account.getRecordTypeInfosByDeveloperName();
        if (byDevName.containsKey('PersonAccount')) return byDevName.get('PersonAccount').getRecordTypeId();
        for (Schema.RecordTypeInfo rti : Schema.SObjectType.Account.getRecordTypeInfos()) {
            if (rti.isAvailable() && rti.getName() == 'Person Account') return rti.getRecordTypeId();
        }
        return null;
    }
}
---

### 1. Umbraco Setup

#### Document Types Created:

- `Produkt`
  - `Produktnavn` (Text)
  - `Beskrivelse` (Rich Text Editor)
  - `ProduktBeskrivelse` (Rich Text – shown only in modal)
  - `Pris` (Decimal)
  - `Produktbillede` (Media Picker)
  - `Produktbillede1` (Optional second Media Picker)

- `Produktside`
  - `Sidetitel` (Text)
  - Uses **child nodes of type `Produkt`** to dynamically list products

- `KontaktSide`
  - `Sidetitel`, `Beskrivelse`, `Email`, `Telefonnummer`, `Adresse`, `Kort` (Google Maps iframe)

- `HomePage`
  - `Title`, `Beskrivelse`

---

### 2. Frontend (Tailwind CSS)

- Responsive **grid layout** using Tailwind: `grid-cols-2`, `grid-cols-3`, `auto-fit`, `minmax(...)`
- Product cards designed as **Partial Views** with image, name, price, and short description
- Modal built with **Alpine.js** for interactive preview with image thumbnails, zoom, and transitions
- **Hover effects**, **mobile-first layout**, and clean design

---

### 3. Extra Features

- Modal supports multiple images with click-to-zoom preview
- Embedded Google Maps iframe on contact page
- Sticky footer and responsive navigation
- Fallbacks and validation for missing fields/images
- Fully styled contact page with SVG icons and Tailwind

---

📦 Installation og opsætning

Klon projektet:

- git clone https://github.com/nspa4921/UmbracoWebShopExample.git

- Åbn i Visual Studio / VS Code (jeg brug IntelliJ)

- Kør følgende kommandoer i terminalen (i projektroden):

- npm install
- dotnet run

Navigér til https://localhost:portnummer (port fremgår af terminalen ved start)

📊 Git-kommandoer jeg brugt

- git init
- git add .
- git commit -m "Commit.."
- git remote add origin https://github.com/nspa4921/UmbracoWebShopExample.git
- git push -u origin main




# UmbracoCaseProject
