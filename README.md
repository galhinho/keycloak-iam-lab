# Keycloak IAM Lab — Open Source Identity & Access Management

## Overview

This lab demonstrates the implementation of an Identity and Access Management 
solution using Keycloak, an open-source IAM platform widely adopted across 
European sovereign cloud environments as an alternative to Microsoft Entra ID.

The lab simulates a small organisation with distinct user personas, security 
groups, and role-based access control, implementing OIDC authentication and 
validating JWT-based role claims.

This lab was built in the context of the EU digital sovereignty movement — 
as European organisations increasingly adopt open-source IAM solutions to 
reduce dependency on US-based cloud providers, Keycloak has emerged as the 
primary enterprise-grade alternative.

---

## Why Keycloak

Microsoft Entra ID and similar US-based IAM platforms are subject to the 
US CLOUD Act, which allows US authorities to access data regardless of 
where it is stored. European organisations in regulated industries — banking, 
healthcare, government — are increasingly required to demonstrate data 
sovereignty, making open-source alternatives like Keycloak essential.

Keycloak supports the same industry-standard protocols as Entra ID:
- OpenID Connect (OIDC)
- SAML 2.0
- OAuth 2.0

This means skills transfer directly between platforms — the protocols are 
identical, only the implementation differs.

---

## Lab Environment

- **Platform:** Keycloak 26.2.4 (standalone, development mode)
- **Runtime:** Java 21 (Eclipse Temurin)
- **Host:** Windows 11, localhost
- **Realm:** Company-lab

---

## Identity Architecture

### Realm
A Keycloak Realm is equivalent to an Entra ID tenant — it is a isolated 
identity domain with its own users, roles, groups, and client applications.

### Groups

| Group | Purpose |
|---|---|
| IT-Team | Groups IT administrators for policy targeting |
| Finance-Team | Groups finance users for scoped access control |

### Roles

| Role | Description |
|---|---|
| `it-admin` | Full access for IT administrators |
| `finance-user` | Standard access for finance team members |
| `external-user` | Limited access for external collaborators |

### Users

| Username | Group | Role | Email Verified |
|---|---|---|---|
| john.admin | IT-Team | it-admin | Yes |
| jane.finance | Finance-Team | finance-user | Yes |
| bob.external | None | external-user | Yes |

**Design Decision — bob.external has no group:**
External collaborators are intentionally excluded from internal groups, 
following least privilege principles. This mirrors the same design decision 
made in the companion Azure Entra ID lab, demonstrating consistent IAM 
thinking across platforms.

---

## Client Application — OIDC Configuration

A Keycloak Client represents an application that delegates authentication 
to Keycloak. This is equivalent to an App Registration in Microsoft Entra ID.

**Client ID:** `company-app`  
**Protocol:** OpenID Connect  
**Authentication flow:** Standard flow + Direct access grants  
**Redirect URI:** `http://localhost:3000/*`  

The client uses a **client secret** for confidential authentication — 
the application must present both its client ID and secret to obtain tokens.

---

## Authentication Testing — JWT Token Validation

Authentication was tested using the OIDC token endpoint directly, 
simulating how a backend application would authenticate users.

### Token Request (PowerShell)

```powershell
$response = Invoke-WebRequest -Method POST `
  -Uri "http://localhost:8080/realms/Company-lab/protocol/openid-connect/token" `
  -ContentType "application/x-www-form-urlencoded" `
  -Body @{
    grant_type="password"
    client_id="company-app"
    client_secret=""
    username="john.admin"
    password=""
  }
```

### Decoded JWT Payload — john.admin

```json
{
  "realm_access": {
    "roles": [
      "it-admin",
      "offline_access",
      "uma_authorization",
      "default-roles-company-lab"
    ]
  },
  "preferred_username": "john.admin",
  "email_verified": true,
  "name": "John Admin",
  "email": "john.admin@company-lab.com"
}
```

**Key observation:** The `it-admin` role is embedded directly in the JWT 
token under `realm_access.roles`. Applications can make authorization 
decisions by reading this claim without making additional calls to Keycloak — 
this is stateless authorization, a core principle of modern IAM architecture.

All three users were authenticated successfully with HTTP 200 responses, 
each returning tokens containing their respective assigned roles:
- `john.admin` → `it-admin`
- `jane.finance` → `finance-user`
- `bob.external` → `external-user`

---

## Comparison — Keycloak vs Microsoft Entra ID

| Feature | Keycloak | Microsoft Entra ID |
|---|---|---|
| Protocol | OIDC, SAML, OAuth2 | OIDC, SAML, OAuth2 |
| Hosting | Self-hosted / EU cloud | Microsoft (US) |
| Data sovereignty | Full control | Subject to US CLOUD Act |
| Cost | Free / open source | Licensed (P1/P2 for advanced features) |
| Conditional Access | Via custom policies | Native (P1 required) |
| PIM | Via extensions | Native (P2 required) |
| MFA | Built-in | Built-in |
| JWT role claims | realm_access.roles | roles claim |
| Enterprise support | Red Hat (paid) | Microsoft (included) |

---

## Production Improvements

In a production environment the following would be added:

- Enable HTTPS — development mode runs HTTP only
- Configure MFA — Keycloak supports TOTP, WebAuthn, and passkeys natively
- Implement Conditional Access policies via Authentication Flows
- Connect to an external user store (LDAP/Active Directory)
- Enable Keycloak clustering for high availability
- Integrate with a secrets manager for client secret rotation
- Add audit logging and SIEM integration

---

## Relation to EU Digital Sovereignty

This lab was built in direct response to the European Union's accelerating 
push for digital sovereignty. Key context:

- The EU Data Act (September 2025) requires cloud providers to support 
switching and block unlawful third-country data access
- The European Commission awarded a €180 million sovereign cloud contract 
in April 2026 to European providers
- NIS2 and DORA require organisations to assess third-country risks in 
their technology supply chains
- France, Germany, Denmark and other EU member states are actively 
migrating public sector workloads away from US-based platforms

Keycloak, maintained by Red Hat (now part of IBM but with European 
operations), is the primary open-source IAM platform used in sovereign 
cloud deployments across Europe.

---

## Related Labs

- [Azure Entra ID Security Hardening Lab](https://github.com/galhinho/azure-entra-id-hardening-lab)
- [Azure Sentinel IAM Threat Detection Lab](https://github.com/galhinho/azure-sentinel-iam-detection)
- [Azure Infrastructure Deployment using Terraform](https://github.com/galhinho/azure-terraform-secure-infra)

---

## Author

Guilherme Alhinho
Cloud Security & IAM | Azure | Keycloak | Sentinel | KQL
[LinkedIn](https://linkedin.com/in/guilhermealhinho) |
[GitHub](https://github.com/galhinho)
