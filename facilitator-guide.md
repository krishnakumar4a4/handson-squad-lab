# Squad Hands-On Workshop Facilitator Guide

**Version:** Squad v0.11.0 · **Duration:** 60 minutes

**Audience:** Workshop facilitators delivering the hands-on Squad session  
**Purpose:** Guidance for preparing, pacing, troubleshooting, and delivering the hands-on workshop

[→ See the participant guide: `hands-on-guide.md`](./hands-on-guide.md)

---

## Pre-Session Checklist

Complete these steps **before participants arrive:**

### Environment Verification (15 min)

- [ ] **Your machine:** Node.js ≥ 22.5.0, npm ≥ 10, Git, GitHub Copilot CLI, `gh` CLI (for optional GitHub lab)
- [ ] **Test a fresh init locally:** Run `npx @bradygaster/squad-cli@latest init` in a temp directory to confirm no blockers
- [ ] **Network:** Verify npm registry is accessible; test behind corporate proxy if applicable
- [ ] **Participants' environment checklist:** Prepare a pre-session email confirming:
  - [ ] Node.js ≥ 22.5.0 (`node --version`)
  - [ ] npm ≥ 10 (`npm --version`)
  - [ ] Git (`git --version`)
  - [ ] GitHub Copilot CLI (standalone) installed and authenticated (`copilot --version`)
  - [ ] GitHub CLI `gh` — **required only for the optional GitHub extension** (`gh --version`)
  - [ ] Internet access (npm install must reach the registry)
  - [ ] A local Git repository to work in (or create one: `git init my-lab; cd my-lab`)
  - [ ] Windows: PowerShell 7+ recommended; avoid legacy cmd.exe

### Demo Prep (30 min, optional)

If you plan to demo Lab 2 (Coordinator Routing via Copilot CLI) live:

1. Create a sample project with Squad installed locally
2. Walk through the Squad agent invocation in advance
3. Prepare the test prompt: `"Task: We need to add unit tests for the auth module."`
4. Verify the agent responds with correct routing decision

If you plan to demo the optional GitHub extension:

1. Create a test GitHub repo with Issues enabled
2. Create a sample issue and apply the `squad` label
3. Walk through `squad triage --execute --max-concurrent 1` and monitor PR creation
4. Note the agent's reasoning in the generated PR description

### Timing Assumptions

- **Total session:** 60 minutes facilitated
- **Setup Variants:** ~10 min (Variant A recommended)
- **Lab 1 (Inspect & Routing):** ~10 min
- **Lab 2 (Coordinator Routing):** ~10 min
- **Lab 3 (Model Config):** ~5 min
- **Lab 4 (Monitoring & Memory):** ~10 min
- **Optional GitHub Extension:** ~10 min (skip if time-constrained)
- **Buffer for Q&A:** ~5 min

If facilitating self-paced (~30 min):
- Focus on Variant A, Labs 1–3 only
- Omit the optional GitHub extension
- Provide participant guide and let them work independently

---

## 60-Minute Agenda

### Welcome & Overview (2 min)

- **Say:** "Squad is a lightweight AI-team framework. By the end of this session, you'll have Squad running in a Git repo and understand how to route work to specialized agents."
- **Show:** Briefly open the participant guide (`hands-on-guide.md`) to orient everyone to the structure.

### Setup Variants (10 min)

**Recommended:** Use **Variant A (Fresh Init)** unless participants have existing Squad installations.

**Delivery notes:**
- Walk through commands step-by-step on screen
- Pause after each command to check for errors
- If `npm install` is slow, narrate what's happening to avoid silence
- Watch for:
  - `npm ERR!` → Check corporate proxy settings; have participant adjust
  - `Not a git repository` → Confirm `git init` was run first
  - On Windows, if someone used `&&` in cmd.exe, remind them to use `;` or switch to PowerShell 7+

**Expected:** Each participant has `.squad/team.md` and `.squad/routing.md` created.

