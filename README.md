# Sysmon Setup Guide

## Objective

- Deploy Microsoft Sysmon on Windows 10 for enhanced security logging
- Configure Sysmon using the SwiftOnSecurity configuration
- Verify process creation, network connection, and file events are being captured
- Document the setup process for future reference and team knowledge sharing
- Build a foundation for threat detection and hunting

---

## Investigation Methodology

### 1. Folder Creation

Created a dedicated directory for Sysmon files:

```powershell
New-Item -ItemType Directory -Path "C:\Sysmon" -Force
```

**Output:**

```
    Directory: C:\

Mode    LastWriteTime    Length Name
----    -------------    ------ ----
d-----  7/9/2026   1:53 PM           Sysmon
```

![Step 1: Create Sysmon Folder](screenshots/1sysmon-folder-creation.png)

---

### 2. Download and Extract Sysmon

Downloaded Sysmon from Microsoft Sysinternals and extracted the ZIP file:

```powershell
# Move Sysmon.zip from Downloads
Move-Item "$env:USERPROFILE\Downloads\Sysmon.zip" -Destination "C:\Sysmon\" -Force

# Extract the ZIP
Expand-Archive -Path "C:\Sysmon\Sysmon.zip" -DestinationPath "C:\Sysmon\" -Force

# Download SwiftOnSecurity configuration
Invoke-WebRequest -Uri "https://raw.githubusercontent.com/SwiftOnSecurity/sysmon-config/master/sysmonconfig-export.xml" -OutFile "C:\Sysmon\config.xml"

# Verify extracted files
Get-ChildItem C:\Sysmon\
```

**Output:**

```
    Directory: C:\Sysmon

Mode    LastWriteTime    Length Name
----    -------------    ------ ----
-a----  7/9/2026   1:59 PM    123257 config.xml
-a----  6/17/2026   7:19 PM      7490 Eula.txt
-a----  6/17/2026   7:21 PM   6258072 Sysmon.exe
-a----  6/26/2026   1:33 PM   2881663 Sysmon.zip
-a----  6/17/2026   7:21 PM   3250120 Sysmon64.exe
-a----  6/17/2026   7:21 PM   3141024 Sysmon64a.exe
```

**Files extracted:**
- `Sysmon64.exe` - Main 64-bit executable
- `Sysmon.exe` - 32-bit version
- `Eula.txt` - License agreement
- `config.xml` - SwiftOnSecurity configuration

![Step 2: Download and Extract Sysmon](screenshots/2unzip-sysmon-files.png)

---

### 3. Install Sysmon

Installed Sysmon with the SwiftOnSecurity configuration:

```powershell
cd C:\Sysmon
.\Sysmon64.exe -accepteula -i config.xml
```

**Output:**

```
System Monitor v15.21 - System activity monitor
By Mark Russinovich and Thomas Garnier
Copyright (C) 2014-2026 Microsoft Corporation
Using libxml2. libxml2 is Copyright (C) 1998-2012 Daniel Veillard. All Rights Reserved.
Sysinternals - www.sysinternals.com

Loading configuration file with schema version 4.50
Sysmon schema version: 4.91
Configuration file validated.
Sysmon64 installed.
SysmonDrv installed.
Starting SysmonDrv.
SysmonDrv started.
Starting Sysmon64.
Sysmon64 started.
```

![Step 3: Install Sysmon](screenshots/3sysmon-installation.png)

---

### 4. Verify Installation

Verified Sysmon was running and logging events:

```powershell
# Check if Sysmon process is running
Get-Process Sysmon*

# Check event log for recent events
Get-WinEvent -LogName "Microsoft-Windows-Sysmon/Operational" -MaxEvents 5

# Check event log status
Get-WinEvent -ListLog *sysmon*
```

**Output:**

```
Handles  NPM(K)  PM(K)  WS(K)  CPU(s)  Id  SI ProcessName
-------  ------  -----  -----  ------  --  -- -----------
    291     528   8048  17112   0.25  18644  0 Sysmon64
```

