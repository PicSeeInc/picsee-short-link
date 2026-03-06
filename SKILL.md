---
name: picsee-short-link
description: PicSee URL shortener with QR code generation, analytics charts, and link management via MCP Server. Use when the user asks to shorten a URL, generate QR codes, visualize analytics, list/search links, or mentions PicSee. Supports unauthenticated mode (basic shortening + QR codes + charts) and authenticated mode (full analytics, editing, search). Token stored with AES-256-CBC encryption.
metadata:
  {
    "openclaw":
      {
        "emoji": "🔗",
        "configPaths": ["skills/picsee-short-link/config.json"],
        "requires": { "bins": ["node", "mcporter"] },
        "externalApis": ["api.pics.ee", "chrome-ext.picsee.tw", "api.qrserver.com", "quickchart.io"],
        "writesPaths": ["skills/picsee-short-link/config.json", "~/.openclaw/.picsee_token", "/tmp/*.png"]
      }
  }
---

# PicSee Short Link

URL shortener with **QR code generation**, **analytics charts**, and link management via **Model Context Protocol (MCP) Server**.

Provides AI agents (Claude Code, Cursor, OpenClaw) with direct access to PicSee's URL shortening, QR code generation, analytics visualization, and link management features through standardized tool calls.

---

## First-Time Setup

### OpenClaw

Execute the following command to register the MCP server (only needed once):

```bash
mcporter config add picsee stdio -- node ~/.openclaw/workspace/skills/picsee-short-link/mcp-server/dist/index.js
```

Verify registration:
```bash
mcporter config list | grep picsee
```

### ClawHub

Install via ClawHub CLI:
```bash
clawhub install picsee-short-link
```

MCP server registration is handled automatically.

---

## Usage

### Basic Shortening
```bash
mcporter call picsee.shorten_url url="https://example.com/long-url"
```

### With Custom Slug
```bash
mcporter call picsee.shorten_url url="https://example.com" encodeId="mylink"
```

### Generate QR Code
```bash
mcporter call picsee.generate_qr_code shortUrl="https://pse.is/mylink" size=300
```

### Check Analytics
```bash
mcporter call picsee.get_analytics encodeId="mylink"
```

### Generate Analytics Chart
```bash
mcporter call picsee.generate_analytics_chart \
  dailyClicks='[{"date":"3/1","clicks":45},{"date":"3/2","clicks":67}]' \
  encodeId="mylink"
```

### List Recent Links
```bash
mcporter call picsee.list_links startTime="2026-03-31T23:59:59" limit=10
```

---

## MCP Tools Reference

### `shorten_url`

Shorten a URL. Auto-detects auth mode: if token is stored, uses authenticated API (`api.pics.ee`) with advanced features; otherwise falls back to unauthenticated (`chrome-ext.picsee.tw`).

**Parameters:**
- `url` (string, **required**): Destination URL
- `encodeId` (string): Custom slug (3-90 chars)
- `domain` (string): Short link domain (default: `pse.is`)
- `title` (string): Custom preview title (Advanced plan)
- `description` (string): Custom preview description (Advanced plan)
- `imageUrl` (string): Custom preview thumbnail (Advanced plan)
- `tags` (string[]): Tags for organization (Advanced plan)
- `utm` (object): `{source, medium, campaign, term, content}` (Advanced plan)

**Returns:** `{ success, shortUrl, encodeId, mode }`

---

### `generate_qr_code`

Generate a QR code URL for any short link. **Only call this if the user explicitly requests a QR code.** Automatically shortens the QR code URL via PicSee (unauthenticated mode, no quota consumed).

**Parameters:**
- `shortUrl` (string, **required**): The short URL to encode in QR code
- `size` (number): QR code size in pixels (100-1000, default: 300)

**Returns:** `{ success, qrCodeUrl, originalQrUrl, shortUrl, size, note }`

**Example:**
```bash
mcporter call picsee.generate_qr_code shortUrl="https://pse.is/mylink" size=300
```

---

### `get_analytics`

Click stats for a short link. **Requires authentication.**

**Parameters:**
- `encodeId` (string, **required**): Slug of the short link (e.g. `abc123` from `pse.is/abc123`)

**Returns:** `{ success, data: { totalClicks, uniqueClicks, dailyClicks[] } }`

---

### `generate_analytics_chart`

Generate a line chart URL visualizing daily click trends. **Only call this if the user explicitly requests a chart or visualization.** Automatically shortens the chart URL via PicSee (unauthenticated mode, no quota consumed).

