---
title: The ARMv8 MMU Overview
layout: default   
date: 2025-09-09
parent: Blog
---
# The ARMv8 MMU

A short look at the ARMv8-A MMU.

Good to know when discussing virtual memory and paging systems to actually have a concrete example.

Notes are focused mostly on the address space layout, page sizes, and page table format.

### Address Space Layout

ARMv8-A divides virtual address space into two parts, each of which is translated using a different page table.

The TTBR0_EL1 register holds the address of the base of the page table.

EL1 because the register can only be accessed at Exception Level 1.

EL0 applications cannot access it, as that would allow unprivileged user programs to tamper with kernel page tables.

Configure TCR_EL1 to use the full 48-bit virtual address space:

![ARMv8 address space layout]({{ "/assets/media/Screenshot%202025-09-22%20at%202.14.23%20PM.png" | relative_url }})

Two registers are used, one for the kernel and one for user space.

On context switch the user space TTBR is changed, but hopefully the kernel one is never written to after initialization and boot.

### Page Sizes and Granules
MMUs on Intel processors have a fixed page size, the smallest unit of memory that can be mapped independently.

ARMv8-A allows 3 different granule sizes.

Granule size can be different for EL0 and EL1 mappings.

Region size is determined by fields in the TCR register.

Multiple granule sizes introduce considerable flexibility.

Granule size is not the same as super pages.

Super pages are a feature of multi-level page tables.

![Granule sizes and super pages]({{ "/assets/media/Screenshot%202025-09-22%20at%202.17.17%20PM.png" | relative_url }})

#### Page Table Format
4 levels, similar to x86.

The final result of a table walk is called a block entry and is allowed at any level of the page table other than level 0.

Page Table Entry:

The dirty bit is absent.

The kernel needs to keep track of accesses to virtual pages using traps.

Functions are provided to install and remove page mappings using CPU driver code.
