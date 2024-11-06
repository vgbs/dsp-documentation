---
title: Transactional Email Service Help
description: SMTP Integration Guide
---

[&larr; back to Overview](/email)

## Brevo SMTP (preferred)

The preferred way to integrate with SMTP is by utilizing Brevo SMTP directly where possible. 
Follow the [SMTP relay integration guide](https://developers.brevo.com/docs/smtp-integration) to integrate with SMTP within your subaccount.

Currently, we do not provide access to the Mailpit SMTP directly. 
If you need a shared sandbox for QA integration, the current way to go would be to self-host an instance on your own. 
Here are some examples:
- [Mailpit](https://github.com/axllent/mailpit)
- [Mailcatcher](https://mailcatcher.me/)
- [Mailhog](https://github.com/mailhog/MailHog)

## SMTP Integration Guide (legacy)

In addition to that, we also provide a centralized SMTP relay, though there are some details we need to pay attention to.

* Currently, The SMTP relay is only able to utilize our DSP subaccount and can not be scaled out to other Sub-Accounts.
* You need to provide us the "tenant" name of the party to integrate. It can be either the team name or application name and will act as the SMTP username.
* You need to provide us the desired sender address, so we can ensure that the domain is authenticated (currently only `vaillant-group.com`). Although our custom SMTP relay does not currently mandate a sender, it is important property to ensure reliability and delivery. We will enforce that property in the future.
* You will receive SMTP username and password.
* Afterward you can access the relays using:

QA (Mailpit):
```
Server: smtp-relay.qa.dsp.vaillant-group.cloud
Port: 587 (STARTTLS)
```
Prod (Brevo):
```
Server: smtp-relay.prod.dsp.vaillant-group.cloud
Port: 587 (STARTTLS)
```
* Mailpit UI can be accessed from [https://mailpit.qa.dsp.vaillant-group.cloud/](https://mailpit.qa.dsp.vaillant-group.cloud/) (Internal) and [https://mailpit-qa.dsp.vaillant-group.com/](https://mailpit-qa.dsp.vaillant-group.com/) (External).

### Whitelist feature
We are currently providing you the option to have some recipient emails whitelisted on QA, so instead of the message going directly to Mailpit, we can send it to real inbox.
If you need such feature for your testing purposes, then please create a ticket in DSP helpdesk and we will configure that for you.

### Important 
* The servers are only available from networks, which are peered to dsp-network at the moment.
* It is now possible to send out emails from QA to real inboxes - whitelisted emails only.