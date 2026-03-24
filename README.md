# whatsapp

This is a WhatsApp MCP server that connects Claude (or Cursor) to your personal WhatsApp account — read messages, search contacts, and send texts or media.

> **Note:** Like most MCP servers, this is vulnerable to [prompt injection attacks](https://simonwillison.net/2025/Jun/16/the-lethal-trifecta/) that could expose private data.

---

## How it works

Three layers talk to each other:

```
Claude ↔ Python MCP Server ↔ Go Bridge ↔ WhatsApp Web API
```

Messages are stored locally in SQLite and only sent to Claude when a tool explicitly requests them.

---

## Requirements

- Go
- Python 3.6+
- [uv](https://astral.sh/uv) — `curl -LsSf https://astral.sh/uv/install.sh | sh`
- Claude Desktop or Cursor
- FFmpeg *(optional)* — needed only for sending `.ogg` voice messages

---

## Setup

### 1. Clone & start the bridge

```bash
git clone https://github.com/lharries/whatsapp-mcp.git
cd whatsapp-mcp/whatsapp-bridge
go run main.go
```

Scan the QR code with WhatsApp on your phone. Re-authentication is needed roughly every 20 days.

### 2. Configure your client

Add this to your config file, filling in the two paths:

```json
{
  "mcpServers": {
    "whatsapp": {
      "command": "{{PATH_TO_UV}}",
      "args": ["--directory", "{{PATH_TO_REPO}}/whatsapp-mcp-server", "run", "main.py"]
    }
  }
}
```

| Client | Config file location |
|--------|----------------------|
| Claude Desktop | `~/Library/Application Support/Claude/claude_desktop_config.json` |
| Cursor | `~/.cursor/mcp.json` |

Restart the app after saving.

### 3. Windows only

CGO must be enabled and a C compiler installed (use [MSYS2](https://www.msys2.org/)):

```bash
go env -w CGO_ENABLED=1
go run main.go
```

---

## Available tools

| Tool | What it does |
|------|--------------|
| `search_contacts` | Find contacts by name or number |
| `list_chats` | List chats with metadata |
| `list_messages` | Fetch messages with filters |
| `get_last_interaction` | Most recent message with a contact |
| `get_message_context` | Surrounding context for a message |
| `send_message` | Send text to a contact or group |
| `send_file` | Send image, video, or document |
| `send_audio_message` | Send a `.ogg` voice message |
| `download_media` | Download media and return local path |

---

## Troubleshooting

| Problem | Fix |
|---------|-----|
| QR code not showing | Restart the bridge; check terminal compatibility |
| Already logged in | Bridge reconnects automatically — no action needed |
| Too many linked devices | Remove a device in WhatsApp → Settings → Linked Devices |
| Messages out of sync | Delete `store/messages.db` and `store/whatsapp.db`, restart bridge |
| `uv` permission error | Add uv to your PATH or use its full path |

See the [MCP docs](https://modelcontextprotocol.io/quickstart/server#claude-for-desktop-integration-issues) for further help.
