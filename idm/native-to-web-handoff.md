---
title: Native to Web Handoff
description: Opening Authenticated WebViews from Native Apps
---

[&larr; back to Overview](/idm)

If your native app needs to open a web application in a WebView or in-app browser tab without requiring the user to log in again, this guide explains the supported approach.
 
## Background
 
Native apps and web apps manage sessions differently. Native apps hold OAuth tokens; web apps use browser cookies. These two worlds do not mix — an embedded WebView has no access to your app's tokens or the system browser's cookies, so a plain redirect to a web app will always show a login screen.
 
The supported solution is _OAuth 2.0 Token Exchange_ (RFC 8693), combined with a session bootstrap on the target application's backend.
 
## How It Works
 
```
Native App
  → opens WebView, passes its access token (e.g. as a URL parameter)
 
Target Web App (Backend)
  → receives the token
  → calls Keycloak token exchange endpoint
  → receives a new access token scoped to the target client
  → creates a web session (sets session cookie)
  → serves the authenticated page
```
 
The user lands directly in the authenticated state. No second login prompt appears.
 
## What You Need to Implement
 
### Native App Side
 
Pass your current access token when opening the WebView. The simplest approach is a short-lived URL parameter:
 
```
https://your-web-app.example.com/auth/bootstrap?token=<access_token>
```

Keep the token short-lived and treat the bootstrap URL as sensitive — do not log it.
 
### Web App Backend (Session Bootstrap Endpoint)
 
Your backend needs an endpoint that:
 
1. Receives the access token from the incoming request.
2. Calls the Keycloak token exchange endpoint to validate the token and obtain one scoped to your client.
3. Creates a server-side session and sets a session cookie.
4. Redirects the user to the intended page.
 
**Keycloak token exchange request:**

Also see [Client Audiences and Token Exchange](client-audiences-and-token-exchange.md) for more details on token exchange.

```
POST /realms/{realm}/protocol/openid-connect/token
 
grant_type=urn:ietf:params:oauth:grant-type:token-exchange
&subject_token=<access_token_from_native_app>
&subject_token_type=urn:ietf:params:oauth:token-type:access_token
&requested_token_type=urn:ietf:params:oauth:token-type:access_token
&client_id=<your_client_id>
&client_secret=<your_client_secret>
```
 
Keycloak will validate the incoming token and return a token issued to your client. Use that to establish the session.
 
> **Note:** Token exchange must be explicitly enabled in Keycloak for your client. Contact the IDM team to configure this.
 
## Logout
 
Token exchange does not propagate logout automatically. You are responsible for:
 
- Invalidating the web session when the user logs out in the native app (e.g. via a backchannel logout endpoint).
- Ensuring session lifetime on the web side does not outlive the native app's session.
 
## Getting Started
 
Request the following from the IDM team to get started:
 
- Enable token exchange for your Keycloak client.
- Confirm allowed token subjects (which client IDs may exchange tokens into yours).
- Review your session bootstrap implementation before going to production.