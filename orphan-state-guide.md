# Orphan State Backend Guide

**Version:** Squad v0.11.0 · **Role:** Linus (Lab Engineer)

[← Back to Alternative Modes Guide](./alternative-modes-guide.md) · [← Back to Hands-On Guide](./hands-on-guide.md)

---

## Key Distinctions

| Concept | Key | What it controls |
|---|---|---|
| **Orphan backend** | `stateBackend: "orphan"` | Mutable state on a separate Git orphan branch — never touches main |
| Two-layer backend | `stateBackend: "two-layer"` | Orphan branch *plus* git notes for commit-scoped annotations |
| Satellite mode | `teamRoot` | Inherits team definition from a hub repo — independent of backend |
| External state | `stateLocation: "external"` | Resolves state under platform appdata — independent of backend |

`orphan` and `two-layer` are not interchangeable. Two-layer = orphan branch + git notes. Satellite mode and external state are separate mechanisms that work with any backend.

---

## When to Use Orphan Mode

Choose based on your needs:

| If you want… | Use |
|---|---|
| Simplest setup; state visible in working tree | **`local`** (default) — [hands-on-guide.md](./hands-on-guide.md) |
| State fully isolated from main; CI-safe; state never in `git log` of main | **`orphan`** (this guide) |
| Everything orphan provides *plus* commit-scoped git notes annotations | **`two-layer`** — [two-layer-state-guide.md](./two-layer-state-guide.md) |

**Use `orphan` when:**
- Contributors should not see raw agent state in the main branch history.
- You need to share mutable state across machines via explicit `squad sync`.
- CI pipelines should not commit state to main.

**Do not use `orphan` if** you just want the simplest setup — use `local` instead.

---

## Windows Prerequisites

```powershell
node --version      # >= 22.5.0
npm --version       # >= 10
git --version
copilot --version   # GitHub Copilot CLI (the standalone copilot command) — must be pre-installed and authenticated
```

Confirm `.mcp.json` at repo root contains a `squad_state` entry (see [MCP Bridge](#the-squad_state-mcp-bridge)). The bridge must be reachable before the first state write.

---

## Safety Gate

> ⚠️ Run `git status` before init. Confirm you have no uncommitted changes you need to keep. Squad will create files and may prompt to overwrite Squad-owned files on an existing installation.

---

## New Repository — Initialization

```powershell
mkdir my-project; cd my-project
git init
npx @bradygaster/squad-cli@latest init --state-backend orphan
```

**Expected outcome:** `.squad\` created; `.squad\config.json` contains `"stateBackend": "orphan"`; `.mcp.json` updated; orphan state branch created.

**Verify initialization:**

```powershell
Get-Content .squad\config.json
# Expected: { "version": 1, "stateBackend": "orphan" }

squad doctor
# Expected: reports stateBackend: orphan, MCP bridge reachable, no errors
```

---

## Existing Repositories — Migration

> ⚠️ **Migration is not covered by this guide.** No documented, safe in-place migration command has been confirmed. If you need to switch backends on a repository with existing state:
>
> 1. **Back up your `.squad\` directory first.**
> 2. Consult official Squad CLI documentation: `squad --help` and `squad init --help`.
> 3. Do **not** manually edit `stateBackend` in `config.json` without using official tooling — this may cause state divergence or loss.

---

## The `squad_state` MCP Bridge

All non-local backends route state reads and writes through the `squad_state` MCP server. The bridge must be declared in `.mcp.json` at the repo root:

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

**Handshake and halt behavior:** Before the first state write, Squad verifies the bridge is reachable. If it is unavailable, Squad **halts** the operation and reports an error — it does not silently fall back to local files or skip the write. Always confirm the bridge is up before starting a session:

```powershell
squad doctor
# or
squad status
```

---

## Static Config vs. Runtime-Owned State

### Safe to edit directly (static config — committed on main)

`.squad\config.json` · `.squad\team.md` · `.squad\routing.md` · `.squad\ceremonies.md` · `.squad\agents\{name}\charter.md` · `.squad\casting\policy.json` · `.github\agents\squad.agent.md` · `.mcp.json`

### Do NOT edit directly (runtime-owned mutable state)

Managed exclusively by the runtime via the `squad_state` bridge. On the orphan backend these live on the state branch — direct edits cause divergence.

`.squad\decisions.md` · `.squad\agents\{name}\history.md` · `.squad\rai\audit-trail.md` · `.squad\fact-checker\audit-trail.md` · `.squad\casting\history.json`

---

## Collaboration with `squad sync`

Collaborators share mutable state via explicit sync — there are no automatic pushes.

- Before starting a session: `squad sync --pull`
- After finishing a session: `squad sync --push`
- Next collaborator: `squad sync --pull` before they begin

```powershell
squad sync --pull   # pull mutable state from the remote state branch
squad sync --push   # push local mutable state to the remote state branch
```

> **Expected:** Both commands may exit silently with no output. This is normal — `squad sync` is a no-op until an active Copilot CLI session has written state through the MCP bridge. Do not expect `"State branch updated"` confirmation. Verify sync results by checking git refs:
> ```powershell
> git branch -a
> ```
> A `squad-state` branch will appear once state has been written through a live session.

> **Flags:** Run `squad help` for the full command reference. (`squad sync --help` does not provide detailed flag help in v0.11.0 — use `squad help` instead.)

---

## Commit and Ignore Guidance

**Commit** `.squad\config.json` — it declares the backend and is team configuration.

**Do not commit** mutable state files from the orphan branch into main. Squad's default `.gitignore` entries:

```gitignore
.squad/orchestration-log/
.squad/log/
.squad/decisions/inbox/
.squad/sessions/
.squad/.scratch/
.squad/.cache/
```

**Casting files (`.squad\casting\policy.json`, `.squad\casting\registry.json`):** Under the orphan backend, Squad's pre-commit hook treats these as runtime-owned state and **blocks them from being committed to main**. Do not include them in a `git add .` when the hook is active.

> **v0.11.0 hook behavior note:** The pre-commit hook message may say "refusing to commit two-layer state into the working tree" even when `stateBackend` is `"orphan"`. This wording is a known CLI issue — the protection behavior is correct regardless of the message text.

---

## Verification

```powershell
Get-Content .squad\config.json    # confirm stateBackend: orphan
squad status                      # confirm backend and version
squad doctor                      # check MCP bridge reachable
```

> **v0.11.0 known limitation:** `squad doctor` may report false failures (e.g., `decisions.md` not found) for non-local backends — this is expected because decisions live on the state branch, not the working tree. Use `Get-Content .squad\config.json` and `squad status` as primary verification. Do not treat doctor warnings as errors unless state writes are actually failing.

---

## Common Mistakes and Recovery

| Mistake | Symptom | Fix |
|---|---|---|
| MCP bridge missing from `.mcp.json` | State write halts | Add `squad_state` entry; restart session |
| No remote configured | `squad sync` fails | Add `origin` remote: `git remote add origin <url>` |
| Editing state files in working tree | State branch diverges | Restore from state branch; avoid direct edits |
| Switching backends without backup | State lost | Always back up `.squad\` before backend changes |

**Recovery:** If state is missing after a sync failure, run `squad sync --pull` to restore from the remote state branch.

---

*Guide authored by Linus (Lab Engineer) · Squad v0.11.0*
