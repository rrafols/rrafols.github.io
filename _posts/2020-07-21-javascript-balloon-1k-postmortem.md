---
id: 652
title: JavaScript Balloon 1k postmortem
date: 2020-07-21T18:50:43+00:00
author: raimon
layout: default
guid: http://ec2-18-232-250-173.compute-1.amazonaws.com/?p=652
permalink: /index.php/2020/07/21/javascript-balloon-1k-postmortem/
image: /wp-content/uploads/2020/07/Screenshot-2020-07-21-at-19.39.19.png
categories:
  - Javascript
tags:
  - demoscene
  - Javascript
---
Last year was the last edition of <a rel="noreferrer noopener" href="https://js1k.com/" target="_blank">js1k</a>, a 1k JavaScript code golfing competition I usually took part. Last month, I saw a tweet by <a rel="noreferrer noopener" href="https://twitter.com/MaximeEuziere" target="_blank">Xem</a> that he was preparing a js1k successor: <a rel="noreferrer noopener" href="https://js1024.fun/" target="_blank">js1024.fun</a> so I decided I must take part as well even though I had very limited time 😎.

The end result can be found <a rel="noreferrer noopener" href="https://js1024.fun/demos/2020/44" target="_blank">here</a>, but I&#8217;ll go through the development process to explain what I wanted to do and what I did at the end (and how!)

<a href="https://js1024.fun/demos/2020/44" target="_blank" rel="noopener noreferrer"><img loading="lazy" width="1024" src="/wp-content/uploads/2020/07/Screenshot-2020-07-21-at-19.39.19-1024x710.png" /></a>

