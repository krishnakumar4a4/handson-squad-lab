# Orphan State Backend — Setup Guide

**Version:** Squad v0.11.0

[← Alternative Modes Guide](../../alternative-modes-guide.md) · [→ Supplementary (concepts, commit guidance, troubleshooting)](./supplementary.md)

---

## Prerequisites

```powershell
node --version      # >= 22.5.0
npm --version       # >= 10
git --version
copilot --version   # GitHub Copilot CLI (the standalone copilot command) — pre-installed and authenticated
```

Confirm `.mcp.json` at repo root contains a `squad_state` entry. The bridge must be reachable before the first state write.

---

## Safety Gate

> ⚠️ Run `git status` before init. Confirm you have no uncommitted changes to protect. Squad creates files and may prompt to overwrite Squad-owned files on an existing installation.

---

## Setup Steps

### 1 — Create and Initialize

```powershell
mkdir my-project; cd my-project
git init
npx @bradygaster/squad-cli@latest init --state-backend orphan
```

> **Expected:** `.squad\` created; `.squad\config.json` contains `"stateBackend": "orphan"`; `.mcp.json` updated; orphan state branch created.

### 2 — Inspect the Generated Configuration

```powershell
code .
```

In Visual Studio Code, open:

- `.squad\config.json` — confirm it contains `"stateBackend": "orphan"`.
- `.mcp.json` — confirm it contains a `squad_state` entry.

### 3 — Check MCP Bridge

```powershell
squad doctor
```

> **Expected:** Reports `stateBackend: orphan`, MCP bridge reachable, no errors.
>
> **v0.11.0 known limitation:** `squad doctor` may report false failures (e.g., `decisions.md` not found) for non-local backends — this is expected because decisions live on the state branch. Use the configuration files in Visual Studio Code and `squad status` as primary verification.

### 4 — Confirm State Branch

```powershell
git branch -a
```

> **Expected:** A `squad-state` branch appears once state has been written through a live session. On a fresh init before any agent session, the branch may not exist yet — this is normal.

```powershell
squad status
```

> **Expected:** Shows the Squad version and configuration. Treat the files inspected in Visual Studio Code as the source of truth if v0.11.0 displays the wrong backend.

---

**→ [Supplementary Guide](./supplementary.md)** — key distinctions, MCP bridge details, commit/ignore rules, static vs runtime-owned files, recovery.
