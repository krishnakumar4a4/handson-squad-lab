# Squad Alternative Modes Guide

**Version:** Squad v0.11.0 · **Audience:** Users who need something other than a default fresh local init

[← Back to hands-on guide](./hands-on-guide.md)

---

This guide is the entry point for **non-default** Squad configurations. If you are doing a fresh init on Windows with the default local state backend, stay in `hands-on-guide.md` — you do not need this guide.

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

> **Expected:** Lists files overwritten, then confirms completion.

**What upgrade preserves vs. overwrites:**

| Preserved | Overwritten |
|---|---|
| `.squad/` directory (team state, charters, routing, decisions) | `squad.agent.md` |
| `.ai-team/` directory if present | `.squad/templates/` directory |

Your roster, charters, and decisions are safe. After upgrading, run `squad status` to confirm.

> **Safety gate:** Run `git status` before upgrading. Stash any uncommitted changes (`git stash`) to avoid merge conflicts with Squad-owned files.

---

## 2 · Orphan Backend

Use when you want mutable state (decisions, history) fully isolated from your main branch — state files never appear in `git log` of main.

```powershell
npx @bradygaster/squad-cli@latest init --state-backend orphan
```

> **Expected:** Init completes. Inspect `.squad\config.json` or run `squad doctor` to confirm `stateBackend: "orphan"`.

To sync state with remote:

```powershell
squad sync --push
```

> **Expected:** `Syncing squad-state branch...` then confirmation.

> ⚠️ **Safety:** Do not manually edit state files (`.squad\decisions.md`, agent history, audit trails) under non-local backends. These are managed by the runtime via the `squad_state` MCP bridge. Direct edits cause state branch divergence.

**→ Full details, prerequisites, MCP bridge setup, sync workflow, file ownership, and common mistakes: [Orphan State Backend Guide](./orphan-state-guide.md)**

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

External state (`stateLocation: "external"`) resolves mutable state under platform appdata rather than inside the repository. It is independent of satellite mode and `stateBackend`.

Set in `.squad\config.json`:

```json
{
  "version": 1,
  "stateLocation": "external",
  "projectKey": "my-project"
}
```

> Use `projectKey` to namespace state when multiple projects share the same appdata location. Confirm the `squad_state` MCP bridge is reachable before the first state write — Squad halts (does not silently fall back) if the bridge is unavailable.

---

## Key Distinctions

- **Satellite mode** and **state backend** are independent dials — a satellite repo can use any backend.
- **Two-layer** = git notes + orphan branch. **Orphan** = orphan branch only. They are not the same.
- **External state** is a third independent concern, separate from both satellite mode and `stateBackend`.
- For any non-local backend, runtime-owned state files must **not** be manually edited.

---

*[← Back to hands-on guide](./hands-on-guide.md) · [Orphan State Guide](./orphan-state-guide.md) · [Two-Layer State Guide](./two-layer-state-guide.md) · [Satellite Mode Guide](./satellite-mode-guide.md)*
