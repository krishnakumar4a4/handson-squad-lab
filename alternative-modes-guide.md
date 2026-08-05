# Squad Alternative Modes Guide

**Version:** Squad v0.11.0 · **Audience:** Users who need something other than a default fresh local init

[← Back to hands-on guide](./hands-on-guide.md)

---

This guide is the entry point for **non-default** Squad configurations. If you are doing a fresh init on Windows with the default local state backend, stay in `hands-on-guide.md` — you do not need this guide.

---

## Key Distinctions

- **Satellite mode** and **state backend** are independent dials — a satellite repo can use any backend.
- **Two-layer** = git notes + orphan branch. **Orphan** = orphan branch only. They are not the same.
- **External state** is a third independent concern, separate from both satellite mode and `stateBackend`.
- For any non-local backend, runtime-owned state files must **not** be manually edited.
- Non-local backends route state through the `squad_state` MCP bridge — a background communication layer. It must be reachable before the first state write. See specific guides for bridge setup.

---

## Modes at a Glance

| Mode | Team definition | Mutable state | Commit to the working branch | Do not commit to the working branch |
|---|---|---|---|---|
| **Default init (`local`)** | Current repo: `.squad\team.md`, routing, charters | Current repo under `.squad\` | Squad configuration, team files, charters, decisions/history, `.github\`, `.copilot\mcp-config.json`, `.mcp.json` | Generated logs, caches, scratch files, and secrets listed in `.gitignore` |
| **Orphan backend** | Current repo | Separate orphan branch | Static Squad configuration, routing, ceremonies, charters, `.github\`, `.mcp.json` | Runtime-owned decisions, histories, audit trails, casting state, and memory |
| **Two-layer backend** | Current repo | Git notes for commit-scoped annotations plus an orphan branch for permanent state | Same static configuration as orphan mode | Same runtime-owned mutable state as orphan mode |
| **Satellite mode** | Shared hub repo selected by `teamRoot` | Determined separately by the satellite's state backend | Satellite pointer/config and local integration files; hub repo commits the shared team definition | Do not duplicate the hub roster, routing, or charters in the satellite repo |

> **Repository policy wins:** Review `git status` and the mode-specific guide before committing. Never store credentials or local secret files in Squad state.

---

## Choosing a Path

| # | I need to… | Go to |
|---|---|---|
| 1 | Upgrade an existing Squad installation | [1 · Upgrade an Existing Installation](#1-upgrade-an-existing-installation) below |
| 2 | Isolate mutable state on a separate branch (`orphan`) | [2 · Orphan Backend](#2-orphan-backend) below |
| 3 | Split config on main / runtime state on a remote branch (`two-layer`) | [3 · Two-Layer State Backend](./two-layer-state-guide.md) |
| 4 | Inherit a team definition from a shared hub repo (satellite) | [4 · Satellite Mode](./satellite-mode-guide.md) |
| 5 | Store state under platform appdata (`external`) | [5 · External State](#5-external-state) below |

---

## 1 · Upgrade an Existing Installation

If Squad is already installed and you want to update Squad-owned files:

```powershell
squad upgrade
```

> **Expected:** Lists files overwritten, then confirms completion. Syncs skills to `.github/skills/`.
>
> If `squad.agent.md` has local customizations, upgrade backs it up to `squad.agent.md.local-backup` before overwriting — your customizations are not lost.

**What upgrade preserves vs. overwrites:**

| Preserved | Overwritten |
|---|---|
| `.squad/` directory (team state, charters, routing, decisions) | `squad.agent.md` |
| `.ai-team/` directory if present | `.squad/templates/` directory |
| | `.github/skills/` (synced/overwritten with 19+ skills) |

Your roster, charters, and decisions are safe. After upgrading, run `squad status` to confirm.

> **Safety gate:** Run `git status` before upgrading. Stash any uncommitted changes (`git stash`) to avoid merge conflicts with Squad-owned files.

---

## 2 · Orphan Backend

Orphan mode stores permanent mutable state on a separate orphan branch, keeping decisions and agent history out of the main branch.

**This mode has its own detailed guide:**

**→ [Orphan State Backend Guide](./orphan-state-guide.md)** — prerequisites, MCP bridge setup, sync workflow, safe-edit rules, and recovery.

---

## 3 · Two-Layer State Backend

Two-layer combines git notes (commit-scoped) with an orphan branch (permanent mutable state). Best for shared team repos where collaborators synchronize state via `squad sync`.

**This mode has its own detailed guide:**

**→ [Two-Layer State Backend Guide](./two-layer-state-guide.md)** — prerequisites, MCP bridge setup, sync workflow, safe-edit rules, and recovery.

---

## 4 · Satellite Mode

Satellite mode lets a repository inherit its team definition (roster, routing, charters) from a hub repository. The satellite owns only a pointer config locally.

**This mode has its own detailed guide:**

**→ [Satellite Mode Guide](./satellite-mode-guide.md)** — prerequisites, step-by-step setup, portability notes, hub vs. satellite ownership, and common mistakes.

---

## 5 · External State

External state (`stateLocation: "external"`) resolves mutable state under platform appdata rather than inside the repository. It is **independent** of satellite mode and `stateBackend`.

**When to use:** Choose external state when you want Squad state outside the repository — for example, on a shared team machine where multiple projects share Squad state, or in CI environments where you cannot commit state to the repo.

Set in `.squad\config.json`:

```json
{
  "version": 1,
  "stateLocation": "external",
  "projectKey": "my-project"
}
```

Use `projectKey` to namespace state when multiple projects share the same appdata location.

**Requirements:** The `squad_state` MCP bridge must be reachable before the first state write. Squad halts (does not silently fall back) if the bridge is unavailable. Confirm with:

```powershell
squad doctor
```

> **Runtime verification note:** Verifying that external state is actually resolving correctly requires an active, authenticated Copilot CLI session. Static config (the JSON above) can be verified by inspection; runtime state read/write can only be confirmed through a live agent session. Do not rely solely on `squad status` output to confirm external state is working.

---

*[← Back to hands-on guide](./hands-on-guide.md) · [Orphan State Guide](./orphan-state-guide.md) · [Two-Layer State Guide](./two-layer-state-guide.md) · [Satellite Mode Guide](./satellite-mode-guide.md)*
