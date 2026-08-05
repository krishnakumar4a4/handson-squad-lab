# Satellite Mode — Supplementary Guide

**Version:** Squad v0.11.0

[← Setup Guide (guide.md)](./guide.md) · [← Alternative Modes Guide](../../alternative-modes-guide.md)

---

## When to Use Satellite Mode

Use satellite mode when a repository needs to **read its team definition from a shared hub repository** rather than defining its own team locally.

**Use satellite mode when:**
- A service repo inherits charters, routing, and decisions from a shared platform repo.
- A workshop or demo repo points at a long-lived, managed team root.

**Not** for peer Squad communication across organizations — that uses the `cross-squad` and `cross-squad-communication` skills (see `.squad/skills/`).

> Satellite mode and state backend are **independent** — a satellite repo can use any backend. See the [Key Distinctions](#key-distinctions) section below for the full distinction table.

---

## Key Distinctions

Satellite mode and state backend are **independent dials** — the most common source of misconfiguration:

| Concept | Config key | Controls |
|---|---|---|
| Satellite mode | `teamRoot` in `.squad/config.json` | **Where** the team root lives (hub repository root) |
| State backend | `stateBackend` in `.squad/config.json` | **How** mutable runtime state is stored and synced |
| External state | `stateLocation: "external"` + `projectKey` | Resolves state under platform appdata; independent of satellite mode and `stateBackend` |

A satellite repo can use any state backend. A non-satellite repo can also use any backend.

---

## Architecture: Hub vs. Satellite

The coordinator reads `{teamRoot}/.squad/team.md` to load the team definition. The satellite repo owns only a pointer config locally.

### What the Satellite Owns vs. the Hub

| Artifact | Owner |
|---|---|
| `.squad/config.json` (pointer), `squad.agent.md`, `.mcp.json`, `.gitattributes` | Satellite |
| `team.md`, `routing.md`, `ceremonies.md`, per-agent `charter.md` | Hub |
| `decisions.md`, agent `history.md` | Hub (local backend) or satellite state branch (orphan/two-layer) |

With `"orphan"` or `"two-layer"` state backend on the satellite, mutable state (decisions, history) lives on a branch in the **satellite repo** — not the hub. Each satellite manages its own mutable state while sharing the hub's team definition.

---

## Commit and Ignore Guidance

```gitignore
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

## JSON Path Escaping Reference

In JSON, each Windows backslash must be written as `\\`:

| Windows path | JSON value |
|---|---|
| `C:\SquadTeams\team-hub` | `"C:\\SquadTeams\\team-hub"` |

**PowerShell `ConvertTo-Json` warning:** May produce `C:\\\\SquadTeams\\\\team-hub` (four backslashes — incorrect). Always verify output, or write the JSON directly:

```powershell
Set-Content .squad\config.json -Value '{"version":1,"teamRoot":"C:\\SquadTeams\\team-hub"}'
```

---

## Common Mistakes and Recovery

| Mistake | Symptom | Fix |
|---|---|---|
| `teamRoot` is a relative path | Init Mode on satellite | Use an absolute path |
| `teamRoot` points to `.squad/` subfolder | Works via fallback; canonical form is repo root | Prefer repo root — coordinator appends `/.squad/team.md`; `.squad/`-direct path resolves via `{teamRoot}/team.md` |
| Committed absolute path, different machine | Init Mode for colleague | Gitignore real file; use placeholder in `.example` |
| Non-local backend with missing MCP bridge | State write fails / silent data loss | Confirm `squad_state` in `.mcp.json`; fall back to `"local"` |

**Recovery checklist** (Init Mode unexpectedly):
1. `teamRoot` is an absolute path
2. `{teamRoot}/.squad/team.md` exists with roster entries
3. `config.json` is valid JSON (no double-escaped backslashes from ConvertTo-Json)

---

*[← Setup Guide](./guide.md) · [← Alternative Modes](../../alternative-modes-guide.md)*
