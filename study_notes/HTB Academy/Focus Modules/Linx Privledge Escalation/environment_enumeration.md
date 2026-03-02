---
title: 2. Environment Enumeration
parent: Linux Privilege Escalation
layout: default
---

## Environment Enumeration

LinPEAS, LinENUM

Say we gained access to Linux host by exploiting unrestricted file upload vulnerability during External Penetration Test

After establishing reverse shell, persistence, start gathering basics about system we are working with.

1. Basics:
whoami, id, hostname, ifconfig, ip a, sudo -l

2. Check user's PATH, where Linux system look severy time command is executed for any executables to match name of what we type, i.e., `id`
`echo $PATH` 
`env`

3. Check kernel version. 
`uname -a`
`cat /proc/version`
CPU type, version
`lscpu`

4, Login shells exist on server? Highlight tmux, Screen
`cat /etc/shells`

5. See defences in place
- exec shield
- iptables
- AppArmor `aa-status`
- SELinux
- Fail2ban
- Snort
- Uncomplicated Firewall
5. Drives and any shares `lsblk`, `lpstat`, printers attached to system. Sensitive info?
6. Check for mounted drives, unmounted drives. Can we cmount an unmounted drive and gain access to sensitive data? Find any types of credentials in `fstab` for mounted drives by grepping for common words such as passowrd, username, credential, in `/etc/fstab`
7. Check routing table by typing `route` or `netstart -rn.` What other networks are avaialble via which interface
8. In domain enviornment check `/etc/resolv.conf` if host is configured to use internal DNS, able to use as starting point to query the Active Directory environment
9. Check arp table to see what other hosts the target communicated with `arp -a`
10.  Existing Users `cat /etc/passwd`, `cat /etc/passwd | cut -f1 -d:`

| **Algorithm** | **Hash**       |
| ------------- | -------------- |
| Salted MD5    | `$1$`...       |
| SHA-256       | `$5$`...       |
| SHA-512       | `$6$`...       |
| BCrypt        | `$2a$`...      |
| Scrypt        | `$7$`...       |
| Argon2        | `$argon2i$`... |

11. Check which users have login shells. Once we see what shells are on the system, we can check each version for vulns. etc. shellshock `grep "sh$" /etc/passwd`
12. Check existing groups `cat /etc/group`, or `getent`
13. Mounted file systems `df -h`
14. Unmounted file systems `cat /etc/fstab | grep -v "#" | column -t`
When unmounted, no longer accessible by system. Such as when disk removed, no longer needed. another reason may be files, scripts, documents, and other important information must not be mounted and viewed by a standard user. Therefore, if we can extend our privilegdes to root user, we could mount and read these file systems ourselves. 
15. `find / -type f -name ".*" -exec ls -l {} \; 2>/dev/null | grep htb-student`
16. `find / -type d -name ".*" -ls 2>/dev/null`
Find hidden files and directories.
17. Temporary folders, /tmp /var/tmp. /var/tmp has longer retention time. All files and fata stored in /var/tmp are retianed for up to 30 days. All temporary files in /tmp are delted on restart. `ls -l /tmp /var/tmp /dev/shm`

