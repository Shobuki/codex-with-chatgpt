# Codex with ChatGPT

> ChatGPT thinks. Codex works.

## What this repository does

This repository provides a CLI and bridge called `c2c` for connecting ChatGPT
web to a local Codex workspace.

- ChatGPT reads the repository, creates plans, and reviews changes.
- Codex edits files, runs shell commands, git operations, and tests.
- ChatGPT receives read-only access through MCP.
- The entire repository is not uploaded and no API key is required.

## HOW TO RUN THIS REPOSITORY

### Windows / PowerShell

Run the commands below from the repository folder:

```powershell
cd D:\kodingan\codex-with-chatgpt
```

### 1. Install, build, and test

```powershell
corepack pnpm install
corepack pnpm build
corepack pnpm test
```

If the output shows `154 passed`, all tests passed.

Requirements:

- Node.js 20 or newer
- Git
- `cloudflared` for the public connection

On Windows, install or update `cloudflared` with:

```powershell
winget upgrade --id Cloudflare.cloudflared -e
```

### 2. Start the bridge and public connection

```powershell
node .\bin\c2c.js setup `
  -w "D:\kodingan\codex-with-chatgpt" `
  --json
```

The command starts the bridge, creates a public connection, and returns:

- `mcpUrl`: enter this as the **Server URL** in the ChatGPT Connector.
- `pairingCode`: enter this when ChatGPT asks for authorization.

Do not type the MCP URL or pairing code directly into PowerShell.

### 3. Connect to ChatGPT web

In ChatGPT web:

1. Open the Connector/MCP settings.
2. Create a new connector.
3. Enter the `mcpUrl` value as the **Server URL**.
4. Click **Connect** or **Authorize**.
5. Enter the `pairingCode` value.

After connecting, use a prompt such as:

```text
Use the "Codex with ChatGPT · codex-with-chatgpt" connector.
Read this repository and explain its purpose, structure, and how to run it.
Do not modify any files.
```

### 4. Check the connection status

```powershell
node .\bin\c2c.js status -w "D:\kodingan\codex-with-chatgpt" --json
node .\bin\c2c.js doctor -w "D:\kodingan\codex-with-chatgpt" --json
```

The connection is ready when the status contains:

```text
"tunnel": { "running": true }
```

### 5. Run it again

If the terminal or computer was closed, run setup again:

```powershell
node .\bin\c2c.js setup `
  -w "D:\kodingan\codex-with-chatgpt" `
  --json
```

Quick Tunnel may generate a new URL and pairing code after every restart.
If the pairing code expires, generate a new one:

```powershell
node .\bin\c2c.js pair -w "D:\kodingan\codex-with-chatgpt" --json
```

### 6. Stop the bridge

```powershell
node .\bin\c2c.js stop -w "D:\kodingan\codex-with-chatgpt"
```

### Optional stable hostname

If you have a Cloudflare account and a domain already on Cloudflare, first-time
setup (and the next coding session, once) will ask whether you want a stable
hostname such as `c2c-<project>.your-domain.com`. That path opens a browser so
you can authorize Cloudflare. After that, the ChatGPT connector keeps working
across restarts. If you skip it, or the login fails, Codex stays on the temporary
address — same features, just a slower repair.

### Local development mode

This mode only tests the local bridge. ChatGPT web cannot access it.

```powershell
node .\bin\c2c.js setup `
  -w "D:\kodingan\codex-with-chatgpt" `
  --no-tunnel `
  --json
```

## How to use it to modify code

The MCP server in this repository is intentionally read-only. ChatGPT web
cannot directly write files.

Use this workflow:

1. Ask ChatGPT web to read the repository and create a plan.
2. Run Codex on the same workspace.
3. Ask Codex to implement the plan and run the tests.
4. Ask ChatGPT web to review the changes.

Example prompt for Codex:

```text
Use Codex with ChatGPT to add a login feature to this repository.
Create a plan first, implement it, and then run the tests.
```

## Install the Codex skill

The skill helps Codex run the planning, execution, and review workflow
automatically.

```powershell
$skillDir = "$env:USERPROFILE\.codex\skills\codex-with-chatgpt"

New-Item -ItemType Directory -Force $skillDir | Out-Null
Copy-Item ".\skill\SKILL.md" "$skillDir\SKILL.md" -Force
```

Open the copied file:

```powershell
notepad "$skillDir\SKILL.md"
```

Replace `<ACTUAL_CHECKOUT_PATH>` with:

```text
D:\kodingan\codex-with-chatgpt
```

Then open a new Codex session so the skill is loaded.

## How it works

```text
ChatGPT web
  | reads, plans, and reviews
  v
C2C Bridge
  | read-only MCP connection and authorization
  v
Local workspace
  ^
  | file edits, shell, git, and tests
Codex
```

ChatGPT and Codex exchange short control messages. File contents, diffs, and
command output are read through MCP tools when needed.

## Security

- MCP does not provide write, delete, shell, commit, or execution tools.
- Each bridge is bound to one workspace.
- Paths outside the workspace are blocked.
- Sensitive files such as `.env`, keys, SSH files, and credentials are blocked
  by default.
- Each connection uses OAuth and a one-time pairing code.
- Pairing codes expire after approximately five minutes and are destroyed after
  use.

See [docs/security.md](docs/security.md) for the full security model.

## Developer commands

```powershell
corepack pnpm install
corepack pnpm build
corepack pnpm typecheck
corepack pnpm test

node .\bin\c2c.js --help
node .\bin\c2c.js status -w "D:\kodingan\codex-with-chatgpt"
node .\bin\c2c.js doctor -w "D:\kodingan\codex-with-chatgpt"
```

To run without building:

```powershell
corepack pnpm dev -- --help
corepack pnpm dev -- status -w .
```

## Project structure

```text
src/
  bridge/      local HTTP server, ports, and admin API
  mcp/         read-only MCP tools
  auth/        OAuth and authentication middleware
  pairing/     one-time pairing codes
  workspace/   path containment, search, and git
  tunnel/      Cloudflare Quick Tunnel and Named Tunnel
  execution/   execution records for review
  process/     daemon lifecycle
  cli/         c2c command-line interface
skill/         Codex skill
tests/         unit and integration tests
docs/          architecture, protocol, and security documentation
```

## Troubleshooting

If the connection fails, run:

```powershell
node .\bin\c2c.js doctor -w "D:\kodingan\codex-with-chatgpt" --json
node .\bin\c2c.js logs --verbose
```

If the tunnel stops after a restart, run `setup` again. Quick Tunnel addresses
are temporary and may change.

If the pairing code is rejected or expired, generate a new code with `pair`.

## Status

Version 1 has been verified for the bridge, OAuth, pairing, public connection,
ChatGPT connector, and first-time setup workflow.

This is an unofficial community project. It is not affiliated with or endorsed
by OpenAI.

## License

[MIT](LICENSE)
