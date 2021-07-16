---
id: 52
title: '&#039;Realtime&#039; raytracing on j2me'
date: 2009-06-16T00:40:41+00:00
author: raimon
layout: default
guid: http://blog.rafols.org/?p=52
permalink: /index.php/2009/06/16/realtime-raytracing-on-j2me/
restapi_import_id:
  - 599477c8431d4
original_post_id:
  - "52"
categories:
  - j2me
---
I have been playing a bit this afternoon with my old raytracer and decided to wrote a a small implementation in j2me.. yes.. it sounds completely useless but it was quite fun to remember the old days.  
It&#8217;s a very basic raytracer, it only supports spheres and planes and then some hard shadows and reflections. I haven&#8217;t done any optimitzation, for example checks if the camera ray intersects all objects for every single pixel and, obviously, it runs pretty slow on the real device (even on the emulator..)  
Just three screenshots of the &#8216;realtime&#8217; (ehem..) raytracing:  
![rt - screenshot 1](http://labs.rafols.org/rt00.png)![rt - screenshot 2](http://labs.rafols.org/rt01.png)  
![rt - screenshot 3](http://labs.rafols.org/rt02.png)  
![](http://labs.rafols.org/img.php?id=rt0-post)
