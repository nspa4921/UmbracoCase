/* force-app/main/default/lwc/ncRegister/ncRegister.js */
import { LightningElement, track } from 'lwc';
import register from '@salesforce/apex/NC_SelfRegService.register';

export default class NcRegister extends LightningElement {
  @track firstName = '';
  @track lastName = '';
  @track email = '';
  @track password = '';
  @track loading = false;
  @track message = '';

  handleChange(e){
    this[e.target.name] = e.target.value;
  }

  async handleSubmit(){
    this.loading = true; this.message = '';
    try{
      const req = {
        firstName: this.firstName,
        lastName:  this.lastName,
        email:     this.email,
        password:  this.password,
        locale:    navigator.language?.replace('-', '_') || 'en_US',
        timeZone:  'GMT',
        language:  (navigator.language?.split('-')[0]) || 'en',
        browserId: localStorage.getItem('browserId') || null,
        newsletterOptIn: false
      };
      const res = await register({ req });
      this.message = res?.message || (res?.success ? 'Success' : 'Failed');
      if(res?.success){
        // opcija: redirect na home ili login
        // window.location.href = '/s/';
      }
    } catch(err){
      this.message = err?.body?.message || 'Registration failed.';
    } finally {
      this.loading = false;
    }
  }
}

# UmbracoWebShopExample

**Case: Umbraco / Frontend Developer**

Dette projekt er en mini produktside bygget med **Umbraco CMS (v15.3.1)** og **Tailwind CSS** som en del af en caseopgave.
---

## 🛠️ Brugt teknologi

- **Umbraco CMS v15.3.1**
- **.NET 7**
- **Tailwind CSS v3**
- **Alpine.js v3**
- **Git & GitHub for version control**

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
