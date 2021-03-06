---
id: 615
title: 'Looping &#8211; js1k 2018 post mortem'
date: 2019-08-09T14:34:37+00:00
author: raimon
layout: revision
guid: http://ec2-18-232-250-173.compute-1.amazonaws.com/index.php/2019/08/09/558-revision-v1/
permalink: /index.php/2019/08/09/558-revision-v1/
---
For this year&#8217;s <a href="http://js1k.com/2018-coins/" target="_blank" rel="noopener noreferrer">js1k</a> I wanted to build a simple ray-tracer to see both how much could I fit in 1k and the performance of js.  
I started by adding a very simple (trivial) camera implementation and adding a sphere primitive:  
<img loading="lazy" class="alignnone size-full wp-image-559" src="http://ec2-18-232-250-173.compute-1.amazonaws.com/wp-content/uploads/2018/03/shim_01.png" alt="shim_01" width="3096" height="1978" srcset="http://blog.rafols.org/wp-content/uploads/2018/03/shim_01.png 3096w, http://blog.rafols.org/wp-content/uploads/2018/03/shim_01-300x192.png 300w, http://blog.rafols.org/wp-content/uploads/2018/03/shim_01-768x491.png 768w, http://blog.rafols.org/wp-content/uploads/2018/03/shim_01-1024x654.png 1024w" sizes="(max-width: 3096px) 100vw, 3096px" />  
Results where not mind blowing but, hey, that was a start. Code was pretty simple:

<pre>//w = canvas width
  //h = canvas height
  //F = pixels skipped. At F=1 we would compute the real value each pixel, at 2, we would compute at every 2 pixels, and so on..
// where the camera is and where the camera is looking
origin=[0,0,-1000]
dst  =[0,0,1000]
fov = 90
...
  // compute direction vector and horizontal & vertical vectors (hv & vv respectively)
  cd=[dst[0]-origin[0],
      dst[1]-origin[1],
      dst[2]-origin[2]]
  u(cd)
  alpha = Math.PI*fov/360.0
  focaldist = w * Math.cos(alpha)/(2*Math.sin(alpha))
  cv=[origin[0]+cd[0]*focaldist,
      origin[1]+cd[1]*focaldist,
      origin[2]+cd[2]*focaldist]
  hv=[-cd[2],0,cd[0]]
  vv=x(hv, cd)
  // compute the view vector
  v=[cv[0]-vv[0]*h2-hv[0]*w2-origin[0],
     cv[1]-vv[1]*h2-hv[1]*w2-origin[1],
     cv[2]-vv[2]*h2-hv[2]*w2-origin[2]]
  for(i=0;i&lt;h;i+=F) {
    vl=[v[0], v[1], v[2]];
    for(j=0;j&lt;w;j+=F) {
      d = [vl[0],vl[1],vl[2]]
      u(d)
      os = it(origin, d)
      if(os[0] != -1) {
        c.fillStyle='#fff'
      } else {
        c.fillStyle='#28A'
      }
      c.fillRect(j, i, F, F)
      // we add the computed camera horizontal vector for each column
      vl[0] += hv[0] * F
      vl[1] += hv[1] * F
      vl[2] += hv[2] * F
    }
    // we add the computed camera vertical vector for each row
    v[0] += vv[0] * F
    v[1] += vv[1] * F
    v[2] += vv[2] * F
  }
</pre>

Functions u & x are two small utilities to calculate the unit vector and cross product:

<pre>function u(v) {
  m = Math.sqrt(v[0] * v[0] + v[1] * v[1] + v[2] * v[2])
  v[0]/=m
  v[1]/=m
  v[2]/=m
}
function x(u,v) {
  k=[]
  k[0]=u[1]*v[2] - u[2]*v[1]
  k[1]=-(u[0]*v[2] - u[2]*v[0])
  k[2]=u[0]*v[1] - u[1]*v[0]
  return k
}
</pre>

and the intersection code was straightforward:

