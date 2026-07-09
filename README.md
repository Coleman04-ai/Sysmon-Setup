# 🛡️ Sysmon Setup Guide

> **Step-by-step guide: Installing Microsoft Sysmon with SwiftOnSecurity configuration on Windows**

![Status](https://img.shields.io/badge/Status-Working-success)
![Sysmon](https://img.shields.io/badge/Sysmon-v15.21-blue)
![Windows](https://img.shields.io/badge/Windows-10%20%7C%2011-brightgreen)

---

## 📌 What is Sysmon?

**Sysmon** (System Monitor) is a Microsoft Sysinternals tool that logs detailed system activity to Windows Event Viewer. It provides deep visibility that standard Windows logging misses.

### What Sysmon Tracks:
- ✅ **Process Creation** - Every program that runs (with full command line)
- ✅ **Network Connections** - Source/destination IP and ports
- ✅ **File Creation/Deletion** - New files and deleted files
- ✅ **Registry Changes** - Registry modifications
- ✅ **DNS Queries** - Domain lookups
- ✅ **Driver Loading** - Kernel driver activity

---

## 📋 What I Used

| Component | Detail |
|-----------|--------|
| **System** | Windows 10/11 |
| **Sysmon Version** | v15.21 |
| **Configuration** | SwiftOnSecurity |
| **Install Location** | `C:\Sysmon` |
| **Log Location** | Microsoft-Windows-Sysmon/Operational |

---

## 🔧 Installation Steps (With Screenshots)
### Step 1: Create Sysmon Folder

Create a folder to store Sysmon files:

```powershell
New-Item -ItemType Directory -Path "C:\Sysmon" -Force

https://1sysmon-folder-creation.png/

Step 2: Download and Extract Sysmon
Download Sysmon from Microsoft and extract the ZIP file:

powershell
# Download Sysmon (or copy from Downloads)
Move-Item "$env:USERPROFILE\Downloads\Sysmon.zip" -Destination "C:\Sysmon\" -Force

# Extract the ZIP
Expand-Archive -Path "C:\Sysmon\Sysmon.zip" -DestinationPath "C:\Sysmon\" -Force

# Download SwiftOnSecurity config
Invoke-WebRequest -Uri "https://raw.githubusercontent.com/SwiftOnSecurity/sysmon-config/master/sysmonconfig-export.xml" -OutFile "C:\Sysmon\config.xml"

# Verify files
Get-ChildItem C:\Sysmon\
Expected files after extraction:

Sysmon64.exe - Main Sysmon executable

Sysmon.exe - 32-bit version

Eula.txt - License agreement

config.xml - SwiftOnSecurity configuration

https://2unzip-sysmon-files.png

Step 3: Install Sysmon
Run the installation command:

powershell
cd C:\Sysmon
.\Sysmon64.exe -accepteula -i config.xml
Expected output:

text
System Monitor v15.21 - System activity monitor
Sysmon64 installed.
SysmonDrv installed.
Starting SysmonDrv.
SysmonDrv started.
Starting Sysmon64.
Sysmon64 started.

https://3sysmon-installation.png

Step 4: Verify Installation
Check if Sysmon is running and logging events:

powershell
# Check if Sysmon process is running
Get-Process Sysmon*

# Check event log
Get-WinEvent -LogName "Microsoft-Windows-Sysmon/Operational" -MaxEvents 5
Expected output:

Sysmon64 process running (PID shown)

Events appearing in event log

https://4sysmon-event-process-verification.png

Step 5: Test Sysmon
Generate test events to confirm Sysmon is working:

powershell
# Test Script
Start-Process notepad.exe
Start-Sleep -Seconds 2
Stop-Process -Name notepad -ErrorAction SilentlyContinue

Start-Process msedge.exe -ErrorAction SilentlyContinue
Start-Sleep -Seconds 3
Stop-Process -Name msedge -ErrorAction SilentlyContinue

# Check recent events
Get-WinEvent -LogName "Microsoft-Windows-Sysmon/Operational" -MaxEvents 5 | Format-Table TimeCreated, Id, LevelDisplayName -AutoSize
Expected output:

Event ID 1 - Process Creation (Notepad)

Event ID 3 - Network Connection (Browser)

https://5testing.png

✅ How to View Logs
Method 1: Event Viewer (GUI)
Press Win + R

Type: eventvwr.msc

Navigate to:

text
Applications and Services Logs → Microsoft → Windows → Sysmon → Operational
Method 2: PowerShell
powershell
Get-WinEvent -LogName "Microsoft-Windows-Sysmon/Operational" -MaxEvents 10
📊 Important Event IDs
Event ID	Description	Why It Matters
1	Process Creation	Detect malware execution
3	Network Connection	Detect C2 communication
11	File Creation	Detect malware droppers
12-14	Registry Changes	Detect persistence
22	DNS Query	Detect malicious domains
📝 My Results
Check	Status	Details
Installation	✅ Success	Sysmon v15.21 installed
Process Running	✅ Yes	PID: 18644
Event Log	✅ Active	141+ events recorded
Test Events	✅ Generated	Event ID 1 and 3 captured
Config	✅ Loaded	SwiftOnSecurity
Sample Event I Captured:
text
Event ID: 1 (Process Creation)
Process: splunk-optimize.exe
User: SYSTEM
Time: 2026-07-09 13:33:51
🔧 Troubleshooting
❌ Sysmon Not Running
powershell
# Reinstall
cd C:\Sysmon
.\Sysmon64.exe -u
.\Sysmon64.exe -accepteula -i config.xml
❌ No Events in Event Viewer
Verify Sysmon is running: Get-Process Sysmon*

Generate test event: Open Notepad

Refresh Event Viewer (F5)

⚠️ Service Not Found in services.msc
This is normal! In newer versions, Sysmon runs as a process, not a service. If Get-Process Sysmon* shows it running, it's working.

🗑️ Uninstall Sysmon
powershell
cd C:\Sysmon
.\Sysmon64.exe -u
📚 Resources
Microsoft Sysmon Documentation

SwiftOnSecurity Config

Sysmon Event ID Reference
