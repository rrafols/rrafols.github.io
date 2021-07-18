---
id: 653
title: JavaScript Butterfly Sakura 1k postmortem
date: 2021-07-18T16:00:43+00:00
author: raimon
layout: default
guid: http://ec2-18-232-250-173.compute-1.amazonaws.com/?p=652
permalink: /index.php/2020/07/21/javascript-butterfly-sakura-1k-postmortem/
image: /wp-content/uploads/2021/07/ButterflySakura00.jpg
categories:
  - Javascript
tags:
  - demoscene
  - Javascript
---
As usual I love to take part on the javascript 1k code golfing competitions and this year was no different, so I managed to do something for 2021 edition the [js1024.fun](https://js1024.fun/demos/2021).

I took inspiration on a Unity3D Experiment by [Max Gittel](https://twitter.com/maxSigma_) I saw in the past, but to recreate it (more or less) in 1k. The original experiment can be seen here in action [![Original Butterfly Sakura](/wp-content/uploads/2021/07/ButterflySakura01_original.jpeg)](https://twitter.com/maxSigma_/status/1264900383081664514)

https://twitter.com/maxSigma_/status/1264900383081664514)

I started by trying to recreate the background, first by manually iterating colors - both horitzontally and vertically:

```javascript
for(i = 0; i < N; i++) {
  for(j = 0; j < N; j++) {
      // vertical factor
      f0 = i / (N/2-1)

      // horizontal factor
      f1 = j / (N-1)

      // adding a bit of exponential factor to the vertical color interpolation
      f0 = f0*f0*f0

      r = (AC[0] * f1 + BC[0] * (1 - f1)) * (1-f0) + CC[0] * f0
      g = (AC[1] * f1 + BC[1] * (1 - f1)) * (1-f0) + CC[1] * f0
      b = (AC[2] * f1 + BC[2] * (1 - f1)) * (1-f0) + CC[2] * f0
      c.fillStyle = 'rgb(' + r + ',' +g+ ',' + b + ')'

      c.fillRect(j * a.width/N, i * a.height/N, a.width/N, a.height/N)
  }
}
```
But the results were not very nice and, even if it was not optimized, I was not happy with the amount of bytes it was taking just for the background.

![1st background try](/wp-content/uploads/2021/07/ButterflySakura02.png)

So, I decided to change the approach and play with the `createLinearGradient` function of the Canvas `Context2D`. The tricky part was to have a mix of two different gradients, one vertical and one horizontal, but I started by creating the two different gradients:


```javascript
// h = canvas height
E = c.createLinearGradient(0, 0, 0, h)
E.addColorStop(0, '#114')
// add the cyan middle
E.addColorStop(.5, '#8dd')
E.addColorStop(1, '#000')

// w = canvas width
F = c.createLinearGradient(0, 0, w, 0)
F.addColorStop(0, '#411')
F.addColorStop(1, '#002')
```

Played a bit merging them with alpha (using `globalAlpha`) without too much success but, at the end, using `globalCompositeOperation` and setting it to `lighter` created quite good result and was quite compact in terms of size:

```javascript
c.fillStyle = E
c.fillRect(0, 0, w, h)
c.globalCompositeOperation = 'lighter'
c.fillStyle = F
c.fillRect(0, 0, w, h)
```

![2nd background try](/wp-content/uploads/2021/07/ButterflySakura03.jpg)

Happy with this, I then created a *simple* main loop with all the elements that had to be drawn:

```javascript
setInterval(_=>{
    // clear screen (by setting canvas width it clears the screen for free 😎
    a.width = w

    // background
    ...

    // draw tree
    ...
    
    // draw butterflies
    ...
    
    // blurred reflection
    ...
})
```

I started by generating some points of what would be the half shape of the butterfly:

![Butterfly shape](/wp-content/uploads/2021/07/ButterflySakura04.jpg)

And mirror them to draw the whole shape

![Butterfly shape](/wp-content/uploads/2021/07/ButterflySakura05.jpg)

Transformed and encoded as chars to be smaller:

```javascript
p = "rlmhifgeefdgdigmgnhoipppmpjqhrhtixkzmzpyqvrr"

for(i = 0; i < 88; ) {
  // using modulo to loop over the data twice without overlowing the array
  x = -(p.charCodeAt(i++%44) - 99) * 20
  y = (p.charCodeAt(i++%44) - 99) * 20
  // points were transformed by adding 99 and there is a small size adjustment here by multiplying the size by 20.

  // mirror x coordinate if second half of data
  x = i<44 ? x : -630 - x
}
```

For the butterfly movement, I wanted them to move in a spiral like movement:

![Butterfly initial movement](/wp-content/uploads/2021/07/ButterflySakura06.gif)