<pre>function it(o, d) {
  os = -1
  maxt = 100000000.0
  for(k=0;k0) {
        t=tca-Math.sqrt(thc);
      }
    }
    if(t&lt;maxt) {
      maxt = t;
      os = k
    }
  }
  return [os, maxt]
}
</pre>

Shading was trivial as well (if we can even call it shading):

<pre>os = it(origin, d)
if(os[0] != -1) {
  c.fillStyle='#fff'
} else {
  c.fillStyle='#28A'
}
</pre>

Draw it white if there is an intersection or background-blueish if it isn&#8217;t.  
Now that we get the basics in place, let&#8217;s add more spheres, colors and some shading:

<pre>os = it(origin, d)
if(os[0] != -1) {
  // point of intersection: origin + direction ray * t
  ol=[origin[0] + d[0] * os[1],
      origin[1] + d[1] * os[1],
      origin[2] + d[2] * os[1]]
  // intersection point normal (point - sphere center)
  ir=sphereList[os[0]*S + 5]
  n=[(ol[0] - sphereList[os[0]*S]) * ir,
     (ol[1] - sphereList[os[0]*S+1]) * ir,
     (ol[2] - sphereList[os[0]*S+2]) * ir,
  ]
  u(n)
  // light position
  ll=[0,-1000,0]
  // intersection point -&gt; light direction vector
  rl=[ll[0] - ol[0],
      ll[1] - ol[1],
      ll[2] - ol[2]]
  u(rl)
  // sphere color
  r=sphereList[os[0]*S + 5]
  g=sphereList[os[0]*S + 6]
  b=sphereList[os[0]*S + 7]
  // color factor = dot product of normal and light direction vector
  al=n[0]*rl[0] + n[1]*rl[1] + n[2]*rl[2]
  if(al&gt;0.0){
    r*=al
    g*=al
    b*=al
  } else {
    r=0
    g=0
    b=0
  }
  r=(r*16)|0
  g=(g*16)|0
  b=(b*16)|0
  c.fillStyle='#'+hexmap[r]+hexmap[g]+hexmap[b];
} else {
  c.fillStyle='#28A'
}
</pre>

Results are quite &#8216;gouradish&#8217;:  
<img loading="lazy" class="alignnone size-full wp-image-560" src="http://ec2-18-232-250-173.compute-1.amazonaws.com/wp-content/uploads/2018/03/shim_02.png" alt="shim_02" width="3096" height="1978" srcset="http://blog.rafols.org/wp-content/uploads/2018/03/shim_02.png 3096w, http://blog.rafols.org/wp-content/uploads/2018/03/shim_02-300x192.png 300w, http://blog.rafols.org/wp-content/uploads/2018/03/shim_02-768x491.png 768w, http://blog.rafols.org/wp-content/uploads/2018/03/shim_02-1024x654.png 1024w" sizes="(max-width: 3096px) 100vw, 3096px" />  
So, let&#8217;s add a specular component to the light:

<pre>// compute the reflection of the ray
al=2*(n[0]*d[0]+n[1]*d[1]+n[2]*d[2])
rx=[d[0] - al * n[0],
    d[1] - al * n[1],
    d[2] - al * n[2]
]
// and the dot product with the light vector
ls=rx[0]*rl[0]+
   rx[1]*rl[1]+
   rx[2]*rl[2]
// the higher the exponent, more focused will be the specular:
ls=ls0.0){
  r=r*al+ls
  g=g*al+ls
  b=b*al+ls
}
</pre>

And here the result:  
<img loading="lazy" class="alignnone size-full wp-image-561" src="http://ec2-18-232-250-173.compute-1.amazonaws.com/wp-content/uploads/2018/03/shim_03.png" alt="shim_03" width="3096" height="1978" srcset="http://blog.rafols.org/wp-content/uploads/2018/03/shim_03.png 3096w, http://blog.rafols.org/wp-content/uploads/2018/03/shim_03-300x192.png 300w, http://blog.rafols.org/wp-content/uploads/2018/03/shim_03-768x491.png 768w, http://blog.rafols.org/wp-content/uploads/2018/03/shim_03-1024x654.png 1024w" sizes="(max-width: 3096px) 100vw, 3096px" />  
We can also play with the specular exponent. Below the differences between 20, 8 and 2:

