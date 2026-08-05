# Scenario 2 - Goal-Based Closed-Loop Development with Squad

**Estimated duration:** 20-30 minutes  
**Prerequisite:** A Windows host with Hyper-V and Visual Studio C++ Build Tools

## Goal

Use Squad to build the native Squad Counter app, deploy its executable to a Windows Hyper-V VM, exercise its UI, and report the results.

## Before You Start

Create the test VM:

1. Open **Hyper-V Manager** and select **Quick Create**.
2. Select **Windows 11 dev environment** and wait for creation to complete.
3. Start the VM, sign in, and leave the desktop unlocked.

Also confirm that:

- GitHub Copilot CLI is installed and authenticated.
- Visual Studio or Visual Studio Build Tools includes **Desktop development with C++**.
- Obtain the separately shared `hyperv-vm` custom agent definition and note its local file path.

## Step 1 - Clone the App and Enable Squad

```powershell
mkdir C:\workshop\scenario-2
cd C:\workshop\scenario-2
git clone https://github.com/krishnakumar4a4/handson-squad-ui-app.git
cd handson-squad-ui-app
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
cd C:\workshop\scenario-2\handson-squad-ui-app
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

In the same Squad session, replace `<VM_NAME>` and run:

```text
Build and test the Squad Counter app by running build.cmd. Confirm that the tests pass and that build\SquadCounter.exe is created. Then deploy it to the running Windows Hyper-V VM named <VM_NAME>.

Use the configured host-to-guest transfer method to copy the executable to C:\SquadCounter\SquadCounter.exe in the VM. Then delegate all guest UI work to the Hyper-V specialist:
1. Run a [PRECHECK] before interacting with the VM.
2. Open PowerShell in the VM.
3. Start C:\SquadCounter\SquadCounter.exe and wait for its UI.
4. Click Increase +1 twice, then click Decrease -1 once.
5. Verify the counter displays 1 and the status says "Counter decreased. Current value: 1".
6. Click Reset to 0 and verify the counter displays 0.
7. Capture visual evidence before and after the UI actions.

Stop and report the exact blocker if the build, transfer, WMI operation, launch, or UI verification fails. Do not invent commands, paths, credentials, or successful results.
```

Avoid using the keyboard or mouse in the VM while the specialist is working. You may watch through **Hyper-V Manager > VM > Connect**.

## Step 6 - Review the Report

The final report should state:

- build and test result;
- artifact path and guest destination;
- copy and launch result;
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

Replace `<VM_NAME>` and run:

```text
Goal: Add a "Double +2" button to Squad Counter and prove it works in the Windows Hyper-V VM named <VM_NAME>. Each click must add 2. The status must show "Counter doubled. Current value: 2" after the first click and "Counter doubled. Current value: 4" after the second.

Work iteratively until the goal is verified:
1. Update the app and its counter tests.
2. Run build.cmd and confirm the tests pass.
3. Replace C:\SquadCounter\SquadCounter.exe in the VM with the new executable.
4. Ask the Hyper-V specialist to run a [PRECHECK], start the updated app, and wait for its UI.
5. Click Reset to 0, then click Double +2 twice.
6. Verify the counter displays 4 and the status says "Counter doubled. Current value: 4".
7. Capture visual evidence and compare the result with the expected outcome.
8. If verification fails, diagnose the cause, fix it, then rebuild, redeploy, and verify again.

Finish only when the expected UI outcome is visibly verified or an unrecoverable blocker is clearly reported. Return a report of the iterations, fixes, deployment result, UI actions, and evidence. Do not invent commands, paths, credentials, evidence, or successful results. Do not push changes or modify unrelated VM settings.
```

When the goal is complete, enter `/autopilot` again to turn Autopilot off.

## Known Issues

### UAC Prompt in the VM

The Hyper-V specialist may be unable to click **Yes** on the Windows secure-desktop UAC prompt. Synthetic input can cancel the prompt instead of approving it.

If this occurs:

1. Connect to the VM through Hyper-V Manager.
2. Manually open **Windows Terminal** or **PowerShell** as administrator and approve UAC.
3. Leave the elevated terminal open and focused.
4. Tell Squad:

```text
Reuse the existing Administrator PowerShell terminal in the VM for all elevated commands. Do not close it or launch another elevated process.
```

The specialist can then type commands into the existing elevated terminal without triggering repeated UAC prompts.

### No Visible VM Progress

The specialist uses headless WMI screenshots and synthetic input, so desktop movement may be intermittent. A long pause can also mean the VM is waiting on a dialog or the agent is stuck.

Ask Squad to capture and report progress explicitly:

```text
Capture a screenshot after every step, success, failure, or stuck state. Report the last action, current screen, result, next action, and any blocker. If no visible progress occurs, capture a fresh screenshot before continuing.
```

You can also watch through **Hyper-V Manager > VM > Connect**. Avoid using the keyboard or mouse unless Squad asks you to resolve a blocker.

---

*[Back to Init Mode Guide](../Readme.md)*
