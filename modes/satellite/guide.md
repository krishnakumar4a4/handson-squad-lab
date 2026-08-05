# Satellite Mode — Setup Guide

**Version:** Squad v0.11.0

[← Alternative Modes Guide](../../alternative-modes-guide.md) · [→ Supplementary (distinctions, ownership, commit guidance, recovery)](./supplementary.md)

---

## Prerequisites and Safety

- `teamRoot` must be an **absolute path**. `.squad/config.json` is **not portable** if committed with an absolute path — see [commit/ignore guidance](./supplementary.md#commit-and-ignore-guidance).
- The hub's `.squad/team.md` must exist with at least one `## Members` entry; otherwise the coordinator enters Init Mode on the satellite.
- Satellites own only pointer config locally — charters, routing, and decisions come from the hub.
- Never commit secrets or personal path tokens into `.squad/config.json`.
- With a non-`local` backend, confirm the `squad_state` MCP bridge is reachable before any state write.

---

## Setup Steps

### 1 — Verify the Hub Exists

```powershell
Get-Item C:\path\to\team-hub\.squad\team.md
```

> **Expected:** File exists with a `## Members` section and at least one entry.

### 2 — Initialize the Satellite Repo

```powershell
mkdir satellite-repo; cd satellite-repo
git init
npx @bradygaster/squad-cli@latest init
```

### 3 — Set `teamRoot` in `.squad\config.json`

```json
{
  "version": 1,
  "teamRoot": "C:\\SquadTeams\\team-hub"
}
```

Replace `C:\\SquadTeams\\team-hub` with your actual hub repository root path on disk.

> **JSON path escaping:** In JSON, each backslash must be written as `\\`. So the Windows path `C:\SquadTeams\team-hub` becomes `"C:\\SquadTeams\\team-hub"` in the JSON file.
>
> **PowerShell `ConvertTo-Json` warning:** May double-escape backslashes, producing `C:\\\\SquadTeams\\\\team-hub` (incorrect). Use a direct editor or `Set-Content` with a literal string:
>
> ```powershell
> Set-Content .squad\config.json -Value '{"version":1,"teamRoot":"C:\\SquadTeams\\team-hub"}'
> ```

> **Portability:** `teamRoot` is an absolute path, so this file is **not portable across machines**. Gitignore the real file and commit a `.squad/config.json.example` with a safe placeholder.

### 4 — Verify Resolution

```powershell
Get-Content .squad\config.json
squad status
```

> **Expected:** `squad status` reports the local `.squad` path and backend. It does **not** verify that `teamRoot` resolves correctly.
>
> **Runtime verification** (confirming the hub's `team.md` is actually loaded) requires an authenticated Copilot CLI session: run `copilot --agent squad` in the satellite repo and confirm the coordinator greets with the **hub team's roster**. If you see Init Mode instead, `teamRoot` is wrong or the hub `team.md` is missing.

### 5 — Set State Backend (Optional)

Add `stateBackend` to `config.json` if needed:

```json
{
  "version": 1,
  "teamRoot": "C:\\SquadTeams\\team-hub",
  "stateBackend": "local"
}
```

| `stateBackend` | Effect |
|---|---|
| `"local"` (default) | Mutable state in local working tree |
| `"orphan"` | Separate orphan branch; sync with `squad sync` |
| `"two-layer"` | Main branch for static config; remote state branch for mutable state |

With `"orphan"` or `"two-layer"`, mutable state (decisions, history) lives on a branch in the **satellite repo** — not the hub.

---

## Verification Checklist

- [ ] `config.json` contains a valid `teamRoot` absolute path with double-escaped backslashes
- [ ] `Get-Item {teamRoot}\.squad\team.md` returns a file with `## Members`
- [ ] `squad status` runs without error
- [ ] `copilot --agent squad` greets with hub team roster (not Init Mode)
- [ ] Real `config.json` is gitignored; a `.example` placeholder is committed

---

**→ [Supplementary Guide](./supplementary.md)** — key distinctions, hub vs. satellite ownership, commit/ignore rules, common mistakes, and recovery.