<div id='gallery-5' class='gallery galleryid-615 gallery-columns-3 gallery-size-thumbnail'>
  <figure class='gallery-item'> 
  
  <div class='gallery-icon landscape'>
    <a href='http://blog.rafols.org/index.php/2018/03/08/looping-js1k-2018-post-mortem/screen-shot-2018-03-08-at-13-41-52/'><img width="150" height="150" src="http://blog.rafols.org/wp-content/uploads/2018/03/screen-shot-2018-03-08-at-13-41-52-150x150.png" class="attachment-thumbnail size-thumbnail" alt="" loading="lazy" srcset="http://blog.rafols.org/wp-content/uploads/2018/03/screen-shot-2018-03-08-at-13-41-52-150x150.png 150w, http://blog.rafols.org/wp-content/uploads/2018/03/screen-shot-2018-03-08-at-13-41-52-100x100.png 100w" sizes="(max-width: 150px) 100vw, 150px" /></a>
  </div></figure><figure class='gallery-item'> 
  
  <div class='gallery-icon landscape'>
    <a href='http://blog.rafols.org/index.php/2018/03/08/looping-js1k-2018-post-mortem/screen-shot-2018-03-08-at-13-41-31/'><img width="150" height="150" src="http://blog.rafols.org/wp-content/uploads/2018/03/screen-shot-2018-03-08-at-13-41-31-150x150.png" class="attachment-thumbnail size-thumbnail" alt="" loading="lazy" srcset="http://blog.rafols.org/wp-content/uploads/2018/03/screen-shot-2018-03-08-at-13-41-31-150x150.png 150w, http://blog.rafols.org/wp-content/uploads/2018/03/screen-shot-2018-03-08-at-13-41-31-100x100.png 100w" sizes="(max-width: 150px) 100vw, 150px" /></a>
  </div></figure><figure class='gallery-item'> 
  
  <div class='gallery-icon landscape'>
    <a href='http://blog.rafols.org/index.php/2018/03/08/looping-js1k-2018-post-mortem/screen-shot-2018-03-08-at-13-41-16/'><img width="150" height="150" src="http://blog.rafols.org/wp-content/uploads/2018/03/screen-shot-2018-03-08-at-13-41-16-150x150.png" class="attachment-thumbnail size-thumbnail" alt="" loading="lazy" srcset="http://blog.rafols.org/wp-content/uploads/2018/03/screen-shot-2018-03-08-at-13-41-16-150x150.png 150w, http://blog.rafols.org/wp-content/uploads/2018/03/screen-shot-2018-03-08-at-13-41-16-100x100.png 100w" sizes="(max-width: 150px) 100vw, 150px" /></a>
  </div></figure>
</div>

So far, things are quite simple, but what would be a raytracer without shadows? Let&#8217;s also add some shadows!  
As we have one single light, it is quite straightforward, before shading each pixel, trace a ray from the intersection point to the light position. If there is any positive intersection of an object it means that point is occluded and shouldn&#8217;t be shadowed. As we already have the function to calculate intersections written, it also very simple:

<pre>// compute the 't' value of the equation. This will be used as the maximum value for the 't'. Any object further than that will not occlude the light
tl=(ll[2] - ol[2])/rl[2]
sh=it(ol, rl, tl, os[0])
// if not intersection, let's shade the pixel normally
if(sh[0]==-1){
  ...
  // shading algorithm
  ...
} else {
  // pixel is shadowed, apply only ambient color
}
</pre>

