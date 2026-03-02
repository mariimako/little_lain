---
title: Analyzing Evil With Sysmon & Event Logs
parent: Windows Event Logs & Finding Evil
layout: default
---

# Analyzing Evil With Sysmon & Event Logs

## Sysmon Basics

As shown previously, event IDs serve common indicators of compromise. We saw `Event ID 4624` insight to logon events. 

Then, `4688` showed information about newly created processses, and so on.

Sysmon (System Monitor) provides additional event logging capabilities. It provides additional information: process creation, network connections, changes to file creation time, and more.
It is:
- A Windows service for monitoring system activity.
- A device driver that assists in capturing the system activity data.
- An event log to display captured activity data.

it is more useful for forensic analysis than event viewer as it provides the above additional information

https://github.com/SwiftOnSecurity/sysmon-config

````shell-session
sysmon.exe -c filename.xml
````

utilize a custom Sysmon config. Sysmon for Linux works similarily


## Detecting DLL Hijiacking with Sysmon
Event type 7, can relate to module load events. 
Change include xml to excldue, ensure nothing is excluded
```
sysmon.exe -c sysmonconfig-export.xml
```

Check Event Viewer, Applications and Services, Microsoft, Windows, Sysmon
Reveals presence of targeted event ID

Event ID 7 event log contains DLLs signing status. Process or image responsible for loading DLL, specific DLL that was loaded
Can see what it was signed by

DLL hijacks in windows:
https://www.wietzebeukema.nl/blog/hijacking-dlls-in-windows

calc.exe is vulnerable with various DLLs to be hijacked. WININET.dll is an example. We simply rename our payload to this.

Some IOCs in the lab:
1. calc.exe should not be in writeable directory, as it usually resides in System32
2. WININET.dll located in System32 should not be loaded outside of System32
3. WININET.dll is not signed by Microsoft


## Detecting Unmanaged Powershell /C-Sharp Injection
C# is managed language, it needs a runtime. CLR serves as the runtime. 
It is compiled into bytecode format that runtime processes and executes.

Process hacker:observe range of processes within environment. 

Checking Microsoft .NET Runtime, clr.dll and clrjit.dll are used when C# code is ran as part of runtime to execute bytecode. If loaded by processes that do not require them, potential execute assembly or PowerShell injection attack

## Detecting Credential Dumping

Mimikatz, often used for credential dumping. 
sekurlsa::logonpasswords, enables dumping of password hashes or plaintext passwords by accessing Local Secuirty Authority Subsystem Service. LSASS is responsible for managing user credentials and primary target from Mimikatz

we can check Sysmon event ID 10, represents ProcessAccess. Identify suspicous attempts to access LSASS

