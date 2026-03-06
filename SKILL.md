---
name: picsee-short-link
description: PicSee URL shortener and link management via MCP Server or CLI scripts. Use when the user asks to shorten a URL, check link analytics, list/search links, or mentions PicSee. Supports unauthenticated mode (basic shortening) and authenticated mode (analytics, editing, search). Token stored with AES-256-CBC encryption.
metadata:
  {
    "openclaw":
      {
        "emoji": "🔗",
        "configPaths": ["skills/picsee-short-link/config.json"],
        "requires": { "bins": ["node"] },
        "externalApis": ["api.pics.ee", "chrome-ext.picsee.tw", "api.qrserver.com", "quickchart.io"],
        "writesPaths": ["skills/picsee-short-link/config.json", "~/.openclaw/.picsee_token", "/tmp/*.png"]
      }
  }
---

# PicSee Short Link

URL shortener with analytics, search, and link management. Two interfaces available:
- **MCP Server** (preferred for Claude Code, Cursor, OpenClaw mcporter)
- **CLI scripts** (legacy, still functional)

---

## MCP Server

Entry point: `skills/picsee-short-link/mcp-server/dist/index.js`

### Register with mcporter (OpenClaw)

```bash
mcporter config add picsee stdio -- node ~/.openclaw/workspace/skills/picsee-short-link/mcp-server/dist/index.js
```

Then call tools via:
```bash
mcporter call picsee.shorten_url url="https://example.com/long-path"
```

### Register with Claude Code

In `.claude/settings.json`:
```json
{
  "mcpServers": {
    "picsee": {
      "command": "node",
      "args": ["<absolute-path>/mcp-server/dist/index.js"]
    }
  }
}
```

### MCP Tools Reference

#### `shorten_url`

Shorten a URL. Auto-detects auth mode: if token is stored, uses authenticated API (`api.pics.ee`) with advanced features; otherwise falls back to unauthenticated (`chrome-ext.picsee.tw`).

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `url` | string (URL) | **yes** | Destination URL |
| `encodeId` | string | no | Custom slug (3-90 chars) |
| `domain` | string | no | Short link domain (default: `pse.is`) |
| `title` | string | no | Custom preview title (Advanced plan) |
| `description` | string | no | Custom preview description (Advanced plan) |
| `imageUrl` | string (URL) | no | Custom preview thumbnail (Advanced plan) |
| `tags` | string[] | no | Tags for organisation (Advanced plan) |
| `utm` | object | no | `{source, medium, campaign, term, content}` (Advanced plan) |

Returns: `{ success, shortUrl, encodeId, mode }`

---

#### `get_analytics`

Click stats for a short link. Requires authentication.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `encodeId` | string | **yes** | Slug of the short link (e.g. `abc123` from `pse.is/abc123`) |

Returns: `{ success, data: { totalClicks, uniqueClicks, dailyClicks[] } }`

---

#### `list_links`

List and search links. Requires authentication. Results are returned in reverse chronological order (newest first).

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `startTime` | string | **yes** | Query backward from this time. **Use the LAST moment** of the desired period. Format: `YYYY-MM-DDTHH:MM:SS`. Example: `2026-03-31T23:59:59` for March 2026 |
| `limit` | number | no | Results per page (1-50, default 50) |
| `isAPI` | boolean | no | Filter API-generated links only |
| `isStar` | boolean | no | Filter starred links only |
| `prevMapId` | string | no | Pagination cursor — links older than this mapId |
| `externalId` | string | no | Filter by external ID |
| `keyword` | string | no | Search title/description (Advanced, 3-30 chars) |
| `tag` | string | no | Filter by tag (Advanced) |
| `url` | string | no | Filter by exact destination URL |
| `encodeId` | string | no | Filter by exact slug |
| `authorId` | string | no | Filter by author ID |
| `utm` | object | no | `{source, medium, campaign, term, content}` (Advanced) |

Returns: `{ success, data: [{ encodeId, domain, url, title, totalClicks, uniqueClicks, createTime, tags, utm }] }`

**Common mistake**: Using the first day of a month as startTime (e.g. `2026-03-01`) will miss most of that month's data. Always use the last day.

---

#### `edit_link`

Edit an existing short link. Requires authentication + Advanced plan. Only include fields to change.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `originalEncodeId` | string | **yes** | Current slug of the link to edit |
| `url` | string (URL) | no | New destination URL |
| `encodeId` | string | no | New custom slug |
| `domain` | string | no | New domain |
| `title` | string | no | New preview title |
| `description` | string | no | New preview description |
| `imageUrl` | string (URL) | no | New preview thumbnail |
| `tags` | string[] | no | New tags |
| `fbPixel` | string | no | Meta Pixel ID |
| `gTag` | string | no | Google Tag Manager ID |
| `utm` | object | no | New UTM parameters |
| `expireTime` | string | no | Expiration time (ISO 8601) |

