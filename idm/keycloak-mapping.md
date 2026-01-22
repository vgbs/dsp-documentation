---
title: Keycloak Mapping
description: Keycloak Mapping Overview
---

[&larr; back to Overview](/idm)

## Keycloak Mapping Overview

In Identity Management (IDM), Keycloak serves as the core identity and access management solution. Keycloak mapping refers to the configuration and customization of how user attributes, roles, and permissions are mapped between Keycloak and various applications or services integrated with IDM.

### Key Components of Keycloak Mapping

1. **User Attributes Mapping**: This involves defining how user attributes (e.g., username, email, roles) in Keycloak correspond to attributes in the connected applications. Proper mapping ensures that user information is accurately represented across systems.

For token claim mappings (how attributes appear in JWT tokens), see [Keycloak Token Claims](keycloak-token-claims.md).  

#### Available User Attributes

##### Salesforce Integration
| Attribute Key | Description |
|---------------|-------------|
| `salesforceAccountId` | Salesforce Account ID |
| `salesforceContactId` | Salesforce Contact ID |
| `salesforceBrandDetailContactId` | Salesforce Brand Detail Contact ID |
| `salesforceLoyaltyId` | Salesforce Loyalty ID |

##### Brand & Country
| Attribute Key | Description |
|---------------|-------------|
| `brandName` | Brand Name |
| `country` | Country |

##### Personal Information
| Attribute Key | Description |
|---------------|-------------|
| `title` | Title |
| `salutation` | Salutation |
| `firstName` | First Name |
| `lastName` | Last Name |
| `email` | Email Address |
| `username` | Username |
| `locale` | Keycloak Locale |
| `isEmailVerified` | Email Verification Status |
| `activationToken` | Activation Token |

##### Contact Information
| Attribute Key | Description |
|---------------|-------------|
| `phone` | Phone Number |
| `homePhone` | Home Phone Number |
| `mobile` | Mobile Number |
| `fax` | Fax Number |
| `company` | Company |
| `department` | Department |
| `function` | Function |

##### Personal Address
| Attribute Key | Description |
|---------------|-------------|
| `addressStreet` | Street Address |
| `addressPostalCode` | Postal Code |
| `addressCity` | City |
| `addressCounty` | County |
| `addressCountry` | Country |
| `addressExtension` | Address Extension |

##### Classification Scope
| Attribute Key | Description |
|---------------|-------------|
| `targetGroup` | Target Group |
| `subTargetGroup` | Sub Target Group |
| `preferredLanguage` | Preferred Language |

##### Company Scope
| Attribute Key | Description |
|---------------|-------------|
| `companyName` | Company Name |
| `companyCustomerNumber` | Company Customer Number |
| `companyWebsite` | Company Website |
| `companyTaxNumber` | Company Tax Number |
| `companyVatCode` | Company VAT Code |

##### Company Address
| Attribute Key | Description |
|---------------|-------------|
| `companyAddressStreet` | Company Street Address |
| `companyAddressHouseNumber` | Company House Number |
| `companyAddressPostalCode` | Company Postal Code |
| `companyAddressCity` | Company City |
| `companyAddressFloor` | Company Floor |
| `companyAddressFlatNumber` | Company Flat Number |
| `companyAddressDistrict` | Company District |
| `companyAddressCounty` | Company County |
| `companyAddressCountry` | Company Country |
| `companyAddressExtension` | Company Address Extension |

##### Company Contact
| Attribute Key | Description |
|---------------|-------------|
| `companyContactPhone` | Company Phone |
| `companyContactAdditionalPhone` | Company Additional Phone |
| `companyContactMobile` | Company Mobile |
| `companyContactFax` | Company Fax |
| `companyContactEmail` | Company Email |