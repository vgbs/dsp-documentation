---
title: Identity Management Help
description: Developer Documentation
---

[&larr; back to Overview](/idm)

## How To Start

What we need from you to set everything up:

* Are you using authentication in a frontend, a backend or both?
* Which Realms do you need access to:
    * Brand
    * Country
    * Environment (development / QA / production)
* List of your redirect URIs separated by realm
* What tech stack are you using
* Whether you want to use backend or frontend integration (see below)

What you will get from us:

Basically all information that is needed in the oauth configuration of your tech stack. Namely:

* The SSO base URLs
* Your client-id
* Your client secret (in case of backend integration)

## Backend vs Frontend Integration

After the user logs in with her credentials,
an [auth code](https://auth0.com/docs/get-started/authentication-and-authorization-flow/authorization-code-flow) is
returned and must be exchanged for an access token. This can be done by the back- or frontend of your application.

While both approaches are valid, please consider the pros and cons in your particular application and let us know which
way you choose. In case of backend integration, you will receive an additional client secret from us.

### Frontend Integration

When your frontend renders the redirect page (e.g https://your-domain.de/auth/callback) after the user logs in, the auth
code is picked up from the url and can then be exchanged via an AJAX call to the IDM Server for an access- and id token.
The frontend application is responsible for storing the tokens e.g. in local storage and refresh the access token,
whenever it expires.

### Backend Integration

Your backend application is called after the user logs in. The backend picks up the auth code from the url. It then
sends it together with
its [client-id and client-secret](https://www.oauth.com/oauth2-servers/client-registration/client-id-secret/), receives
an access token and typically creates a server-side session where the access token is stored. It then sends back the
sessionId to the frontend. The frontend doesn't have to handle refresh or access tokens.

## SSO Adapters

We are using a standard OAuth2 SSO implementation. Adapters are available for
most programming languages and technology stacks. There is not one generally
valid documentation for all. Generally, a good starting point is the offical
Keycloak Integration documentation which provides tutorials for different tech
stacks:

[Keycloak Securing Applications and Services Guide](https://www.keycloak.org/docs/latest/securing_apps/)

As we are supporting teams to implement, we will add more and more links to
available documentations:

* Java, Spring Boot: [Official Docs](https://spring.io/guides/tutorials/spring-boot-oauth2/)

### Javascript Adapter

If you're integrating SSO on the frontend side of your Application (i.e. your frontend application will exchange the
auth code for an access token and handle token refreshs), you can use
the [Keycloak Javascript adapter](https://www.keycloak.org/docs/latest/securing_apps/#_javascript_adapter).
Feel free to use the configuration below as a starting point for you configuration:

```javascript
const keycloak = new Keycloak({
    url: 'https://identity-qa.vaillant-group.com/auth',
    realm: 'your-realm',
    clientId: 'your-qa-client-id',
    "enable-pkce": true
});

keycloak.init({onLoad: 'login-required', "pkceMethod": "S256"})
    .then(function (authenticated) {
        if (!authenticated) {
            alert('not authenticated');
        } else {
            console.log(keycloak.idTokenParsed);
            document.getElementById('name').innerHTML = keycloak.idTokenParsed.name;
        }
    }).catch(function (e) {
        console.log(e)
        alert('failed to initialize');
    });
```

## Available Information in the Token and the Userinfo

The JWT Tokens you get from the SSO contains several Vaillant-specific pieces
of information about the user. The following information are provided
automatically in each JWT:

```yaml
{
  "exp": int, # Expiration of this token (unix timestamp)
  "iat": int, # Issued at (unix timestamp)
  "auth_time": int, # time of authentication
  "jti": UUID, # Unique ID of this token
  "iss": String, # Issuer (URL of the originating REALM)
  "sub": String, # Internal USER id

  # a few keycloak internals are skipped here

  "realm_access": {
    "roles": [ # all salesforce web roles assigned to the contact (name and id)
      "Kompetenzpartner Programm 2019-2020",
      "a5p1r0000000DYjAAM",
      "HeizungOnline",
      "a5p690000008TvkAAE"
      # ...
    ]
  },
  # The following information are Vaillant specific
  # They are added to each token if the corresponding information
  # can be found in Salesforce. If not, the key will not be added
  # to the token.
  "salesforceContactId": "6301",
  "country": "DE",
  "brandName": "Vaillant",
  "email_verified": true,
  "companyName": "First-Last GmbH", # deprecated - please use company.name from openid-connect userinfo endpoint (see below)
  "preferred_username": "user@example.org",
  "locale": "de",
  "given_name": "First",
  "name": "First Last",
  "family_name": "Last",
  "email": "user@example.org",
  "salesforceAccountId": "1234",
  "salesforceBrandDetailContactId": "7981"
}
```

Additionally, the SSO provides a openid-connect userinfo endpoint to retrieve
additional information about a given user. As with the JWT, this userinfo
enpoint only provides information that is present in salesforce. If it is
missing, the key is omitted.

In case you need additional information in the userinfo response, feel free to
contact the team via [DSC service desk](https://service.dsp.vaillant-group.com).

These information are contained in the response of this endpoint:

### B2B

```yaml
{
  "salesforceContactId": "6301",
  "sub": String, # Internal USER id
  "country": "DE",
  "salesforceLoyaltyId": "9955",
  "brandName": "vaillant",
  "email_verified": true,
  "preferred_username": "user@example.org",
  "locale": "de",
  "given_name": "First",
  "realm_access": {
    "roles": [
      "Kompetenzpartner Programm 2019-2020",
      "a5p1r0000000DYjAAM",
      "HeizungOnline",
      "a5p690000008TvkAAE"
    ]
  },
  "name": "First Last",


  "company": { # Company data = Salesforce account
    "website": "example.org",  # if set the brand specific website, otherwise website from the account
    "address": {
      "country": "DE",
      "city": "Stadthausen",
      "street": "Straßenweg",
      "houseNumber": "123",
      "floor": "3",
      "flatNumber": "1B",
      "district": "Downtown",
      "county": "Example county",
      "postalCode": "12345",
      "extension": "B1337"
    },
    "contact": {
      "phone": "+49 123456",
      "additionalPhone": "+49 654321",
      "fax": "12345",
      "mobile": "123123213",
      "email": "user@example.org"
    },
    "name": "First-Last GmbH",
    "customerNumber": "11223",
    "taxNumber": "1337",
    "vatCode": "1337"
  },
  "title": "Dr",
  "salutation": "Herr",
  "family_name": "First",
  "email": "user@example.org",
  "phone": "1234",
  "mobile": "321313",
  "fax": "583585",
  "street": "Main street",
  "postalCode": "12345",
  "city": "Stadthausen",
  "county": "County1",
  "countryName": "DE",
  "extension": "abc",
  "salesforceAccountId": "1234",
  "salesforceBrandDetailContactId": "3344"
}
```

### B2C

```yaml
{
  "salesforceContactId": "6301",
  "sub": String, # Internal USER id
  "country": "DE",
  "brandName": "vaillant",
  "email_verified": true,
  "preferred_username": "user@example.org",
  "locale": "de",
  "given_name": "First",
  "name": "First Last",
  "title": "Dr",
  "salutation": "Herr",
  "family_name": "First",
  "email": "user@example.org",
  "phone": "1234",
  "mobile": "321313",
  "fax": "583585",
  "street": "Main street",
  "postalCode": "12345",
  "city": "Stadthausen",
  "county": "County1",
  "countryName": "DE",
  "salesforceAccountId": "1234",
  "salesforceBrandDetailContactId": "3344"
}
```

## Webhooks

Webhooks are used to notify consuming services of certain events. Currently, we notify our consumers, whenever a B2C user
account is deleted. 
In order to become a consumer, please let us know the WEBHOOK_URL we should call, so we can set it up for you.
If you are looking for other events, such as email address updates or logins, please let us know.

The webhook url will receive POST requests of the format below:
```http request
POST <WEBHOOK_URL>
Content-Type: application/json
{
  "action": "user_delete",
  "userId": "user-id",
  "deletedAt": "2020-12-31T10:15:30Z"
  "deletionKind": "soft"
}
```
**Note**
In case of a hard deletion, "deletionKind" will have value "hard".