**Parameters:**
- `dailyClicks` (array, **required**): Array of `{ date, clicks }` objects. Date format: `YYYY-MM-DD` or `MM/DD`
- `encodeId` (string): Link slug (used in chart title)

**Returns:** `{ success, chartUrl, originalChartUrl, dataPoints, note }`

**Example:**
```bash
mcporter call picsee.generate_analytics_chart \
  dailyClicks='[{"date":"3/1","clicks":45},{"date":"3/2","clicks":67}]' \
  encodeId="mylink"
```

---

### `list_links`

List and search links. **Requires authentication.** Results are returned in reverse chronological order (newest first).

**Parameters:**
- `startTime` (string, **required**): Query backward from this time. **Use the LAST moment** of the desired period. Format: `YYYY-MM-DDTHH:MM:SS`. Example: `2026-03-31T23:59:59` for March 2026
- `limit` (number): Results per page (1-50, default 50)
- `isAPI` (boolean): Filter API-generated links only
- `isStar` (boolean): Filter starred links only
- `prevMapId` (string): Pagination cursor — links older than this mapId
- `externalId` (string): Filter by external ID
- `keyword` (string): Search title/description (Advanced, 3-30 chars)
- `tag` (string): Filter by tag (Advanced)
- `url` (string): Filter by exact destination URL
- `encodeId` (string): Filter by exact slug
- `authorId` (string): Filter by author ID

**Returns:** `{ success, data: [{ encodeId, domain, url, title, totalClicks, uniqueClicks, createTime, tags, utm }] }`

**Common mistake:** Using the first day of a month as startTime (e.g. `2026-03-01`) will miss most of that month's data. Always use the last day.

---

### `edit_link`

Edit an existing short link. **Requires authentication + Advanced plan.** Only include fields to change.

**Parameters:**
- `originalEncodeId` (string, **required**): Current slug of the link to edit
- `url` (string): New destination URL
- `encodeId` (string): New custom slug
- `domain` (string): New domain
- `title` (string): New preview title
- `description` (string): New preview description
- `imageUrl` (string): New preview thumbnail
- `tags` (string[]): New tags
- `fbPixel` (string): Meta Pixel ID
- `gTag` (string): Google Tag Manager ID
- `utm` (object): New UTM parameters
- `expireTime` (string): Expiration time (ISO 8601)

**Returns:** `{ success, message }`

---

### `delete_link`

Delete or recover a short link. **Requires authentication.**

**Parameters:**
- `encodeId` (string, **required**): Slug of the short link
- `action` (string): `"delete"` (default) or `"recover"`

**Returns:** `{ success, message }`

---

### `setup_auth`

Store and verify a PicSee API token. Token is encrypted with AES-256-CBC using a machine-specific key (hostname + username → SHA-256) and saved to `~/.openclaw/.picsee_token`.

**Parameters:**
- `token` (string, **required**): PicSee API token

**Returns:** `{ success, plan, quota, usage, message }`

**Token source:** https://picsee.io/ → avatar → Settings → API → Copy token

---

## Auth Modes

| Mode | API Host | Features |
|------|----------|----------|
| **Unauthenticated** | `chrome-ext.picsee.tw` | Create short links only |
| **Authenticated** | `api.pics.ee` | Create + analytics + list + search + edit + delete |

**Auto-detection:** If encrypted token exists at `~/.openclaw/.picsee_token`, authenticated mode is used. Otherwise unauthenticated.

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

**After successfully shortening a URL**, proactively ask the user: "Would you like a QR code for this short link?"

If yes, generate it:

```bash
curl -s -o /tmp/<ENCODE_ID>_qr.png "https://api.qrserver.com/v1/create-qr-code/?size=300x300&data=https://pse.is/<ENCODE_ID>"
```

Send via `message` tool with `filePath: "/tmp/<ENCODE_ID>_qr.png"`.

### Analytics Chart

**After calling `get_analytics` and displaying click statistics**, proactively ask the user: "Would you like to see a visual chart of the daily clicks?"

If yes, generate it:

1. Extract `dailyClicks` array from the response
2. Build a QuickChart URL with the data:

```bash
# Example with 3 data points — construct dynamically from actual dailyClicks
curl -s -o /tmp/<ENCODE_ID>_chart.png "https://quickchart.io/chart?w=600&h=300&c=\
{type:'line',data:{labels:['3/1','3/2','3/3'],datasets:[{label:'Clicks',data:[45,67,23],borderColor:'rgb(59,130,246)',fill:false}]}}"
```

Send via `message` tool with `filePath: "/tmp/<ENCODE_ID>_chart.png"`.