Result starts to look pretty decent:  
<img loading="lazy" class="alignnone size-full wp-image-562" src="http://ec2-18-232-250-173.compute-1.amazonaws.com/wp-content/uploads/2018/03/shim_05.png" alt="shim_05" width="3096" height="1978" srcset="http://blog.rafols.org/wp-content/uploads/2018/03/shim_05.png 3096w, http://blog.rafols.org/wp-content/uploads/2018/03/shim_05-300x192.png 300w, http://blog.rafols.org/wp-content/uploads/2018/03/shim_05-768x491.png 768w, http://blog.rafols.org/wp-content/uploads/2018/03/shim_05-1024x654.png 1024w" sizes="(max-width: 3096px) 100vw, 3096px" />  
I&#8217;ve also added another primitive: a horizontal plane:

<pre>if(sl[k*S+7] == 0) {
  ...
  // sphere intersection
  ...
} else {
  vd=sl[k * S + 0] * d[0] + sl[k * S + 1] * d[1] + sl[k * S + 2] * d[2];
  vo=-sl[k * S + 0] * o[0] + sl[k * S + 1] * o[1] + sl[k * S + 2] * o[2] + sl[k * S + 3];
  t=vo/vd
}
</pre>

Just had to add another entry to the object definition array:

<pre>sl = [1000, 500, 2000, 250000, 0.8, 0.2, 0.3, 0,
      -1000, 500, 2000, 250000, 0.3, 0.2, 0.8, 0,
      0, 500, 3000, 250000, 0.3, 0.8, 0.2, 0,
      0, 1, 0, 1000, 0.6, 0.6, 0.6, 1
 ]
</pre>

Entries in this array have the following meaning:

<pre>// x
// y
// z
// squared radius (radius * radius) or displacement if primitive is a plane
// red
// green
// blue
// object type (0 - sphere, 1 - horizontal plane)
</pre>

Normal has to be computed in a different way depending on the primitive type. In the case of the plane I was planning to do a checkerboard, but to save some space I ended up creating stripes:

<pre>r=sl[os[0]*S + 4]
g=sl[os[0]*S + 5]
b=sl[os[0]*S + 6]
// normal reset to vertical vector
n=[0,-1,0]
if(sl[os[0]*S+7]==0) {
  // if we intersect a sphere, let's recalculate the normal
  n=[(ol[0] - sl[os[0]*S]),
      (ol[1] - sl[os[0]*S+1]),
      (ol[2] - sl[os[0]*S+2])
  ]
  u(n)
} else {
  //tweak color depending on distance - the 1000 was chosen in the most scientific way possible - keep playing with the value until it feels right
  cl=(ol[2]/1000|0)%2
  r=1
  b=0
  if(cl==0) {
    g=1
  } else {
    g=0
  }
}
</pre>

Result was the following:  
<img loading="lazy" class="alignnone size-full wp-image-563" src="http://ec2-18-232-250-173.compute-1.amazonaws.com/wp-content/uploads/2018/03/shim_07.png" alt="shim_07" width="3096" height="1978" srcset="http://blog.rafols.org/wp-content/uploads/2018/03/shim_07.png 3096w, http://blog.rafols.org/wp-content/uploads/2018/03/shim_07-300x192.png 300w, http://blog.rafols.org/wp-content/uploads/2018/03/shim_07-768x491.png 768w, http://blog.rafols.org/wp-content/uploads/2018/03/shim_07-1024x654.png 1024w" sizes="(max-width: 3096px) 100vw, 3096px" />  
We&#8217;re still missing one key feature of raytracers, reflections! In order to add reflections, we need to create a function that calculates the intersection & computes the resulting color. We&#8217;ll call it recursively:

<pre>// if reflection index is smaller than 3, compute a reflected ray color & merge it with current color (70% + 30%)
// we limit the amount of reflections to 3 levels
if(rc &lt; 3) {
  rxc = cil(ol, rx, rc+1)
  r=r*0.7+rxc[0]*0.3
  g=g*0.7+rxc[1]*0.3
  b=b*0.7+rxc[2]*0.3
}
</pre>

