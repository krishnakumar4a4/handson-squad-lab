# Squad Hands-On Workshop Guide — Backup Notes

This document contains reference material and supplementary guidance removed from the participant flow to reduce noise while maintaining useful content for reference.

---

## Practical Adoption

### When to Use Squad

| Scenario | Squad Fit | Notes |
|----------|-----------|-------|
| Solo dev with recurring tasks (tests, docs, reviews) | 🟢 Good fit | Route to specialized agents; review their output |
| Small team with clear domain separation | 🟢 Good fit | Each member covers a domain; coordinator routes issues |
| Large team with existing PR workflow | 🟡 Needs review | Overlay Squad on existing process; start with routing only |
| Security-critical or ambiguous requirements | 🔴 Not suitable | Route to a human; Squad agents are not suitable for these |
| Architecture decisions requiring cross-team input | 🔴 Not suitable | Use ceremonies to capture decisions, but humans decide |

### Anti-Patterns to Avoid

- **Over-routing:** Do not label every issue `squad:*` — reserve it for work clearly within an agent's domain.
- **Expecting agents to act without invocation:** Built-in agents (Scribe, Ralph, Rai, Fact Checker) run when explicitly triggered by ceremonies or the coordinator — they do not autonomously act on your repo without being invoked.
- **Claiming productivity metrics:** Squad helps organize AI-assisted work. Do not cite unverified time-savings or success-rate numbers to your team.

### Responsible-Use Notes

> **Data residency:** When using non-local state backends (orphan, two-layer), Squad state may be synchronized to a remote repository. Apply your organization's repository access and data-handling policies accordingly.

> **Orchestration/cost logs:** `squad cost` and the orchestration-log directory can expose operational metadata (token counts, agent activity). Treat these like any other committed artifact and apply appropriate repository access controls.

---

## Troubleshooting

| Problem | Cause | Fix |
|---------|-------|-----|
| `squad: command not found` | CLI not in PATH | Run `npx @bradygaster/squad-cli@latest <command>` or `npm install -g @bradygaster/squad-cli` |
| npm install fails with proxy/network error | Corporate firewall | Set `npm config set proxy http://proxy:port`; verify registry access |
| `Not a git repository` error on `squad init` | No `.git` dir | Run `git init` first, then `squad init` |
| Permission denied on `.squad/` | File ACL issue | Check directory ownership; on Windows run PowerShell as user (not admin) |
| Windows PowerShell: `&&` not recognized | Syntax issue: PowerShell 5.x does not support `&&` | Use `;` to chain commands in PowerShell 5.x (e.g., `mkdir lab; cd lab; git init`). PowerShell 7+ and cmd.exe support `&&`. Upgrade to PowerShell 7+ for the best experience. |
| npm scripts blocked by execution policy | PowerShell script execution policy prevents npm | If you see "running scripts is disabled", run PowerShell as Administrator and run `Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy RemoteSigned`. This is a troubleshooting step, not a required workshop step. |
| Squad agent not available in Copilot CLI | MCP bridge not configured | Check `.mcp.json` in your project root; ensure `squad_state` server is configured |
| Canary token missing warning | `squad.agent.md` truncated or absent | Run `squad upgrade` to restore; restart your Copilot CLI session |
| `squad cost` shows no data | No orchestration logs yet | Normal on fresh install; data appears after agent sessions run |
| `squad triage` hard-errors with "Cannot create platform adapter" | No git `origin` remote configured | Add a GitHub remote: `git remote add origin <url>`. `squad triage` requires a detectable GitHub platform. On a local-only repo this error is expected — triage is optional. |
