# 🖥️ monitor-lid-guard

> Keeps your laptop awake when the lid is closed with an external monitor connected. Automatically switches to sleep when unplugged.

**Problem:** You close your laptop lid while an external monitor is connected — and Windows puts it to sleep, killing your workflow.

**Solution:** A lightweight PowerShell script that watches for display changes and dynamically switches the lid close action:

| Scenario | Lid Close Action |
|---|---|
| 🔌 External monitor **connected** | **Do nothing** — laptop keeps running |
| 💤 No external monitor | **Sleep** — normal behavior |

## ✨ Features

- 🔄 Real-time monitor detection (polling every 5 seconds via WMI)
- 🔋 Works on both AC power and battery
- 👻 Runs silently in the background (no windows, no tray icons)
- 🛡️ Safe defaults — restores "Sleep" action on exit
- 🪟 Windows Task Scheduler integration for auto-start at logon
- ♻️ Auto-restart on failure (3 attempts, 1 min interval)

## 📋 Requirements

- Windows 10 / 11
- PowerShell 5.1+
- Administrator privileges (required for `powercfg`)

## 🚀 Installation

### 1. Download the script

```powershell
git clone https://github.com/YOUR_USERNAME/monitor-lid-guard.git
```

Or just download [`LidAction-MonitorAware.ps1`](LidAction-MonitorAware.ps1) directly.

### 2. Register as a scheduled task

Open **PowerShell as Administrator** and run:

```powershell
$scriptPath = 'C:\path\to\LidAction-MonitorAware.ps1'

$action = New-ScheduledTaskAction `
    -Execute 'powershell.exe' `
    -Argument "-NoProfile -ExecutionPolicy Bypass -WindowStyle Hidden -File `"$scriptPath`""

$trigger = New-ScheduledTaskTrigger -AtLogOn -User $env:USERNAME

$settings = New-ScheduledTaskSettingsSet `
    -AllowStartIfOnBatteries `
    -DontStopIfGoingOnBatteries `
    -ExecutionTimeLimit ([TimeSpan]::Zero) `
    -RestartCount 3 `
    -RestartInterval (New-TimeSpan -Minutes 1)

$principal = New-ScheduledTaskPrincipal -UserId $env:USERNAME -RunLevel Highest -LogonType Interactive

Register-ScheduledTask `
    -TaskName 'LidAction-MonitorAware' `
    -Action $action `
    -Trigger $trigger `
    -Settings $settings `
    -Principal $principal `
    -Description 'Lid close: do nothing if external monitor, sleep otherwise.' `
    -Force
```

> **Note:** Replace `C:\path\to\` with the actual path to the script.

### 3. Start immediately (optional)

```powershell
Start-ScheduledTask -TaskName 'LidAction-MonitorAware'
```

The script will also start automatically at every logon.

## 🧪 Manual run (for testing)

Run in an elevated PowerShell window to see live output:

```powershell
powershell -ExecutionPolicy Bypass -File .\LidAction-MonitorAware.ps1
```

Example output:

```
══════════════════════════════════════════════════════════
  monitor-lid-guard  —  Press Ctrl+C to stop
══════════════════════════════════════════════════════════
[21:58:30] External monitors: 1
[21:58:30] Lid close action: Do nothing
[21:58:30] Watching for changes...

[21:59:05] Change detected — external monitors: 0
[21:59:05] Lid close action: Sleep
```

## 🗑️ Uninstall

Run as Administrator:

```powershell
Unregister-ScheduledTask -TaskName 'LidAction-MonitorAware' -Confirm:$false
```

Then delete the script file.

## ⚙️ How it works

1. Queries `WmiMonitorBasicDisplayParams` via WMI to count active displays
2. Subtracts 1 for the built-in laptop screen → external monitor count
3. Uses `powercfg` to set the lid close action for both AC and DC power:
   - `SETACVALUEINDEX` / `SETDCVALUEINDEX` on subgroup `4f971e89-eebd-4455-a8de-9e59040e7347` (Power buttons and lid), setting `5ca83367-6e45-459f-a27b-476b1d01c936` (Lid close action)
   - Value `0` = Do nothing, `1` = Sleep
4. Polls every 5 seconds and only applies changes when the state actually changes
5. On exit (Ctrl+C or process termination), restores "Sleep" as a safe default

## 📄 License

MIT
