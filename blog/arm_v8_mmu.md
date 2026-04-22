---
title: The ARMv8 MMU Overview
layout: default   
date: 2025-09-09
parent: Blog
---

# The ARMv8 MMU

Short look in the ARMv8-A MMU

Good to know when discussing virtual memory and paging systems to actually have a concrete example.

Notes are moreso focused on the address space layout, page sizes, page table format.

### Address Space Layout

ARMv8-A divides virtual address space into two parts, each of which is translated using a different page table

TTBR0_EL1 register holds address of base of page table

EL1 beacuse register can only be accessed at execution level 1

EL0 application cannot access, as it would allow unprivledged user programs 

Configure TCR_EL1 to use full 48 bit virtual address space

![[Screenshot 2025-09-22 at 2.14.23 PM.png]]

Use two registers, one for kernel, one for user space. 

Context switch, suer space TTBR is changed, but hopefully kernel is never written to after initialization and boot
### Page Sizes and Granules
MMUs on Intel processors have fixed page size, smalelst unit of memeory that can be mapped independently

ARMV*-A allows 3 different granule sizes

Granule size can be different for EL0 and EL1 mappings

Region size is determined by fields in TCR register

Multiple granule sies introduce considerable flexbility

Granule size not the same as super pages.

Super pages are a feature of multi level page tables

![[Screenshot 2025-09-22 at 2.17.17 PM.png]]

#### Page Table Format
4 levels, similar to x86

Final result of table walk alled a block entry and is allowed at any elvel of page table other than level 0

Page Table Entry

dirty bit is absent.

Need to keep track of accesses to virtual pages using traps

Provided  functions to install and remove page mappings using CPU drivers code.
