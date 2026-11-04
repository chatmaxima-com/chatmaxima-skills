---
name: get-started
description: First-time setup for ChatMaxima. Confirms the connection, picks the workspace to work in, and suggests a first task.
---

# Get started with ChatMaxima

Run this once after the user installs ChatMaxima.

1. Call `list_workspaces` to confirm the connection works and see which ChatMaxima workspaces the user can reach.
2. If there is more than one workspace, ask the user which one to work in, then call `switch_workspace` with that workspace. Never pick one for them.
3. Call `list_bots` and tell the user how many chatbots the workspace has, naming a few.
4. Offer three starting points:
   - build a new chatbot and test it,
   - explain what an existing chatbot does,
   - design a WhatsApp message template.

If a tool says the user's role cannot use a section, pass that message on as it is and suggest asking a workspace admin. Do not retry with other tools to get around it.
