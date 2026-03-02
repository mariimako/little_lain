---
title: SIEM Use Case Development
parent: Security Monitoring & SIEM Fundamentals
layout: default
---
# SIEM Use Case Development

Utilizing SIEM use cases is a fundamental aspect of crafting a robust cybersecurity strategy, as they enable the effective identification and detection of potential security incidents

When developing SIEM Use Case, consider
1. Requirements: comprehending the use case of which and alert or notification is needed
2. Data points: identify all data points within network where a user account can be used to log in  
3. Log validation: verify logs that they contain the right information
4. Design and Implementation: Consider three primary parameters: Condition, Aggregation, and Priority, and determine alert condition
5. Documentation: Standard Operating Procedures (SOP) detail when working on alerts
6. Onboarding
7. Periodic Update/Fine-tuning

Furthermore, establish the Time to Detection (TTD) and Time to Response (TTR) for the alert to assess the SIEM's effectiveness and analysts' performance.
Map the alert to the kill chain or MITRE framework.
Develop incident response plan.

## Examples

### MSBuild Engine Started by Office Application
MSBuild assembles applications according to XML input file. Attackers exploit MSBuilds ability to include malicous source code within its configuration or project file.
Unusual MSBuild calls should be easily identificable.
We create a detection use case in our SIEM regarding MSBuild initiated by Excel or Word, as this behavior could indicate a malicious script payload execution.

Referred to LoLBins (living off the binary) 

**LolBins Recap**
	DLL hijacking, hiding payloads, rocess dumping, downloading files, bypassing UAC keylogging, code compiling, log evasion, code execution, and persistence. 
	To be considered a LoL, the binary, library, or script in question must be on the system by default or put on the system by the user.
	One notable is the [Astaroth fileless trojan attack](https://www.bleepingcomputer.com/news/security/microsoft-discovers-fileless-astaroth-trojan-campaign/), which has been spreading since early 2018. It targets users across Europe and Brazil, can intercept OS calls, and monitors clip bards to steal data. It also features keylogging capabilities. As for its LOL component, it abuses the WMI command line to download and install malware without arousing suspicion. The stolen credentials allow attackers to move across networks to conduct other, more damaging attacks unnoticed.

Map to MITRE: Defense Evasion ([TA0005](https://attack.mitre.org/tactics/TA0005/)) tactic, Trusted Developer Utilities Proxy Execution ([T1127](https://attack.mitre.org/techniques/T1127/)) technique, and the Trusted Developer Utilities Proxy Execution: MSBuild ([T1127.001](https://attack.mitre.org/techniques/T1127/001/)) sub-technique



To define TTD and TTR, we need to focus on the rule's execution interval and the data ingestion pipeline discussed earlier. For this example, we set the rule to run every five minutes, monitoring all incoming logs.

### MSBuild Making Network Connections

Would be lower severity, as it is possible that MSBuild connects to an outbound IP for updates. Pulling off this attack would involve MsBuild binary on the endpoint, which falls under the Execution (TA0002) 