---
title: Identity Management Help
description: Security Admin Panel Documentation
---

[&larr; back to Overview](../index.md)

## Security Admin Panel

Security Admin Panel was created to enable taking active actions against impacts of cyberattacks and data breaches that would otherwise affect the IDM end users.
The panel provides means to block the users affected, so that a potential attack can be prevented.

## Temporary User Blockage

To temporarily block the user, so that the login is possible only after resetting credentials follow the steps below:
1. Search for the user concerned using the Block User section of the [Identity Management Dashboard](https://dashboards.dsp.vaillant-group.cloud/):

    ![Search for the user concerned using the IDM dashboard.](/idm/security/user-search.png "User Search")
2. Wait for the search results to be displayed - the list would be shortly filled with per-realm results for the email provided:

    ![Search results at the Security Admin Panel.](/idm/security/user-search-results.png "User Search Results")
3. Provide the reason for the blockage, or any other information sufficient to make the blockage transparent to all parties involved:

    ![Provide the reason for the blockage.](/idm/security/modifying-blockage-reason.png "Modifying Blockage Reason")
4. If needed, modify the number of days, for which the reset password email link would be valid:

    ![Optionally, modify the reset password email link expiration time.](/idm/security/modifying-expiration-time.png "Modifying Expiration Time")
5. Click the "Block" button - that's it! :)

The Temporary User Blockage logic works as follows:
* The current user credentials are shuffled, in other words the user password is set to a random value,
* All current user sessions are invalidated, in other words the user is logged out from all associated IDM clients (apps),
* An email containing a password reset link is sent out to the users inbox, so that the user can reset the password and log back in.

## How does the Security Panel triggered reset password email look like?

As for today, we're using a default Keycloak generated email message, which looks like this:

![Security Admin Reset Password Email Example.](/idm/security/reset-password-email.png "Reset Password Email")

## Permanent User Blockage

This feature is not yet available.

## Get Access To The Security Admin Panel

If you'd want to be given access to the Panel, please request it here: [DSC Service Desk](https://service.dsp.vaillant-group.com) 
