<!-- force-app/main/default/lwc/ncRegister/ncRegister.html -->
<template>
  <div class="slds-p-around_large slds-card">
    <h2 class="slds-text-heading_medium slds-m-bottom_medium">Create account</h2>

    <div class="slds-grid slds-wrap slds-gutters">
      <div class="slds-col slds-size_1-of-2">
        <lightning-input name="firstName" label="First name" value={firstName} onchange={handleChange} required></lightning-input>
      </div>
      <div class="slds-col slds-size_1-of-2">
        <lightning-input name="lastName" label="Last name" value={lastName} onchange={handleChange} required></lightning-input>
      </div>
      <div class="slds-col slds-size_1-of-1 slds-m-top_small">
        <lightning-input name="email" type="email" label="Email" value={email} onchange={handleChange} required></lightning-input>
      </div>
      <div class="slds-col slds-size_1-of-1 slds-m-top_small">
        <lightning-input name="password" type="password" label="Password" value={password} onchange={handleChange} required></lightning-input>
      </div>
    </div>

    <div class="slds-m-top_medium">
      <lightning-button variant="brand" label={loading ? 'Creating…' : 'Create account'}
                        onclick={handleSubmit} disabled={loading}></lightning-button>
    </div>

    <template if:true={message}>
      <div class="slds-m-top_medium">{message}</div>
    </template>
  </div>
</template>




/* force-app/main/default/lwc/ncLogin/ncLogin.js */
import { LightningElement } from 'lwc';
export default class NcLogin extends LightningElement {
  startUrl = '/s/'; // gde da ode posle logina
}

<!-- force-app/main/default/lwc/ncLogin/ncLogin.html -->
<template>
  <div class="slds-p-around_large slds-card">
    <h2 class="slds-text-heading_medium slds-m-bottom_medium">Log in</h2>

    <!-- Standard Experience Cloud endpoint -->
    <form method="POST" action="/login">
      <input type="hidden" name="startURL" value={startUrl} />
      <lightning-input name="username" type="email" label="Email" required></lightning-input>
      <lightning-input name="password" type="password" label="Password" class="slds-m-top_small" required></lightning-input>
      <div class="slds-m-top_medium">
        <lightning-button variant="brand" type="submit" label="Log in"></lightning-button>
      </div>
    </form>

    <div class="slds-m-top_small">
      <a href="/login/SelfRegister">Create account</a> •
      <a href="/secur/forgotpassword.jsp">Forgot password?</a>
    </div>
  </div>
</template>
--‐---------
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
