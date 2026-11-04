---
name: whatsapp-flow
description: Create or edit a WhatsApp Flow, a multi-screen form shown inside WhatsApp or the ChatMaxima website chat. Use when the user wants a booking form, lead or sign-up form, survey, feedback form, or appointment request that customers fill in without leaving the chat.
---

# Create a WhatsApp Flow

## 1. Ask where it will be used

Ask whether the flow is for **WhatsApp** or for the **website chat**. This choice is permanent:

- `whatsapp`: created on WhatsApp as a Draft. The user publishes it from the WhatsApp Flows page, and only a published flow can be sent. It needs a WhatsApp number from `list_whatsapp_accounts`.
- `web`: works in the website chat straight away, but can never be published to WhatsApp.

## 2. Design the screens

Call `get_flow_schema` for the exact format. Use the Form Builder (`screens`) unless the user wants later questions to depend on earlier answers in ways the Form Builder cannot express, in which case use the Dynamic Data Flow (`steps_config`).

Form Builder rules:

- 1 to 8 screens. Screen ids use uppercase letters and underscores only, no digits, for example `DETAILS`.
- The first screen is the entry. Every screen must be reachable from it.
- Inputs: text input, text area, dropdown, radio buttons, checkboxes, date picker and opt-in. Each input's `name` must be unique across the whole flow, because the answers arrive under those names.
- Text and images: heading, subheading, body text, caption, and public images up to 300 KB.
- At most one Nav Button (`Footer`) per screen. It can go to a fixed screen, or branch on a dropdown or radio answer.
- The last screen ends the flow. Answers from every screen are sent together at the end.

Keep it short. Ask only for what the business will use, and put the most important question first.

## 3. Create and hand over

1. Show the user the screens as a simple outline and get a yes.
2. Call `create_whatsapp_flow`. It returns the flow alias, status and a link.
3. For a WhatsApp flow, remind the user to publish it from the WhatsApp Flows page.
4. To send it in a bot, add a WhatsApp Flow block (see the `build-chatbot` skill). The flow itself can only be tested on WhatsApp, by messaging the connected number.

## Editing a flow

Read it with `get_whatsapp_flow`, then call `update_whatsapp_flow` with the complete new screens. Editing a published Form Builder flow creates a new draft of the same flow on WhatsApp, which the user publishes again. A published Dynamic Data Flow cannot change its steps. Never create a copy to edit unless the user asks for a new flow.