```
   ProviderName: Microsoft-Windows-Sysmon

TimeCreated                      Id LevelDisplayName Message
-----------                      -- ---------------- -------
7/9/2026 2:05:25 PM               1 Information      Process Create:...
7/9/2026 2:05:25 PM               1 Information      Process Create:...
7/9/2026 2:05:25 PM               1 Information      Process Create:...
7/9/2026 2:05:25 PM               1 Information      Process Create:...
7/9/2026 2:05:25 PM               1 Information      Process Create:...

LogName            : Microsoft-Windows-Sysmon/Operational
MaximumSizeInBytes : 67108864
RecordCount        : 141
LogMode            : Circular
```

**Note:** The Sysmon service did not appear in `services.msc`. This is expected behavior in newer versions—Sysmon runs as a process instead. `Get-Process Sysmon*` is the reliable way to check status.

![Step 4: Verify Installation](screenshots/4sysmon-event-process-verification.png)

---

### 5. Test Event Generation

Generated test events to confirm Sysmon was logging properly:

```powershell
# Generate Process Creation event (Event ID 1)
Start-Process notepad.exe
Start-Sleep -Seconds 2
Stop-Process -Name notepad -ErrorAction SilentlyContinue

# Generate Network Connection event (Event ID 3)
Start-Process msedge.exe -ErrorAction SilentlyContinue
Start-Sleep -Seconds 3
Stop-Process -Name msedge -ErrorAction SilentlyContinue

# Check recent events
Get-WinEvent -LogName "Microsoft-Windows-Sysmon/Operational" -MaxEvents 5 | Format-Table TimeCreated, Id, LevelDisplayName -AutoSize
```

**Output:**

```
=== TESTING SYSMON ===
✅ Created Event ID 1 (Process Creation)
✅ Created Event ID 3 (Network Connection)

📊 Recent Sysmon Events:

TimeCreated              Id LevelDisplayName
-----------              -- ----------------
7/9/2026 2:29:38 PM       1 Information
7/9/2026 2:29:37 PM       1 Information
7/9/2026 2:29:35 PM       1 Information
7/9/2026 2:29:35 PM       3 Information
7/9/2026 2:29:33 PM       1 Information
```

**Results:** Event ID 1 (Process Creation) and Event ID 3 (Network Connection) were successfully captured.

![Step 5: Test Event Generation](screenshots/5testing.png)

---

## What It Detects

### Concrete Example: Process Creation (Event ID 1)

One of the most valuable events Sysmon captures is **process creation** with full command-line details.

**Event Captured:**

```
Event ID: 1 (Process Creation)
Process: splunk-optimize.exe
User: SYSTEM
Time: 2026-07-09 13:33:51
Image: C:\Program Files\Splunk\bin\splunk-optimize.exe
```

### MITRE ATT&CK Mapping

| MITRE Tactic | MITRE Technique | How Sysmon Detects It |
|--------------|-----------------|----------------------|
| **Execution** | T1059 - Command and Scripting Interpreter | Event ID 1 captures every process start with full command line |
| **Execution** | T1047 - Windows Management Instrumentation | WMI process creation events via Event ID 1 |
| **Persistence** | T1547 - Boot or Logon Autostart Execution | Registry changes via Event ID 12-14 |
| **Command & Control** | T1071 - Application Layer Protocol | Network connections via Event ID 3 |

### Why This Matters

Without Sysmon, Windows Event Log only shows basic process start events (Event ID 4688) with **limited command-line information**. Sysmon provides:

- ✅ **Full command-line arguments** (critical for detecting malicious commands)
- ✅ **Process hashes** (MD5, SHA1, SHA256) for file reputation checks
- ✅ **Parent process tracking** (identifying suspicious process chains)
- ✅ **Network connection details** (detecting C2 communication)

---

## Key Events Being Logged

