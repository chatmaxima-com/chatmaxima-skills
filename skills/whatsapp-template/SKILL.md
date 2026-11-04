---
name: whatsapp-template
description: Write a WhatsApp message template in ChatMaxima and submit it to Meta for approval. Use when the user wants a WhatsApp template for order updates, reminders, offers, one-time login codes, or a carousel, wants to message customers outside the 24-hour window, or asks why a template was rejected.
---

# Write and submit a WhatsApp template

A business can message a customer freely only within 24 hours of the customer's last message. After that, only a Meta-approved template can open the conversation. That is what this skill makes.

## 1. Gather the brief

- Which WhatsApp number it is for: `list_whatsapp_accounts`. Ask if there is more than one.
- What it should say, and what changes per customer (name, order number, date).
- The category:
  - **UTILITY**: an update about something the customer already did, such as an order, booking, payment or account change.
  - **MARKETING**: offers, announcements, anything promotional. Meta can move a "utility" template that contains promotion into marketing.
  - **AUTHENTICATION**: one-time codes only. The wording is fixed by Meta.
- To start from a proven example, use `browse_template_library` (filter by industry or category).

## 2. Draft

Call `get_template_schema` for the exact fields, then draft:

- `name`: lowercase letters, digits and underscores, for example `order_shipped`. It cannot change after submission.
- Variables: either numbered `{{1}}`, `{{2}}` in order from 1, or named like `{{order_id}}`. Never mix the two. Every variable needs a sample value in `example`.
- Header (optional): text up to 60 characters with at most one variable, or an image, video or document.
- Body: up to 1024 characters. Footer: up to 60 characters, no variables.
- Buttons (optional, up to 10): quick replies, website links (at most 2), a phone number (at most 1), or a copy-code button on marketing templates. Button text up to 25 characters.
- Carousel: 2 to 10 cards with the same media type (all images or all videos) and the same buttons. Card text is up to 160 characters, with no variables. Send an approved carousel with a WhatsApp Carousel block.

Write it the way the business talks, and keep it specific to the customer's situation.

## 3. Check, confirm, submit

1. Call `compose_whatsapp_template`. It has no side effects. Fix every error it returns.
2. Show the user the final template, the category and the number it is for. Ask for a clear yes.
3. Only then call `create_whatsapp_template` with `user_confirmed: true`. Submission goes to Meta and cannot be undone.
4. The template stays PENDING until Meta decides, usually within minutes and at most 24 hours. `get_whatsapp_template` shows the status and any rejection reason.

Images and videos must be public links. Use `upload_media` to host a file first if the user only has it locally.

## A rejected template

Read the reason with `get_whatsapp_template` and explain it plainly. Then submit a corrected copy under a new name, because a name and language pair can be used only once on a number.
