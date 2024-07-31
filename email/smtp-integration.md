---
title: Transactional Email Service Help
description: SMTP Integration Guide
---

[&larr; back to Overview](/email)

## SMTP Integration Guide
Integration with Team Trixie's SMTP is relatively straightforward, though there are some details we need to pay attention to.

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
* Mailpit UI can be accessed from `https://mailpit.qa.dsp.vaillant-group.cloud/` at the moment, but it will be protected by Vaillant SSO in the near future.


### Important 
* The servers are only available from networks, which are peered to dsp-network at the moment.
* It is not possible right now to send out emails from QA to real inboxes.