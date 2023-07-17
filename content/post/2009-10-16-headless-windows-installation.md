---
title: Headless Windows installation
date: 2009-10-16T09:35:12+00:00
tags:
  - tricks
---
I recently faced a problem with installing Windows on a headless (no monitor, no keyboard) computer. The computer in question was HP MediaSmart Server LX195 &#8211; a nice and cheap piece of hardware. The server comes with Windows Home Server preinstalled &#8211; a very neat consumer-grade server operating system. But being a geek &#8211; the operating system was not for me. I desperately wanted the new features and feel that came with Windows Vista. So I chose the path of installing something else on this server.

# Installing Windows Server 2008 R2

My reinstallation adventures started with Windows Server 2008 R2. It didn&#8217;t take long to figure out that the server won&#8217;t boot from USB and having no DVD drive, the only option was a disk-based install. The other problem was the absence of a keyboard and a monitor. The only option was some kind of unattended installation. This is quite simple to do with Windows:

  * On the Windows Server 2008 R2 installation DVD you&#8217;ll find a headlessunattend.xml file (in Support\Samples). This is an unattended installation script that will automate the installation process. Copy this file somewhere and edit it to suit your needs. (I modified it to format partition 1 of disk 0, mark it as active and install Windows on it.) You can also use the Windows AIK (Automated Installation Kit) if you prefer GUIs and want to avoid errors.
  * Rename the file to autounattend.xml and copy it to the root of a USB flash drive.
  * Copy the contents of the Windows installation DVD to the root of the second partition on home server&#8217;s disk (D:).
  * Use bootsect.exe to modify the boot sector of the second partition on the home server (we need a boot sector that will fetch a BOOTMGR style loader (NT 6.0 style) instead of the NTLDR style (pre-NT 6). Bootsect.exe is a part of Windows AIK but maybe you can find it somewhere else too.
  * Open disk management tool on the home server and mark the second partition as active.
  * Plug in the flash drive with the XML file and restart home server

If everything went well, the new boot sector on the now active partition will fetch the Windows installation on the second partition as if you booted from a DVD. The XML file on the thumb drive will be automatically detected and processed by the installation. After ~30 minutes, you will have a working Windows 2008 R2 installation on the home server.

I really recommend to try this out in a virtual environment first. If you use Windows Virtual PC, you can use a virtual floppy drive instead of a flash drive. Use WinImage to to create a floppy image.

# Installing Windows Server 2008

The problem with Server 2008 R2 is that it&#8217;s 64-bit only. I don&#8217;t have much RAM in my home server so it didn&#8217;t really make much sense. So I decided to roll back to Windows Server 2008. I tried the same approach as above &#8211; and failed. The problem is that if you try installing Server 2008 from a hard drive, it will hang asking you for a driver. I never figured out what driver it wants. So I had to try another unattended installation option: sysprep.

  * I took out the hard drive from my home server, connected it to a standard computer, popped in a Server 2008 installation DVD and installed it.
  * I downloaded the drivers for my home server chipset (Intel 945G), storage (Intel Matrix Storage driver floppy) and network card (Marvell Yukon 88E8071). I used pnputil.exe (part of Windows installation) to integrate these drivers in the system driver storage (pnputil -a path\to\driver.inf).
  * I enabled remote desktop.
  * I created an unattended XML file to automate the specialization and OOBE phases that would run after sysprep and require user interaction (setting computer name, passwords,&#8230;). Use internets to find out what you need to automate.
  * I executed c:\windows\system32\sysprep\sysprep.exe /generalize /oobe /shutdown /unattend:full\path\to\xml. This command removed all the drivers for the machine I used to install Windows Server with and set the system to a hardware-neutral state. On the next boot a full hardware detection and specialization will take place.
  * Then I popped the hard drive back to my home server, powered it on and after a few minutes I had a Server 2008 installation on it that I could use remotely.