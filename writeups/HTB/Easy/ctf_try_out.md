---
title: ctf_try_out
parent: HTB
---

Some easy hardware ones. Introduction to hardware styled CTFs which I haven't touched.

# 1
Very Easy: "backdoor" contains the 16bit signal to get into the device

# 2
Very Easy: open the PCB related ASCII files, Gerber (.gbr) files in some application. I used PCBWay. I then downloaded the copper layers and viewed the svg files. Two of the layers contained the flag. In a typical PCB Gerber files the following exist:
- **F = Front / B = Back**
- **Cu = Copper layers** (top, bottom, and inner copper planes/traces)
- **Mask = Soldermask** (defines where soldermask is applied)
- **Paste = Stencil openings** (for solder paste application on SMD pads)
- **Silkscreen = Printed text/graphics on the board**
- **Fab = Fabrication drawing** (mechanical info, assembly markings)
- **Edge_Cuts = Board outline**
- **In1_Cu, In2_Cu = Internal copper layers** (for multilayer boards)
The bottom copper layer and the internal copper layer contained the flag.

# 3
Given a .sal file. This is a Saleae Logic Analyzer capture file (from the Saleae Logic/Logic Pro devices).  They are portable USB logic analyzers used for debugging and analyzing digital and analog signals in embedded systems. 

Me with no hardware experience be like:
**What even are logic analyzers**
electronic instrument for monitoring and analyzing multiple digital signals within a digital system simultaneously, unlike an oscilloscope which focuses on analog signals

- Analog signals are smooth, continuous waves representing a vast range of values, like a dimmer switch or a phonograph record, and are ideal for capturing real-world nuances but are prone to noise
- Digital signals, conversely, use discrete binary values (0s and 1s), creating square-wave patterns and offering greater accuracy, noise resistance, easier storage, and better long-distance transmission, though they simplify complex information

Tx stands for transmission, and Rx represents reception Dx for diagnosis


I see signal in RX. I can apply an analyzer. I used async serial (UART) analyzer. A bunch of hexits appeared. 
I set the baud rate to default, hex streams show error. 
Set to 115200 bps, apparently it is common for Embedded boards / MCU.

So the analyzer was connected to the digital signals sent via UART in some boot process. I set the Baud rate to for a typical  embedded board/ MCU. Converted the signal to ASCII, and it showed a madeup boot process, including the flag.