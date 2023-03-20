---
title: Identity Management Help
description: Login Issues
---

[&larr; back to Overview](/idm)

On issues with the usual login of customers you can follow the steps below to investigate on a reason why there is a problem with the specific login of the customer.

## Ask For The Support ID

After a failed login of the customer ask for the support ID. The customer will find it on the login page at the bottom in light grey:

![The support ID on the login page.](/support-id.jpg "Support ID")

If your customer already left the login page after the failed try, please ask him for trying to login again and sending you the support ID, if it failed again.

## Check Support ID In Elastic

Elastic is our logging system. Here you can easily search for the support ID and get anonymous information about the login with potential reasons why it could have failed. Log in with your Elastic user, choose the "IDM Logs"-Dashboard and search for the support ID the customer gave you:

[Elastic IdM Log](https://logs.idm.vaillant-group.com/)

![Search for support ID in Elastic.](/idm/elastic-support.png.png "Support")

You find possible reasons for the failed login in the message column of the log entry.

## Get Access To Elastic

If you like to have an Elastic user, please write a short e-mail to identity@vaillant-group.com or write a message into our Slack channel #idm-support.