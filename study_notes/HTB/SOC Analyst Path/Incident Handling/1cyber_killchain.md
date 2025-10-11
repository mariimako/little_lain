---
title: 1. Cyber Kill Chain, Incident Handling Process
layout: default   
parent: Incident Handling
---


Cyber Kill Chain consists of 7 different stages, attackers process

![[Screenshot 2025-05-07 at 4.08.38 PM.png]]

Recon is OSINT, choose target
Weaponize is malware to initial access is developed and embedded into payload, remote access
Delivery is the actual delivery of payload to victim, phishing email, etc, 
Exploitation is the momemnt where payload is triggered
Installation: initial stage is executed and running
- droppers: small piece of code that is designed to install malware on system deliver malware
- Backdoors: malware designed to rpvodie attacker with ongoing access to compromised system
- Rootkits: malware that is designed to hide its presence on a compromised system, rootkits are often used in installation stage to evade detection
Command and Control is establishing remote access capability
ActionL exfiltering condiential data, etc.

# Inclident Handling Process 
 By NIST![[Screenshot 2025-05-07 at 4.16.17 PM.png]]
TIme is spent mostly on first two, prep and detect analysis

Always have resources operaitng on first two stages so that there is no disruption of preparation and detection capabailities
- again consider it as cyclic process, not always the case you want to immediately take action

Investigating and Recovering:
- investigation: initial patient zero victim and create incident timeline
- Determine what tools and malware the adversary used
- Document compromised systems and what adversay has done
