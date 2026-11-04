---
name: build-chatbot
description: Build and test a ChatMaxima chatbot or AI agent from a description. Use when the user wants a new bot, an AI agent that answers from their website or documents, a lead capture or booking bot, a FAQ bot, or wants to add steps to an existing bot.
---

# Build a ChatMaxima chatbot or AI agent

## 1. Agree the design first

Ask only what you cannot reasonably assume:

- What the bot is for and who talks to it.
- Which channel it runs on (website chat, WhatsApp, Instagram and so on). Channels are connected accounts: see them with `list_accounts`.
- How a conversation starts: specific keywords, or any message.
- What it must collect (name, email, phone, date) and where that goes (contacts, a Google Sheet, a calendar booking, a ticket, an email).

Then describe the flow back in a short numbered list and get a yes before building.

## 2. Pick the blocks

1. Call `list_block_types` to see what exists. Common choices:
   - **MaxIA AI Agent** (`ai_prompt`): an AI agent with a system prompt, optional knowledge bases, variable extraction and tool calling. Use it when replies should be open-ended.
   - **Knowledge Source** (`knowledge_source`): answers one question from the selected knowledge bases and stores the answer in a variable.
   - Text, buttons, questions (ask for name, email, phone, date), conditions, and integration blocks such as Google Sheets, Google Calendar and MaxCRM.
2. Call `get_block_schema` for every block type you will use, so each field is filled correctly.
3. Fill dropdown-style fields from the real lists, never by guessing:
   - knowledge bases: `list_knowledge_bases`
   - channel accounts: `list_accounts`
   - integrations and their actions: `list_integrations` (or `list_integration_actions` for an app that is not connected yet)
   - calendars: `list_calendars`; sheet columns: `list_sheet_columns`
   - contact lists and attributes: `list_contact_lists`, `list_contact_attributes`
   - labels, statuses, agents: `list_labels`, `list_statuses`, `list_team_members`
   - voices for a voice agent: `list_voice_options`

## 3. Build

1. `create_bot`, with `welcome_message` if the user gave one. The bot starts with a Starting Step (node `7`) and a Start Message.
2. Set the trigger on node `7` with `update_block`: `trigger_type` with keywords (`exact_match`, `starts_with`, `contains`, `ends_with` or `any`) or `allow_any` for no keyword. Never leave it unset. Set the inactivity timeout if the user gave one.
3. Add each block with `add_block`, passing the previous block's id as `after_node` so it connects automatically.
4. For branches, wire each extra output (`output_2` and up, for example one per button) to its target with `connect_blocks`.
5. In message text, use saved variables with single braces, for example `{customer_name}`. Never use double braces in bot messages.
6. Finish with `organize_bot`, and `rename_bot` if the name no longer fits.

## 4. Test before handing over

1. Call `run_test` with a short scripted conversation. The first message must match the trigger. Email and SMS are simulated.
2. Read the transcript. If a step is wrong, fix it with `get_block` then `update_block` (send only the changed fields), and test again.
3. A WhatsApp Flow form cannot be opened in the test chat. Say so, and tell the user to test that part on WhatsApp itself.
4. Share the transcript summary and the builder link. Tell the user which channel the bot still needs to be connected to, if any.
