---
title: 1. Enumeration
parent: Linux Privilege Escalation
layout: default
---

## Enumeration
[LinEnum](https://github.com/rebootuser/LinEnum)

### Basics:
- OS Version
- Kernel Version (targetting specific kernel version)
- Running services (Nagios, Exim, Smaba, ProFTP) `ps aux | grep root`
- Installed Packages and Versions: ex. Screen (similar to tmux), version 4.05 suffers from privledge escalation vulnerability
- Logged in users: Knowing which other users are logged into system and what they are

`ps au` List current terminal attached processes

**User Home Directories** 
Are other user's home directories acccessible? SSH Keys? 
`ls /home`
Check individual user directories and see if files such as .bash_history files are readable

**SSH Directory Contents**
If i find SSH key for current user, can be used to open SSH session on host, and gain stable and interactive session. 
Check ARP cache to see what other hosts are being accessed and cross reference these gainst any useable SSH private keys

ARP cache is temporary storage table on device that maps IP addresses to MAC addresses for devices in same local network

**Bash History**

Bash history may be passing passwords as an argument on command line, working with git repo, setting up cron jobs
`history`

**Sudo - List User's Privileges**

Can user run any commands as another user or as root

If no credentials, you wont be able to, often sudoer entries include NOPASSWD
Not uncommon to gain access as a user with full sudo privileges 

`sudo su` might immediately give root session

`sudo -l`

**Configuration Files**
Configuration files can hold a wealth of information. 
Worth searching through all files that end in extensions such as .conf and .config


**Readable Shadow File**
shadow files is readable, gather password hashes for all users.
Hashes can be subjected for offline brute force to recover cleartext

**Password hashes in /etc/passwd**
Occasionally, password hashes will be seen directly in /etc/passwd 
File is readable by all users, and as with hashes in shadow file, can be subjected to offline password cracking attack.
Uncommon but sometimes be seen on embedded devices and routers

`cat /etc/passwd`

**Cron Jobs**
Combined with misconfigs such as relative paths or weak permissions, they can leverage to escalate priviledges when scheduled cron job runs

`ks -la /etc/cron.daily/`

**File Systems & Additional Drives**
Unmounted file systems, additional drives
If you discover and can mount an additional drive or unmounted file system, may find sensitive files, passwords, backups
`lsblk`

**SETUID and SETGID Permissions**

Binaries are set with these permissions to allow suer to run command as root, without having to grant root-level access to user

Many binaries contain functionality that can be exploited to get a root shell


**Writeable Directories**
Important to discover which directories are writeable, if downloading tools into systems. Find cron job places files, Provides idea of how often cron jobs runs and could be sued to elevate privledges, if script that runs cron jobs runs is also writeable

`find / -path /proc -prune -o -type d -perm -o+w 2>/dev/null `

**Writable Files**
Are any scripts or configuration files world writable?
Any scripts run as root using cron jobs can be modified to append a command

`find / -path /proc -prune -o -type f -perm -o+w 2>/dev/null`


[[Linux_Privilege_Escalation_Module_Cheat_Sheet.pdf]]




