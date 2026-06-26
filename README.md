# @exitanty/mcp

Use **ExitAnty** on your iPhone from Claude Desktop, Claude Code, Cursor,
VS Code, OpenAI Codex, Ollama, and other MCP-compatible clients.

ExitAnty is the first WebKit-based antidetect browser: real Safari/WebKit on a
real iPhone, not a headless Chromium.

## Requirements

- **ExitAnty** installed on an iPhone — https://exitanty.com
- The iPhone and the computer running the MCP client on the **same Wi-Fi network**
- **Node.js 18+** on the computer (the client runs the server via `npx`)
- The ExitAnty app open on the iPhone while using the client

> **Note for nvm users:** GUI apps (Claude Desktop, Cursor, VS Code) usually do
> not see a Node.js installed through `nvm`, because they do not load your shell
> `PATH`. In those apps, replace `"npx"` with the absolute path from `which npx`
> (often `/opt/homebrew/bin/npx`). Command-line tools such as Claude Code, Codex
> and `ollmcp` usually inherit your shell `PATH`, so plain `npx` normally works.

## 1. Get Token, Port And IP

On the iPhone, open **ExitAnty → Settings → Automation**:

1. Copy the token from this screen.
2. Note the port from this screen (default `9519`).

Then find the iPhone's **local LAN IP address** in iOS settings:
**Settings → Wi-Fi → ⓘ** next to your connected network → **IP Address**
(for example `192.168.1.72`). This must be the iPhone's local Wi-Fi IP, not the
computer IP, not a public IP, and not `127.0.0.1` / `localhost`. If the iPhone is
sharing its own Personal Hotspot, the phone address is usually `172.20.10.1`.

You now have the three values the server needs:

| Value            | Where                                         | Example        |
| ---------------- | --------------------------------------------- | -------------- |
| `EXITANTY_HOST`  | iPhone local IP from iOS Settings → Wi-Fi → ⓘ | `192.168.1.72` |
| `EXITANTY_PORT`  | ExitAnty → Settings → Automation → Port       | `9519`         |
| `EXITANTY_TOKEN` | ExitAnty → Settings → Automation → Token      | `your-token`   |

## 2. Add the server to your MCP client

The server is published to npm and runs with `npx` — no manual install needed.

### Claude Code

```bash
claude mcp add exitanty -s user \
  -e EXITANTY_HOST=192.168.1.72 \
  -e EXITANTY_PORT=9519 \
  -e EXITANTY_TOKEN=your-token \
  -- npx -y @exitanty/mcp
```

`-s user` makes it available in every Claude Code project. Verify with
`claude mcp list`.

### Claude Desktop

Edit `claude_desktop_config.json` (Settings → Developer → Edit Config). Quit
Claude Desktop completely before editing, because a running app can overwrite
the file.

```json
{
  "mcpServers": {
    "exitanty": {
      "command": "npx",
      "args": ["-y", "@exitanty/mcp"],
      "env": {
        "EXITANTY_HOST": "192.168.1.72",
        "EXITANTY_PORT": "9519",
        "EXITANTY_TOKEN": "your-token"
      }
    }
  }
}
```

Reopen Claude Desktop after saving the config.

### Cursor

Edit `~/.cursor/mcp.json` (global) or `.cursor/mcp.json` (per project):

```json
{
  "mcpServers": {
    "exitanty": {
      "command": "npx",
      "args": ["-y", "@exitanty/mcp"],
      "env": {
        "EXITANTY_HOST": "192.168.1.72",
        "EXITANTY_PORT": "9519",
        "EXITANTY_TOKEN": "your-token"
      }
    }
  }
}
```

Then open Cursor → Settings → **MCP** and confirm `exitanty` is enabled.

### VS Code

Edit `.vscode/mcp.json`:

```json
{
  "servers": {
    "exitanty": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@exitanty/mcp"],
      "env": {
        "EXITANTY_HOST": "192.168.1.72",
        "EXITANTY_PORT": "9519",
        "EXITANTY_TOKEN": "your-token"
      }
    }
  }
}
```

### Codex (OpenAI Codex CLI)

```bash
codex mcp add \
  --env EXITANTY_HOST=192.168.1.72 \
  --env EXITANTY_PORT=9519 \
  --env EXITANTY_TOKEN=your-token \
  exitanty -- npx -y @exitanty/mcp
```

Verify with `codex mcp list`.

### Ollama

Use an MCP-aware Ollama client such as `ollmcp`:

```bash
pip install --upgrade ollmcp

ollmcp mcp add \
  --env EXITANTY_HOST=192.168.1.72 \
  --env EXITANTY_PORT=9519 \
  --env EXITANTY_TOKEN=your-token \
  exitanty -- npx -y @exitanty/mcp

ollmcp --model qwen2.5:14b
```

Use an Ollama model that supports tool calling.

Restart the client. Ask it something like:

> Show my ExitAnty profiles.

## Environment variables

| Name             | Default | Required | Description                                   |
| ---------------- | ------- | -------- | --------------------------------------------- |
| `EXITANTY_HOST`  | —       | yes      | iPhone local IP from iOS Settings → Wi-Fi → ⓘ |
| `EXITANTY_PORT`  | `9519`  | no       | Port from ExitAnty → Settings → Automation    |
| `EXITANTY_TOKEN` | —       | yes      | Token from ExitAnty → Settings → Automation   |

## Troubleshooting

- **Cannot reach iPhone** — open the ExitAnty app, check the setting in
  Settings → Automation, and make sure `EXITANTY_HOST` is the iPhone local IP
  from iOS Settings → Wi-Fi → ⓘ, not the computer IP, public IP, `127.0.0.1`, or
  `localhost`. Direct check:
  `curl -i -H "Authorization: Bearer $EXITANTY_TOKEN" http://$EXITANTY_HOST:$EXITANTY_PORT/status`
- **Unauthorized** — the token in the config does not match the token in the
  ExitAnty app.
- **Captcha / 2FA** — complete it manually on the iPhone.
