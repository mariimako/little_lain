---
title: Windows Kernel Callbacks and ETW
layout: default
parent: OS / Systems
---

# Windows Kernel Callbacks 

General way for drivers to request and provide notificaiton when certain conditions are satisfied. 

Drive can register a callback in its code for any supported action. Will recieve pre or post noitifcation when targeted action is performed.

Callbacks will not eprform any modification to underlying Windows Kernel thus, avoiding any BSOD due to KPP. (kernel patch protection)

https://otterhacker.github.io/Malware/Kernel%20callback.html