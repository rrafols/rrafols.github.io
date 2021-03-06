---
id: 629
title: JS1k 2014 entries
date: 2019-08-09T14:56:27+00:00
author: raimon
layout: revision
guid: http://ec2-18-232-250-173.compute-1.amazonaws.com/index.php/2019/08/09/316-revision-v1/
permalink: /index.php/2019/08/09/316-revision-v1/
---
Last year I [said](http://blog.rafols.org/?p=209) I had a very ambitious idea for the js1k 2013 competition but I didn&#8217;t had the time and I thought to leave it for the following year. This year, once again, I could only work on my entry the day before the deadline, so I decided to do something else and start from scratch.  
I saw this animated gif long time ago (which, by the way, I don&#8217;t know the original source or author)  
![](http://www.myconfinedspace.com/wp-content/uploads/2013/06/road-lights.gif)  
So I thought it would be good to make a 1k version of it.  
To be honest I&#8217;m not very happy with the result, I&#8217;m pretty sure it could be done better & nicer. Anyway I submitted it to the competition and here is the [result](http://js1k.com/2014-dragons/demo/1973):  
<http://js1k.com/2014-dragons/demo/1973>  
Find below the full source code or check the [details](http://js1k.com/2014-dragons/details/1973) page on js1k

<pre>//values are just trial & error
w=window
f=w.innerHeight/w.innerWidth
w=1343
h=w*f
_=Math
C=_.cos
S=_.sin
R=_.random
F=512
Z=2200
P=6.28
l=[]
A=[]
B=[]
L=54
p=P/L
Y=N=0
//generate lamp posts in a circle on both sides and pointing to the center
df='000100110010000001011010010110110010010011011010'.split('')
for(i=0;i=3;k--) {
    x=k*F*S(p*i)
    z=k*F*C(p*i)
    x2=(k+v)*F*S(p*i)
    z2=(k+v)*F*C(p*i)
    for(j=0;j9&&j&2){
        xv=x2
        zv=z2
      }
      A[N++]=xv+df[j*3  ]*10
      A[N++]=-df[j*3+1]*512
      A[N++]=zv+df[j*3+2]*8
    }
    i+=.5
    v=-v
  }
}
//store where lamp posts end
K=N
//generate random lines
k=0
for(i=L*8;i--;) {
  l[k++]=R()*P
  l[k++]=R()
  l[k++]=-R()*128-64
  l[k++]=R()*.2+.2;
}
N+=L*96;	//8*12
setInterval(function(){
  Y+=.004
  b.style.background='#000'
  //clear screen
  a.width=w
  a.height=h
  //generate trail lines & move them
  j = K
  k=i=0;
  for(;i&lt;l*8;i++) {
    d=l[k++]
    p=l[k++]
    y=l[k++]
    e=l[k++]
    r=(3.1+p*.8)*F
    v=4*S(y+i)
    g=[0,v,v,0]
    for(v=0;v&lt;4;v++) {
      x=r*S(d)
      z=r*C(d)
      A[j++]=x
      A[j++]=y+g[v]
      A[j++]=z
      if(v&1) d+=e
    }
    l[k-4]+=.02
    // // move lines in both directions
    // if(i= K - trail lines, half red #f22, half white #fff
  k=i=0
  for(;i=K) {
      if(i*12=2) {
        c.lineTo(B[k],B[k+1]);
      }
      k+=3
    }
    c.fill()
    //draw lamp post light after drawing the last top horizontal quad
    //lamp posts are formed of 4 quads:
    //0 & 1 vertical
    //2 & 3 top horizontal
    //avoid drawing lights on trail lights
    if(i % 4 == 3 && i*12&lt;k) {
      c.fillStyle="#fff"
      c.beginPath()
      c.arc(B[k-3], B[k-2], (2-B[k-1])*13,0,P)
      c.fill()
    }
  }
}, 20)
</pre>

Code is compressed under 1k thanks to [Google Closure Compiler](http://closure-compiler.appspot.com/home) and [RegPack v3](http://siorki.github.io/regPack.html)  
As I wasn&#8217;t really happy with the result I did another quick entry based on the old good [daCube2](https://www.youtube.com/watch?v=OYABLIPjF7Q) colors & idea:  
[youtube https://www.youtube.com/watch?v=OYABLIPjF7Q?rel=0&w=640&h=480]  
Find below the [result](http://js1k.com/2014-dragons/demo/1969):  
<http://js1k.com/2014-dragons/demo/1969>  
and the source code or check the [details](http://js1k.com/2014-dragons/details/1969) page on js1k:

<pre>w=window
h=w.innerHeight
w=w.innerWidth
_=Math
mC=_.cos
mS=_.sin
R=_.random
F=512
Z=2048
N=L=X=Y=0
A=[]
B=[]
C=[]
D=[]
addCube=function(s) {
  k=N/3
  j='000100010110001101011111'.split('')
  for(i=0;i&lt;24;i++) {
    A[N++]=(j[i]-.5)*2*s
  }
  l=&#039;011332200445514662577367&#039;.split(&#039;&#039;)
  for(i=0;i&lt;24;i++) {
    D[L++]=k+(l[i]|0)
  }
}
for(z=200;z--;){
  addCube(100+z*5)
}
anim=.004
setInterval(function(){
  Y+=anim*R()+anim
  X+=anim*R()+anim
  y=Y
  x=X
  //X+=.01
  b.style.background=&#039;#444&#039;
  //clear screen
  a.width=w
  a.height=h
  k=0
  //dy rotation
  for(i=N;i--;) {
    if(k%24*3==0) y-=anim*2
    C[k+2]=A[k+2]*mC(y)-A[k]*mS(y)
    C[k  ]=A[k+2]*mS(y)+A[k]*mC(y)
    k+=3
  }
  k=0
  //dx rotation
  for(i=N;i--;) {
    if(k%24*3==0) x+=anim*2
    B[k+1]=A[k+1]*mC(x)-C[k+2]*mS(x)
    B[k+2]=A[k+1]*mS(x)+C[k+2]*mC(x)
    k+=3
  }
  //transform 3d-2d
  k=p=0
  for(i=N;i--;) {
    C[k++]=F*C[p  ]/(C[p+2]+Z)+w/2
    C[k++]=F*B[p+1]/(C[p+2]+Z)+h/2
    p+=3
  }
  p=0
  for(i=0;i&lt;l/24;i++) {
    c.strokeStyle="#fff"
    c.globalAlpha=(i/(L/24))
    c.beginPath()
    for(k=24;k--;){
      j=D[p++]
      c.moveTo(C[j*2],C[j*2+1]);
      j=D[p++]
      c.lineTo(C[j*2],C[j*2+1]);
    }
    c.stroke()
  }
}, 20)
</pre>
