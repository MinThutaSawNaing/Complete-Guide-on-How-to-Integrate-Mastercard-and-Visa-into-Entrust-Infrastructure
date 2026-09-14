<link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;600;700;800&family=JetBrains+Mono:wght@500&display=swap" rel="stylesheet">

<style>
  :root {
    --brand-navy: #0b1f3a;
    --brand-gold: #c9a227;
    --brand-blue: #1f6feb;
    --brand-red: #eb001b;   /* Mastercard red  */
    --brand-amber: #ff5f00; /* Mastercard amber */
    --brand-visa: #1a1f71;  /* Visa blue */
    --ink: #24292f;
    --paper: #f6f8fa;
  }
  .doc { font-family: 'Inter', -apple-system, 'Segoe UI', Roboto, sans-serif; color: var(--ink); line-height: 1.65; }
  .hero { text-align: center; padding: 28px 16px 20px; border-radius: 14px;
          background: linear-gradient(135deg, #0b1f3a 0%, #153e75 55%, #1a1f71 100%); color: #fff; }
  .hero h1 { font-weight: 800; letter-spacing: -0.5px; margin: 8px 0 4px; color: #fff; }
  .hero p.sub { font-family: 'JetBrains Mono', monospace; font-size: 13px; color: #cdd9ef; margin-top: 6px; }
  .pill { display: inline-block; padding: 3px 12px; margin: 3px; border-radius: 999px; font-size: 12px; font-weight: 600; }
  .pill-mc   { background: linear-gradient(90deg, var(--brand-red), var(--brand-amber)); color: #fff; }
  .pill-visa { background: var(--brand-visa); color: #fff; }
  .pill-en   { background: var(--brand-gold); color: #1b1b1b; }
  h2 { font-family: 'Inter', sans-serif; font-weight: 800; border-bottom: 3px solid var(--brand-gold);
       padding-bottom: 6px; margin-top: 40px; letter-spacing: -0.3px; }
  h3 { font-weight: 700; color: var(--brand-navy); }
  code, pre { font-family: 'JetBrains Mono', 'Cascadia Code', Consolas, monospace !important; }
  .callout { border-left: 5px solid var(--brand-red); background: #fff5f5; padding: 12px 16px; border-radius: 0 8px 8px 0; }
  .note    { border-left: 5px solid var(--brand-blue); background: #eef5ff; padding: 12px 16px; border-radius: 0 8px 8px 0; }
  .tip     { border-left: 5px solid #2da44e; background: #f0fff4; padding: 12px 16px; border-radius: 0 8px 8px 0; }
  table { border-collapse: collapse; width: 100%; }
  th { background: var(--brand-navy) !important; color: #fff !important; font-weight: 700; }
  tr:nth-child(even) { background: var(--paper); }
</style>

<div class="doc">

<div class="hero">

# 🏦 The Complete Guide to Integrating Mastercard &amp; Visa into Entrust Infrastructure

`Prepared by MinThutaSawNaing` · `Card Personalization &amp; Key Management Series`

<span class="pill pill-mc">MASTERCARD</span> <span class="pill pill-visa">VISA</span> <span class="pill pill-en">ENTRUST KEY MANAGER</span> <span class="pill pill-en">EMV</span>

*A hands-on field guide for preparing and configuring the Entrust software stack for Mastercard or Visa issuance — token tables, issuer key pairs, CA requests, and EMV profiles.*

</div>

---

## 📑 Table of Contents

1. [Overview](#-overview)
2. [Before You Start — The Three Entrust Portals](#-before-you-start--the-three-entrust-portals)
3. [Step 1 — Token Table Configuration](#step-1--token-table-configuration)
4. [Step 2 — Generate the Issuer Key Pair (Issuer_PK / Issuer_SK)](#step-2--generate-the-issuer-key-pair-issuer_pk--issuer_sk)
5. [Step 3 — Create &amp; Submit the CA Request](#step-3--create--submit-the-ca-request)
6. [Step 4 — Import the CA Certificate (.hep / .sep) and Issuer Certificate (.cEF)](#step-4--import-the-ca-certificate-hep--sep-and-issuer-certificate-cef)
7. [Step 5 — Import the Required Keys into the Token Table](#step-5--import-the-required-keys-into-the-token-table)
8. [Step 6 — EMV Profile &amp; PQDF Tool Files](#step-6--emv-profile--pqdf-tool-files)
9. [Delivery Checklist](#-delivery-checklist)
10. [Common Pitfalls](#-common-pitfalls)

---

## 🔎 Overview

Bringing a bank onto **Mastercard** or **Visa** rails using **Entrust** (nShield HSM + Key Manager) is mostly a *key-ceremony* discipline: the right keys, in the right token table, with the right attributes — followed by the right certificates, in the right order.

This guide walks the full happy path:

```
Create Token Table  →  Generate Issuer Key Pair  →  Build CA Request (ZIP)
        →  Mastercard returns Issuer Certificate (.cEF)
        →  Import CA Certificate (.hep + .sep)  →  Import Issuer Certificate (.cEF)
        →  Import Request/Session Keys (2DES, 2 components)
        →  Import EMV Profile + PQDF Tool Files  →  🎉 Ready for issuance
```

> ⏱ Roughly **50% of the mission is complete** once the CA and issuer certificates are imported. The remaining keys and EMV profile files finish the job.

---

## 🧭 Before You Start — The Three Entrust Portals

Our mission uses **all three** portals in the Entrust ecosystem:

| # | Portal | Primary Role in This Guide |
|---|--------|----------------------------|
| 1 | 🔐 **Key Manager Portal** | Token tables, key pairs, CA requests, certificate &amp; key imports |
| 2 | 🧩 **EMV Profile Manager Portal** | EMV kernel/profile configuration for the card applet |
| 3 | 💳 **Card Wizard Portal** | Card personalization workflow &amp; tool-file generation (PQDF) |

**Client-side prerequisites to confirm first:**

- [ ] Client maintains **separate token tables for Testing (UAT) and Live (PROD)** — see Step 1.
- [ ] The bank's **BIN number** is available (it becomes the key *Owner*).
- [ ] A bank **HOST** operator is available to generate 2DES keys **with two key components**.
- [ ] Access to the **Mastercard Portal** to download the `.hep` and `.sep` files.

---

## Step 1 — Token Table Configuration

🔐 *Key Manager Portal*

Some clients run **everything in one token table**. It is not a bad thing, but it is **not practical** — you want a clean blast-radius separation between testing and production material.

> **Rule of thumb:** create **two separate token tables** — one for **Testing (UAT)**, one for **Live**.

**To create a new token table:**

```
Entrust Key Manager Portal
  └─ Setup
      └─ HSM Configurations
          └─ Right-click on the HSM  →  Login
              └─ Create a New Token Table  →  Name the table
```

✅ When the table appears in the HSM tree, the first milestone is done.

---

## Step 2 — Generate the Issuer Key Pair (Issuer_PK / Issuer_SK)

🔐 *Key Manager Portal*

Next, generate the **Issuer Public Key (Issuer_PK)** — the key used to build the CA request — and the matching **Issuer Private Key (Issuer_SK)**.

> ⚠️ **This is the most critical step in the entire integration — roughly 80% of CA requests fail because of mistakes made here.** Double-check every attribute before clicking *Generate*.

**Navigation:**

```
Key Manager Portal
  └─ Home
      └─ Choose the correct token table  →  Login
          └─ Login as Security Officer (SO)
              └─ "Generate Key Pair" (side bar)  →  Apply the parameters below
```

**Required key parameters (Mastercard):**

| Parameter | Value | Notes |
|-----------|-------|-------|
| Key Type | Any **RSA-based** key type | Per your HSM's key-type naming |
| Key Length | **1976 bits** | As specified for Mastercard in this guide |
| Owner | The bank's **BIN number** | Ties the key material to the issuer identity |
| Version | `06` = **LIVE** &nbsp;·&nbsp; `EF` = **UAT** | Version encodes the environment |
| Usages | **Encrypt, Wrap, Verify** | Select all three |
| Attributes | **Extractable, Exportable, Deletable** | Select all three |
| Exponent | **03** | Standard RSA public exponent |

> 📝 **Generate both keys with the same procedure** — `Issuer_PK` and `Issuer_SK` follow the identical guide and identical parameters; only the **name** differs.

---

## Step 3 — Create &amp; Submit the CA Request

🔐 *Key Manager Portal* → 🌐 *Mastercard Portal*

Create the certificate authority request that will be submitted to Mastercard:

```
Key Manager Portal
  └─ Certificates
      └─ Request a Certificate
          └─ Mastercard
              └─ Choose the Issuer Public Key created in Step 2
                  └─ Generate Request Certificate  →  (ZIP output)
```

**Hand-off flow:**

1. 📦 Entrust produces the CA request as a **ZIP** archive.
2. 🏦 Give the ZIP to the **bank**; the bank submits it to **Mastercard**.
3. 📨 Mastercard replies with the **Issuer Certificate** (`.cEF` file).

> 🚨 **Most banks mistakenly believe the `.cEF` is everything they need to go live with Mastercard. It is not.** You still need two additional files — both downloadable from the Mastercard Portal:

| Artifact | Format | Where It Comes From | Used For |
|----------|--------|---------------------|----------|
| Issuer Certificate | `.cEF` | Mastercard reply to the CA request | Imported as the issuer certificate (Step 4) |
| Hardware Encryption Program | `.hep` | **Mastercard Portal** download | Building the CA certificate in Entrust (Step 4) |
| Security Program | `.sep` | **Mastercard Portal** download | Building the CA certificate in Entrust (Step 4) |

> ✅ **Make sure you get all three:** `.cEF` + `.hep` + `.sep`.

---

## Step 4 — Import the CA Certificate (.hep / .sep) and Issuer Certificate (.cEF)

🔐 *Key Manager Portal*

Once all three files are in hand, import them into Entrust **in this order**:

**1. Import the CA certificate (HEP + SEP):**

```
Key Manager Portal
  └─ Import Certificate
      └─ CA Certificate
          └─ Choose both .hep and .sep  →  Import
```

**2. Import the Issuer Certificate:**

```
Key Manager Portal
  └─ Import Certificate
      └─ Issuer Certificate
          └─ Choose .cEF  →  Import
```

🎯 **At this point, 50% of the integration is complete.**

---

## Step 5 — Import the Required Keys into the Token Table

🔐 *Key Manager Portal* → 🏦 *Bank HOST*

The next phase is requesting and importing the **necessary operational keys** (request/session keys) into the token table.

> 💡 **Don't worry about guessing which keys you need** — Entrust will tell you exactly which keys must be configured. Once you have the list, pass it to the bank, and the bank generates them with its HOST.

> ⚠️ **Important instruction for the bank:** request **2DES keys generated with two components** — Entrust requires **at least two key components** to import a key.

**Import procedure (once the components arrive):**

```
Key Manager Portal
  └─ Import Secret Key or Key Pair
      └─ Import Key with Components
          └─ Type in the plain key components
              └─ Enter the components twice  →  Generate Key
```

---

## Step 6 — EMV Profile &amp; PQDF Tool Files

🧩 *EMV Profile Manager Portal* · 💳 *Card Wizard Portal*

After the keys are in place, Entrust provides:

| Deliverable | Purpose |
|-------------|---------|
| 🧩 **EMV Profile file** | Card applet / kernel configuration |
| 💳 **PQDF tool files** | Personalization data tool files for card issuance |

These are straightforward to import — simply follow the guidelines shipped with them.

> 🔧 **Tuning tip:** expect to make small adjustments to the configuration files based on **your customer's card design and applet requirements** before the first production run.

---

## 📋 Delivery Checklist

Use this to sign off an integration:

- [ ] **Two token tables** exist — one **UAT**, one **LIVE**
- [ ] **Issuer_PK** &amp; **Issuer_SK** generated with the correct BIN owner, version (`06`/`EF`), usages, attributes, and exponent `03`
- [ ] **CA request ZIP** generated and handed to the bank
- [ ] Bank submitted the request; Mastercard returned the **Issuer Certificate (`.cEF`)**
- [ ] **`.hep`** and **`.sep`** downloaded from the Mastercard Portal
- [ ] **CA certificate imported** (both `.hep` + `.sep`)
- [ ] **Issuer certificate imported** (`.cEF`)
- [ ] Required keys requested from the bank — **2DES, two components each**
- [ ] Keys imported via *Import Key with Components* (entered twice)
- [ ] **EMV profile** imported
- [ ] **PQDF tool files** imported &amp; tuned for the customer's cards

---

## ⚠️ Common Pitfalls

| # | Pitfall | Consequence | Avoidance |
|---|---------|-------------|-----------|
| 1 | Wrong key-pair attributes (usages / attributes / exponent / version) | ❌ **~80% of CA requests fail here** | Re-read the Step 2 parameter table before generating |
| 2 | Thinking the `.cEF` alone is enough | Integration stalls at personalization | Remember: you need `.cEF` **+ `.hep` + `.sep`** |
| 3 | One shared token table for UAT &amp; Live | Operationally impractical; test material leaks into production risk | Always create separate tables (Step 1) |
| 4 | Bank generates keys as a single component | Entrust **cannot import** the key | Instruct the bank: **2DES keys with two components** |
| 5 | Importing EMV/PQDF files without tuning | Cards misbehave for the specific customer design | Follow vendor guidelines **and** tune per customer cards |

---

<div align="center">

**Happy issuing! 🎉**

*Written from hands-on field notes for payment-scheme engineers working with Entrust, Mastercard and Visa infrastructure.*

Made with ❤️ — [MinThutaSawNaing](https://github.com/MinThutaSawNaing)

</div>

</div>



