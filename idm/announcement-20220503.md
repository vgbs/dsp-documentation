---
title: Identity Management Help
description: SSO Announcement 03.05.2022
---

[&larr; back to Overview](/idm)

## OpenID Connect Logout

Previous versions of Keycloak had supported automatic logout of the user and redirecting to the application by opening logout endpoint URL such as

```
http(s)://example-host/auth/realms/my-realm-name/protocol/openid-connect/logout?redirect_uri=encodedRedirectUri
```

While that implementation was easy to use, it had potentially negative impact on performance and security. The new version has better support for logout based on the OpenID Connect RP-Initiated Logout specification. The parameter `redirect_uri` is no longer supported; also, in the new version, the user needs to confirm the logout. It is possible to omit the confirmation and do automatic redirect to the application when you include parameter `post_logout_redirect_uri` together with the parameter `id_token_hint` with the ID Token used for login.

The existing deployments are affected in the following ways:
* If your application directly uses links to logout endpoint with the `redirect_uri` parameter, you may be required to change this as described above. Consider either removing the `redirect_uri` parameter entirely or replacing it with the `id_token_hint` and `post_logout_redirect_uri` parameters.
* If you use java adapters and your application does logout by call `httpServletRequest.logout()`, you are not affected because this call uses the backchannel variant of the logout endpoint and that one was not changed.
* If you use the latest javascript adapter, you are also not affected. However if your application uses an older version of the JavaScript adapter, you are affected as this adapter uses the variant of the logout endpoint with the deprecated ´redirect_uri´ parameter. In this case, you may need to upgrade to the latest version of the JavaScript adapter.
* For the Node.js adapter, the same guideline applies as for the JavaScript adapter. You are encouraged to update to the latest version as the older version of the adapter uses the deprecated `redirect_uri` parameter. With the latest Node.js adapter, you are not affected as long as you use the logout based on the /logout URL as described in the documentation or in the Node.js adapter example. However, in the case when your application directly uses the method `keycloak.logoutUrl`, you can consider adding `idTokenHint` as the second argument to this method. The possibility to add `idTokenHint` as second argument was newly added in this version. The `idTokenHint` needs to be a valid ID Token that was obtained during the login. Adding `idTokenHint` is optional, but if you omit it, your users will need to confirm the logout screen as described earlier. Also they will not be redirected back to the application after logout.