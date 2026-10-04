# 🖥️ lid-monitor-switch

> Keeps your laptop awake and unlocked when the lid is closed with an external monitor connected. Automatically switches to sleep when unplugged.

**Problem:** You close your laptop lid while an external monitor is connected — and Windows puts it to sleep or locks the screen, killing your workflow (especially with tools like Mouse Without Borders).

**Solution:** A lightweight PowerShell script that watches for display changes and dynamically switches the lid close action and screen lock behavior:

| Scenario | Lid Close Action | Screen Lock |
|---|---|---|
| 🔌 External monitor **connected** | **Do nothing** | **Disabled** |
| 💤 No external monitor | **Sleep** | **Enabled** |

## ✨ Features

- 🔄 Real-time monitor detection (polling every 5 seconds via WMI)
- 🔓 Disables screen lock on lid close when external monitor is connected
- 🔋 Works on both AC power and battery
- 👻 Runs silently in the background (no windows, no tray icons)
- 🛡️ Safe defaults — restores sleep and lock on exit
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
==========================================================
  monitor-lid-guard  --  Press Ctrl+C to stop
==========================================================
[21:58:30] External monitors: 1
[21:58:30] Lid close action: Do nothing
[21:58:30] Console lock: OFF (no lock on lid close)
[21:58:30] Watching for changes...

[21:59:05] Change detected - external monitors: 0
[21:59:05] Lid close action: Sleep
[21:59:05] Console lock: ON (lock on lid close)
```

## 🗑️ Uninstall

Run as Administrator:

```powershell
Unregister-ScheduledTask -TaskName 'LidAction-MonitorAware' -Confirm:$false
```

Then delete the script file.

## ⚙️ How it works

1. Queries `WmiMonitorBasicDisplayParams` via WMI to count active displays
2. Subtracts 1 for the built-in laptop screen = external monitor count
3. Uses `powercfg` to configure two settings (both for AC and DC power):
   - **Lid close action** — subgroup `4f971e89-...` / setting `5ca83367-...` — `0` (Do nothing) or `1` (Sleep)
   - **Console lock on display off** — subgroup `fea3413e-...` / setting `0e796bdb-...` — `0` (Don't lock) or `1` (Lock)
4. Polls every 5 seconds and only applies changes when the state actually changes
5. On exit (Ctrl+C or process termination), restores Sleep + Lock as safe defaults

## 🤝 Use case

Perfect for setups where you:
- Use your laptop as a desktop with an external monitor
- Share keyboard and mouse across machines with [Mouse Without Borders](https://www.microsoft.com/en-us/garage/wall-of-fame/mouse-without-borders/)
- Want to close the lid without losing your session

## 📄 License

MIT
