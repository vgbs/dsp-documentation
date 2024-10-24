---
title: Transactional Email Service Help
description: REST API Integration Guide
---

[&larr; back to Overview](/email)

## REST Integration Guide
Integration with Team Trixie's REST Proxy is relatively straightforward:

* You can generate an API Key for your Subaccount from within the Brevo UI
* The API Key can be used to integrate with our E-Mail proxy by providing it in a header field. Consult the [API Documentation](api-documentation.html) on the usage of the API
* Afterward you can access the E-Mail Proxy API using:

QA:
```
Base URL: https://email-proxy-qa.dsp.vaillant-group.com/api
```
Prod:
```
Base URL: https://email-proxy.dsp.vaillant-group.com/api
```

* Mailpit UI can be accessed from [https://mailpit.qa.dsp.vaillant-group.cloud/](https://mailpit.qa.dsp.vaillant-group.cloud/) (Internal) and [https://mailpit-qa.dsp.vaillant-group.com/](https://mailpit-qa.dsp.vaillant-group.com/) (External).


### Important 
* It is not possible to send out emails from QA to real inboxes.