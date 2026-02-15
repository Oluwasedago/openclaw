![OpenClaw](https://img.shields.io/badge/OpenClaw-2026.2.13-red?style=for-the-badge)
![WSL2](https://img.shields.io/badge/WSL-2-blue?style=for-the-badge)
![Ubuntu](https://img.shields.io/badge/Ubuntu-24.04-orange?style=for-the-badge)
![WhatsApp](https://img.shields.io/badge/WhatsApp-Gateway-green?style=for-the-badge)

# 🦞 OpenClaw Complete Setup, Autostart & Migration Guide

> **Version:** 2026.2.13
> **Platform:** Windows 10/11 + WSL2 (Ubuntu)
> **Author:** Sedago
> **Last Updated:** February 2026

---

## 📑 Table of Contents

| # | Section | Description |
|---|---------|-------------|
| 1 | [Prerequisites](#-1-prerequisites) | What you need before starting |
| 2 | [Fresh Install](#-2-fresh-install-on-a-new-pc) | Installing OpenClaw from scratch |
| 3 | [Configuration](#-3-configuration) | openclaw.json and workspace setup |
| 4 | [WhatsApp Linking](#-4-whatsapp-linking) | Pairing your assistant number |
| 5 | [Gateway Operations](#-5-gateway-operations) | Starting, stopping, status checks |
| 6 | [Autostart Setup](#-6-autostart-setup-windows--wsl2) | Full auto-start on Windows login |
| 7 | [API Keys & Providers](#-7-api-keys--providers) | Changing keys and switching models |
| 8 | [Migration to Another PC](#-8-migration-to-another-pc) | Moving your entire setup |
| 9 | [Troubleshooting](#-9-troubleshooting) | Common errors and fixes |
| 10 | [Quick Reference](#-10-quick-reference) | Cheat sheet of common commands |

---

## 🔧 1. Prerequisites

### Hardware & Software

- [x] Windows 10/11 with WSL2 enabled
- [x] Ubuntu installed via Microsoft Store (or `wsl --install`)
- [x] Node.js 18+ installed inside WSL
- [x] A second phone number (SIM/eSIM/prepaid) for the WhatsApp assistant
- [x] An API key from a supported LLM provider

### Verify WSL Setup (PowerShell)

    wsl -l -v

Expected output:

    NAME              STATE           VERSION
    * Ubuntu            Running         2

### Verify Node.js (inside WSL)

    node --version
    npm --version

---

## 📦 2. Fresh Install on a New PC

### Step 2.1: Install OpenClaw globally

    sudo npm install -g openclaw

### Step 2.2: Run initial setup

    openclaw setup

This creates the workspace at `~/.openclaw/` with starter files:

    ~/.openclaw/
    ├── openclaw.json          # Main configuration
    ├── workspace/
    │   ├── AGENTS.md          # Agent operating instructions
    │   ├── SOUL.md            # Persona/personality
    │   ├── TOOLS.md           # Available tools
    │   ├── IDENTITY.md        # Agent identity
    │   ├── USER.md            # User info
    │   ├── HEARTBEAT.md       # Heartbeat instructions (optional)
    │   └── MEMORY.md          # Agent memory (optional, not auto-created)
    ├── agents/
    │   └── main/
    │       └── sessions/
    │           └── sessions.json
    └── scripts/
        └── start_openclaw_with_prompt.sh

### Step 2.3: Set your API key as environment variable

Edit your `~/.bashrc`:

    nano ~/.bashrc

Add your provider's API key (see Section 7 for all providers):

    # For OpenRouter
    export OPENROUTER_API_KEY="sk-or-v1-your-key-here"

    # OR for Anthropic direct
    export ANTHROPIC_API_KEY="sk-ant-your-key-here"

    # OR for OpenAI direct
    export OPENAI_API_KEY="sk-your-key-here"

Reload:

    source ~/.bashrc

### Step 2.4: Verify installation

    openclaw status

---

## ⚙️ 3. Configuration

### Main config file: `~/.openclaw/openclaw.json`

    {
      "logging": { "level": "info" },
      "agent": {
        "model": "anthropic/claude-opus-4-6",
        "workspace": "~/.openclaw/workspace",
        "thinkingDefault": "high",
        "timeoutSeconds": 1800,
        "heartbeat": { "every": "0m" }
      },
      "channels": {
        "whatsapp": {
          "allowFrom": ["+1XXXXXXXXXX"],
          "groups": {
            "*": { "requireMention": true }
          }
        }
      },
      "routing": {
        "groupChat": {
          "mentionPatterns": ["@openclaw", "openclaw"]
        }
      },
      "session": {
        "scope": "per-sender",
        "resetTriggers": ["/new", "/reset"],
        "reset": {
          "mode": "daily",
          "atHour": 4,
          "idleMinutes": 10080
        }
      }
    }

### Key configuration options

| Setting | Description | Example |
|---------|-------------|---------|
| `agent.model` | LLM model to use | `anthropic/claude-opus-4-6` |
| `agent.workspace` | Path to workspace directory | `~/.openclaw/workspace` |
| `agent.heartbeat.every` | Heartbeat interval (`0m` = disabled) | `30m` |
| `channels.whatsapp.allowFrom` | Allowlisted phone numbers | `["+15551234567"]` |
| `session.resetTriggers` | Commands to reset session | `["/new", "/reset"]` |

> ⚠️ **SECURITY:** Always set `channels.whatsapp.allowFrom`. Never run open-to-the-world.

---

## 📱 4. WhatsApp Linking

### Step 4.1: Login (pair via QR code)

    openclaw channels login

This displays a QR code. Scan it with the **assistant phone** (second number), not your personal phone.

### Step 4.2: Verify link

    openclaw status

You should see:

    WhatsApp: linked (auth age Xm)
    Web Channel: +XXXXXXXXXXX

### Two-Phone Architecture

    ┌─────────────────────┐         ┌──────────────────────┐
    │  Your Personal Phone│         │  Assistant Phone      │
    │  +1-555-YOU         │────────>│  +1-555-ASSIST        │
    │  (sends messages)   │ WhatsApp│  (linked via QR)      │
    └─────────────────────┘         └──────────┬───────────┘
                                               │
                                               │ WhatsApp Web
                                               │
                                    ┌──────────▼───────────┐
                                    │  Your PC (WSL)       │
                                    │  OpenClaw Gateway     │
                                    │  Pi Agent             │
                                    └──────────────────────┘

---

## 🚀 5. Gateway Operations

### Start the gateway

    openclaw gateway --port 18789

### Stop the gateway

    openclaw gateway stop

### Check status

    openclaw status
    openclaw status --all     # Full diagnosis
    openclaw status --deep    # Includes health probes
    openclaw health --json    # JSON health snapshot

### Open the dashboard

    openclaw dashboard

### Session management

| Command | Action |
|---------|--------|
| `/new` or `/reset` | Start fresh session (send via WhatsApp) |
| `/compact [instructions]` | Compact session context |

### Logs

    # View logs
    cat /tmp/openclaw/openclaw-$(date +%Y-%m-%d).log

    # Tail logs in real-time
    tail -f /tmp/openclaw/openclaw-$(date +%Y-%m-%d).log

---

## 🔄 6. Autostart Setup (Windows + WSL2)

This section provides the complete auto-start configuration so the OpenClaw gateway starts automatically when you log into Windows.

### Architecture Overview

    Windows Login
        │
        ▼
    Task Scheduler triggers "OpenClaw Autostart"
        │
        ▼
    cmd.exe runs C:\Users\<USER>\start_openclaw_prompt.bat
        │
        ▼
    Batch file calls: wsl -d Ubuntu bash -lc "<script>"
        │
        ▼
    WSL script checks if gateway is running
        │
        ├── Already running? → Exit gracefully
        │
        └── Not running? → Prompt user → Start gateway

### Step 6.1: Create the WSL startup script

Inside WSL:

    mkdir -p ~/.openclaw/scripts

    cat << 'SCRIPT' > /home/sedago/.openclaw/scripts/start_openclaw_with_prompt.sh
    #!/usr/bin/env bash
    set -euo pipefail

    # Check if gateway is already running
    WSL_GATEWAY_PID=$(pgrep -f "openclaw gateway" | head -n 1 || echo "")
    if [ -n "${WSL_GATEWAY_PID}" ]; then
      echo "OpenClaw gateway already running (PID ${WSL_GATEWAY_PID})."
      exit 0
    fi

    # Prompt for user consent to start
    read -p "Start OpenClaw gateway? (y/n): " answer
    case "${answer}" in
      [yY][eE][sS]|[yY])
        echo "Starting OpenClaw gateway..."
        openclaw gateway start
        echo "Gateway start command issued."
        echo "Check status with: openclaw gateway status"
        ;;
      *)
        echo "Gateway start cancelled."
        ;;
    esac
    SCRIPT

    chmod +x /home/sedago/.openclaw/scripts/start_openclaw_with_prompt.sh

### Step 6.2: Create the Windows batch file

Exit WSL (`exit`), then in **Command Prompt**:

    echo @echo off > C:\Users\<USER>\start_openclaw_prompt.bat
    echo REM Launch OpenClaw prompt in WSL (Ubuntu) >> C:\Users\<USER>\start_openclaw_prompt.bat
    echo wsl -d Ubuntu bash -lc "/home/sedago/.openclaw/scripts/start_openclaw_with_prompt.sh" >> C:\Users\<USER>\start_openclaw_prompt.bat

> **Replace `<USER>` with your actual Windows user folder name.**
> To find it, run: `echo %USERNAME%` in Command Prompt.
> Note: The Windows username and the user folder name may differ.
> Check `C:\Users\` to see the actual folder name.
> In the original setup, username was `Sedago` but folder was `a`.

Verify:

    type C:\Users\<USER>\start_openclaw_prompt.bat

Expected output:

    @echo off
    REM Launch OpenClaw prompt in WSL (Ubuntu)
    wsl -d Ubuntu bash -lc "/home/sedago/.openclaw/scripts/start_openclaw_with_prompt.sh"

### Step 6.3: Create the Task Scheduler task

In **Administrator Command Prompt**:

    schtasks /create /tn "OpenClaw Autostart" /tr "cmd.exe /c C:\Users\<USER>\start_openclaw_prompt.bat" /sc onlogon /rl highest /f

> **Replace `<USER>` with your actual Windows user folder name.**

Verify:

    schtasks /query /tn "OpenClaw Autostart"

Expected output:

    TaskName                     Next Run Time          Status
    ============================ ====================== ===============
    OpenClaw Autostart           N/A                    Ready

### Step 6.4: Test the autostart

    schtasks /run /tn "OpenClaw Autostart"

A new Command Prompt window should appear asking:

    Start OpenClaw gateway? (y/n):

Type `y` to confirm.

### Step 6.5: (Alternative) .bashrc auto-start

If you prefer the gateway to start automatically every time you open a WSL terminal (no prompt), add this to `~/.bashrc`:

    # Auto-start OpenClaw gateway
    if ! pgrep -f "openclaw gateway" > /dev/null 2>&1; then
        echo "OpenClaw gateway not running. Starting..."
        openclaw gateway --port 18789 &
        disown
        echo "OpenClaw gateway started."
    fi

Add it with:

    cat << 'EOF' >> ~/.bashrc

    # Auto-start OpenClaw gateway
    if ! pgrep -f "openclaw gateway" > /dev/null 2>&1; then
        echo "OpenClaw gateway not running. Starting..."
        openclaw gateway --port 18789 &
        disown
        echo "OpenClaw gateway started."
    fi
    EOF

### Autostart Method Comparison

| Method | Trigger | Prompt? | Needs Terminal? |
|--------|---------|---------|-----------------|
| Task Scheduler + Batch | Windows login | Yes (y/n) | Auto-opens one |
| .bashrc | Opening WSL terminal | No (auto) | Yes (manual) |
| Both combined | Either event | Mixed | Flexible |

### Managing the Scheduled Task

| Action | Command |
|--------|---------|
| Check status | `schtasks /query /tn "OpenClaw Autostart"` |
| Run manually | `schtasks /run /tn "OpenClaw Autostart"` |
| Disable | `schtasks /change /tn "OpenClaw Autostart" /disable` |
| Enable | `schtasks /change /tn "OpenClaw Autostart" /enable` |
| Delete | `schtasks /delete /tn "OpenClaw Autostart" /f` |

---

## 🔑 7. API Keys & Providers

### Currently Supported Providers & Environment Variables

| Provider | Environment Variable | Model Example |
|----------|---------------------|---------------|
| OpenRouter (multi-provider) | `OPENROUTER_API_KEY` | `anthropic/claude-opus-4-6` |
| Anthropic (direct) | `ANTHROPIC_API_KEY` | `claude-opus-4-6` |
| OpenAI (direct) | `OPENAI_API_KEY` | `gpt-4o` |
| Google (direct) | `GOOGLE_API_KEY` | `gemini-pro` |

### How to Change Your API Key

#### Step 1: Edit `.bashrc`

    nano ~/.bashrc

#### Step 2: Find and replace the key line

**Remove or comment out the old key:**

    # OLD (comment out or delete)
    # export OPENROUTER_API_KEY="sk-or-v1-old-key-here"

**Add the new key:**

    # NEW
    export OPENROUTER_API_KEY="sk-or-v1-new-key-here"

#### Step 3: Reload

    source ~/.bashrc

#### Step 4: Restart the gateway

    openclaw gateway stop
    openclaw gateway --port 18789

### How to Switch Providers Entirely

#### Step 1: Set the new provider's API key in `~/.bashrc`

    # Example: switching from OpenRouter to Anthropic direct
    # Comment out old:
    # export OPENROUTER_API_KEY="sk-or-v1-old-key"

    # Add new:
    export ANTHROPIC_API_KEY="sk-ant-your-new-key-here"

#### Step 2: Update the model in `~/.openclaw/openclaw.json`

    nano ~/.openclaw/openclaw.json

Change the `agent.model` field:

    // OpenRouter format (provider/model):
    "model": "anthropic/claude-opus-4-6"

    // Anthropic direct format (just model name):
    "model": "claude-opus-4-6"

    // OpenAI direct:
    "model": "gpt-4o"

#### Step 3: Reload

    source ~/.bashrc
    openclaw gateway stop
    openclaw gateway --port 18789

### Where to Get API Keys

| Provider | URL | Notes |
|----------|-----|-------|
| OpenRouter | [openrouter.ai/keys](https://openrouter.ai/keys) | Access to 100+ models, pay-per-use |
| Anthropic | [console.anthropic.com](https://console.anthropic.com/) | Direct Claude access |
| OpenAI | [platform.openai.com/api-keys](https://platform.openai.com/api-keys) | Direct GPT access |
| Google | [aistudio.google.com](https://aistudio.google.com/) | Gemini models |

> ⚠️ **SECURITY:** Never commit API keys to git. Never share screenshots containing keys. Rotate keys immediately if exposed. Use `.env` files or environment variables only.

---

## 🚚 8. Migration to Another PC

### What to Back Up

| Item | Path (WSL) | Purpose |
|------|------------|---------|
| Config | `~/.openclaw/openclaw.json` | All settings |
| Workspace | `~/.openclaw/workspace/` | Agent memory, persona, tools |
| Sessions | `~/.openclaw/agents/` | Chat history |
| Scripts | `~/.openclaw/scripts/` | Autostart scripts |
| Env vars | `~/.bashrc` | API keys (lines with `export`) |
| Batch file | `C:\Users\<USER>\start_openclaw_prompt.bat` | Windows autostart |

### Step-by-Step Migration

#### On the OLD PC (inside WSL):

**Step 1: Create a backup archive**

    cd ~
    tar czf openclaw-backup-$(date +%Y%m%d).tar.gz \
      .openclaw/openclaw.json \
      .openclaw/workspace/ \
      .openclaw/agents/ \
      .openclaw/scripts/

**Step 2: Export your environment variables**

    grep -E "^export (OPENROUTER|ANTHROPIC|OPENAI|GOOGLE)_API_KEY" ~/.bashrc > ~/openclaw-env-backup.txt

**Step 3: Copy the backup to a USB drive or cloud**

    # Copy to Windows filesystem (accessible from Windows)
    cp ~/openclaw-backup-*.tar.gz /mnt/c/Users/<USER>/Desktop/
    cp ~/openclaw-env-backup.txt /mnt/c/Users/<USER>/Desktop/

**Step 4: Also save the Windows batch file**

In Windows, copy `C:\Users\<USER>\start_openclaw_prompt.bat` to USB/cloud.

#### On the NEW PC:

**Step 1: Install prerequisites**

    # In WSL (Ubuntu)
    sudo npm install -g openclaw
    openclaw setup

**Step 2: Restore the backup**

    # Copy backup file into WSL first
    cp /mnt/c/Users/<NEWUSER>/Desktop/openclaw-backup-*.tar.gz ~/

    # Extract (overwrites defaults from setup)
    cd ~
    tar xzf openclaw-backup-*.tar.gz

**Step 3: Restore environment variables**

    cat ~/openclaw-env-backup.txt >> ~/.bashrc
    source ~/.bashrc

**Step 4: Re-link WhatsApp**

> ⚠️ WhatsApp credentials do NOT transfer. You must re-pair on the new PC.

    openclaw channels login

Scan the QR code with your assistant phone.

**Step 5: Set up autostart (follow Section 6)**

Create the batch file and Task Scheduler task on the new PC.

Adjust paths if the Windows username/folder is different:

    # Find the new Windows user folder
    # In PowerShell:
    echo $env:USERNAME
    # In Command Prompt:
    echo %USERNAME%
    # Check actual folder: dir C:\Users\

**Step 6: Verify everything**

    openclaw status --all

### Migration Checklist

- [ ] OpenClaw installed on new PC (`sudo npm install -g openclaw`)
- [ ] `openclaw setup` run on new PC
- [ ] Backup archive extracted to `~/.openclaw/`
- [ ] API key environment variable added to `~/.bashrc`
- [ ] `source ~/.bashrc` run
- [ ] WhatsApp re-paired (`openclaw channels login`)
- [ ] Gateway starts successfully (`openclaw gateway --port 18789`)
- [ ] Messages send and receive on WhatsApp
- [ ] Autostart batch file created (Section 6.2)
- [ ] Task Scheduler task created (Section 6.3)
- [ ] Autostart tested (`schtasks /run /tn "OpenClaw Autostart"`)
- [ ] Old PC cleaned up (optional)

### Cleaning Up the Old PC (Optional)

    # In WSL on old PC
    openclaw gateway stop
    rm -rf ~/.openclaw

    # In Windows (Admin Command Prompt)
    schtasks /delete /tn "OpenClaw Autostart" /f
    del C:\Users\<USER>\start_openclaw_prompt.bat

---

## 🔧 9. Troubleshooting

### Common Errors & Fixes

| Error | Cause | Fix |
|-------|-------|-----|
| `Permission denied` on `npm install` | Need sudo for global install | `sudo npm install -g openclaw` |
| `Gateway already running (pid XXXX)` | Previous instance still active | `openclaw gateway stop` then start again |
| `Port 18789 is already in use` | Same as above | `openclaw gateway stop` |
| `systemctl --user unavailable` | WSL2 doesn't have full systemd | Ignore; start gateway manually |
| `-bash: home/linuxbrew/.linuxbrew/bin/brew: No such file or directory` | Missing `/` in brew path | Fix in `.bashrc`: change `home/` to `/home/` |
| `Set-Content is not recognized` | Running PowerShell command in cmd | Switch to PowerShell or use `echo >` syntax |
| `wsl -d bash` fails | `bash` is not a distro name | Use `wsl -d Ubuntu` |
| `%USERNAME%` prints literally | Using cmd syntax in PowerShell | Use `$env:USERNAME` in PowerShell |
| `EACCES` / npm permission errors | Global npm dir owned by root | Use `sudo` or fix npm prefix |
| WhatsApp not receiving messages | Gateway not running or not linked | Run `openclaw status` and `openclaw channels login` |

### Permission Fix for npm (Permanent)

    mkdir -p ~/.npm-global
    npm config set prefix '~/.npm-global'
    echo 'export PATH=~/.npm-global/bin:$PATH' >> ~/.bashrc
    source ~/.bashrc
    npm install -g openclaw

### Reset Everything

    openclaw gateway stop
    rm -rf ~/.openclaw
    openclaw setup
    openclaw channels login

---

## 📋 10. Quick Reference

### Essential Commands

    # Status
    openclaw status
    openclaw status --all
    openclaw status --deep

    # Gateway
    openclaw gateway --port 18789    # Start
    openclaw gateway stop            # Stop
    openclaw health --json           # Health check

    # WhatsApp
    openclaw channels login          # Pair via QR

    # Dashboard
    openclaw dashboard               # Open web UI

    # Update
    sudo openclaw update             # Update to latest

    # Sessions (send via WhatsApp)
    /new                             # New session
    /reset                           # Reset session
    /compact                         # Compact context

### Key File Locations (WSL)

    ~/.openclaw/openclaw.json                              # Config
    ~/.openclaw/workspace/                                 # Agent workspace
    ~/.openclaw/workspace/SOUL.md                          # Persona
    ~/.openclaw/workspace/AGENTS.md                        # Instructions
    ~/.openclaw/workspace/HEARTBEAT.md                     # Heartbeat tasks
    ~/.openclaw/agents/main/sessions/sessions.json         # Session metadata
    ~/.openclaw/scripts/start_openclaw_with_prompt.sh      # Autostart script
    /tmp/openclaw/openclaw-YYYY-MM-DD.log                  # Daily logs

### Key File Locations (Windows)

    C:\Users\<USER>\start_openclaw_prompt.bat              # Autostart batch
    Task Scheduler > "OpenClaw Autostart"                  # Scheduled task

### Environment Variables

    OPENROUTER_API_KEY=sk-or-v1-...       # OpenRouter
    ANTHROPIC_API_KEY=sk-ant-...          # Anthropic
    OPENAI_API_KEY=sk-...                 # OpenAI
    GOOGLE_API_KEY=...                    # Google

---

> 📝 **Tip:** Treat `~/.openclaw/workspace/` as a git repo for version-controlled agent memory. Run `cd ~/.openclaw/workspace && git init && git add -A && git commit -m "initial"` to start tracking changes.

> 🔒 **Security Reminder:** Add `.env` and any files containing API keys to `.gitignore`. Never push secrets to public repositories.
