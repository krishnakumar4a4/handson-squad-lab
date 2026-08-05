# Two-Layer State Backend — Supplementary Guide

**Version:** Squad v0.11.0

[← Setup Guide (guide.md)](./guide.md) · [← Alternative Modes Guide](../../alternative-modes-guide.md)

---

## When to Use Two-Layer Mode

Two-layer combines git notes (commit-scoped annotations) with an orphan branch (permanent mutable state). Best for shared team repos where collaborators synchronize state via `squad sync`.

**Use `two-layer` when:**
- You want static config (team.md, routing, charters) on main as regular repository content.
- You want mutable state (decisions, history) in two complementary layers: git notes for commit-scoped annotations and an orphan branch for permanent state.
- Multiple collaborators need to sync state via `squad sync`.

**Use `orphan` instead** if you just want state isolated from main without the git notes layer: [modes\orphan\guide.md](../orphan/guide.md).

**Use `local`** (default) for the simplest setup: [Readme.md](../../Readme.md).

> For the full backends comparison, see the [Backends at a Glance](#backends-at-a-glance) section below.

---

## Three Independent Dials

Understand what these concepts control — they are **not** aliases:

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

**`orphan` vs `two-layer`:** Use `orphan` if you want mutable state fully isolated from main. Use `two-layer` if you also need commit-scoped git notes annotations alongside the orphan branch.

---

## The `squad_state` MCP Bridge

All non-local backends route state reads and writes through the `squad_state` MCP server. Declared in `.mcp.json` at the repo root:

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

**Halt behavior:** If the bridge is unreachable when a state write is attempted, Squad **halts** the operation and reports an error. It does not silently skip the write or fall back to a local file. Always confirm the bridge is up before starting a session on a non-local backend.

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

Managed by the runtime state tools via the MCP bridge. Editing these directly under `two-layer` or `orphan` backends causes state branch divergence:

- `.squad/decisions.md`
- `.squad/agents/{name}/history.md`
- `.squad/rai/audit-trail.md`
- `.squad/fact-checker/audit-trail.md`
- `.squad/casting/history.json`

---

## `squad sync` Collaboration Workflow

```powershell
squad sync --push   # push local mutable state to remote
squad sync --pull   # pull mutable state from remote
```

1. Before session: `squad sync --pull`
2. After session: `squad sync --push`
3. Next collaborator: `squad sync --pull` before their session
4. No manual merging needed — the bridge owns conflict resolution.

> **Expected:** Commands may exit silently with no output. This is normal — sync is a no-op until an active Copilot CLI session has written state through the MCP bridge. Do not expect `"State branch updated"` confirmation. Verify by checking git refs:
>
> ```powershell
> git branch -a
> ```
>
> A `squad-state` branch will appear once state has been written through a live session.

> Run `squad help` for the full command reference. (`squad sync --help` does not provide detailed help in v0.11.0.)

---

## Commit and Ignore Guidance

**Commit** `.squad/config.json` — it contains the `stateBackend` setting, which is team configuration.

**Do not commit** mutable state files from the state branch into main:

```gitignore
.squad/orchestration-log/
.squad/log/
.squad/decisions/inbox/
.squad/sessions/
.squad/.scratch/
.squad/.cache/
```

**Casting files (`.squad\casting\policy.json`, `.squad\casting\registry.json`):** Under the two-layer backend, the pre-commit hook treats these as runtime-owned state and blocks them from being committed to main. Do not include them in `git add .` when the hook is active. The hook message "two-layer state" is correct for this backend.

---

## Existing Repositories — Migration

> ⚠️ **Migration is not covered by this guide.** No documented, safe in-place migration command has been confirmed. If you need to switch backends on a repository with existing state:
>
> 1. **Back up your `.squad\` directory first.**
> 2. Consult official Squad CLI documentation: `squad --help` and `squad init --help`.
> 3. Do **not** manually edit `stateBackend` in `config.json` without using official tooling — this may cause state divergence or loss.

---

## v0.11.0 Known Limitations

- **`squad status` display issue:** May show `Mode: local` even when `stateBackend` is `"two-layer"`. The actual backend is set in `config.json` — use `Get-Content .squad\config.json` to verify.
- **`squad doctor` false warnings:** May report `decisions.md` not found — expected because decisions live on the state branch, not the working tree. Not an error unless state writes are actually failing.
- **`squad sync --help`:** Does not provide detailed flag help in v0.11.0. Use `squad help` instead.

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

*[← Setup Guide](./guide.md) · [← Alternative Modes](../../alternative-modes-guide.md)*
