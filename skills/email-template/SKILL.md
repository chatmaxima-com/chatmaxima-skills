---
name: email-template
description: Write or edit a reusable email template in ChatMaxima for bots and automations to send. Use when the user wants a welcome, confirmation, follow-up or notification email that a ChatMaxima bot sends, or wants to change the wording of one.
---

# Email templates in ChatMaxima

Email templates need no approval and are ready as soon as they are saved. A bot sends one through a Send Email block.

## Create one

1. `list_email_templates` returns the existing templates and the workspace's email accounts. A template belongs to one email account: ask the user which one if there are several. If there are none, the user has to connect an email account in ChatMaxima first.
2. Draft the subject and the HTML body. Use `{variables}` with single braces for values filled in at send time, for example `{customer_name}` or `{booking_date}`. Use only variables the bot actually collects or stores.
3. Keep it short and plain: one purpose per email, the key detail near the top, and one clear next step.
4. Show the draft to the user, then call `create_email_template`.
5. To send it from a bot, add or update a Send Email block with `broadcast_type: "email_template"` and the template's `email_template_alias` (see the `build-chatbot` skill).

## Edit one

1. `get_email_template` to read the current subject and body.
2. For a small change, send `replace` edits to `update_email_template` instead of a whole new body.
3. Send Email blocks keep their own copy of the subject and body. After editing a template, update each block that uses it with `update_block`, setting `email_template_alias` again, or the bots keep sending the old wording.
