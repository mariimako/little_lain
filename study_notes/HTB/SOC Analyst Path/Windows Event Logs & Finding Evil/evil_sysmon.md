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


## Detecting Unmanaged Powershell /C-Sharp Injection

## Detecting Credential Dumping



