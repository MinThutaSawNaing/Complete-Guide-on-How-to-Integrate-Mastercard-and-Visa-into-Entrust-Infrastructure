<div align="center">

# 🏦 Complete Guide: Integrating Mastercard &amp; Visa into Entrust Infrastructure

[![Mastercard](https://img.shields.io/badge/Mastercard-EB001B?style=for-the-badge&logo=mastercard&logoColor=FF5F00)](#)
[![Visa](https://img.shields.io/badge/Visa-1A1F71?style=for-the-badge&logo=visa&logoColor=white)](#)
[![Entrust](https://img.shields.io/badge/Entrust_Key_Manager-C9A227?style=for-the-badge&logo=lockdotdev&logoColor=black)](#)
[![HTML5](https://img.shields.io/badge/HTML5-E34F26?style=for-the-badge&logo=html5&logoColor=white)](index.html)
[![CSS3](https://img.shields.io/badge/CSS3-1572B6?style=for-the-badge&logo=css3&logoColor=white)](index.html)
[![JavaScript](https://img.shields.io/badge/JavaScript-F7DF1E?style=for-the-badge&logo=javascript&logoColor=black)](index.html)

### ✨ [👉 Open the Interactive Guide (index.html) 👈](./index.html)

*A fully styled, interactive HTML/CSS/JavaScript edition of the guide — dark/light mode, live progress bar, animated delivery checklist and more.*

**📥 Prefer offline? [Download the PDF version of this guide](./Entrust-Mastercard-Visa-Integration-Guide.pdf)**

</div>

---

## 📖 About This Guide

Bringing a bank onto **Mastercard** or **Visa** rails using **Entrust** (nShield HSM + Key Manager) is a *key-ceremony* discipline: the right keys, in the right token table, with the right attributes — followed by the right certificates, in the right order.

The full integration pipeline:

```
Create Token Tables  →  Generate Issuer Key Pair  →  Build CA Request (ZIP)
        →  Mastercard returns Issuer Certificate (.cEF)
        →  Import CA Certificate (.hep + .sep)  →  Import Issuer Certificate (.cEF)
        →  Import Request/Session Keys (2DES, 2 components)
        →  Import EMV Profile + PQDF Tool Files  →  🎉 Ready for issuance
```

> ⏱ Roughly **50% of the mission is complete** once the CA and issuer certificates are imported.

---

## 🧭 The Three Entrust Portals

| # | Portal | Primary Role |
|---|--------|--------------|
| 1 | 🔐 **Key Manager Portal** | Token tables, key pairs, CA requests, certificate &amp; key imports |
| 2 | 🧩 **EMV Profile Manager Portal** | EMV kernel / profile configuration for the card applet |
| 3 | 💳 **Card Wizard Portal** | Card personalization workflow &amp; tool-file generation (PQDF) |

---

## 🪜 The Six Steps

### Step 1 — Token Table Configuration
Create **two separate token tables** — one for **Testing (UAT)**, one for **Live**. Running everything in one table is possible but not practical.

```
Key Manager Portal → Setup → HSM Configurations
  → Right-click HSM → Login → Create a new token table → Name the table
```

### Step 2 — Generate the Issuer Key Pair (Issuer_PK / Issuer_SK)

> ⚠️ **Roughly 80% of CA requests fail because of mistakes in this step.** Double-check every attribute.

```
Key Manager Portal → Home → Choose token table → Login as Security Officer
  → Generate Key Pair (side bar)
```

| Parameter | Value |
|-----------|-------|
| Key Type | Any **RSA-based** key type |
| Key Length | **1976 bits** (Mastercard) |
| Owner | The bank's **BIN number** |
| Version | `06` = **LIVE** · `EF` = **UAT** |
| Usages | **Encrypt, Wrap, Verify** |
| Attributes | **Extractable, Exportable, Deletable** |
| Exponent | **03** |

*Issuer_PK and Issuer_SK follow the identical procedure — only the name differs.*

### Step 3 — Create &amp; Submit the CA Request

```
Key Manager Portal → Certificates → Request a Certificate → Mastercard
  → Choose the Issuer Public Key → Generate Request Certificate (ZIP)
```

The bank submits the ZIP to Mastercard, which replies with the **Issuer Certificate (`.cEF`)**.

> 🚨 The `.cEF` alone is **not** enough. You also need **`.hep`** and **`.sep`**, both downloadable from the Mastercard Portal.

### Step 4 — Import CA &amp; Issuer Certificates

```
Key Manager Portal → Import Certificate → CA Certificate → choose .hep + .sep → Import
Key Manager Portal → Import Certificate → Issuer Certificate → choose .cEF → Import
```

🎯 **50% of the integration is done at this point.**

### Step 5 — Import the Required Keys
Entrust tells you which keys are needed; the bank generates them with its HOST.

> ⚠️ Tell the bank to generate **2DES keys with two components** — Entrust needs at least two components to import a key.

```
Key Manager Portal → Import Secret Key or Key Pair → Import Key with Components
  → Type the plain key components (twice) → Generate Key
```

### Step 6 — EMV Profile &amp; PQDF Tool Files
Entrust provides the **EMV profile file** and **PQDF tool files** — import them per the shipped guidelines, then fine-tune the configuration files for your customer's cards.

---

## ⚠️ Top Pitfalls

| # | Pitfall | Avoidance |
|---|---------|-----------|
| 1 | Wrong key-pair attributes | ~80% of CA failures live here — re-verify the Step 2 table |
| 2 | Believing `.cEF` is enough | You need `.cEF` **+ `.hep` + `.sep`** |
| 3 | One shared token table | Always separate UAT and LIVE |
| 4 | Single-component keys | Entrust can't import them — require two components |
| 5 | Untuned EMV/PQDF files | Adjust per customer card design before production |

---

<div align="center">

**Made with ❤️ by [MinThutaSawNaing](https://github.com/MinThutaSawNaing)**

🌐 [Interactive HTML Guide](./index.html) · 📥 [PDF Download](./Entrust-Mastercard-Visa-Integration-Guide.pdf) · 📄 [Original Field Notes (.txt)](<./Full Guide on preparing or configuring the Entrust Software for Master or Visa.txt>)

</div>
