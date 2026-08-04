# Squad Satellite Mode Guide

**Version:** Squad v0.11.0 · **Role:** Linus (Lab Engineer)

[← Back to the hands-on guide](./hands-on-guide.md)

---

## Satellite Mode vs. State Backend — Key Distinction

These are **independent dials** and the most common source of misconfiguration:

| Concept | Config key | Controls |
|---|---|---|
| Satellite mode | `teamRoot` in `.squad/config.json` | **Where** the team root lives (hub repository root) |
| State backend | `stateBackend` in `.squad/config.json` | **How** mutable runtime state is stored and synced |
| External state | `stateLocation: "external"` + `projectKey` | Resolves state under platform appdata; independent of satellite mode and `stateBackend` |

A satellite repo can use any state backend. A non-satellite repo can also use any backend.

---

## When to Use Satellite Mode

Use satellite mode when a repo needs to **read its team definition from elsewhere**:

- A service repo that inherits charters, routing, and decisions from a shared platform repo.
- A workshop or demo repo pointing at a long-lived, managed team root.

**Not** for peer Squad communication across organizations — that uses the `cross-squad` and `cross-squad-communication` skills (see `.squad/skills/`).

---

## Prerequisites and Safety

- `teamRoot` must be an **absolute path**. `.squad/config.json` is **not portable** if committed — see commit/ignore guidance below.
- The hub's `.squad/team.md` must exist with at least one `## Members` entry; otherwise the coordinator enters Init Mode on the satellite.
- Satellites own only pointer config locally — charters, routing, and decisions come from the hub.
- Never commit secrets or personal path tokens into `.squad/config.json`.
- With a non-`local` backend, confirm the `squad_state` MCP bridge is reachable before any state write.

---

## Step-by-Step Setup

### 1 — Verify the Hub Exists

```bash
# macOS/Linux
ls /path/to/team-hub/.squad/team.md
# Windows PowerShell
Get-Item C:\path\to\team-hub\.squad\team.md
```

> 🟢 **Expected:** File exists with a `## Members` section and at least one entry.

### 2 — Initialize the Satellite Repo

```bash
mkdir satellite-repo && cd satellite-repo
git init
npx @bradygaster/squad-cli@latest init
```

### 3 — Set `teamRoot` in `.squad/config.json`

```json
{
  "version": 1,
  "teamRoot": "/absolute/path/to/team-hub"
}
```

Windows:
```json
{ "version": 1, "teamRoot": "C:\\Users\\yourname\\projects\\team-hub" }
```

> ⚠️ **Portability:** Set `teamRoot` to the **hub repository root** — the coordinator reads `{teamRoot}/.squad/team.md`. A direct absolute path to the hub's `.squad/` directory (e.g. `…/team-hub/.squad`) is also supported via fallback resolution (`{teamRoot}/team.md`); it is not an error. Because this is an absolute path, gitignore the real file and commit a `.squad/config.json.example` with a safe placeholder instead.

### 4 — Verify Resolution

Start a Copilot CLI session in the satellite repo.

> 🟢 **Expected:** Coordinator greets with the **hub team's roster**. Init Mode means `teamRoot` is wrong or hub `team.md` is missing.

```bash
squad status          # reports active squad and backend
cat .squad/config.json
```

### 5 — Set State Backend (Optional)

```json
{
  "version": 1,
  "teamRoot": "/absolute/path/to/team-hub",
  "stateBackend": "local"
}
```

| `stateBackend` | Effect |
|---|---|
| `"local"` (default) | Mutable state in local working tree |
| `"orphan"` | Separate orphan branch; sync with `squad sync` |
| `"two-layer"` | Main branch for static config; remote state branch for mutable state |

With `"orphan"` or `"two-layer"`, mutable state (decisions, history) lives on a branch in the **satellite repo** — not the hub. Each satellite manages its own mutable state while sharing the hub's team definition.

---

## What the Satellite Owns vs. the Hub

| Artifact | Owner |
|---|---|
| `.squad/config.json` (pointer), `squad.agent.md`, `.mcp.json`, `.gitattributes` | Satellite |
| `team.md`, `routing.md`, `ceremonies.md`, per-agent `charter.md` | Hub |
| `decisions.md`, agent `history.md` | Hub (local backend) or satellite state branch (orphan/two-layer) |

---

## Commit and Ignore Guidance

```
# .gitignore — satellite repos with non-portable config
.squad/config.json
```

- **Commit:** `squad.agent.md`, `.mcp.json`, `.gitattributes`, `.squad/config.json.example` (safe placeholder)
- **Do not commit:** The real `config.json` if it contains an absolute path, real usernames, API keys, or secrets.

Document the placeholder in your README:
```
# Set locally before using Squad — not committed
teamRoot: "<TEAM_HUB_PATH>/team-hub"
```

---

## Common Mistakes and Recovery

| Mistake | Symptom | Fix |
|---|---|---|
| `teamRoot` is a relative path | Init Mode | Use an absolute path |
| `teamRoot` points to `.squad/` subfolder | Works via fallback; canonical form is repo root | Prefer repo root — coordinator appends `/.squad/team.md`; `.squad/`-direct path resolves via `{teamRoot}/team.md` |
| Committed absolute path, different machine | Init Mode for colleague | Gitignore real file; use placeholder in `.example` |
| Non-local backend with missing MCP bridge | State write fails / silent data loss | Confirm `squad_state` in `.mcp.json`; fall back to `"local"` |

**Recovery checklist** (Init Mode unexpectedly): (a) `teamRoot` is absolute, (b) `{teamRoot}/.squad/team.md` exists with roster entries, (c) `config.json` is valid JSON.

---

*Guide authored by Linus (Lab Engineer) · Squad v0.11.0*
