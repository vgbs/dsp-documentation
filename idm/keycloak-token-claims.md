---
title: Keycloak Token Claims
description: Keycloak Token Claims Mapping
---

[&larr; back to Overview](/idm)

## Keycloak Token Claims Overview

This document describes how Keycloak user attributes are mapped to JWT token claims. When a user authenticates, these claims are included in the access token or ID token based on the configured scopes.

For available user attributes, see [Keycloak Mapping](keycloak-mapping.md).  

### Available Client Scopes

| Scope | Type | Description |
|-------|------|-------------|
| `b2b` | Default (B2B) | Core B2B user claims |
| `b2c` | Default (B2C) | Core B2C user claims |
| `company` | Default (B2B) | Company-related claims |
| `classification` | Optional | Target group and classification claims |
| `loyalty` | Optional | Loyalty program claims |
| `offline_access` | Optional | Enables offline access tokens |

---

### Company Scope

The `company` scope provides access to company-related information. These claims appear in the **UserInfo** response with a nested structure.

#### Company Information (UserInfo)
| User Attribute | Token Claim | Description |
|----------------|-------------|-------------|
| `companyName` | `company.name` | Company Name |
| `companyCustomerNumber` | `company.customerNumber` | Company Customer Number |
| `companyWebsite` | `company.website` | Company Website |
| `companyTaxNumber` | `company.taxNumber` | Company Tax Number |
| `companyVatCode` | `company.vatCode` | Company VAT Code |

#### Company Address (UserInfo)
| User Attribute | Token Claim | Description |
|----------------|-------------|-------------|
| `companyAddressStreet` | `company.address.street` | Company Street Address |
| `companyAddressHouseNumber` | `company.address.houseNumber` | Company House Number |
| `companyAddressPostalCode` | `company.address.postalCode` | Company Postal Code |
| `companyAddressCity` | `company.address.city` | Company City |
| `companyAddressFloor` | `company.address.floor` | Company Floor |
| `companyAddressFlatNumber` | `company.address.flatNumber` | Company Flat Number |
| `companyAddressDistrict` | `company.address.district` | Company District |
| `companyAddressCounty` | `company.address.county` | Company County |
| `companyAddressCountry` | `company.address.country` | Company Country |
| `companyAddressExtension` | `company.address.extension` | Company Address Extension |

#### Company Contact (UserInfo)
| User Attribute | Token Claim | Description |
|----------------|-------------|-------------|
| `companyContactPhone` | `company.contact.phone` | Company Phone |
| `companyContactAdditionalPhone` | `company.contact.additionalPhone` | Company Additional Phone |
| `companyContactMobile` | `company.contact.mobile` | Company Mobile |
| `companyContactFax` | `company.contact.fax` | Company Fax |
| `companyContactEmail` | `company.contact.email` | Company Email |

#### Deprecated Claims (Access Token)

| User Attribute | Token Claim | Notes |
|----------------|-------------|-------|
| `companyName` | `companyName` | **Deprecated** - Use `company.name` from UserInfo instead. |

---

### B2B Scope

The `b2b` scope is a **default** scope for B2B realms that provides core user identity claims.

#### Access Token Claims
| User Attribute | Token Claim | Description |
|----------------|-------------|-------------|
| `firstName` + `lastName` | `name` | Full name (e.g., "Julian Installer") |
| `salesforceContactId` | `salesforceContactId` | Salesforce Contact ID |
| `salesforceAccountId` | `salesforceAccountId` | Salesforce Account ID |
| `salesforceBrandDetailContactId` | `salesforceBrandDetailContactId` | Salesforce Brand Detail Contact ID |
| `brandName` | `brandName` | Brand Name (lowercase, e.g., "vaillant") |
| `country` | `country` | Country Code |

#### Roles Claims

| Token Location | Claim Path | Description |
|----------------|------------|-------------|
| Access Token / ID Token | `realm_access.roles` | Realm roles (including Salesforce webroles) |
| Access Token / ID Token | `resource_access.{clientId}.roles` | Client-specific roles |
| UserInfo | `realm_access.roles` | Realm roles |

---

### Classification Scope

The `classification` scope is an **optional** scope that provides target group and classification claims.

| User Attribute | Token Claim | Description |
|----------------|-------------|-------------|
| `targetGroup` | `classification.targetGroup` | Target Group |
| `subTargetGroup` | `classification.subTargetGroup` | Sub Target Group |
| `preferredLanguage` | `classification.preferredLanguage` | Preferred Language |

> **Note:** Token claim names are assumed based on the naming convention. Verify with actual mapper configuration.

---

### Loyalty Scope

The `loyalty` scope is an **optional** scope that provides loyalty program claims.

#### UserInfo Claims
| User Attribute | Token Claim | Description |
|----------------|-------------|-------------|
| `salesforceLoyaltyId` | `salesforceLoyaltyId` | Salesforce Loyalty ID |

> **Note:** The loyalty claim appears at the root level in UserInfo, not nested.
