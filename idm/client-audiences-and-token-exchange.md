---
title: Client Audiences and Token Exchange
description: How to configure client audiences and token exchange in IDM
---

[&larr; back to Overview](/idm)

## What is an Audience?

An audience (`aud`) is a claim embedded in tokens that specifies the intended recipient(s) of the token that may contain multiple audiences. It prevents token replay attacks (e.g., a token for Service A being used at Service B) and ensures that only the correct client or resource server will accept the token.
In short: The audience defines "who this token is for."

### Why Audience Matters

1. **Security Boundary and Trust Relationship**: Prevents tokens from being used outside their intended scope.
2. **Client Isolation**: Ensures one client cannot use tokens issued for another.
3. **API Protection**: Ensures APIs only accept tokens explicitly minted for them.

When a client or resource server receives a token, it must validate the aud claim and must check that its identifier is included in the aud claim.

> ##### WARNING
>
> Currently, IDM only supports setting other IDM clients as audiences.


## What is Token Exchange?

Token Exchange is a protocol that allows a client to exchange one token for another, typically to obtain a token with a different audience or scope. This is useful in scenarios where a service needs to act on behalf of a user or another service, and it needs to obtain a token that is valid for the target service.

In many real-world systems, a single token is not always enough.
- A client may need a token for a different audience than the one it originally requested.
- A service acting on behalf of a user might need a new token for another downstream service.
- A machine-to-machine workflow may require token delegation or impersonation.

The OAuth 2.0 Token Exchange specification ([RFC 8693](https://www.rfc-editor.org/rfc/rfc8693.html)) defines a standardized way to request a new security token by presenting an existing one. This mechanism is sometimes referred to as "token delegation" or "token impersonation".

Client Audiences enable the use of Token Exchange in IDM, allowing clients to request tokens for different audiences securely as it ensures that the token is only valid for the intended audience.

### Example Request

```
POST https://identity-qa.vaillant-group.com/auth/realms/<REALM_ID>/protocol/openid-connect/token HTTP/1.1
Host: auth.example.com
Content-Type: application/x-www-form-urlencoded

grant_type=urn:ietf:params:oauth:grant-type:token-exchange&
subject_token=eyJhbGciOi...&
subject_token_type=urn:ietf:params:oauth:token-type:access_token&
requested_token_type=urn:ietf:params:oauth:token-type:access_token&
audience=example-client
```

### Example Response

```
{
  "access_token": "eyJhbGciOi...",
  "issued_token_type": "urn:ietf:params:oauth:token-type:access_token",
  "token_type": "Bearer",
  "expires_in": 3600,
  "scope": "read write"
}
```

## Token Exchange Flow

Assuming we have 3 clients (A, B, C) in place.

- **Client A** authenticates the user Alice and passes the token to ingrate with **Client B** 
- **Client B** receives a token issued to and can perform actions on behalf the user.
- **Client B** is now re-using the token and sends it to **Client C**.
- **Client C** validates the token but rejects it because the audience does not match its own Client ID.
- **Client B** must request a new token using Token Exchange to obtain a token that is valid for **Client C**.

This is where client audiences come into play. We can configure Client A so that it includes audiences ("aud" claim) in its token which protects against this misuse. The claim includes all the Client IDs the token is meant to be used for. Client libraries will then validate the audience and reject a request with a non-matching audience.

![Token Exchange Flow](/idm/token-exchange-flow.png)

## How to configure Client Audiences and Token Exchange

Create a [Helpdesk ticket](https://service.dsp.vaillant-group.com) to request the configuration of client audiences and token exchange for your application. The DSP team will assist you with setting up the necessary client audiences in IDM.
