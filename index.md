---
title: Home
layout: home
nav_order: 1
toc: true
---
# Hello

My name is Marius. 

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

I'm a student at UBC. My passion is systems and low-level engineering and defensive security, specifically OS-level telemetry and detection.

I completed three internships as part of my co-op program at UBC, a program that intertwines 16 months of internship-based work experience with the four-year computer science bachelor's program, for a total of five years.



My internship experience is as follows:
Cyberium is a Vancouver-based cybersecurity consulting firm. I worked there both as a developer for some new projects and tooling, and doing auditing work for local clients.

At NTT, I worked as a research assistant in NTT R&D's security research team. This team focused on eBPF based fuzzing, and system call based anomaly intrusion detection systems for multi-tenant container environments. I worked in the latter. 

I focused on exploring various characterizations of system calls to minimize FPR and compute cost. This involved analysis of system call traces in exploits such as DirtyCOW, DirtyPipe, and many other CVEs, and creating various models that characterize them differently: using the sequence (n-grams), frequency, arguments, entropy of arguments, and mapping as graphs.

These were all unsupervised models. The premise of the research is to develop a detection engine that does not require prior knowledge of attacks, but relies purely on the normal behaviour of the relatively constant container environment.

At TELUS, I worked in incident response. While NTT was research focused and deeply technical, I wanted to explore what detections and triage look like at the SOC and incident response level (and also learn from closed source EDR engines!). My idea was that I needed a broader understanding of how detection ties into real security operations.

What I enjoyed most was applying my previous experience to analyzing tuning opportunities for EDRs: while my research focused on false positives, detection accuracy, and the compute cost of a new specialized detection engine for a specific environment, the detection engineering of generalized EDRs has different challenges and considerations: implementation difficulty, ROI in terms of false positives, and worrying about introducing security gaps due to the implementation of naive rules.




