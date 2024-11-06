---
title: Template Migrator Help
description: Getting Started Guide
---

[&larr; back to Overview](/templatemigrator)

## Template Migrator Getting Started Guide
#### How it works
1. Input and Output Directories: The tool reads HTML templates from the input_dir and writes the converted templates to the output_dir. If no directories are specified, it uses the default paths from config.json.
2. Merge Tags: The tool converts Mandrill/Mailchimp merge tags to Brevo-compatible format.
3. Conditional Blocks: Conditional logic in templates (e.g., `````*|IF|*`````) is transformed to Brevo's syntax.

##### Your steps
1. Download the latest release binary of the [template-migrator tool](https://github.com/vgbs/template-migrator).
2. You can either specify the input_dir and output_dir (input/output directories) via command-line arguments, or rely on the defaults provided in config.json.
3. You also need to provide your config.json in command-line arguments. You can find an example below.

##### Linux
Ensure the binary file has execute permissions. You can grant them with:

```chmod +x /path/to/template-migrator-linux-amd64```

Then run the script from the terminal:

```./template-migrator-linux-amd64 --input_dir /path/to/input --output_dir /path/to/output --config /path/to/config.json```

##### Windows
No special actions are needed. Simply run the .exe file. You may do that either with double-click or via terminal:

```template-migrator-windows-amd64.exe --input_dir C:\path\to\input --output_dir C:\path\to\output --config /path/to/config.json```

##### MacOS
If encountering access not permitted error, macOS users may need to bypass Gatekeeper security features by running the following command in terminal:

```sudo xattr -rd com.apple.quarantine /path/to/template-migrator-macos-amd64```

Then run the script from the terminal:

```./template-migrator-macos-amd64 --input_dir /path/to/input --output_dir /path/to/output --config /path/to/config.json```

##### Note
Linux and macOS binaries names may end with arm64 instead of amd64. It depends on which architecture you need.

### config.json example:
```{
  "mappings": {
    "*|FNAME|*": "params.first_name",
    "*|LNAME|*": "params.last_name",
    "*|EMAIL|*": "params.email",
    "*|PHONE|*": "params.phone_number",
    "*|ADDRESS|*": "params.address",
    "*|LIST:NAME|*": "params.audience_name",
    "*|LIST:COMPANY|*": "params.company_name",
    "*|LIST:SUBSCRIBERS|*": "params.subscriber_count",
    "*|USER:COMPANY|*": "params.user_company",
    "*|CURRENT_YEAR|*": "params.current_year",
    "*|UNSUB|*": "params.unsubscribe_url",
    "*|UPDATE_PROFILE|*": "params.update_profile_url",
    "*|OPTIN_DATE|*": "params.opt_in_date",
    "*|MC:SUBJECT|*": "params.subject",
    "*|MC:PREVIEW_TEXT|*": "params.preview_text",
    "*|MC_LANGUAGE|*": "params.language_code",
    "*|MC_LANGUAGE_LABEL|*": "params.language_label",
    "*|MC_PREVIEW_TEXT|*": "params.preview_text",
    "*|ARCHIVE|*": "params.archive_url",
    "*|POLL:RATING:x|*": "params.poll_rating",
    "*|SURVEY|*": "params.survey",
    "*|REWARDS|*": "params.rewards_badge",
    "*|REWARDS_TEXT|*": "params.rewards_text",
    "*|PROMO_CODE:[$store_id=x, $rule_id=x, $code_id=x]|*": "params.promo_code",
    "*|SUBJECT|*": "params.subject",
    "*|LIST:ADDRESS|*": "params.address",
    "*|MERGE|*": "params.merge"
  },
  "input_dir": "templates/original",
  "output_dir": "templates/converted"
}
```