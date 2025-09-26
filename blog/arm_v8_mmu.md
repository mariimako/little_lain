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

use two registers, one for kernel, one for user space. 

Context switch, suer space TTBR is changed, but hopefully kernel is never written to after initialization and boot
### Page Sizes and Granules
MMUs on Intel processors have fixed page size, smalelst unit of memeory that can be mapped independently

ARMV*-A allows 3 idfferent granule sizes

Granykze size can be different for EL0 and EL1 mappings

Region size is determined by fields in TCR register

Multiple granule sies introduce consdierable flexbility

Barrrefish, awlays use granule size of 4KiB


grannule size not the same as superpages.

Superpages are feature of multi level page tables

![[Screenshot 2025-09-22 at 2.17.17 PM.png]]

#### Page Table Foramt
4levels, similar to x86

Final result of table walkc alled a block entyr and is allowed at any elvel of page table other than level 0

Page Table Entry

dirty bit, si absent.

Need to keep track of accesses to virtual pages using traps

prvouded functions to install and remvoe page mappings using CPU drivers code.
