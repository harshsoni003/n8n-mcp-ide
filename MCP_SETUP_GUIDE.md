# N8N MCP Setup Guide

> **Purpose**: Connect your n8n instance to any AI IDE or agent (Antigravity, Cursor, VS Code + Copilot, Claude Desktop, etc.) via the Model Context Protocol (MCP).  
> **Result**: Your AI assistant can create, read, update, and manage n8n workflows directly from the IDE — no copy-paste, no switching tabs.

---

## What Is MCP?

**Model Context Protocol (MCP)** is an open standard that lets AI assistants connect to external tools and data sources via a simple server/client protocol.

```
[ AI IDE / Agent ]  ←→  [ MCP Server: n8n-mcp ]  ←→  [ Your n8n Instance ]
```

The `n8n-mcp` server translates the AI's tool calls into n8n REST API calls, so the AI can:
- List, create, update, and delete workflows
- Search n8n nodes and templates
- Validate workflow configurations
- Trigger test executions

---

## Prerequisites

| Requirement | Notes |
|---|---|
| **Node.js** ≥ 18 | Check: `node -v` |
| **n8n instance** | Cloud or self-hosted, with the API enabled |
| **n8n API Key** | Generated from your n8n settings (see below) |
| **n8n-mcp package** | Installed globally via npm |

---

## Step 1 — Generate Your n8n API Key

1. Open your n8n instance (e.g. `https://n8n-dev1.customaistudio.io`)
2. Click your **profile icon** (top-right) → **Settings**
3. Go to **API** → **Create API Key**
4. Copy the key — **you won't see it again**

> [!CAUTION]
> Never commit your API key to Git or share it publicly. Treat it like a password.

---

## Step 2 — Install n8n-mcp

Open a terminal and run:

```powershell
npm install -g n8n-mcp
```

Verify installation:

```powershell
n8n-mcp --version
```

---

## Step 3 — Create the MCP Config File

Create the config file at the path your IDE expects (see IDE-specific paths below).

### Config Template

```json
{
  "mcpServers": {
    "n8n-mcp": {
      "command": "n8n-mcp",
      "args": [],
      "env": {
        "MCP_MODE": "stdio",
        "LOG_LEVEL": "error",
        "DISABLE_CONSOLE_OUTPUT": "true",
        "N8N_API_URL": "https://YOUR-N8N-INSTANCE-URL",
        "N8N_API_KEY": "YOUR_N8N_API_KEY_HERE"
      }
    }
  }
}
```

Replace:
- `https://YOUR-N8N-INSTANCE-URL` → your actual n8n URL (e.g. `https://n8n-dev1.customaistudio.io`)
- `YOUR_N8N_API_KEY_HERE` → the API key from Step 1

---

## Step 4 — Configure Your IDE

### 🟣 Antigravity (Google DeepMind)

**Config file path:**
```
C:\Users\<YourName>\.gemini\antigravity\mcp_config.json
```

Paste the config template above into that file and save.  
Antigravity will auto-detect and connect on next load.

---

### 🟦 Cursor

**Config file path:**
```
C:\Users\<YourName>\.cursor\mcp.json
```

Or via UI: **Cursor Settings** → **MCP** → **Add Server** → paste the config.

---

### 🟩 VS Code (GitHub Copilot / Claude extension)

**Config file path:**
```
C:\Users\<YourName>\AppData\Roaming\Code\User\settings.json
```

Add under `"mcp.servers"`:

```json
"mcp": {
  "servers": {
    "n8n-mcp": {
      "command": "n8n-mcp",
      "args": [],
      "env": {
        "MCP_MODE": "stdio",
        "LOG_LEVEL": "error",
        "DISABLE_CONSOLE_OUTPUT": "true",
        "N8N_API_URL": "https://YOUR-N8N-INSTANCE-URL",
        "N8N_API_KEY": "YOUR_N8N_API_KEY_HERE"
      }
    }
  }
}
```

---

### 🟤 Claude Desktop

**Config file path:**
```
C:\Users\<YourName>\AppData\Roaming\Claude\claude_desktop_config.json
```

Paste the config template inside the existing JSON (merge with any existing `mcpServers` block).

---

### 🟡 Windsurf (Codeium)

**Config file path:**
```
C:\Users\<YourName>\.codeium\windsurf\mcp_config.json
```

Same format as the template above.

---

## Step 5 — Verify the Connection

After restarting your IDE, ask the AI:

```
Check n8n health
```

or

```
List all my n8n workflows
```

You should see live data from your n8n instance in the response.

---

## Environment Variables Reference

| Variable | Required | Description |
|---|---|---|
| `MCP_MODE` | ✅ | Must be `stdio` for IDE integration |
| `N8N_API_URL` | ✅ | Full URL of your n8n instance (no trailing slash) |
| `N8N_API_KEY` | ✅ | Your n8n REST API key |
| `LOG_LEVEL` | ⬜ | `error` recommended to suppress verbose logs |
| `DISABLE_CONSOLE_OUTPUT` | ⬜ | `true` to keep IDE output clean |

---

## Multiple n8n Instances

You can connect multiple n8n instances by adding more server entries:

```json
{
  "mcpServers": {
    "n8n-dev": {
      "command": "n8n-mcp",
      "args": [],
      "env": {
        "MCP_MODE": "stdio",
        "LOG_LEVEL": "error",
        "DISABLE_CONSOLE_OUTPUT": "true",
        "N8N_API_URL": "https://n8n-dev1.customaistudio.io",
        "N8N_API_KEY": "YOUR_DEV_API_KEY"
      }
    },
    "n8n-prod": {
      "command": "n8n-mcp",
      "args": [],
      "env": {
        "MCP_MODE": "stdio",
        "LOG_LEVEL": "error",
        "DISABLE_CONSOLE_OUTPUT": "true",
        "N8N_API_URL": "https://n8n-prod.customaistudio.io",
        "N8N_API_KEY": "YOUR_PROD_API_KEY"
      }
    }
  }
}
```

> [!WARNING]
> Be careful when the AI is connected to PROD — any create/update/delete operations will affect live workflows.

---

## Troubleshooting

| Problem | Fix |
|---|---|
| AI says "MCP not found" | Run `npm install -g n8n-mcp` and restart IDE |
| `command not found: n8n-mcp` | Add npm global bin to PATH: `npm config get prefix` → add `/bin` to PATH |
| Connection refused | Verify `N8N_API_URL` is reachable and the API is enabled in n8n settings |
| 401 Unauthorized | API key is wrong or expired — regenerate in n8n → Settings → API |
| Workflows not updating | Restart the IDE / MCP server after config changes |

---

## Security Best Practices

> [!IMPORTANT]
> Follow these to keep your n8n instance secure.

- ✅ Store API keys only in the config file — never in workflow code or notes
- ✅ Use **separate API keys** for DEV and PROD
- ✅ Rotate keys periodically (n8n → Settings → API → Regenerate)
- ✅ Add the config file path to `.gitignore` if your dotfiles are version-controlled
- ❌ Never share your `mcp_config.json` in screenshots, Slack, or public repos

---

## Changelog

| Date | Change | Author |
|---|---|---|
| 2026-03-01 | Initial guide created | Antigravity |
