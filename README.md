// NC_SelfRegService.cls
public with sharing class NC_SelfRegService {
    public class RegRequest {
        @AuraEnabled public String firstName;
        @AuraEnabled public String lastName;
        @AuraEnabled public String email;
        @AuraEnabled public String password;
        @AuraEnabled public String locale;      // npr. 'en_US' / 'ja_JP'
        @AuraEnabled public String timeZone;    // npr. 'GMT'
        @AuraEnabled public String language;    // npr. 'en'
        @AuraEnabled public String browserId;   // za kasnije merge assessment-a (opciono)
        @AuraEnabled public Boolean newsletterOptIn;
    }
    public class RegResponse {
        @AuraEnabled public Boolean success;
        @AuraEnabled public String message;
    }

    @AuraEnabled(cacheable=false)
    public static RegResponse register(RegRequest req) {
        RegResponse res = new RegResponse();
        if (String.isBlank(req.email) || String.isBlank(req.password) || String.isBlank(req.firstName) || String.isBlank(req.lastName)) {
            res.success = false; res.message = 'Missing required fields.'; return res;
        }

        // 1) Ako već postoji Contact/PA za taj email:
        Contact existingC = [SELECT Id, AccountId,
                                    Account.PersonEmail,
                                    (SELECT Id, IsActive FROM Users WHERE IsActive = true LIMIT 1)
                               FROM Contact
                               WHERE Email = :req.email
                               LIMIT 1];
        if (existingC != null) {
            if (!existingC.Users.isEmpty()) {
                res.success = false; res.message = 'An account already exists. Please log in or reset your password.'; return res;
            }
            // Kreiraj community user nad postojećim kontaktom
            createCommunityUser(existingC, req);
            // (opciono) Newsletter/consent/assessment merge
            res.success = true; res.message = 'User created for existing contact.'; return res;
        }

        // 2) Kreiraj Person Account (FORCIRANO)
        Id personRtId = getPersonAccountRecordTypeId();
        if (personRtId == null) {
            res.success = false; res.message = 'Person Account record type not found.'; return res;
        }

        Account pa = new Account();
        pa.RecordTypeId = personRtId;
        pa.FirstName     = req.firstName;
        pa.LastName      = req.lastName;
        pa.PersonEmail   = req.email;
        insert pa;

        // Dohvati automatski kreirani Contact
        Contact c = [SELECT Id, AccountId FROM Contact WHERE AccountId = :pa.Id LIMIT 1];

        // 3) Kreiraj Experience Cloud User (portal user) i setuj lozinku
        createCommunityUser(c, req);

        // (opciono) ovde pozovi AssessmentMergeService.merge(req.browserId, c.Id);
        // (opciono) Newsletter/Consent upis

        res.success = true;
        res.message = 'Registration successful.';
        return res;
    }

    private static void createCommunityUser(Contact c, RegRequest req) {
        // PROFIL za community korisnike — prilagodi svojoj org konfiguraciji!
        // (npr. 'Customer Community Plus User' ili custom community profil)
        Id portalProfileId = [SELECT Id FROM Profile WHERE Name = 'Customer Community Plus User' LIMIT 1].Id;

        User u = new User();
        u.Username            = uniqueUsername(req.email);
        u.Alias               = (req.firstName.substring(0,1) + req.lastName.substring(0,1)).toLowerCase();
        u.Email               = req.email;
        u.EmailEncodingKey    = 'UTF-8';
        u.TimeZoneSidKey      = String.isBlank(req.timeZone) ? 'GMT' : req.timeZone;
        u.LocaleSidKey        = String.isBlank(req.locale) ? 'en_US' : req.locale;
        u.LanguageLocaleKey   = String.isBlank(req.language) ? 'en' : req.language;
        u.ProfileId           = portalProfileId;
        u.ContactId           = c.Id;

        // Važno: za Experience Cloud koristimo Site.createExternalUser
        // Ovo mora da se izvršava iz konteksta guest sajta/portala
        System.runAs(new User(Id=UserInfo.getUserId())) {
            Site.createExternalUser(u, c.Account, req.password);
        }
    }

    private static String uniqueUsername(String base) {
        // U sandboxovima Username mora biti jedinstven org-wide; dodaj sufiks ako postoji
        Integer tries = 0;
        String candidate = base;
        while (true) {
            List<User> u = [SELECT Id FROM User WHERE Username = :candidate LIMIT 1];
            if (u.isEmpty()) return candidate;
            tries++;
            candidate = base.replace('@','+'+tries+'@');
        }
    }

    private static Id getPersonAccountRecordTypeId() {
        Map<String, Schema.RecordTypeInfo> byDevName =
            Schema.SObjectType.Account.getRecordTypeInfosByDeveloperName();
        if (byDevName.containsKey('PersonAccount')) return byDevName.get('PersonAccount').getRecordTypeId();

        // Fallback: traži po labeli
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
