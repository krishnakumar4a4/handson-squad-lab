# Scenario 2 - Deploy and Test an App in Hyper-V

**Estimated duration:** 20-30 minutes  
**Prerequisite:** A buildable Windows app repository and a running, unlocked Windows Hyper-V VM

## Goal

Use Squad to build an app, deploy its artifact to a Windows Hyper-V VM, install and launch it through the VM UI, perform a few UI actions, and report the results.

## Before You Start

Create the test VM:

1. Extract the downloaded **Windows 11 dev environment** image.
2. Open **Hyper-V Manager** and select **New > Virtual Machine**.
3. Choose **Generation 2**, select **Use an existing virtual hard disk**, and browse to the extracted `.vhdx` file.
4. Start the VM, complete Windows sign-in, and leave the desktop unlocked.

Also confirm that:

- GitHub Copilot CLI is installed and authenticated.
- A host-to-guest artifact transfer method is available, such as an Enhanced Session shared drive.
- Obtain the separately shared `hyperv-vm` custom agent definition and note its local file path.

## Step 1 - Clone the App and Enable Squad

Replace the repository URL and folder name:

```powershell
git clone git@github.com:OWNER/REPOSITORY.git REPOSITORY
cd REPOSITORY
npx @bradygaster/squad-cli@latest init
```

Complete the interactive setup, then start Squad:

```powershell
copilot --agent squad
```

## Step 2 - Grant Hyper-V Permission

Open PowerShell as Administrator and add the current host user to **Hyper-V Administrators**:

```powershell
Add-LocalGroupMember `
    -Group "Hyper-V Administrators" `
    -Member "$env:USERDOMAIN\$env:USERNAME"
```

## Step 3 - Refresh the Host Session

Save your work, exit Copilot CLI, log off Windows, and sign back in with the same account. This creates a new access token containing the Hyper-V permission.

Return to the repository and start Squad:

```powershell
cd C:\path\to\REPOSITORY
copilot --agent squad
```

Use `/resume` and select this session. Then verify the group is present:

```powershell
whoami /groups | findstr "S-1-5-32-578"
```

> **Expected:** The command displays the Hyper-V Administrators group.

If you cannot log off, exit Copilot CLI and open **PowerShell as Administrator**. Return to the repository, start Squad using the commands above, then use `/resume` to continue this session from the elevated prompt.

## Step 4 - Add the Hyper-V Specialist

The specialist can be added through an internal hosted MCP registry or from a custom agent definition file. This exercise uses the separately shared `hyperv-vm` custom definition directly.

Give Squad the definition file path and this prompt:

```text
Add a Hyper-V specialist to this Squad using the hyperv-vm agent definition at <AGENT_DEFINITION_PATH>. The specialist must read that definition before every VM task. Update the team and routing so all Hyper-V GUI interaction is assigned to this specialist.
```

In your IDE, confirm that `.squad\team.md`, `.squad\routing.md`, and the specialist's agent folder were created or updated.

## Step 5 - Build, Deploy, and Test

In the same Squad session, replace the placeholders and run:

```text
Build and test this application using the repository's documented commands, then deploy it to the running Windows Hyper-V VM named <VM_NAME>.

Use the configured host-to-guest transfer method to copy the installable artifact to <GUEST_DESTINATION>. Then delegate all guest UI work to the Hyper-V specialist:
1. Run a [PRECHECK] before interacting with the VM.
2. Open PowerShell in the VM.
3. Install the copied artifact using the repository's documented installation command.
4. Start the application and wait for its UI to be ready.
5. Perform these UI actions: <UI_ACTIONS>.
6. Capture visual evidence before and after the UI actions.

Stop and report the exact blocker if the build, transfer, WMI operation, installation, launch, or UI verification fails. Do not invent commands, paths, credentials, or successful results.
```

Avoid using the keyboard or mouse in the VM while the specialist is working. You may watch through **Hyper-V Manager > VM > Connect**.

## Step 6 - Review the Report

The final report should state:

- build and test result;
- artifact path and guest destination;
- installation and launch result;
- UI actions performed and observed outcome;
- captured evidence and any blockers.

Confirm that the report matches what was visibly observed in the VM.

## Step 7 - Enable Autopilot

In Copilot CLI, enter:

```text
/autopilot
```

Enable Autopilot so Squad can continue without waiting for approval after every task.

## Step 8 - Run a Closed-Loop Goal

Choose a follow-up goal, replace the placeholders, and run:

```text
Goal: Implement <FOLLOW_UP_GOAL> and prove that <EXPECTED_UI_OUTCOME> works in the Windows Hyper-V VM named <VM_NAME>.

Work iteratively until the goal is verified:
1. Make the required code changes, then build and test the application.
2. Copy the new installable artifact to <GUEST_DESTINATION>.
3. Ask the Hyper-V specialist to run a [PRECHECK], install or update the app, start it, and wait for its UI.
4. Perform these UI actions: <UI_ACTIONS>.
5. Capture visual evidence and compare the result with the expected outcome.
6. If verification fails, diagnose the cause, fix it, then rebuild, redeploy, and verify again.

Finish only when the expected UI outcome is visibly verified or an unrecoverable blocker is clearly reported. Return a report of the iterations, fixes, deployment result, UI actions, and evidence. Do not invent commands, paths, credentials, evidence, or successful results. Do not push changes or modify unrelated VM settings.
```

When the goal is complete, enter `/autopilot` again to turn Autopilot off.

---

*[Back to Init Mode Guide](../Readme.md)*
