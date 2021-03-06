---
id: 596
title: Image Rotation in j2me
date: 2018-07-05T18:43:57+00:00
author: raimon
layout: revision
guid: http://ec2-18-232-250-173.compute-1.amazonaws.com/index.php/2018/07/05/58-revision-v1/
permalink: /index.php/2018/07/05/58-revision-v1/
---
Lately I&#8217;ve been creating some low level image functions for Java ME just to see if low level bitmap manipulation was way too slow for doing it on real time on Java ME or it was, at least, usable. I&#8217;ll publish more functions but I&#8217;ll start with Image Rotation.  
In Java ME is easy to rotate images if you want to rotate by an angle multiple of 90° but it doesn&#8217;t provide any mechanism to rotate it by an arbitrary angle (yes, ok&#8230; you could do the same using the Mobile 3D API)  
I created a small function that fills that gap and allows image rotation by any angle. The resulting image will have the same size as the original (watch out for the corners..)  
[code lang=&#8221;java&#8221;]  
public static void rotateImage(Image src, float angle, Graphics g) {  
int sw = src.getWidth();  
int sh = src.getHeight();  
int[] srcData = new int[sw * sh];  
src.getRGB(srcData, 0, sw, 0, 0, sw, sh);  
int[] dstData = new int[sw * sh];  
double rads = angle * Math.PI / 180.f;  
float sa = (float) Math.sin(rads);  
float ca = (float) Math.cos(rads);  
int isa = (int) (256 * sa);  
int ica = (int) (256 * ca);  
int my = &#8211; (sh >> 1);  
for(int i = 0; i > 1);  
for(int j = 0; j > 8;  
int srcy = (-mx \* isa + my \* ica) >> 8;  
srcx += sw >> 1;  
srcy += sh >> 1;  
if(srcx < 0) srcx = 0;  
if(srcy sw &#8211; 1) srcx = sw &#8211; 1;  
if(srcy > sh &#8211; 1) srcy = sh &#8211; 1;  
dstData[j + i \* sw] = srcData[srcx + srcy \* sw];  
mx++;  
}  
my++;  
}  
g.drawRGB(dstData, 0, sw, 0, 0, sw, sh, true);  
}  
[/code]  
If we move all the calculations that doesn&#8217;t need to be done in the inner loop to the external loop we will have a speed improvement:  
[code lang=&#8221;java&#8221;]  
public static void rotateImage(Image src, float angle, Graphics g) {  
int sw = src.getWidth();  
int sh = src.getHeight();  
int[] srcData = new int[sw * sh];  
src.getRGB(srcData, 0, sw, 0, 0, sw, sh);  
int[] dstData = new int[sw * sh];  
double rads = angle * Math.PI / 180.f;  
float sa = (float) Math.sin(rads);  
float ca = (float) Math.cos(rads);  
int isa = (int) (256 * sa);  
int ica = (int) (256 * ca);  
int my = &#8211; (sh >> 1);  
for(int i = 0; i > 1) \* ica + ((sw >> 1) <> 1) \* isa + ((sh >> 1) << 8);  
for(int j = 0; j > 8);  
int srcy = (yacc >> 8);  
if(srcx < 0) srcx = 0;  
if(srcy sw &#8211; 1) srcx = sw &#8211; 1;  
if(srcy > sh &#8211; 1) srcy = sh &#8211; 1;  
dstData[wpos++] = srcData[srcx + srcy * sw];  
xacc += ica;  
yacc -= isa;  
}  
my++;  
}  
g.drawRGB(dstData, 0, sw, 0, 0, sw, sh, true);  
}  
[/code]  
And if we know beforehand the size of the image we want to rotate we could do some extra tricks (for example, here assumes the source image will be 256&#215;256 pixels and will only work with that resolution):  
[code lang=&#8221;java&#8221;]  
public static void rotateImage(Image src, float angle, Graphics g) {  
int sw = src.getWidth();  
int sh = src.getHeight();  
int[] srcData = new int[sw * sh];  
src.getRGB(srcData, 0, sw, 0, 0, sw, sh);  
int[] dstData = new int[sw * sh];  
double rads = angle * Math.PI / 180.f;  
float sa = (float) Math.sin(rads);  
float ca = (float) Math.cos(rads);  
int isa = (int) (256 * sa);  
int ica = (int) (256 * ca);  
int my = &#8211; (sh >> 1);  
for(int i = 0; i > 1) \* ica + ((sw >> 1) <> 1) \* isa + ((sh >> 1) << 8);  
for(int j = 0; j > 8) & 0xff;  
int srcy = yacc & 0xff00;  
dstData[wpos++] = srcData[srcx + srcy];  
xacc += ica;  
yacc -= isa;  
}  
my++;  
}  
g.drawRGB(dstData, 0, sw, 0, 0, sw, sh, true);  
}  
[/code]  
These functions paints the rotated image in a Graphics object directly, but doing some minor changes it could generate a new Image with the content:  
* Change the function declaration to:  
[code lang=&#8221;java&#8221;]  
public static Image rotateImage_img(Image src, float angle) {  
[/code]  
* Replace the drawRGB call with:  
[code lang=&#8221;java&#8221;]  
return Image.createRGBImage(dstData, sw, sh, true);  
[/code]  
Feel free to use it for whatever you want but it would be nice if you drop me a line and put me somewhere in the credits 🙂  
![](http://labs.rafols.org/img.php?id=imgrot0-post)
