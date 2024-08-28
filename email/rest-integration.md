---
title: Transactional Email Service Help
description: REST API Integration Guide
---

[&larr; back to Overview](/email)

## REST Integration Guide
Integration with Team Trixie's REST Proxy is relatively straightforward, though there are some details we need to pay attention to.

* You need to provide us the "tenant" name of the party to integrate. It can be either the team name or application name and will act as the SMTP username.
* You need to provide us the desired sender address, so we can ensure that the domain is authenticated (currently only `vaillant-group.com`). Although our custom SMTP relay does not currently mandate a sender, it is important property to ensure reliability and delivery. We will enforce that property in the future.
* You will receive API username and API Key.
* Consult the [API Documentation](api-documentation.html) on the usage of the API
* Afterward you can access the E-Mail Proxy API using:

QA:
```
Base URL: https://email-proxy.qa.dsp.vaillant-group.cloud/api
```
Prod:
```
Base URL: https://email-proxy.prod.dsp.vaillant-group.cloud/api
```

* Mailpit UI can be accessed from [https://mailpit.qa.dsp.vaillant-group.cloud/](https://mailpit.qa.dsp.vaillant-group.cloud/) (Internal) and [https://mailpit-qa.dsp.vaillant-group.com/](https://mailpit-qa.dsp.vaillant-group.com/) (External).


### Important 
* The servers are only available from networks, which are peered to dsp-network at the moment.
* It is not possible right now to send out emails from QA to real inboxes.