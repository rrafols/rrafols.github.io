---
id: 254
title: 'What your mom didn&#039;t tell you about autoboxing&#8230;'
date: 2013-04-16T17:56:32+00:00
author: raimon
layout: default
guid: http://blog.rafols.org/?p=254
permalink: /index.php/2013/04/16/what-your-mom-didnt-tell-you-about-autoboxing/
shareaholic_disable_share_buttons:
  - "0"
shareaholic_disable_open_graph_tags:
  - "0"
restapi_import_id:
  - 599477c8431d4
original_post_id:
  - "254"
categories:
  - Java
tags:
  - mobile
---
[Autoboxing](http://en.wikipedia.org/wiki/Object_type_(object-oriented_programming)#Autoboxing) is a nice feature added to Java 5 to avoid writing _boxing_ code to add, for example, primitive data types to a Collection as you can only add the appropriate wrapper class (Integer for int, Long for long, &#8230;). I&#8217;ve recently seen the autoboxing feature being widely used all over the place so I decided to write this post.  
Oracle itself [warns](http://docs.oracle.com/javase/1.5.0/docs/guide/language/autoboxing.html) that &#8220;_It is plenty fast enough for occasional use, but it would be folly to use it in a performance critical inner loop._&#8221; and &#8220;_An Integer is not a substitute for an int; autoboxing and unboxing blur the distinction between primitive types and reference types, but they do not eliminate it._&#8220;. So that basically means do not use autoboxing on portions of code where performance counts.  
To illustrate the problem I&#8217;ve written an example:  
`<br />
public class Autoboxing {<br />
    // method using "accidentally" autoboxing<br />
    public void method1() {<br />
        long t = System.currentTimeMillis();<br />
        Long total = 0l;<br />
        for(Integer i = 0; i < 10000000; i++) {<br />
            total += i;<br />
        }<br />
        System.out.println(" total method1: "  + total + " time:" + (System.currentTimeMillis() - t));<br />
    }<br />
    // method not using autoboxing<br />
    public void method2() {<br />
        long t = System.currentTimeMillis();<br />
        long total = 0;<br />
        for(int i = 0; i < 10000000; i++) {<br />
            total += i;<br />
        }<br />
        System.out.println(" total method2: "  + total + " time:" + (System.currentTimeMillis() - t));<br />
    }<br />
    public static void main(String[] args) {<br />
        Autoboxing abox = new Autoboxing();<br />
        abox.method1();<br />
        abox.method2();<br />
    }<br />
}<br />
`  
The first method _method1_ uses some autoboxing (the _Integer_ and the _Long_). The second method _method2_ only uses primitive data types. Both methods iterate 1.000.000 times and when they finish they print the total and the amount of milliseconds since they started.  
The bytecode of the for loop of both methods already indicates, subtly, that _method2_ will be more efficient:  
Bytecode for loop method1:  
`<br />
 9: iconst_0<br />
10: invokestatic    #4;     //Method java/lang/Integer.valueOf:(I)Ljava/lang/Integer;<br />
13: astore           4<br />
15: aload            4<br />
17: invokevirtual   #5;     //Method java/lang/Integer.intValue:()I<br />
20: ldc             #6;     //int 10000000<br />
22: if_icmpge       65<br />
25: aload_3<br />
26: invokevirtual   #7;     //Method java/lang/Long.longValue:()J<br />
29: aload            4<br />
31: invokevirtual   #5;     //Method java/lang/Integer.intValue:()I<br />
34: i2l<br />
35: ladd<br />
36: invokestatic    #3;     //Method java/lang/Long.valueOf:(J)Ljava/lang/Long;<br />
39: astore_3<br />
40: aload            4<br />
42: astore           5<br />
44: aload            4<br />
46: invokevirtual   #5;     //Method java/lang/Integer.intValue:()I<br />
49: iconst_1<br />
50: iadd<br />
51: invokestatic    #4;     //Method java/lang/Integer.valueOf:(I)Ljava/lang/Integer;<br />
54: dup<br />
55: astore           4<br />
57: astore           6<br />
59: aload            5<br />
61: pop<br />
62: goto            15<br />
`  
Bytecode for loop method2:  
`<br />
 4: lconst_0<br />
 5: lstore_3<br />
 6: iconst_0<br />
 7: istore           5<br />
 9: iload            5<br />
11: ldc             #6;      //int 10000000<br />
13: if_icmpge       28<br />
16: lload_3<br />
17: iload            5<br />
19: i2l<br />
20: ladd<br />
21: lstore_3<br />
22: iinc          5, 1<br />
25: goto             9<br />
`  
In terms of timing, if we execute this class we get the following result:  
`<br />
rrafols$ java Autoboxing<br />
 total method1: 49999995000000 time:110<br />
 total method2: 49999995000000 time:9<br />
`  
So _method2_ takes only 8.18% of the total time of _method1_. Pretty big difference huh?  
What happens if we run the same without enabling the JIT (Just in Time compiler)? Most mobile devices have very limited or non-existent JIT.  
`<br />
rrafols$ java -Xint Autoboxing<br />
 total method1: 49999995000000 time:7387<br />
 total method2: 49999995000000 time:141<br />
`  
_Method2_ takes only 1.91% of the total time of _method1_. Not only that, imagine we&#8217;re processing something that we have to show back to the user, there is a huge impact between having the user to wait for ~8 seconds or to wait for 150ms.  
Please use common sense and be extra careful when using code you are actually not controlling or the compiler is generating for you.
