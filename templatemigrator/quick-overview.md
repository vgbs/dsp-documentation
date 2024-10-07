---
title: Template Migrator Help
description: Quick Overview
---

[&larr; back to Overview](/templatemigrator)

## Template Migrator Quick Overview
This tool converts Mailchimp/Mandrill HTML templates into Brevo-compatible templates by replacing Mailchimp-specific tags with Brevo's parameters and conditional logic.

## Features

- Convert Mailchimp merge tags (e.g., `*|FNAME|*`) to Brevo format (`{{ params.first_name }}`)
- Handle conditional blocks (e.g., `*|IF:MC_PREVIEW_TEXT|*`) with Brevo's templating language
- Customizable tag mappings via `config.json`

## Usage
This tool is available as binary file for linux, macOS and Windows. You can find more information here:
[Getting Started Guide](getting-started.md)
