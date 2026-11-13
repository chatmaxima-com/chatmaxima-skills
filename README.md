# ChatMaxima skills

Agent skills for [ChatMaxima](https://chatmaxima.com/), the AI agent and shared inbox platform for WhatsApp, Instagram, Messenger, Telegram, SMS, email and website chat.

The skills teach your AI assistant how to do real ChatMaxima work through the ChatMaxima MCP connector: build and test chatbots and AI agents, fix a bot from its run history, write WhatsApp templates and Flows, write email templates, and triage the inbox. A developer skill also helps coding agents write integrations against the ChatMaxima REST API. They follow the open [Agent Skills](https://agentskills.io) format, so the same folders work in Claude, Codex, Gemini CLI, Cursor and other tools that read `SKILL.md`.

## Skills

| Skill | Use it to |
|---|---|
| `get-started` | Connect, pick a workspace and choose a first task |
| `chatmaxima` | Ground rules for any task in a ChatMaxima workspace |
| `build-chatbot` | Build and test a chatbot or AI agent from a description |
| `fix-chatbot` | Find and fix what is wrong with a bot from its real runs |
| `whatsapp-template` | Write a WhatsApp template and submit it to Meta for approval |
| `whatsapp-flow` | Build a multi-screen form for WhatsApp or website chat |
| `inbox-triage` | See what is waiting, summarize, assign, label and hand chats to a human |
| `email-template` | Write or edit the email templates your bots send |
| `chatmaxima-api` | Write code against the ChatMaxima REST API: sends, WhatsApp templates, webhooks, contacts, exports |

## Install

**Claude (claude.ai, Cowork, Claude Code):** add the ChatMaxima plugin from the Claude directory, then connect ChatMaxima from the plugin's **Connectors** tab and sign in.

**Claude Code from this repository:**

```bash
claude --plugin-dir ./chatmaxima-skills
```

**Any agent that reads Agent Skills:**

```bash
npx skills@1.7.6 add chatmaxima-com/chatmaxima-skills
```

(`skills` is the open-source CLI from Vercel Labs; use whatever version is current.)

The skills call the ChatMaxima MCP server, so connect it in your tool as well: `https://chatmaxima.com/mcp` (OAuth sign-in with your ChatMaxima account). Setup guides: [docs.chatmaxima.com/mcp](https://docs.chatmaxima.com/mcp).

## What you need

- A ChatMaxima account. New workspaces start with a 7-day free trial, no card required.
- For WhatsApp templates and Flows, a WhatsApp number connected in ChatMaxima.

## Data

This plugin contains only Markdown instructions and one MCP server reference. It runs no scripts, hooks or local programs.

- It connects to one server, `https://chatmaxima.com/mcp`, operated by ChatMaxima, after you sign in with OAuth. No API keys are stored in this repository, and the plugin reads no credentials from your machine.
- When you ask for a task, your assistant sends the details of that task (for example a bot design, template text or a conversation id) to your ChatMaxima workspace and reads back what the tools return, such as bot configuration, templates and inbox conversations, which can include your customers' messages and contact details.
- The `chatmaxima-api` skill only guides the code your assistant writes. It makes no calls itself; the code it helps write calls `https://chatmaxima.com/api/v2` with your own API key. Its examples use the placeholder `YOUR_API_KEY`, which you replace with the key from your backend's secret store.
- Actions that cannot be undone (deleting a bot block, submitting a WhatsApp template to Meta, adding a column to your own spreadsheet) need your confirmation first.

ChatMaxima's privacy policy: [chatmaxima.com/privacy/](https://chatmaxima.com/privacy/)

## Support

[support.chatmaxima.com](https://support.chatmaxima.com/en/) or support@chatmaxima.com.

## License

MIT. See [LICENSE](LICENSE).
