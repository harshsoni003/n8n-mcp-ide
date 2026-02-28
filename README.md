<div align="center">

# 🤖 n8n MCP + IDE

**Production-ready n8n workflow development kit**  
Enforced naming conventions · Error handling · Note-taking · Best practices  
All applied automatically by your AI agent via MCP

[![n8n](https://img.shields.io/badge/n8n-Workflow%20Automation-orange?style=flat-square&logo=n8n)](https://n8n.io)
[![MCP](https://img.shields.io/badge/MCP-Model%20Context%20Protocol-blue?style=flat-square)](https://modelcontextprotocol.io)
[![License: MIT](https://img.shields.io/badge/License-MIT-green?style=flat-square)](LICENSE)

</div>

---

## 🧠 What Is This?

This repo is the **standards and rules layer** for building n8n workflows with an AI IDE (Antigravity, Cursor, VS Code Copilot, Claude Desktop, etc.) connected via the **[Model Context Protocol (MCP)](https://modelcontextprotocol.io)**.

```
[ You (in IDE) ]  →  [ AI Agent ]  →  [ n8n-mcp Server ]  →  [ n8n Instance ]
                         ↑
               Reads rule files from this repo
               and enforces them on every workflow
```

Instead of manually following conventions — the AI reads these rule files and **automatically enforces them** on every workflow it creates or edits.

---

## 📁 Project Structure

```
n8n-mcp-ide/
│
├── 📄 MCP_SETUP_GUIDE.md          ← How to connect n8n to your AI IDE
├── 📄 README.md                   ← You are here
├── 📄 .gitignore
│
├── 📂 .agents/
│   └── workflows/                 ← Agent rule files (auto-enforced)
│       ├── n8n_naming_conventions.md
│       ├── n8n_error_handling.md
│       ├── n8n_note_taking.md
│       └── n8n_best_practices.md
│
└── 📂 docs/                       ← Human-readable reference docs
    ├── N8N_NAMING_CONVENTIONS.md
    ├── N8N_ERROR_HANDLING.md
    ├── N8N_NOTE_TAKING.md
    └── N8N_BEST_PRACTICES.md
```

---

## ⚡ Quick Start

### 1. Connect n8n to your AI IDE
See **[MCP_SETUP_GUIDE.md](MCP_SETUP_GUIDE.md)** for full setup instructions for:
- 🟣 Antigravity (Google DeepMind)
- 🟦 Cursor
- 🟩 VS Code
- 🟤 Claude Desktop
- 🟡 Windsurf

### 2. Clone this repo into your workspace
```bash
git clone https://github.com/harshsoni003/n8n-mcp-ide.git
```

### 3. Open the folder in your AI IDE
The agent automatically reads `.agents/workflows/*.md` and applies all rules.

---

## 📜 Agent Rules

| Rule | File | Mandatory? | What It Enforces |
|---|---|---|---|
| Naming Conventions | `n8n_naming_conventions.md` | ✅ Yes | Workflow names, node names, PascalCase attributes, credential names |
| Error Handling | `n8n_error_handling.md` | ✅ Yes | retryOnFail on API nodes, errorWorkflow, design-level validation |
| Note Taking | `n8n_note_taking.md` | ✅ Yes | Summary sticky notes, node notes explaining "why", notesInFlow |
| Best Practices | `n8n_best_practices.md` | ⚠️ Soft | Workflow size limits, Code node usage, readability guidelines |

---

## 🏷️ Naming Reference

| Area | Pattern | Example |
|---|---|---|
| **Workflow (Dev)** | `DEV - <Who> - <Trigger> - <Purpose>` | `DEV - Harsh - Webhook - Sync Leads` |
| **Workflow (Prod)** | `PROD - <Trigger> - <Purpose>` | `PROD - Webhook - Sync Leads` |
| **Node** | `<Verb> <Object>` | `Fetch Leads`, `Send Slack Alert` |
| **Attribute** | `PascalCase ≤ 4 words` | `CustomerID`, `CreatedAt` |
| **Credential** | `<Client>-<Service>-<Purpose>-<Env>` | `Harsh-OpenAI-APIkey-DEV` |

> `<Who>` = first name of the developer building the workflow.  
> In PROD, no person name is used — the workflow belongs to the product.

---

## 🔴 Error Handling Rules (Every Workflow)

```
✅ Every API node must have:
   retryOnFail: true
   maxTries: 3
   waitBetweenTries: 1000
   onError: "stopWorkflow"

✅ Every workflow must have:
   settings.errorWorkflow → linked to "<Client> - Error Workflow"

✅ Data validation before every API node:
   → IF / Filter nodes to guard missing fields
   → Set nodes to apply defaults
```

---

## 📝 Note Taking Rules (Every Workflow)

```
✅ EVERY workflow must have:
   → Sticky note (stickyNote) at the top with: Name, Purpose, Trigger,
     Connections, Dependencies, Setup Instructions, Outputs

✅ EVERY API / HTTP / external service node must have:
   → notes: "Why this node exists + rate limits + docs link"
   → notesInFlow: true

✅ Code nodes ALWAYS have a note — no exceptions
```

---

## 🤖 Workflows Built With These Rules

| Workflow | Type | Description |
|---|---|---|
| `DEV - Harsh - WhatsApp - AI Echo Bot` | WhatsApp Bot | Receives messages → GPT-4o-mini → replies |
| `DEV - Harsh - Slack - AI Reply Bot` | Slack Bot | @mentions → GPT-4o-mini → thread reply |
| `DEV - Harsh - Error Workflow` | Error Handler | Centralized error logging for all workflows |

---

## 🔒 Security

- API keys are **never stored in this repo**
- `mcp_config.json` (which contains your n8n API key) is excluded via `.gitignore`
- Use separate credentials for DEV and PROD environments
- Rotate API keys periodically from n8n → Settings → API

---

## 🗂️ Changelog

| Date | Change |
|---|---|
| 2026-03-01 | Initial repo created — naming, error handling, note-taking, best practices |
| 2026-03-01 | MCP Setup Guide added |
| 2026-03-01 | `<Who>` in DEV naming made dynamic (any developer's name) |

---

<div align="center">

Made by **[Harsh Soni](https://github.com/harshsoni003)** · Powered by [n8n](https://n8n.io) + [MCP](https://modelcontextprotocol.io)

</div>