My first idea was very different, in fact I wanted to build something with flowers. I had the idea of building kind of procedural 3D transparent flowers (water lilies&#8230;ish?) and move them slowly in a &#8216;lake like&#8217; background &#8211; by just rendering ripples to simulate the water instead of rendering it.

<img loading="lazy" width="1024" src="/wp-content/uploads/2020/07/image-1024x622.png" />

I managed to get something spinning on the screen, but after reaching this point, I got quite uninspired and didn&#8217;t know how to continue. On the last weekend before the deadline, I remembered I had some ideas stored for future inspiration and especially the three.js <a rel="noreferrer noopener" href="https://alexanderperrin.com.au/triangles/ballooning/" target="_blank">ballooning demo</a> by <a rel="noreferrer noopener" href="https://twitter.com/alexanderperrin" target="_blank">Alexander Perrin</a>. I really liked the concept and wanted to recreate something similar in just 1k of JavaScript &#8211; using the Canvas2D API &#8211; not using WebGL or three.js, so I gave it a try&#8230;

<img loading="lazy" alt="original ballooning demo by Alexander Perrin" width="1024" src="/wp-content/uploads/2020/07/Screenshot-2020-07-21-at-19.30.06-1024x807.png" />

I started trying to create a 3D terrain with a valley in the middle, in the first version of the code, I was pre-generating N segments of the terrain, but later I realised I didn&#8217;t know how to loop it nicely or pre-generate an absurd number of segments just for the demo (hey.. it&#8217;s 1k after all!). Below, the early results after playing a bit with numbers to get it decently looking:

<img loading="lazy" width="1024" src="/wp-content/uploads/2020/07/Screenshot-2020-07-21-at-17.52.12-1024x696.png" />

Or, to see more easily the generated geometry, the same render without filling the generated tiles or quads:

<img loading="lazy" width="1024" src="/wp-content/uploads/2020/07/Screenshot-2020-07-21-at-17.51.29-1024x695.png" />

At the end, I did the right thing and I decided to generate/remove terrain segments on the render loop. At the beginning of every render loop, I&#8217;m checking the number of segments and, while it&#8217;s below N (hardcoded constant) it&#8217;ll generate a new segment and smooth it out with its neighbours. Also, when a vertex of a segment was too close to the screen (closer than the simulated camera FOV) I&#8217;ll remove the whole segment, so a new one will be regenerated at the back on the next frame. Below, a short animation of the result if I was only generating one single segment per each frame:

<img loading="lazy" width="480"  src="/wp-content/uploads/2020/07/anim.gif" />

To improve the flatness of the result, I added some lightning by doing a cross product of each tile vectors to compute the quad/tile normal and then the dot product with the light vector &#8211; hardcoded to [.2,1.0]:

```javascript
v0=[pp[k+N].x-pp[k].x,pp[k+N].y-pp[k].y,pp[k+N].z-pp[k].z]
v1=[pp[k+1].x-pp[k].x,pp[k+1].y-pp[k].y,pp[k+1].z-pp[k].z]
v2=cross(v0,v1)
v2=.4+.6*dot(v2,unit([.2,1,0]))

c.fillStyle='rgb(' + ((cols[pp[k].c][0]*v2)|0) + ',' + ((cols[pp[k].c][1]*v2)|0) + ',' + ((cols[pp[k].c][2]*v2)|0) + ')'
```

Result was a bit too dark sometimes, so at the end I&#8217;ve left 40% ambient light + 60% based on lighting calculations which made the end result nicer:

<img loading="lazy" width="1024" src="/wp-content/uploads/2020/07/Screenshot-2020-07-21-at-18.04.03-1024x776.png" />

The segments generated at the back were appearing too abruptly, so the last touch to the terrain was to add some fog by applying alpha blending to the segments based on their distance. As the segments were drawn from the back to the front &#8211; to have a natural z sorting without writing any specific code, it was trivial to add this alpha blending calculation by just adding the following code to each segment before rendering it:

```javascript
c.globalAlpha= 1 - i / (pp.length / N)
```

Where i is the current segment and pp.length / N is the total number of segments (pp.length is an array containing all segments with N vertices each). The result introduces some artifacts like parts of the mountains that overlap with the previously rendered mountains, but also introduces nice touches (the blending with the background, water looked nicer&#8230;) and, overall, I liked the end result.

<img loading="lazy" width="1024" src="/wp-content/uploads/2020/07/Screenshot-2020-07-21-at-18.39.05-1024x780.png" />

After the terrain was done, I started creating the balloon. I took a very naive approach, not reusing what I already had but aiming to use a similar code structure so I could merge it later on. I generated S vertical stripes with S quads on each stripe, just on the front part of the balloon, I&#8217;m not generating the back part of the balloon as it would be never rendered.

Color is switched on each horizontal segment to recreate the white/red stripes. Also, as I was pretty sure I did not had enough space to generate a basked, I just rendered 2 additional vertical iterations, giving the impression of something similar-enough to a basket. Also, these additional iterations were rendered with darker colours as the normals are inverted. To be honest, this was just pure luck, didn&#8217;t thought about it when I devised this dirty hack, but the result was quite ok though:

```javascript
S=12 // segments
for(i=0;i&lt;=S+1;i++) {
  for(jj=0;jj&lt;=S;jj++){
    cl=3+jj%2 // colour
    gl=[]
    cj = Math.cos(jj*.3)
    cj1 = Math.cos((jj+1)*.3)
    sj = Math.sin(jj*.3)
    sj1 = Math.sin((jj+1)*.3)
    pr = 200 * Math.sin(i*.3)
    nr = 200 * Math.sin((i+1)*.3)
    gl.push({x:pr * cj,  y:i * 40, z:pr * sj + 600, c:cl})
    gl.push({x:pr * cj1,  y:i * 40, z:pr * sj1 + 600, c:cl})
    gl.push({x:nr * cj,  y:(i+1) * 40, z:nr * sj + 600, c:cl})
    gl.push({x:nr * cj1,  y:(i+1) * 40, z:nr * sj1 + 600, c:cl})
    dr(gl,0,2,4,0) // draw function
  }
}
```

<img loading="lazy" width="1024" src="/wp-content/uploads/2020/07/Screenshot-2020-07-21-at-19.00.26-1024x778.png" />

I assumed, I would not have enough space to render some trees and add the birds (as the original demo), in my original plan I would have used the same function to draw the balloon to draw the trees by changing location, size and colour, but when I first used [terser + regpack online](https://xem.github.io/terser-online/) to see the final packed size.. I just saw I would have some trouble to reduce it below 1k as I was around 1500 bytes at that time.

<img loading="lazy" width="1024" src="/wp-content/uploads/2020/07/Screenshot-2020-07-21-at-19.11.43-1024x194.png" />

From here to the day before the deadline was a byte crushing race. The first obvious part was to use the same rendering and 3D projection code both for the terrain & the balloon. At that point, the packed result was around 1250 bytes, so still more than 200 bytes to reduce, but I had the idea to animate the water, as I believed it could be done quite easily without adding too much bytes by simply displacing the Y & Z coordinates of the water vertices by a sine function of the time. It seemed to work quite well and didn't add too many bytes, so decided to keep it in.

<img loading="lazy" width="320" src="/wp-content/uploads/2020/07/water2.gif" />

After this, it started to get ugly from source code point of view. I removed the 'sand' colour, hardcoded even more the light vector so no dot product function was needed, reduced some - not very relevant - calculations of the cross product (these changes didn't allow me to move it or change it anymore) and started golfing the source code which some parts become quite unreadable. At the end, there were some minor impacts on lighting and balloon 'design' but after playing with RegPack options, I managed to get it just on 1024 bytes.

```
{
  withMath: false,
  hash2DContext: true,
  hashWebGLContext: true,
  hashAudioContext: true,
  contextVariableName: "c",
  contextType: 0,
  reassignVars: true,
  varsNotReassigned: "a b c d", // js1024
  crushGainFactor: 8,
  crushLengthFactor: 18,
  crushCopiesFactor: 20,
  crushTiebreakerFactor: 1,
  wrapInSetInterval: false,
  timeVariableName: "t",
  useES6: true
}
```

The resulting source code can be found on GitHub: <https://github.com/rrafols/js1024_balloon>