Returns: `{ success, message }`

---

#### `delete_link`

Delete or recover a short link. Requires authentication.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `encodeId` | string | **yes** | Slug of the short link |
| `action` | string | no | `"delete"` (default) or `"recover"` |

Returns: `{ success, message }`

---

#### `setup_auth`

Store and verify a PicSee API token. Token is encrypted with AES-256-CBC using a machine-specific key (hostname + username → SHA-256) and saved to `~/.openclaw/.picsee_token`.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `token` | string | **yes** | PicSee API token |

Returns: `{ success, plan, quota, usage, message }`

Token source: <https://picsee.io/> → avatar → Settings → API → Copy token

---

## CLI Scripts (Legacy)

All scripts in `skills/picsee-short-link/scripts/`. Output JSON. Can be used standalone without MCP.

| Script | Purpose | Usage |
|--------|---------|-------|
| `shorten.mjs` | Shorten URL | `node shorten.mjs "<URL>"` |
| `analytics.mjs` | Link stats | `node analytics.mjs "<ENCODE_ID>"` |
| `list.mjs` | List/search links | `node list.mjs "<START_TIME>" [LIMIT] [--flags]` |
| `edit.mjs` | Edit link | `node edit.mjs "<ENCODE_ID>" "<NEW_URL>"` |
| `delete.mjs` | Delete/recover | `node delete.mjs "<ENCODE_ID>" [delete\|recover]` |
| `auth.mjs` | Store token | `node auth.mjs "<TOKEN>"` |
| `setup.mjs` | Set unauth mode | `node setup.mjs` |

For `list.mjs` advanced flags, run `node list.mjs --help`.

---

## Auth Modes

| Mode | API Host | Features |
|------|----------|----------|
| **Unauthenticated** | `chrome-ext.picsee.tw` | Create short links only |
| **Authenticated** | `api.pics.ee` | Create + analytics + list + search + edit + delete |

Auto-detection: if encrypted token exists at `~/.openclaw/.picsee_token`, authenticated mode is used. Otherwise unauthenticated.

`config.json` tracks the mode preference:
```json
{ "mode": "authenticated", "setupDate": "2026-03-04" }
```

---

## Security

- **Token encryption**: AES-256-CBC, IV stored alongside ciphertext (`iv_hex:ciphertext_hex`)
- **Key derivation**: `SHA-256(hostname + "-" + username)` — machine-specific, no shared secrets
- **File permissions**: `0600` on token file
- **No plaintext tokens** stored anywhere
- **Compatible** between MCP Server and CLI scripts (same keychain module, same file path)

---

## Error Handling

| Scenario | Response |
|----------|----------|
| No token + auth-required tool | "This requires authentication. Use setup_auth with your PicSee API token." |
| Non-Advanced plan + edit | "Editing requires the Advanced plan. Create a new link instead." |
| Invalid token | "Invalid token. Get one from https://picsee.io/ → Settings → API" |

---

## Agent Recipes (OpenClaw)

These are **not MCP tools** — they are post-processing actions the agent performs after getting results from the tools above. Use `exec` to run the curl commands.

### QR Code Generation

When the user asks for a QR code after shortening:

```bash
curl -s -o /tmp/<ENCODE_ID>_qr.png "https://api.qrserver.com/v1/create-qr-code/?size=300x300&data=https://pse.is/<ENCODE_ID>"
```

Send via `message` tool with `filePath: "/tmp/<ENCODE_ID>_qr.png"`. Only generate when explicitly requested.

### Analytics Chart

After calling `get_analytics`, if the user wants a visual chart of daily clicks:

1. Extract `dailyClicks` array from the response
2. Build a QuickChart URL with the data:

```bash
# Example with 3 data points — construct dynamically from actual dailyClicks
curl -s -o /tmp/<ENCODE_ID>_chart.png "https://quickchart.io/chart?w=600&h=300&c=\
{type:'line',data:{labels:['3/1','3/2','3/3'],datasets:[{label:'Clicks',data:[45,67,23],borderColor:'rgb(59,130,246)',fill:false}]}}"
```

Send via `message` tool with `filePath: "/tmp/<ENCODE_ID>_chart.png"`. Only generate when the user asks for visualization — don't auto-generate for every analytics query.
