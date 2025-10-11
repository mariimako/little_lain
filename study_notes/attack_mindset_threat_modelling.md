---
title: Threat Modelling, Incident Response
layout: default           
nav_order: 1
parent: Security+ Review    
---

 Detection Logging, Large Scale Defense, Network and Host Security, Real-World Cases & Detection Logic


Random Read Links:
https://cloud.google.com/beyondcorp
https://www.crowdstrike.com/blog/category/threat-intel/
https://learn.microsoft.com/en-us/training/courses/sc-100t00
https://attack.mitre.org/
Linux/Windows persistence methods (GTFOBins, MITRE ATT&CK T1547–T1053)

https://cheatsheetseries.owasp.org/
https://owasp.org/API-Security/
https://attack.mitre.org/

https://www.cisa.gov/resources-tools/resources/incident-response-playbook

https://github.com/SigmaHQ/sigma
https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-61r2.pdf

https://docs.aws.amazon.com/whitepapers/latest/aws-security-best-practices/aws-security-best-practices.pdf
https://cloud.google.com/security/incident-response


**Rooms:** “Network Fundamentals”, “Linux Privilege Escalation”

Tlemetry refers to the **automatic collection and transmission of data from devices, applications, or systems to a central location** for monitoring, analysis, or alerting.

## Cheat Sheet
---

## **Attacker Mindset & Threat Modeling**

**Goal:** Think like an attacker — persistence, lateral movement, data exfiltration, reconnaissance.

**Common Attack Techniques:**

- **Persistence:** backdoors, cron/systemd tasks, registry Run keys, bootkits/UEFI malware, malicious containers, cloud functions.
- **Privilege Escalation:** exploiting misconfigured services, weak passwords, kernel exploits.
- **Lateral Movement:** SSH/Telnet reuse, SMB/remote desktop, vulnerable IoT devices, ARP poisoning.
- **Exfiltration:** encrypted channels (HTTPS, DNS tunneling), cloud storage abuse, insider threats.
    
**Threat Modeling Framework: STRIDE**

|Threat|Description|
|---|---|
|**S**poofing|Pretending to be someone else (credentials, tokens)|
|**T**ampering|Modifying data or system integrity|
|**R**epudiation|Denying actions without traceability|
|**I**nformation Disclosure|Data leaks, exposed secrets|
|**D**enial of Service|System/resource unavailability|
|**E**levation of Privilege|Gaining higher access than authorized|

**Mitigation Principles:**
- Zero Trust: always verify identity, segment networks, least privilege.
- MFA, credential hygiene, secrets rotation.
- Monitoring, alerting, and telemetry for abnormal activity.

---

## ** IR Lifecycle**

1. **Preparation** – policies, backups, playbooks, logging, monitoring.
2. **Identification** – detect anomalies via SIEM/telemetry/logs.
3. **Containment** – isolate affected systems, block malicious IPs, preserve volatile evidence.
4. **Eradication** – remove malware/backdoors, patch vulnerabilities.
5. **Recovery** – restore from clean backups, validate systems, bring services online safely.
6. **Post-Incident / Lessons Learned** – root cause analysis, update playbooks, communicate findings.
    

**Telemetry in IR:** Continuous collection of logs, metrics, and events to detect abnormal behavior.
- **Examples:** Process creation, network connections, file access, login attempts, API calls.
    

**Quick Tips:**
- Preserve evidence before eradication.
- Use MITRE ATT&CK for mapping adversary behavior.
- Communicate clearly under management pressure — phased recovery if needed.

---

## **3️ Large-Scale System Defense**

**Logging & Monitoring:**

- Collect structured logs + traces + metrics (OpenTelemetry).
- Central ingestion → enrichment → SIEM/SOAR → dashboards/alerts.
- Monitor for abnormal authentication, process behavior, and network traffic.
    

**Cloud / Microservices / APIs:**

- Secure APIs: OAuth2/mTLS auth, rate-limiting, quotas, WAF, field-level validation.
- Secure architecture: microsegmentation, zero trust network, telemetry & alerting.
- High-scale detection: aggregate events, prioritize alerts, anomaly detection, automated responses.
    

**Authentication / Traffic Monitoring:**

- Signals for attack vs legit: IP reputation, geo diversity, proxy/VPN use, timing patterns, success/failure ratio, user-agent analysis.
    
**Common IoT & Network Risks:**

- Default/weak creds, open debug ports, insecure firmware updates.
- DNS spoofing, MITM, lateral movement, SYN floods.
- Defenses: device identity, signed firmware, OTA security, network segmentation, monitoring.
    
---

## **4️ Key Frameworks & References**

**MITRE ATT&CK Highlights**

- **Initial Access:** Phishing, exploitation of public-facing apps, IoT vulnerabilities.
- **Persistence:** Bootkits, services, cron jobs, scheduled tasks.
- **Privilege Escalation:** Exploiting OS/service vulnerabilities.
- **Lateral Movement:** RDP/SSH reuse, SMB, remote admin tools.
- **Exfiltration:** HTTPS, DNS, cloud storage abuse, remote transfer.
    

**Zero Trust Principles**

1. Verify explicitly (identity + device + context). 
2. Least privilege access
3. Assume breach, segment networks.
4. Continuous monitoring and telemetry.
    

**OWASP Top 10 / API Risks**

- Broken Object Level Access
- Broken Authentication
- Excessive Data Exposure
- Lack of Rate Limiting / Throttling
- Injection & RCE
- Security Misconfiguration
    

**Quick Cheat: Logs & Telemetry to Know**

- Endpoint: process start/stop, file access, login events
- Network: connections, failed auth, unusual ports
- Cloud: API calls, object access, policy changes
- Alerts: anomalies, threshold triggers, behavioral deviations
    
---

## **5️⃣ Rapid Interview Prep Tips**
- **Scenario-Based:** “If a server shows X, I’d check Y → contain → preserve evidence → remediate → report.”
- **Use MITRE & STRIDE terminology** for precision
- **Mention automation & scale** for large systems (SIEM, SOAR, cloud logs).
- **Highlight trade-offs:** speed vs evidence preservation, user experience vs security
- **Keep Zero Trust in mind** when discussing network design or API protection.
