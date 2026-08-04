# Two-Layer State Backend Guide

**Version:** Squad v0.11.0 · **Role:** Linus (Lab Engineer)

[← Back to the hands-on guide](./hands-on-guide.md) · [← Satellite Mode Guide](./satellite-mode-guide.md)

---

## Three Independent Dials

Before diving in, understand what these concepts control — they are **not** aliases:

| Concept | Key in `config.json` | Controls |
|---|---|---|
| **State backend** | `stateBackend` | *How* mutable runtime state is stored and synced |
| Satellite mode | `teamRoot` | *Which* team definition is used (hub vs local) |
| External state | `stateLocation: "external"` + `projectKey` | Resolves state under platform appdata |

This guide covers **`stateBackend: "two-layer"`** only. Satellite mode and external state are separate concerns.

---

## Backends at a Glance

| Backend | Static config | Mutable state | `squad sync` needed? |
|---|---|---|---|
| `local` (default) | main branch | main branch (working tree) | No |
| `orphan` | main branch | Separate orphan branch | Yes |
| `two-layer` | main branch | Git notes (commit-scoped) + orphan branch (permanent mutable) | Yes — sync to share |

**When to use `orphan`:** You want mutable state (decisions, history) fully isolated from main — no state files ever touch main. Suitable for CI environments or repos where contributors must not see raw agent logs.

**When to use `two-layer`:** You want static config (team.md, routing, charters) on main as normal repository content, while mutable state lives in two complementary layers: **git notes** for commit-scoped annotations and an **orphan branch** for permanent mutable state. The orphan branch exists as repository state and can be synchronized with a configured remote through `squad sync`. Best fit for shared team repos where collaborators sync state via `squad sync`.

---

## Prerequisites and Safety Gate

- A supported, modern version of Git
- The `squad_state` MCP bridge configured in `.mcp.json` (see below)
- A remote (`origin`) reachable for push/pull
- No uncommitted mutable state you want to keep before changing backends

> ⚠️ **Safety gate:** Confirm the `squad_state` bridge is reachable *before* the first state write. If it is unavailable, Squad **halts** state writes — it does not silently fall back or corrupt data. Verify with `squad status` or `squad doctor`.

---

## Setup

### New Repository

```powershell
npx @bradygaster/squad-cli@latest init --state-backend two-layer
```

Verify the backend was set:

```powershell
Get-Content .squad\config.json
# Expected: { "version": 1, "stateBackend": "two-layer" }
squad status
```

### Changing an Existing Repository

> ⚠️ **Not verified by this lab.** No documented, safe in-place migration command has been confirmed. If you need to change backends on a repo with existing state, **back up your `.squad/` directory first**, consult the official Squad CLI documentation (`squad --help`), and avoid manually editing state files. Editing mutable state files directly under non-local backends may cause data loss or desync.

---

## The `squad_state` MCP Bridge

All non-local backends route state reads and writes through the `squad_state` MCP server. The bridge is declared in `.mcp.json` at the repo root:

```json
{
  "mcpServers": {
    "squad_state": {
      "command": "npx",
      "args": ["-y", "@bradygaster/squad-cli@insider", "state-mcp"],
      "env": {},
      "tools": ["*"]
    }
  }
}
```

**Halt behavior:** If the bridge is unreachable when a state write is attempted, Squad halts the operation and reports an error. It does not silently skip the write or fall back to a local file. Always confirm the bridge is up before starting a session on a non-local backend.

---

## What to Edit Directly vs. What Not To

### Safe to edit directly (static config — main branch)

- `.squad/team.md`
- `.squad/routing.md`
- `.squad/ceremonies.md`
- `.squad/agents/{name}/charter.md`
- `.squad/casting/policy.json`
- `.squad/config.json`
- `.github/agents/squad.agent.md`
- `.mcp.json`

### Do NOT edit directly under non-local backends

These are managed by the runtime state tools via the MCP bridge:

- `.squad/decisions.md`
- `.squad/agents/{name}/history.md`
- `.squad/rai/audit-trail.md`
- `.squad/fact-checker/audit-trail.md`
- `.squad/casting/history.json`

Editing these files directly while on `two-layer` or `orphan` backends can cause the state branch to diverge from your working tree.

---

## `squad sync` Usage

```powershell
squad sync --push   # push local mutable state to remote
squad sync --pull   # pull mutable state from remote
# squad sync supports push, pull, or both — see: squad sync --help
```

**Expected output:**

```
Syncing squad-state branch...
✓ State branch updated.
```

**Collaboration workflow:**

1. Collaborator A runs `squad sync --pull` before starting a session.
2. After session ends, Collaborator A runs `squad sync --push`.
3. Collaborator B pulls before their session.
4. No manual merging of state files is needed — the bridge owns conflict resolution.

---

## Commit and Ignore Guidance

```gitignore
# .gitignore — mutable state is on the state branch, not main
.squad/orchestration-log/
.squad/log/
.squad/decisions/inbox/
.squad/sessions/
.squad/.scratch/
.squad/.cache/
```

**Commit** `.squad/config.json` (it contains the `stateBackend` setting, which is team configuration).

---

## Verification Steps

```powershell
squad status          # confirm stateBackend: two-layer
squad doctor          # verify MCP bridge reachable and backend healthy
squad sync --push     # first push after init
```

---

## Common Mistakes and Recovery

| Mistake | Symptom | Fix |
|---|---|---|
| MCP bridge not in `.mcp.json` | State write halts | Add `squad_state` entry; restart session |
| No remote configured | `squad sync` fails | Add `origin` remote and push |
| Editing state files directly | State branch diverges | Restore from state branch; avoid direct edits |
| Switching backends mid-project without backup | State lost | Always backup `.squad/` before backend change |

**Recovery:** If state is missing after a sync failure, run `squad sync --pull` to restore from the remote state branch. If the remote branch is missing, restore from your `.squad/` backup.

---

*Guide authored by Linus (Lab Engineer) · Squad v0.11.0*
