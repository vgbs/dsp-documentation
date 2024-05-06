---
title: User Import
description: User Import from Legacy Systems (B2C Only)
---

[&larr; back to Overview](/idm)

We offer a way to import users from legacy systems into our IDM for B2C users.
This import is done manually and requires an export file we can use to import these users in our system. 

The following lightweight JSON scheme using comments as description should show the file structure.

```json
[
  {
    // One of: AL,AT,BA,BE,BG,CH,CZ,DE,DK,EE,ES,FI,FR,GB,GE,GR,HR,HU,IT,LT,LU,LV,MD,MK,NL,NO,NZ,PL,PT,RO,RS,SE,SI,SK,TR,UA,UZ,XK
    "countryCode": "string",
    // One of: AWB, Bulex, DemirDoekuem, Glow-worm, HermannSaunierDuval, Protherm, SaunierDuval, Vaillant
    "brand": "string",
    "emailAddress": "string",
    "salesforceBrandDetailContactId": "string",

    // Supported: PBKDF2-SHA-512 / Bcrypt
    "hash": "string",
    "salt": "string",
    "hashIterations": 0,
    
    // Optional properties
    "locale": "string",
    "firstName": "string",
    "lastName": "string",
  },
  // ...
]
```

An import can then be requested and coordinated using our Helpdesk. 
For further questions and customizations, also just use our Helpdesk.