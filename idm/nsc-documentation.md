---
title: Identity Management Help
description: NSC Documentation
---

[&larr; back to Overview](/idm)

## Realm Configuration

A realm is the combination of environment (QA or production), brand, country and user group - for example Saunier Duval France for B2C on Production. All applications in this context are connected with this realm so that a user has a real Single Sign On experience across all his relevant frontends.

The following configurations can be made for a realm:

* Realm display name (seen in the tab title)
* Title picture
* Link to registration page (also see chapter User Activation)
* Link to imprint
* Link to privacy statement
* Link to terms of use
* Sender name of outgoing e-mails (e-mail address identity@vaillant-group.com can not be changed)
* Reply-to address of outgoing e-mails

If you like to change configurations for your realm, please write an e-mail to identity@vaillant-group.com or leave a post into our Slack channel #idm-support

Translations are language specific and not country specific. German is shared accross Germany, Switherland and Austria. See more in chapter Translations.

Logo and design is brand specific and usually set by central marketing.

## Translations

All texts on the shown frontends and in the outgoing e-mails can be adapted by yourself. To get access to the translation tool, please write to identity@vaillant-group.com or leave a message in our Slack channel #idm-support.

Please be aware that translations are always valid for a complete language and might also affect other realms and countries.

## User Activation

Users can only get activated if all required pre-conditions in Salesforce are fulfilled. A user always needs:

* An account (usual B2B or person account for B2C)
* Account Brand Details
* A contact
* Contact Brand Details
* Web Login Permissions
* Deleted flag must be inactive

The user registration and the user activation are two different steps. The registration is usually handled by the partnerNET and the lead management process in Salesforce for B2B or by myVAILLANT Web for B2C.

The user activation can be done automatically or manually via the lead management process or the Contact object in Salesforce for B2B or by myVAILLANT Web for B2C.

In both cases the user will get the activation e-mail after.

## User Support

As the identity management is a complex system also many different types of errors can occur. Most of them are problems with the login, the activation or the password reset. For those cases you can directly look up the e-mail address or a Salesforce ID in our customer history log. For access please write to identity@vaillant-group.com or leave a message in our Slack channel #idm-support