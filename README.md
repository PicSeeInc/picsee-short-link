# 🔗 PicSee MCP Server

[![MCP Native](https://img.shields.io/badge/MCP-Native-blue.svg)](https://modelcontextprotocol.io)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

A **Model Context Protocol (MCP)** server for [PicSee](https://picsee.io) — URL shortening, click analytics, and link management. Built for AI agents like **Claude Code**, **OpenClaw**, and **Cursor**.

---

## 📖 Resources

- **PicSee Website:** [https://picsee.io](https://picsee.io)
- **API Documentation:** [https://picsee.io/developers/docs/public-api.html](https://picsee.io/developers/docs/public-api.html)

---

## 🌟 Features

- **Dual-Mode Operation** — Unauthenticated (basic shortening) and Authenticated (full management) with automatic detection
- **Secure Token Storage** — AES-256-CBC encryption with machine-specific key derivation (hostname + username → SHA-256). Tokens never stored in plaintext
- **Analytics** — Total clicks, unique clicks, and daily traffic trends
- **Link Management** — Search, filter (tags, stars, keywords), edit, and delete
- **Agent Recipes** — Built-in instructions for QR code generation and analytics chart visualization

---

## 🧩 MCP Tools

| Tool | Description | Auth |
|:-----|:------------|:-----|
| `shorten_url` | Create a `pse.is` short link with optional custom slug, tags, UTM, and preview metadata. Auto-detects auth mode | Optional |
| `get_analytics` | Click statistics — total, unique, and daily breakdown for the past 60 days | Required |
| `list_links` | List and search link history with filters (tags, keywords, UTM, stars, author, date range) | Required |
| `edit_link` | Update destination URL, slug, title, description, thumbnail, tags, UTM, tracking pixels, expiration (Advanced plan) | Required |
| `delete_link` | Delete or recover a short link | Required |
| `setup_auth` | Verify and encrypt your PicSee API token locally | No |

---

## ⚙️ Installation

### Claude Code

Add to `.claude/settings.json`:

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

### OpenClaw (via mcporter)

```bash
mcporter config add picsee stdio -- node /path/to/picsee-short-link/mcp-server/dist/index.js
```

Then call tools:

```bash
mcporter call picsee.shorten_url url="https://example.com/long-url"
```

### Smithery

> Coming soon. The server is designed to be Smithery-compatible — see `smithery.yaml` in the repository.

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

The encryption format (`iv_hex:ciphertext_hex`) is shared between the MCP server and the legacy CLI scripts, so existing tokens work without re-authentication.

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
├── scripts/              # Legacy CLI scripts (.mjs)
├── references/           # API documentation
├── SKILL.md              # OpenClaw skill definition
├── README.md             # This file
└── _meta.json            # OpenClaw registry metadata
```

---

## 📄 License

MIT
