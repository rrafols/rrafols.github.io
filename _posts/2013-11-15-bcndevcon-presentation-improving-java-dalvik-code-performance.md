---
id: 307
title: 'BcnDevCon Presentation &#8211; Improving Java &#038; Dalvik Code Performance'
date: 2013-11-15T01:53:50+00:00
author: raimon
layout: default
guid: http://blog.rafols.org/?p=307
permalink: /index.php/2013/11/15/bcndevcon-presentation-improving-java-dalvik-code-performance/
shareaholic_disable_share_buttons:
  - "0"
shareaholic_disable_open_graph_tags:
  - "0"
restapi_import_id:
  - 599477c8431d4
original_post_id:
  - "307"
categories:
  - Android
  - Events
  - Java
tags:
  - bytecode
  - Dalvik
  - mobile
  - performance
  - Speaking
---
Last week I did a presentation at <a href="http://bcndevcon.org/" target="_blank" rel="noopener noreferrer">BcnDevCon</a> about improving Java Code Performance. The focus of the presentation was showing some examples of compiled java sources and evaluate the performance impact of different ways of looping, string concatenation or using Java 1.5 features as autoboxing or foreach loops. According to java the performance optimizations are always left to the JVM, but we will see we can do many things to improve our code performance by knowing how the compiler works.  
Some of these examples are also shown on Davlvik bytecode and performance tests are executed on both a computer and an android device. Even if Dalvik is register based and standard java bytecode is stack base in general terms what works for standard java can be also applied for Android apps.  
On future posts I will explain in more detail the performance graphs and other topics I didn&#8217;t had time to include in the presentation.  
[<img loading="lazy" src="http://blog.rafols.org/wp-content/uploads/Screen-Shot-2013-11-15-at-00.51.53-300x168.png" alt="Screen Shot 2013-11-15 at 00.51.53" width="300" height="168" class="alignnone size-medium wp-image-311" />  
Slides](http://ec2-18-232-250-173.compute-1.amazonaws.com/wp-content/uploads/2013/11/improving-java-code-performance-bcndevcon_final.pdf)