### Core Capabilities Overview (3 min)

**Say:** "Now that Squad is initialized, let's explore what you've got. Squad surfaces four key concepts: your team roster, routing rules, memory/ceremonies, and configuration."

- Open and point to `.squad/team.md` (roster)
- Open and point to `.squad/routing.md` (routing rules)
- Explain: "These files are your source of truth. If an issue gets a `squad:{member}` label, that member picks it up per the routing table."

### Lab 1 — Inspect, Cast, and Routing (~10 min)

**Facilitator role:** Guide, do not solve

- Have participants run the three commands from Lab 1 independently
- While they work, circulate and ask: "Which member handles documentation? Where would a bug-fix issue route?"
- Collect answers; discuss any confusion
- **Reflection stop:** "Notice the routing table maps work types to agents. This is how Squad knows who does what. Any questions?"

### Lab 2 — Coordinator Routing via Copilot CLI (~10 min)

**Facilitator role:** Demonstrate first, then guide participants

1. **Demonstrate on screen:**
   - Run `copilot` in your sample Squad project
   - Select the Squad agent from the picker
   - Enter: `"Task: We need to add unit tests for the auth module."`
   - Show the agent's routing decision and reasoning

2. **Debrief:** "The coordinator analyzed the task, identified it as a testing concern, and routed to the QA agent per `routing.md`. You can invoke this same agent in your own projects."

3. **Have participants try:** Open `copilot` in their squad-lab directory, select Squad, enter their own task description (any work type), and observe the routing.

**Troubleshooting note:** If the Squad agent is not available, check `.mcp.json` in the project root to confirm `squad_state` server is configured.

### Lab 3 — Model and Reasoning Configuration (~5 min)

- Have participants run `squad start --help` and skim the output
- Explain: "These flags control which AI model Squad uses and how it reasons about work. You can customize this per session."
- **Optional demo:** Run `squad start --model gpt-4o` and explain how to choose models based on your task complexity

### Lab 4 — Monitoring, Memory, and Ceremonies (~10 min)

