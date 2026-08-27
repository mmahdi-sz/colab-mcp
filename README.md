# Colab MCP (Enhanced Edition)

[![Python](https://img.shields.io/badge/Python-3.10%2B-blue.svg)](https://python.org)
[![MCP Protocol](https://img.shields.io/badge/MCP-Protocol%20Compatible-green.svg)](https://modelcontextprotocol.io)
[![License](https://img.shields.io/badge/License-Apache%202.0-orange.svg)](LICENSE)
[![Tested On](https://img.shields.io/badge/Tested%20With-Google%20Antigravity%20%7C%20Claude%20Code-purple.svg)](#supported-clients)

**[English](README.md)** | **[فارسی (Persian)](README.fa.md)**

A robust, production-ready [Model Context Protocol (MCP)](https://modelcontextprotocol.io) server that bridges local AI agents to active [Google Colab](https://colab.research.google.com) notebook sessions in the browser.

---

## 🌟 Key Improvements in this Edition

This fork addresses real-world networking and runtime issues found in Linux, remote SSH/xrdp, and modern agentic coding environments:

1. **Clean STDIO JSON-RPC Streams:**
   - Silences FastMCP's ANSI/Rich styling and banner on startup (`show_banner=False`).
   - Suppresses third-party deprecation warnings so stdio pipes are never corrupted by non-JSON text.
2. **Dual-Stack IPv4 / IPv6 Port Alignment:**
   - Resolves Linux ephemeral port collisions where `localhost` allocated disparate ports for `127.0.0.1` and `::1`.
3. **Headless & Remote Environment Safety:**
   - Replaces bare `webbrowser.open_new` with `_safe_open_browser` to prevent TTY-based browsers (`lynx`, `w3m`) from hijacking stdio pipes.
   - Automatically writes connection links to `/tmp/colab_mcp_url.txt` for headless/remote hosts.
4. **Resilient Handshake Handling:**
   - Gracefully ignores Colab frontend extension messages (e.g., `server/discover`) without crashing the underlying MCP stream.

---

## 🖥️ Supported Clients

Requires client support for local MCP transports and `notifications/tools/list_changed`:
* **Google Antigravity**
* **Claude Code & Claude Desktop**
* **Gemini CLI**
* **Windsurf & Cursor**
* **Goose / Zed**

---

## 🚀 Quick Start & Installation

### 1. Prerequisites
* Python 3.10+
* [`uv`](https://docs.astral.sh/uv/) (recommended) or standard `pip` / `venv`
* Google Chrome or any modern web browser

### 2. Install Package

#### Option A: Direct from Git Repository (Recommended)
```bash
uv venv ~/.local/share/colab-mcp-venv --python python3
uv pip install git+https://github.com/googlecolab/colab-mcp.git --python ~/.local/share/colab-mcp-venv/bin/python
```

#### Option B: Editable / Local Clone
```bash
git clone https://github.com/googlecolab/colab-mcp.git ~/colab-mcp
uv venv ~/.local/share/colab-mcp-venv --python python3
uv pip install -e ~/colab-mcp --python ~/.local/share/colab-mcp-venv/bin/python
```

### 3. Create the Production Wrapper Script
To guarantee pristine STDIO communication, create a clean wrapper:

```bash
cat << 'WRAPPER_EOF' > ~/.local/share/colab-mcp-venv/bin/colab-mcp-wrapper
#!/bin/bash
export NO_COLOR=1
export PYTHONWARNINGS="ignore"
export FASTMCP_DISABLE_VERSION_CHECK=1
exec /home/$USER/.local/share/colab-mcp-venv/bin/colab-mcp "$@" 2>>/tmp/colab_mcp_stderr.log
WRAPPER_EOF

chmod +x ~/.local/share/colab-mcp-venv/bin/colab-mcp-wrapper
```

---

## ⚙️ Client Configurations

### Google Antigravity (`~/.gemini/config/mcp_config.json`)
```json
{
  "mcpServers": {
    "colab-mcp": {
      "command": "/home/YOUR_USER/.local/share/colab-mcp-venv/bin/colab-mcp-wrapper",
      "args": [],
      "env": {
        "FASTMCP_DISABLE_VERSION_CHECK": "1",
        "NO_COLOR": "1"
      },
      "timeout": 30000
    }
  }
}
```

### Claude Desktop (`claude_desktop_config.json`)
```json
{
  "mcpServers": {
    "colab-mcp": {
      "command": "/home/YOUR_USER/.local/share/colab-mcp-venv/bin/colab-mcp-wrapper",
      "args": []
    }
  }
}
```

---

## 📖 How to Connect to Your Notebook

### Method 1: Automatic Link Connection
1. Ask your AI agent to connect (triggers `open_colab_browser_connection`).
2. Open the generated Colab URL in your browser:
   ```text
   https://colab.research.google.com/notebooks/empty.ipynb#mcpProxyToken=TOKEN&mcpProxyPort=PORT
   ```
3. A security dialog with title **"Connect to a local Colab MCP server"** will appear. Click **Connect**.

### Method 2: Connect an Existing Open Notebook
1. In your open Colab tab, press **`Ctrl + Shift + P`** (Command Palette).
2. Type **`Connect to a local Colab MCP server`** and press **Enter**.
3. Paste the `TOKEN&PORT` provided by your agent and click **Connect**.
4. A green notification **`Connected to the local Colab MCP server`** will appear in the bottom-left corner.

---

## 📄 License
Licensed under the [Apache License, Version 2.0](LICENSE).
