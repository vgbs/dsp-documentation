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

## Using templates

Before you are able to send out an E-Mail template in prod, it needs to be activated in Brevo.

If you plan to use templates using the Proxy, please note that the following limitations:

### Template Attributes

Brevo offers a way to template attributes like the sender, Reply-To, Subject, etc. Currently, our E-Mail Proxy does not take these attributes actively into consideration.
When sending out templates, we mandate the use of required headers, and pass it to the Brevo / Mailpit API, therefore overwriting the templated attributes.
In some situations, the template attributes might be implicity used in the Prod Environment.
This might happen for some optional parameters (e.g. if attachments are specified in the template but not passed to the Proxy). 
At the moment, it is only safe to assume that the templated Body is used.

### Templating in QA

Templating in QA is not fully compliant with the templating in Brevo.
The reason for that is, that we're fetching the template and render it using custom logic, which might lead to a slightly different result.

Currently, the following limitations are known on QA:

**Implementation Differences**
- The `truncatechars` filter puts a whitespace in front of the truncation

**Placeholders**
Brevo predefines placeholders, that are not known to us during template rendering. Basically, we are only able to access the `params` variables.
All unknown variables will be omitted, including:
{% raw %}
- Contact attributes like `{{contact.FIRSTNAME}}`, `{{contact.EMAIL}}`
- `{{mirror}}`
- `{{unsubscribe}}`
- `{{update_profile}}`
{% endraw %}

**Unsupported Filters**
- `time_parse`
- `date`
- `date_i18n`
- `time_add_date`
- `time_in_location`

### Important for QA
* It is not possible to send out emails from QA to real inboxes.
* If your account exceeds more than 1000 templates, only the first 1000 can be used.
* All templates are cached for 3 minutes, meaning updates and creations of templates take 3 minutes to be available for use in the proxy.