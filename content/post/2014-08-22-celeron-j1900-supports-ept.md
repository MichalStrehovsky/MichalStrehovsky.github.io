---
title: Celeron J1900 supports EPT
date: 2014-08-22T05:58:48+00:00
tags:
  - misc
---
Quick post. I was shopping for a new home server and was eyeing the new Intel Bay Trail CPUs. The only thing that deterred me from buying it was [Intel ARK][1] insisting that Celeron J1900 doesn&#8217;t support the Extended Page Table (aka Nested Page Table) feature.

From the past I know that ARK is not exactly the best source of knowledge on EPT (go figure) so I kept searching, but I couldn&#8217;t find a confirmation. I found a mention of EPT in someone&#8217;s /proc/cpuinfo dump, so I bit the bullet and bought it.

```
C:\Users\Administrator>Coreinfo.exe -v

Coreinfo v3.31 - Dump information on system CPU and memory topology
Copyright (C) 2008-2014 Mark Russinovich
Sysinternals - www.sysinternals.com

Intel(R) Celeron(R) CPU  J1900  @ 1.99GHz
Intel64 Family 6 Model 55 Stepping 8, GenuineIntel
Microcode signature: 00000811
HYPERVISOR      -       Hypervisor is present
VMX             *       Supports Intel hardware-assisted virtualization
EPT             *       Supports Intel extended page tables (SLAT)
```

There you have it &#8211; both VT-x and EPT are supported by this CPU.

 [1]: http://ark.intel.com/