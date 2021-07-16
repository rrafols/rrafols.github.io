---
id: 209
title: My js1k entry
date: 2013-04-01T00:18:25+00:00
author: raimon
layout: default
guid: http://blog.rafols.org/?p=209
permalink: /index.php/2013/04/01/my-js1k-entry/
shareaholic_disable_share_buttons:
  - "0"
shareaholic_disable_open_graph_tags:
  - "0"
restapi_import_id:
  - 599477c8431d4
original_post_id:
  - "209"
categories:
  - Javascript
tags:
  - HTML5
---
I was thinking to take part into the javascript 1k competition (<a href="http://js1k.com/2013-spring/" target="_blank" rel="noopener noreferrer">here</a>) but as I didn&#8217;t had too much free time and never really did that much in javascript targeting at 1k, I discarded the idea. Until I got a very ambitious idea the last day before the deadline.. how appropriate.. So I started trying few things and, to be honest, they didn&#8217;t work as I expected.. so I saved my ambitious idea for the next competition 😉  
At the end, as I was already programming in Javascript I did a waving catalan flag in roughly 1k without paying attention to optimization. Pretty simple but enough for being my first time on a 1k javascript competition. Anyway, after doing some basic optimizations I still got a couple hundred bytes left so I&#8217;ve decided to add some extras. Check the demo for more details 🙂  
Click the image below for the demo:  
<a href="http://js1k.com/2013-spring/demo/1537" target="_blank" rel="noopener noreferrer"><img loading="lazy" src="http://blog.rafols.org/wp-content/uploads/Screen-Shot-2013-04-01-at-1.11.46-AM-300x161.png" alt="Screen Shot 2013-04-01 at 1.11.46 AM" width="300" height="161" class="alignnone size-medium wp-image-218" /></a>  
Just in case anyone is interested here is the fully _uncompressed_ code with few comments.

<pre>k = 0
w=window
h=w.innerHeight
w=w.innerWidth
h2=h/2
W=H=40
setInterval(function(){
    // clear screen
    c.width=w
    c.height=h
    o = 0;
    p = []
    //Generate flag points. Before size-optimizing the code they were generated only once and stored in an external array.
    //To save one loop from the code they're generated every single frame.. oh well..
    for(j = 0; j &lt; H; j++) {
        for(i = 0; i &lt; W; i++) {
            x = 1.2 * i * w / W +8 * Math.sin((j + k) * 0.15)
            y = 1.2 * j * h / H + 8 * Math.sin((i + k) * 0.3)
            z = 512 + 32*Math.sin((i+j+k) * 0.2)
            p[o] = (512 * x) / z
            p[o+1] = (512 * y) / z
            p[o+2] = z;
            o+=3
        }
    }
    // draw the flag
    o = 0
    for(j = 0; j &lt; H-4; j++) {
        for(i = 0; i &lt; W-1; i++) {
            // compute cross product to compute light factor
            // this can be probably done in waaaay less bytes
            l = []
            l[0] = p[o+3] - p[o]
            l[1] = p[o+4] - p[o+1]
            l[2] = p[o+5] - p[o+2]
            m = []
            m[0] = p[o+W*3] - p[o]
            m[1] = p[o+W*3+1] - p[o+1]
            m[2] = p[o+W*3+2] - p[o+2]
            n =[]
            n[0] = l[1]*m[2] - l[2]*m[1]
            n[1] = l[2]*m[0] - l[0]*m[2]
            n[2] = l[0]*m[1] - l[1]*m[0]
            n = n[2] / Math.sqrt(n[0] * n[0] + n[1] * n[1] + n[2] * n[2]);
            // square it to make results more dramatic
            n *= n;
            // choose flag color (red/yellow) to render the different color stripes depending on the vertical coordinate
            g = 221
            b = 9
            if(((j / 4) % 2|0) == 0){
                r = 253
            } else {
                r = g
                g = b
            }
            // apply the lightning factor and convert it back to integer using "|0"
            r=r*n|0
            g=g*n|0
            b=b*n|0
            a.fillStyle="rgba("+r+","+g+","+b+",1)"
            a.strokeStyle=a.fillStyle
            // actual render of the quad. Fill & stroke to polish (antialiasing, subpixel,..)
            a.beginPath()
            a.lineTo(p[o], p[o + 1])
            a.lineTo(p[o+3], p[o + 4])
            a.lineTo(p[o + 3 + W*3], p[o + 4 + W*3])
            a.lineTo(p[o + W*3], p[o + 1 + W*3])
            a.fill()
            a.stroke()
            o+=3
        }
        o+=3
    }
    // draw the "nyan cat" like trail
    w4=4*w/6
    w1=w/10
    cols=["#5E96FF","#387DFF","#0F47AF","#0D3E99","#0B347F","#08265B"]
    for(j = 0; j &lt; 32; j++) {
        for(i = 0; i&lt; 6;i++) {
            a.fillStyle=cols[i]
            a.strokeStyle=a.fillStyle
            wb=w4 / 32
            hb=w1 / 6
            x=j * wb
            y=h2 + (i-3) *hb - hb*Math.sin(k*0.2 + j *0.6)
            a.fillRect(x,y,wb,hb)
            a.strokeRect(x,y,wb,hb)
        }
    }
    // draw the star
    a.fillStyle="#ffffff"
    a.strokeStyle=a.fillStyle
    a.beginPath()
    for(i = 0; i &lt; 5; i++) {
        s = Math.sin(k * 0.2)*32
        l = w1 * Math.sin(i* 1.256) + s/4
        m = w1 * -Math.sin(i* 1.256 + 1.57) + s
        n = w/25 * Math.sin(i* 1.256 + 0.628)
        o = w/25 * -Math.sin(i* 1.256 + 2.19) + s
        a.lineTo(w4 + l, h2 + m)
        a.lineTo(w4 + n, h2 + o)
    }
    a.fill()
    a.stroke()
    k+=0.5
},20);
</pre>

And the compressed result thanks to [Google Closure Compiler](http://closure-compiler.appspot.com/home) and [JSCrush](http://www.iteral.com/jscrush/):

<pre>k=wJdow;h&HV;w&Wid^;h2=h/2;H=W=4setInterval(function_{c.wid^;c.hV=h;o=p=[]j&lt;hui&lt;wx=~i*w/W+815*(j!y=~j*h/H+83*`!zK2+Z`+j!]Kx/z,1]Ky/z,2]=z,o=ji=cols[i,wb4/Z,hb1/6,x=j*wb,y=N`-3)*hb-hbk+6*j,;="#ffffff";;;5&gt;is=Zkl1*O/4,m1*-+~57O,nY+628oY-+2.19Ow4+l,Nm)w4+n,No);_;_;k+=5},20);GJ(+X,a.lJeTo(a.strokefor`=~256*i[2]	[0][1]a.fillX=Rect(x,y,wb,hb)],++)3*W*;for(j=a.begJPa^_=w),p[o*n|0,0;2*F #;i]-o+=3!+k)$])&.Jner@*mC=GMa^.sJinK=51L+",Nh2+O)+sQ=[U;jVeightXStyleY/25*Z32^th_()`(i~1.0.';for(Y in $='~`_^ZYXVUQONLKJGC@&$!	')with(_.split($[Y]))_=join(pop());eval(_)
</pre>