| Event ID | Description | MITRE Mapping | Why It's Important |
|----------|-------------|---------------|-------------------|
| **1** | Process Creation | T1059 (Execution) | Detect malware execution |
| **3** | Network Connection | T1071 (C2) | Detect beaconing/C2 traffic |
| **11** | File Creation | T1105 (Ingress Transfer) | Detect malware drops |
| **12-14** | Registry Changes | T1547 (Persistence) | Detect persistence mechanisms |
| **22** | DNS Query | T1071 (C2) | Detect malicious domain lookups |

---

## Takeaways

### What I Learned

**1. Sysmon installation is straightforward once you know the steps.**

The hardest part was finding the correct filename after downloading (the ZIP files had `(1)` and `(2)` suffixes from multiple downloads).

**2. The SwiftOnSecurity config is perfect for beginners.**

It provides comprehensive logging without overwhelming you. It's well-documented and community-vetted.

**3. Service doesn't show in services.msc - and that's OK.**

In newer versions, Sysmon runs as a process, not a service. `Get-Process Sysmon*` is the reliable way to check status.

**4. Testing is essential.**

You can't assume it's working just because installation succeeded. Generating test events (Notepad, browser) confirms everything works.

### What Went Wrong

| Issue | How I Fixed It |
|-------|---------------|
| Downloaded multiple ZIP files (`Sysmon (1).zip`, `Sysmon (2).zip`) | Identified and used the original `Sysmon.zip` |
| Service not showing in `services.msc` | Learned this is expected behavior; used `Get-Process` instead |
| Initially no events showing | Generated test events to confirm logging |

### Key Insight

> **Sysmon provides visibility that Windows Event Log alone cannot.** The ability to see full command lines, process hashes, and network connections is essential for detecting modern threats. This setup is the foundation for building effective detection rules.

---

## Next Steps: Building a Detection

### Sigma Rule for Event ID 1 (Suspicious Process Creation)

A Sigma rule converts your logging into a detection. Here's a Sigma rule for detecting suspicious PowerShell execution:

```yaml
title: Suspicious PowerShell Execution with Encoded Command
id: 00000000-0000-0000-0000-000000000000
status: experimental
description: Detects PowerShell execution with encoded command (common malware technique)
references:
    - https://attack.mitre.org/techniques/T1059/001/
author: Coleman04-ai
date: 2026-07-09
logsource:
    product: windows
    service: sysmon
    definition: Sysmon Event ID 1 - Process Creation
detection:
    selection:
        EventID: 1
        Image|endswith: '\powershell.exe'
        CommandLine|contains: '-EncodedCommand'
    condition: selection
falsepositives:
    - Legitimate administrative scripts using encoded commands
level: medium
tags:
    - attack.execution
    - attack.t1059.001
```

### How This Sigma Rule Works

| Component | Explanation |
|-----------|-------------|
| **EventID: 1** | Looks for process creation events |
| **Image\|endswith: '\powershell.exe'** | Specifically targets PowerShell |
| **CommandLine\|contains: '-EncodedCommand'** | Flags base64-encoded commands (common obfuscation technique) |
| **Level: medium** | Alert severity level |

This detection would fire when Sysmon logs a PowerShell process with an encoded command, helping you identify potential malicious activity.

---

## Resources

- [Microsoft Sysmon Documentation](https://docs.microsoft.com/en-us/sysinternals/downloads/sysmon)
- [SwiftOnSecurity Config](https://github.com/SwiftOnSecurity/sysmon-config)
- [Sysmon Event ID Reference](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/)
- [Sigma Rules Guide](https://github.com/SigmaHQ/sigma)
- [MITRE ATT&CK Framework](https://attack.mitre.org/)

---

## Repository Structure

```
📁 Sysmon-Setup/
├── 📄 README.md (this file)
├── 📁 screenshots/
│   ├── 1sysmon-folder-creation.png
│   ├── 2unzip-sysmon-files.png
│   ├── 3sysmon-installation.png
│   ├── 4sysmon-event-process-verification.png
│   └── 5testing.png
└── 📁 sigma-rules/
    └── suspicious-powershell-encodedcommand.yml
```

---

**Documented on: 2026-07-09 | Sysmon v15.21**
