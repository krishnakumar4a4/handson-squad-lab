# Two-Layer State Backend — Setup Guide

**Version:** Squad v0.11.0

[← Alternative Modes Guide](../../alternative-modes-guide.md) · [→ Supplementary (concepts, MCP bridge, commit guidance, troubleshooting)](./supplementary.md)

---

## Prerequisites

- Node.js ≥ 22.5.0 (`node --version`)
- npm ≥ 10 (`npm --version`)
- Git (a supported, modern version)
- GitHub Copilot CLI (standalone `copilot` command) — pre-installed and authenticated
- A remote (`origin`) reachable for push/pull
- The `squad_state` MCP bridge configured in `.mcp.json` (see [Supplementary](./supplementary.md))

---

## Safety Gate

> ⚠️ Confirm the `squad_state` bridge is reachable *before* the first state write. If it is unavailable, Squad **halts** state writes — it does not silently fall back or corrupt data. Verify with `squad status` or `squad doctor`.

> ⚠️ Run `git status` and confirm no uncommitted mutable state you want to keep before changing backends.

---

## Setup Steps

### 1 — Initialize (New Repository)

```powershell
mkdir my-project; cd my-project
git init
npx @bradygaster/squad-cli@latest init --state-backend two-layer
```

### 2 — Inspect the Generated Configuration

```powershell
code .
```

In Visual Studio Code, open:

- `.squad\config.json` — confirm it contains `"stateBackend": "two-layer"`.
- `.mcp.json` — confirm it contains a `squad_state` entry.

### 3 — Confirm Status

```powershell
squad status
```

> **Expected:** Shows backend (`two-layer`) and Squad version.
>
> **v0.11.0 known display issue:** `squad status` may show `Mode: local` — this is a display bug. The actual configured backend is `two-layer` as set in `config.json`.

### 4 — Check MCP Bridge

```powershell
squad doctor
```

> **Expected:** Reports MCP bridge reachable.
>
> **v0.11.0 known false warnings:** Doctor may report `decisions.md` not found — expected because decisions live on the state branch. Use the configuration files in Visual Studio Code as primary verification.

### 5 — Push Initial State

After a Copilot CLI session has run, push state to the remote:

```powershell
squad sync --push
```

## Sync Workflow (Summary)

```powershell
squad sync --pull   # before a session
squad sync --push   # after a session
```

> Commands may exit silently — this is normal. See [Supplementary](./supplementary.md) for full collaboration workflow and verification steps.

---

**→ [Supplementary Guide](./supplementary.md)** — three independent dials, MCP bridge details, what to edit vs. not, commit/ignore rules, recovery.
