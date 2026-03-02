---
title: Home
layout: home
nav_order: 1
toc: true
---
# Hello, Konnichiwa

My name is Marius. This is my personal site,

 It is definitely a WIP with a lot of content not imported from my previous site, hopefully I am consistent in updating it. This is more so a hobby thing rather than a professional portfolio or some official writeup page so take it as it is. I mainly created this site to motivate and showcase my learning and career path.

Please feel free to check out:
- My [Projects]({{ "/projects/projects_base.html" | relative_url }})
- My [Writeups]({{ "/writeups/Writeups.html" | relative_url }})
- My [Study Notes]({{ "/study_notes/study_notes.html" | relative_url }})
- My [Blog]({{ "/blog/blogs.html" | relative_url }})
<p style="display: flex; gap: 1rem; flex-wrap: wrap;">
  <a href="https://github.com/mariimako">GitHub</a> 
  <a href="https://www.linkedin.com/in/mariiimako/">LinkedIn</a> 
  <a href="mailto:msheph02@student.ubc.ca">Mail</a> 
  <a href="{{ 'assets/resumes/Resume_CYB-17.pdf' | relative_url }}">Resume</a>
</p>

## About Me

My name is Marius. I'm a student at UBC. My passions is security, specifically in detection engineering, DFIR, and blue teaming. Even more if it mixes frontline research and raw operating system telemetry. I originally come from a development background. I was mainly interested in firmware and systems development but a few internships later I shifted to security in said systems.

A little more professionally:

My background combines deep operating systems expertise with applied security engineering. Through research and development into the mechanics of state-of-the-art detection systems and teaching advanced OS coursework, I have developed a strong theoretical foundation, which I have applied in corporate DFIR and SOC teams to design, evaluate, and triage high-signal detections grounded in real operational constraints. 

I am looking for a deeply technically oriented and high impact team to apply my combination of research and corporate blue team work to have a leave a large impact in the industry by improving detection, triage pipelines, and research.


### SOC Analyst Intern, TELUS

**Skills:** Elastic SIEM/Kibana, CrowdStrike Falcon, Google Chronicle (SecOps), Cloudflare Security, Obsidian Security, AWS GuardDuty, Azure Log Analytics, Juniper SRX, Cisco ISE, Sigma, YARA

At TELUS I worked in the internal corporate SOC as a SOC analyst, but the role was much more technical and coincided with IR, detection engineering and automation than “just triage.” I was on the front line for incidents across corporate endpoints, and I didn’t only close alerts — I worked directly with automation, detection, and IR teams to improve posture through tuning, rule logic, and signal fidelity.  
I spent a lot of time analyzing telemetry across EDR/SIEM sources, mapping activity to ATT&CK tactics/techniques, and surfacing patterns that made real improvements to coverage and response. I also acted as a flexible team member that worked on bridging between the three teams to improve noise, prioritize automation based on impact, and work closely with IR on malicious true positives remediation.

---

### Detection Research & Development Intern, NTT

**Skills:** System call anomaly detection, Linux security (seccomp), IDS, Python, ML/Deep learning

At NTT I did R&D on system‑call‑based intrusion detection for Linux container environments. That meant a deep dive into ML and DL anomaly detection papers, system call extraction tooling, and building/testing IDS prototypes using different syscall features and n‑gram/argument patterns.  
I also built a pentest‑style testbed to evaluate detectors in realistic attack scenarios (exploit chains, evasions, noisy benign workloads). This internship is what pushed me toward DFIR and OS security, but I wanted to bridge this experience with real alert triage as I felt the need to know how real alert triage connects to SOTA research  — which is why I wanted the frontline SOC experience next.

---

### UBC Subbots

At UBC Subbots, I work on firmware for the Steelhead AUV. I’ve written C++ drivers for IMUs into ROS2, built Gazebo sims for underwater testing, and stitched together CV + state estimation + PID nodes for autonomous gate navigation.  
We hit semis at RoboSub 2025 and had the best team performance in our history, despite being the bottom quartile out of other US and Canadian universities for funding. Not to through shade but some teams literally outsourced key parts to external companies!

---

### Technical Analyst Intern, Cyberium

At Cyberium I bounced between blue team ops and security tooling. I analyzed and reported on Nessus/nmap results, helped build security training tools, and worked on ISO 27001 / SOC 2 requirement rationalization using LLMs.  
Later I joined a startup team working on a mobile app that interfaced with a Bluetooth health device with Swift and later Flutter and Kotlin, along with bluetooth protocols.

---

### Computer Systems Teaching Assistant, UBC

I TA’d CPSC 213 (Computer Systems). I wrote new questions and autograding tests on RISC‑V assembly, C synchronization, CPU pipelining, and common C exploitation patterns. Everything was Python or C driven, containerized, and shipped through PrairieLearn to around 400 students.

I probably spent more time debugging weird edge‑cases in student submissions than writing the questions — but that’s also how I got really sharp on low‑level behaviour and correctness.

## Projects

Please see my various personal, professional, co-curricular and course [Projects]({{ "/projects" | relative_url }})