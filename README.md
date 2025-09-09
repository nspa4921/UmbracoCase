public without sharing class NCSelfRegService {

    // ---- helper: safe nick (<=40) ----
    private static String nick(String firstName, String lastName) {
        String n = (lastName != null ? lastName.left(20) : '') +
                   (firstName != null ? firstName.left(20) : '');
        if (String.isBlank(n)) n = 'user' + String.valueOf(System.now().getTime());
        return n.left(40);
    }

    // Helper to build a new User record for registration
    private static User getPopulatedNewUserRecord(
        String firstName, String lastName, String email, Id contactId, Id profileId
    ) {
        return new User(
            FirstName          = firstName,
            LastName           = String.isBlank(lastName) ? 'User' : lastName.trim(),
            Email              = email,
            Username           = String.valueOf(System.currentTimeMillis()) + '.' + email, // globalno unikatno
            Alias              = (String.isBlank(email) ? 'user' : email.replace('@','_')).left(8),
            ContactId          = contactId,
            CommunityNickname  = nick(firstName, lastName),
            ProfileId          = profileId,
            // NEMA UserRoleId za customer community user-a
            TimeZoneSidKey     = 'GMT',
            LocaleSidKey       = 'en_US',
            LanguageLocaleKey  = 'en_US',
            EmailEncodingKey   = 'UTF-8'
        );
    }

    // User koji će biti owner PA (mora imati Role)
    private static Id getPaOwnerId() {
        String ownerEmail = 'qenp+ncjpdev1@novonordisk.com';
        List<User> usersWithRole = [
            SELECT Id
            FROM User
            WHERE Email = :ownerEmail AND IsActive = true AND UserRoleId != null
            LIMIT 1
        ];
        if (usersWithRole.isEmpty()) {
            throw new AuraHandledException('No user with a Role found for email: ' + ownerEmail);
        }
        return usersWithRole[0].Id;
    }

    // RecordType za Person Account
    private static Id getPersonAccountRecordTypeId() {
        for (Schema.RecordTypeInfo rti : Schema.SObjectType.Account.getDescribe().getRecordTypeInfos()) {
            if (rti.isPersonType()) return rti.getRecordTypeId();
        }
        throw new AuraHandledException('Person Account RecordType not found / PA not enabled.');
    }

    public class RegResponse {
        @AuraEnabled public Boolean success;
        @AuraEnabled public String  message;
        @AuraEnabled public String  accountId;
        @AuraEnabled public String  userId;
    }

    @AuraEnabled(cacheable=false)
    public static RegResponse register(String firstName, String lastName, String email, String password) {
        RegResponse res = new RegResponse();
        if (String.isBlank(email) || String.isBlank(password) || String.isBlank(firstName) || String.isBlank(lastName)) {
            res.success = false; res.message = 'Missing required fields.'; return res;
        }

        try {
            // 1) Person Account (owner mora imati Role)
            Account pa = new Account();
            pa.RecordTypeId = getPersonAccountRecordTypeId();
            pa.FirstName    = firstName;
            pa.LastName     = lastName;
            pa.PersonEmail  = email;
            pa.OwnerId      = getPaOwnerId();   // KLJUČNO
            insert pa;
            res.accountId = pa.Id;

            // 2) Contact automatski kreiran za PA
            Contact c = [SELECT Id FROM Contact WHERE AccountId = :pa.Id LIMIT 1];

            // 3) Profil za external user-a
            Id profileId = [SELECT Id FROM Profile WHERE Name = 'NovoCare Community Plus Login User' LIMIT 1].Id;

            // 4) Pripremi User
            User newUser = getPopulatedNewUserRecord(firstName, lastName, email, c.Id, profileId);

            // 5) Kreiraj external user-a (self-registration, radi iz guest konteksta)
            //    Ako si u internom kontekstu, možeš umesto ovoga: insert newUser; System.setPassword(newUser.Id, password);
            Id createdUserId = Site.createExternalUser(newUser, c.Id, password);
            res.userId = createdUserId;

            res.success = true;
            res.message = 'Registration successful.';
        } catch (Exception e) {
            res.success = false;
            res.message = 'Error: ' + e.getMessage();
        }
        return res;
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