By adding reflections, looks slightly better:  
<img loading="lazy" class="alignnone size-full wp-image-564" src="http://ec2-18-232-250-173.compute-1.amazonaws.com/wp-content/uploads/2018/03/shim_08.png" alt="shim_08" width="3096" height="1978" srcset="http://blog.rafols.org/wp-content/uploads/2018/03/shim_08.png 3096w, http://blog.rafols.org/wp-content/uploads/2018/03/shim_08-300x192.png 300w, http://blog.rafols.org/wp-content/uploads/2018/03/shim_08-768x491.png 768w, http://blog.rafols.org/wp-content/uploads/2018/03/shim_08-1024x654.png 1024w" sizes="(max-width: 3096px) 100vw, 3096px" />  
Also, as you could see in this last screenshot, there was an small intent of CSG (Constructive Solid Geometry). I added a sphere that was &#8216;substracting&#8217; from other primitives.  
At the end, I managed to fix the CSG intersection code. In order to make it work we&#8217;ve to calculate both the minimum and the maximum &#8216;t&#8217; of the ray intersection and work out all the different options:

<pre>---- direction ray
* intersection of original object
# intersection of the subtracting object

trivial case, normal intersection
1)---------*----*----#---#-------
trivial case, normal intersection
2)---------#----#----*---*-------
trivial case, normal intersection
3)---------*----#----#---*-------
trivial case, normal intersection
4)---------*----#----*---#-------
trivial case, no intersection (subtracting object is wrapping the whole object)
5)---------#----*----*---#-------
intersection would be the exit point of subtracting object (second #)
6)---------#----*----#---*-------
</pre>

And the updated intersection code. Used a &#8216;magic&#8217; code -2 to detect we&#8217;re checking the CSG intersection: (Kids don&#8217;t do that at home!)

<pre>var csi=0
if(z!=-2 &&t!=t2){
  var csg=it(o,d,maxt,-2)
  if(csg[0]==5) {
    if(csg[1] &lt; t && csg[2] &gt; t && csg[2] &lt; t2) {
      t=csg[2]
      csi = 1
    }  else if(csg[1] &lt; t && csg[2] &gt; t2) {
      t = maxt
    }
  }
}
</pre>

In addition, I also added another primitive: a vertical cylinder.

<pre>if(sl[k*S+7] == 0) {
 // sphere intersection
} else if(sl[k*S+7] == 1) {
 // plane intersection
} else {
 // cylinder
  var rd2=1.0-d[1]*d[1]
  var b=sl[k*S + 3]*rd2
  var b2=d[0]*(o[2]-sl[k*S + 2]) - d[2]*(o[0]-sl[k*S + 0])
  b-=b2*b2
  if(b&gt;0) {
    b2 = Math.sqrt(b)
    b=d[0]*(o[0]-sl[k*S + 0]) + d[2]*(o[2]-sl[k*S + 2])
    t=-(b+b2)/rd2
    t2=-(b-b2)/rd2
  }
}
</pre>

And here is the result:  
<img loading="lazy" class="alignnone size-full wp-image-565" src="http://ec2-18-232-250-173.compute-1.amazonaws.com/wp-content/uploads/2018/03/shim_09.png" alt="shim_09" width="2076" height="1328" srcset="http://blog.rafols.org/wp-content/uploads/2018/03/shim_09.png 2076w, http://blog.rafols.org/wp-content/uploads/2018/03/shim_09-300x192.png 300w, http://blog.rafols.org/wp-content/uploads/2018/03/shim_09-768x491.png 768w, http://blog.rafols.org/wp-content/uploads/2018/03/shim_09-1024x655.png 1024w" sizes="(max-width: 2076px) 100vw, 2076px" /> 

