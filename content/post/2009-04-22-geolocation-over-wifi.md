---
title: Geolocation over WiFi
date: 2009-04-22T09:36:40+00:00
tags:
  - misc
---
I was playing with my iPod Touch today, skimming throught Google maps, when I accidentaly hit the _Show current location_ button. I knew that my iPod doesn&#8217;t have a GPS so I never really expected it to work. I was really stunned when after 3 seconds it showed me a map of Uppsala, with a circle around a place that actually was my location.

My first reaction was: _what_?! And I immediately went to check if my iPod really does not have a GPS. It didn&#8217;t. So I started googling and after a few links I finally knew what was going on: iPod uses WiFi to provide geolocation.

Still, this was not an answer for me. WiFi access points usually don&#8217;t report their own GPS coordinates. That would make them too expensive with very little benefit to the user. Something different had to be behind this.

A few more minutes after that I found the answer: there is a company that runs cars with a GPS and WiFi on board, that scan MAC addresses of WiFi access points and stores them together with their positions. When a device like my iPod Touch needs to find out it&#8217;s location, it sends a list of all access points around it to a web service of this company. If you are lucky, the MAC addresses are already in their database and using triangulation, they can find your location with precision of ~100 meters.

That&#8217;s pretty impressive for a device with no real geolocation hardware.