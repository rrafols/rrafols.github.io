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

```javascript
// butterfly points (x, y, x, y, ...)
points = [30, 19, 20, 10, 13, 6, 8, 5, 4, 6, 3, 8, 3, 12, 8, 20, 8, 23, 10, 25, 13, 26, 27, 26, 20, 27, 15, 28, 11, 31, 10, 35, 12, 42, 16, 47, 21, 46, 26, 44, 29, 38, 30, 31]
```

Mirroring these points horizontally we can draw the whole shape (not the best drawn butterfly in world though - but hopefully good enough for 1k of javascript code):

![Butterfly shape](/wp-content/uploads/2021/07/ButterflySakura05.jpg)


For size purposes, points were transformed and encoded as `chars` to be smaller (this is a preprocess, did not end up on the final code):
```javascript
points = [30, 19, 20, 10, 13, 6, 8, 5, 4, 6, 3, 8, 3, 12, 8, 20, 8, 23, 10, 25, 13, 26, 27, 26, 20, 27, 15, 28, 11, 31, 10, 35, 12, 42, 16, 47, 21, 46, 26, 44, 29, 38, 30, 31]

str = ""
for(i = 0; i < points.length; i++) {
  // 33 to map it to visible chars after space
  // 66 adjust for a lower case ascii string
  // points are divided by 2 - losing a bit of precision in the process
  // but for space optimization purposes
  str += String.fromCharCode(33 + 66 + ((points[i]/2)|0))
}

console.log('p="'+str+'"')
```

and, when drawing the butterflies, `chars` are decoded again into coordinates by substracting 99 and then multiplying by 20 to get an adequate size.

```javascript
p = "rlmhifgeefdgdigmgnhoipppmpjqhrhtixkzmzpyqvrr"

// iterate twice the amount of data
for(i = 0; i < 88; ) {
  // using modulo to loop over the index and avoid overlowing the array
  x = -(p.charCodeAt(i++%44) - 99) * 20
  y = (p.charCodeAt(i++%44) - 99) * 20

  // mirror x coordinate if second half of data
  x = i<44 ? x : -630 - x
}
```

To get some butterflies on the screen, I randomized their position (on the top part only):
```javascript
for(i = 0; i < 607; i++) {
  B[i] = [Math.random()*900 - 450, -Math.random()*600]
}
```
![Butterfly initial position](/wp-content/uploads/2021/07/ButterflySakura08.jpg)

For the movement, I wanted them to move in a spiral like path from their starting point:
![Butterfly initial movement](/wp-content/uploads/2021/07/ButterflySakura06.gif)

but also rotated so they face the direction they are going:
```javascript
  C = Math.cos(f)
  S = Math.sin(f)

// rotate the x & z coordinates by the Y axis
  xr = x * C - z * S
  zr = x * S + z * C
```

and adding the radius and some `z-offset`, gives us the following result:
```javascript
  C = Math.cos(f)
  S = Math.sin(f)

  // z-offset 6500 (displace all points further from the camera)
  // horizontal rotation radius is wider than depth rotation radius
  xr = x * C - z * S + 3*radius * C
  zr = x * S + z * C + radius * S + 6500
```
![Butterfly  movement](/wp-content/uploads/2021/07/ButterflySakura09.gif)

Also, we need to add some _flapping_ to the wings, for that instead of having a constant flat `y` coordinate, let's modify it based on `x` coordinate and time (`t`):
```javascript
  // 80 is the strength of the 'flap'
  y = 80 * Math.sin(x / 200 + t * .1)
```

see the animation in action:

![Butterfly  movement](/wp-content/uploads/2021/07/ButterflySakura11.gif)

and the final animation for each single buttefly:

![Butterfly  movement](/wp-content/uploads/2021/07/ButterflySakura10.gif)

If we put all together, with the randomized butterfly positions, this is the result:

```javascript
// draw butterflies
for(k = 0; k < 607; k++) {
  c.beginPath()
  for(i = 0; i < 80; ) {
                        
    x = -(p.charCodeAt(i++%44) - 99) * 20
    z = -(p.charCodeAt(i++%44) - 99) * 20
    

    x = i<44 ? x : -630 - x

    butterflyTime = Math.max(t - 20 * k, 0)
    angle = butterflyTime ? butterflyTime / 80 : 0
    
    y = (butterflyTime ? 80*Math.sin((x + k) / 200 + t * .1) : 0)
    
    radius = Math.min(150 * angle, 2000)
    C = Math.cos(angle)
    S = Math.sin(angle)

    xr = x * C - z * S + 3*radius * C
    zr = x * S + z * C + radius * S + 6500

    //500 is a FOV for doing the 3D to 2D projection
    c.lineTo(
      500 * (B[k][0] + xr) / zr + w/2, 
      500 * (B[k][1] + y) / zr + h/2)
  }
  c.fill()
}
```

![Multiple butterfly movement](/wp-content/uploads/2021/07/ButterflySakura07.gif)

Time-based movement of the butterfly is modified with the index and some negative offset, so they do not start moving at the same time:
```javascript
  // index is the butterfly index in the array
  butterflyTime = Math.max(t - 20 * index, 0)

  //angle is 0 if butterflyTime is below or equal to 0
  angle = butterflyTime ? butterflyTime / 80 : 0
```

Let's park the butteflies for a moment and let's focus on the tree. I wanted to do something very simple but with good results. I thought on a recursive tree I used on a presentation I did back on Oracle Code One 2019:

[Rendering Art on the Web - A performance compendium
![Rendering Art on the Web - A performance compendium](/wp-content/uploads/2021/07/ButterflySakura12.png)](https://www.slideshare.net/RaimonRls/rendering-art-on-the-web-a-performance-compendium)

but with some adaptations, definitely not performance optimized, unbalanced and decreasing branch size:

```javascript
// draw a recursive tree
tree = (x, y, l, n=0, i=0) => {
  // reduce line width on each level (wide base, thinner branches)
  // increase level by one
  c.lineWidth = 20 / ++i

  n += Math.cos(t * .01) * .01

  // calculate end coordinates based on length * sin/cos angle.
  // these will be used as starting point for sub-branches
  let x1 = x + l * Math.sin(n)
  let y1 = y - l * Math.cos(n)

  c.beginPath()
  c.moveTo(x, y)
  c.lineTo(x1, y1)
  c.stroke()

  // reduce length on each level
  l *= .7

  // recursivelly draw branches depending (unbalanced on purpose)
  if(i<7) tree(x1, y1, l, n - 1, i)
  if(i<5) tree(x1, y1, l, n + 1, i)
  if(i<7) tree(x1, y1, l, n + .4, i)
}
```

Temporarily removing the butterflies and adding calling this method on the drawing loop:
```javascript
  c.strokeStyle = '#f8c'
  c.globalAlpha = .5
  tree(w/2, h/2, h/8)
```

give us the following result:

![Rendered tree](/wp-content/uploads/2021/07/ButterflySakura13.png)

Let's now change the code to generate the butterflies at each branch node and remove the random position code from before:

```javascript
  ...
  let x1 = x + l * Math.sin(n)
  let y1 = y - l * Math.cos(n)

  // add butterfly coordinates (centered on the middle of the screen)
  B[k++] = [x1 - w/2, y1 - h/2]

  c.beginPath()
  c.moveTo(x, y)
  ...
```
Now it is starting to look better:

![Rendered tree with butterflies](/wp-content/uploads/2021/07/ButterflySakura14.gif)

with the end result, after all butterflies have departed form their original position:

![Butterflies circling the tree](/wp-content/uploads/2021/07/ButterflySakura15.gif)
