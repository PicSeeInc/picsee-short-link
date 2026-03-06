# 🔗 PicSee MCP Server

[![MCP Native](https://img.shields.io/badge/MCP-Native-blue.svg)](https://modelcontextprotocol.io)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

A **Model Context Protocol (MCP)** server for [PicSee](https://picsee.io) — URL shortening with QR code generation, click analytics, and link management. Built for AI agents like **Claude Code**, **OpenClaw**, and **Cursor**.

---

## 📖 Resources

- **PicSee Website:** [https://picsee.io](https://picsee.io)
- **API Documentation:** [https://picsee.io/developers/docs/public-api.html](https://picsee.io/developers/docs/public-api.html)

---

## 🌟 Features

- **Dual-Mode Operation** — Unauthenticated (basic shortening) and Authenticated (full management) with automatic detection
- **URL Shortening + QR Codes** — Create short links and instantly generate QR codes (300x300px, customizable)
- **Visual Analytics** — Total clicks, unique clicks, daily trends, and optional chart generation
- **Secure Token Storage** — AES-256-CBC encryption with machine-specific key derivation (hostname + username → SHA-256). Tokens never stored in plaintext
- **Link Management** — Search, filter (tags, stars, keywords), edit, and delete
- **Agent-Ready** — Built-in instructions for proactive QR code and chart generation workflows

---

## 🧩 MCP Tools

| Tool | Description | Auth |
|:-----|:------------|:-----|
| `shorten_url` | Create a `pse.is` short link with optional custom slug, tags, UTM, and preview metadata. Auto-detects auth mode | Optional |
| `generate_qr_code` | Generate a QR code URL for any short link (300x300px default, customizable size) | No |
| `get_analytics` | Click statistics — total, unique, and daily breakdown for the past 60 days | Required |
| `generate_analytics_chart` | Generate a line chart URL visualizing daily click trends | No |
| `list_links` | List and search link history with filters (tags, keywords, stars, author, date range) | Required |
| `edit_link` | Update destination URL, slug, title, description, thumbnail, tags, UTM, tracking pixels, expiration (Advanced plan) | Required |
| `delete_link` | Delete or recover a short link | Required |
| `setup_auth` | Verify and encrypt your PicSee API token locally | No |

---

## ⚙️ Installation

### Claude Code

```bash
claude mcp add picsee -- node /path/to/picsee-short-link/mcp-server/dist/index.js
```

Replace `/path/to/` with the actual path to the skill directory.

### Cursor

Edit Cline MCP settings file:
- **macOS**: `~/Library/Application Support/Cursor/User/globalStorage/saoudrizwan.claude-dev/settings/cline_mcp_settings.json`
- **Windows**: `%APPDATA%\Cursor\User\globalStorage\saoudrizwan.claude-dev\settings\cline_mcp_settings.json`
- **Linux**: `~/.config/Cursor/User/globalStorage/saoudrizwan.claude-dev/settings/cline_mcp_settings.json`

Add:
```json
{
  "mcpServers": {
    "picsee": {
      "command": "node",
      "args": ["/path/to/picsee-short-link/mcp-server/dist/index.js"]
    }
  }
}
```

Then restart Cursor.

### OpenClaw (via mcporter)

```bash
mcporter config add picsee stdio -- node /path/to/picsee-short-link/mcp-server/dist/index.js
```

Then call tools:

```bash
mcporter call picsee.shorten_url url="https://example.com/long-url"
```

### ClawHub

Install via ClawHub CLI:

```bash
clawhub install picsee-short-link
```

MCP server registration is handled automatically.

### Smithery

Install via Smithery registry:

```bash
npx @smithery/cli install picsee/short-link
```

Or browse: [https://smithery.ai/skills/picsee/short-link](https://smithery.ai/skills/picsee/short-link)

---

## 🎨 Agent Recipes

Beyond the core MCP tools, this skill includes instructions for AI agents to handle post-processing:

- **QR Code** — Generate a 300×300 QR code image via `api.qrserver.com` after shortening
- **Analytics Chart** — Parse daily click data from `get_analytics` and render a trend chart via `quickchart.io`

These recipes are documented in `SKILL.md` for agents that support the OpenClaw skill format.

---

## 🔒 Security

| Aspect | Detail |
|:-------|:-------|
| **Storage** | `~/.openclaw/.picsee_token` |
| **Encryption** | AES-256-CBC, random IV per write |
| **Key Derivation** | `SHA-256(hostname + "-" + username)` — unique per machine and user |
| **File Permissions** | `0600` (owner read/write only) |
| **Logging** | None — no URLs or metadata are logged by this server |

---

## 📁 Project Structure

```
picsee-short-link/
├── mcp-server/           # MCP Server (TypeScript)
│   ├── src/
│   │   ├── index.ts      # Server entry point + tool definitions
│   │   ├── api.ts        # PicSee REST API client
│   │   └── keychain.ts   # AES-256-CBC token storage
│   ├── dist/             # Compiled output
│   ├── package.json
│   ├── tsconfig.json
│   └── smithery.yaml
├── references/           # API documentation
├── SKILL.md              # OpenClaw skill definition
├── README.md             # This file
└── _meta.json            # OpenClaw registry metadata
```

---

## 📄 License

MIT
