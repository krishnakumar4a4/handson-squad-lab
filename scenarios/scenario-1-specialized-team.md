# Scenario 1 - Build a Codebase-Specialized Squad

**Version:** Squad v0.11.0  
**Estimated duration:** 15-20 minutes  
**Prerequisite:** Complete the [Init Mode hands-on guide](../Readme.md)

## Goal

Clone a repository, initialize Squad, then ask the coordinator to create a team based on the repository's actual code, conventions, and workflows.

## Prerequisites

- Git, Node.js, and npm
- GitHub Copilot CLI installed and authenticated
- Access to the repository you want to clone
- A clean local folder outside this workshop repository

## Step 1 - Create a Workspace

```powershell
mkdir C:\workshop\scenario-1
cd C:\workshop\scenario-1
```

> **Expected:** PowerShell is now in `C:\workshop\scenario-1`.

## Step 2 - Clone Your Repository

Replace the URL and folder name with your repository details.

```powershell
git clone git@github.com:OWNER/REPOSITORY.git REPOSITORY
cd REPOSITORY
```

If SSH is unavailable, use the HTTPS clone URL:

```powershell
git clone https://github.com/OWNER/REPOSITORY.git REPOSITORY
cd REPOSITORY
```

> **Expected:** The repository is cloned and PowerShell is inside its root directory.

## Step 3 - Initialize Squad

```powershell
npx @bradygaster/squad-cli@latest init
```

Complete the interactive prompts. Adding `@copilot` is optional.

> **Expected:** Squad creates `.squad\`, `.github\agents\squad.agent.md`, `.copilot\mcp-config.json`, and related configuration files.

## Step 4 - Inspect the Default Team

Open the repository in your preferred IDE. For Visual Studio Code:

```powershell
code .
```

In the IDE, inspect:

- `.squad\team.md`
- `.squad\routing.md`
- `.squad\agents\`

Check the initial roster, routing rules, and agent folders before specialization.

## Step 5 - Create a Specialized Team

Start Squad:

```powershell
copilot --agent squad
```

Paste this prompt exactly:

```text
Go through the codebase and create a specialised team for handling issues, feature requests. Also factor in distinct code patterns, styles from the codebase to ensure the fixes and reviews are more natural looking and human friendly.
```

Review the proposed roster before confirming it. Names and team size may vary; evaluate whether the roles match the repository.

> **Expected:** Squad creates or updates the roster, routing rules, and agent charters using codebase-specific information.

## Step 6 - Validate the Specialized Team

### Check the roster

Open `.squad\team.md` in your IDE.

Confirm:

- The file contains the exact `## Members` heading.
- Roles relate to the repository's languages, components, and workflows.
- Members are not limited to generic labels such as Developer or Tester.

### Check routing

Open `.squad\routing.md` in your IDE.

Confirm:

- Work types are concrete and repository-specific.
- Issue, feature, test, documentation, and review work has clear ownership.
- Placeholder values have been removed.

### Check agent charters

Open `.squad\agents\` in your IDE and review each member's `charter.md`.

Confirm the charters reference real repository details, such as:

- languages, frameworks, modules, or packages;
- build, test, lint, and formatting commands;
- existing coding patterns and style rules;
- safety-sensitive or platform-specific areas;
- review boundaries and expected handoffs.

### Check generated state

In your IDE, inspect `.squad\agents\` and `.squad\casting\registry.json`. Then run:

```powershell
git status --short
```
Confirm:

- each active member has one agent folder;
- no duplicate members were created;
- only Squad-related files changed;
- source files were not modified.

### Run a configuration check

```powershell
squad doctor
```

Use this as a sanity check. Squad v0.11.0 may not validate every runtime-state detail.

## Completion Checklist

- [ ] Repository cloned successfully
- [ ] Squad initialized
- [ ] Default team inspected
- [ ] Specialization prompt completed
- [ ] Roster reflects the codebase
- [ ] Routing contains concrete work types
- [ ] Charters reference actual repository conventions
- [ ] No source code or remote repository was changed

## Repeat Safely

To repeat the scenario, clone the repository into a new folder. Do not use `git reset --hard` or delete a working copy that may contain changes.

This scenario does not require code changes, issue creation, commits, or remote pushes.

---

*[Back to Init Mode Guide](../Readme.md)*
