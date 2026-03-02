---
title: 7. Internals Based Priviledge Escalation
parent: Linux Privilege Escalation
layout: default
---
## Kernel Exploits
`uname -a`
```shell-session
cat /etc/lsb-release 
```

Careful as some exploits break production

Demo with 4.4.0-116, Ubuntu 
https://vulners.com/zdt/1337DAY-ID-30003

`wget`. But this isn't feasible in a corporate environments, EDRs will easily detect wgets
```shell-session
gcc kernel_exploit.c -o kernel_exploit && chmod +x kernel_exploit
```
```shell-session
./kernel_exploit 
```
**DirtyCow**

- Allowed write to read only files, due to a race condition inside the kernel functions which handle copy on write (this is when kernel does not create the actual copy until it sbeing written into)

1. Create private copy of read only file
2. `write` to the private copy
3. COW takes place
4. `write` actually consists of two non-atomic actions:
	- locate physical address
	- write to physical address
5. Get right in the middle via another thread, and tell kernel to throw away our private copy using `madvise`. This results in writing into orignal read only file




## Shared Libraries

##  Shared Object Hijacking

## Python Library Hijacking