- Have participants run `squad nap --dry-run` (show preview mode)
- Run `squad cost --all` together (may show no data on fresh install — that's normal)
- Have them inspect `.squad/decisions.md` to see the governance structure
- **Optional triage dry-run:** Run `squad triage` (no `--execute` flag) to show how Squad scans for actionable work

**Note:** With no GitHub issues configured, `squad triage` will likely report nothing to do — this is correct and not an error.

### Optional Extension — GitHub Issue to PR (10 min, skip if time-constrained)

**Prerequisites:** Participant has `gh` CLI authenticated and a GitHub repo with Issues enabled.

**Facilitator role:** Hands-on guidance

1. Have participant create an issue in their GitHub repo with title like `"Add unit tests for auth module"` and apply the `squad` label
2. **Demonstrate:** Run `squad triage --execute --max-concurrent 1` on your own test repo
3. Show the resulting draft PR on GitHub and explain the agent's reasoning
4. Have participants try this flow if time permits
5. **If it fails:** Common causes are missing GitHub CLI auth or no `squad` label on the issue. Troubleshoot with participant.

---

## Delivery Notes Per Lab

### Lab 1 — Inspect, Cast, and Routing

**Common questions:**
- *"Why does my `routing.md` look different?"* → Squad allows custom routing rules per team. The default varies by setup. Show them how to read and understand their specific routing table.
- *"Can I add my own agent?"* → Yes, `squad cast --name MyAgent --role "Custom Role"` adds an agent. We'll see this in later workshops.

**If someone goes off-track:**
- Refocus: "Let's read the routing table and identify which agent handles testing. That's the main idea for this lab."

---

### Lab 2 — Coordinator Routing via Copilot CLI

**Common issues:**
- Squad agent not in picker → Check `.mcp.json`; confirm `squad_state` server is configured. Have them run `squad upgrade` if needed.
- Copilot CLI not installed → Confirm with `copilot --version`. If missing, installation may be needed; reference GitHub Copilot documentation.
- Prompt yields generic response (no routing explanation) → The agent may need clarification. Try: `"Route this task: 'Add unit tests for auth module'"`

**If participant gets stuck on Squad agent invocation:**
- Simplify: "Just select Squad from the agent picker and type your task description. Squad will tell us which team member should handle it."

---

### Lab 3 — Model and Reasoning Configuration

**Key point:** This lab is about awareness, not action. Participants don't need to change models to pass. They just need to know flags exist.

**If someone wants to try `squad start --model`:**
- Explain: "You can try this after the session. For now, let's focus on understanding the options."

---

### Lab 4 — Monitoring, Memory, and Ceremonies

**Expected behaviors on a fresh install:**
- `squad cost` shows **no data** → Normal; data accumulates after agent sessions run
- `.squad/orchestration-log/` is **empty** → Normal; logs appear after ceremonies execute
- `squad triage` finds **nothing** → Normal; no issues labeled yet
- `.squad/decisions.md` shows **governance template** → Normal; decisions accumulate over time

**Reflection question:** "Notice Squad keeps a log of decisions and token usage. As your team grows, you'll use these to stay aligned and track AI-assisted work."

---

## Safety Gate Note (for Variants B & C)

If a participant is upgrading an existing Squad installation or switching state backends:

1. Ensure they run `git status` first and review any uncommitted changes
2. Have them stash uncommitted work: `git stash`
3. Proceed with upgrade or backend switch
4. After completion, verify `.squad/config.json` reflects the new backend

**Why:** Squad state is committed to your repo. Upgrading without a clean working tree can cause merge conflicts.

---

## Troubleshooting & Recovery

| Problem | On-the-Fly Fix | Root Cause |
|---------|----------------|-----------|
| Participant misses setup; gets `Not a git repository` on Lab 1 | Have them run `git init` right now, then `squad init` | They skipped Variant A step 1 |
| `npm install` stalls behind proxy | Set proxy: `npm config set proxy http://proxy:port` | Corporate firewall blocking npm registry |
| Squad agent not available in Copilot CLI | Have them check `.mcp.json` exists; run `squad upgrade` if needed | MCP bridge missing or outdated |
| Windows participant uses `&&` and gets error | Remind them: PowerShell uses `;` not `&&` | Legacy cmd.exe syntax |
| Optional GitHub lab: `squad triage` finds nothing | Check: did they apply the `squad` label to the issue? | Issue not labeled or not in GitHub yet |
| Participant gets stuck on Lab 2 routing prompt | Suggest: "Try: 'Route this task: [any work description]'" | Prompt phrasing doesn't trigger clear response |

---

## Post-Session Checklist

After the workshop:

- [ ] Collect feedback (Google Form, Slack poll, or verbal)
- [ ] Note any participant blockers that recurred (proxy, CLI version, etc.)
- [ ] Follow up with participants who dropped out or had unresolved errors
- [ ] Update this guide if you discover new common issues
- [ ] Share notes with Danny (Workshop Lead) for next iteration

---

## Useful Facilitator Commands

| Command | Purpose |
|---------|---------|
| `squad doctor` | Diagnostic check of Squad health and configuration |
| `squad upgrade` | Force update Squad-owned files (if MCP bridge is stale) |
| `npx @bradygaster/squad-cli@latest init --help` | See all init flags and state backend options |
| `squad triage --help` | See triage command options, including `--execute` for live demo |
| `git stash && git stash pop` | Quickly save/restore uncommitted work if participant gets confused |

---

## Feedback Integration

After each session, capture:
- Participant setup time (actual vs. 10-min estimate)
- Most common blockers
- Questions participants asked repeatedly
- Any labs that took longer than estimated

Share findings with Danny and Linus to refine future iterations.
