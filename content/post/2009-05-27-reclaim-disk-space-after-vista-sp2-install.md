---
title: Reclaim disk space after SP2 install
date: 2009-05-27T08:43:18+00:00
tags:
  - tricks
---
Quick post. If you already installed Vista SP2, you probably noticed the decrease in free disk space on your system drive. The reason for that is that the SP2 installer stores all data that is neccessary for uninstalling it.

If you don&#8217;t plan on removing SP2, you can reclaim the disk space pretty easily: start an elevated command prompt and type `compcln`. Say yes to the question it asks you, wait a few seconds and enjoy your reclaimed disk space.