Unfortunately, after using Uglify (<https://github.com/mishoo/UglifyJS>) and RegPack (<http://siorki.github.io/regPack.html>) the resulting size was over 1600 bytes so I had to start a heavy optimization to get it under 1024. I removed the CSG intersection code :(, simplified the RGB filling color to:

<pre>r=(col[0]*255)|0
g=(col[1]*255)|0
b=(col[2]*255)|0
c.fillStyle='rgba('+r+','+g+','+b+',1)'
c.fillRect(j, i, F, F)
</pre>

instead of using the hexmap and 16 different shades per component &#8211; I kind of liked it the pixelated/oldschool effect, but it was slightly shorter & definitely faster.  
After some time optimizing and doing some sacrifices I managed to get it under 1024. It was a pity I couldn&#8217;t spend any more time on it as I would really tried to squeeze the CSG code in! ????  
Here is the final source code:

<pre>w=window.innerWidth
K=500
J=K*K
E=1000
D=1500
G=0
l=n=[]
O = [0, K, 0, J*2, 1, 0, 0, 0,
     0, K, 0, J, 0, 0, 1, 0,
     0, K, 0, J, 0, 1, 0, 0,
     0, E, E*2, J, 1, 1, 1, 2,
     0, 1, 0, E, 1, 0, 0, 1]

// dot product
dp = (u,v) =&gt; u[0]*v[0]+u[1]*v[1]+u[2]*v[2]

// unit vector
u = v =&gt; {
  m = Math.sqrt(dp(v,v))
  v[0]/=m
  v[1]/=m
  v[2]/=m
}

it = (o, d, T) =&gt; {
  u(d)
  s = -1
  t = T
  for(z=0;z&lt;40;z+=8){
    t=(-O[z] * o[0] + O[z + 1] * o[1] + O[z + 2] * o[2] + O[z + 3])/(O[z] * d[0] + O[z + 1] * d[1] + O[z + 2] * d[2])
    if(O[z+7] == 0) {
      e=[O[z] - o[0],
         O[z + 1] - o[1],
         O[z + 2] - o[2]]
      p=dp(e,d)
      t=p-Math.sqrt(O[z+3]-dp(e,e)+p*p)
    }
    if(O[z+7] == 2) {
      p=1-d[1]*d[1]
      q=d[0]*(o[2]-O[z + 2]) - d[2]*(o[0]-O[z])
      t=-(d[0]*(o[0]-O[z]) + d[2]*(o[2]-O[z + 2])+Math.sqrt(O[z + 3]*p-q*q))/p
    }
    if(t&gt;.1&&t&lt;T) {
      T = t
      s = z
    }
  }
  return [s, T]
}

cil = (o,d) =&gt; {
  var r=[.4,.4,.5]
  I = it(o, d, E*E)
  if(I[0]&gt;=0) {
      y=I[0]
      for(k=0;k&lt;3;k++) {
        o[k]=o[k] + d[k] * I[1]
        n[k]=o[k] - O[y+k]
        l[k]=O[2-k]-o[k]-D*k
      }
      if(O[y+7]==1) {
        n=[0,-1,0]
        O[y + 5]=(o[2]/E|0)%2
      }
      if(O[y+7]==2) n[1] = 0
      u(n)
      H=it(o, l, E*E)
      A=2*dp(n,d)
      rx=[d[0] - A * n[0], d[1] - A * n[1], d[2] - A * n[2]]
      A=L=0
      if(H[0]&lt;0) {
        L=dp(rx,l)
        L=L&lt;0?0:Math.pow(L,9)
        A=dp(n,l)
      }
      for(k=0;k&lt;3;k++) r[k]=O[y+4+k]*A+L
      R = cil(o, rx)
      for(k=0;k&lt;3;k++) r[k]=(r[k] + R[k])/2
  }
  return r
}

setInterval(() =&gt; {
  //animation
  for(i=2;i&gt;=0;i--) {
    O[i*8] = l[2] = D*Math.sin(G + i*2)
    O[i*8 + 2] = l[0] = D*Math.cos(G + i*2) + D;
  }
  //render
  for(;i&lt;window.innerHeight;i+=6) {
    for(j=0;j&lt;w;j+=6) {
      x=cil([0,0,-D*2], [w/2-j, i-w/6, D-w/6])
      c.fillStyle='rgb('+[x[0]*255|0,x[1]*255|0,x[2]*255|0]+')'
      c.fillRect(j, i, 6, 6)
    }
  }
  G+=.02
}, 10)
</pre>

and the result can be checked here: <https://js1k.com/2018-coins/demo/3101>
