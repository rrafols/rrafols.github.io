---
id: 97
title: 'Augmented Reality &#8211; II'
date: 2011-01-08T19:23:33+00:00
author: raimon
layout: post
guid: http://blog.rafols.org/?p=97
permalink: /index.php/2011/01/08/augmented-reality-ii/
restapi_import_id:
  - 599477c8431d4
original_post_id:
  - "97"
categories:
  - Android
  - Augmented Reality
tags:
  - mobile
---
In my last post I showed how to get position and orientation updates. In this short post (also because it&#8217;s quite simple) I&#8217;ll show how to integrate it with the camera preview.  
To show the camera preview in Android it&#8217;s quite easy, just create a class that extends from SurfaceView and implements the SurfaceHolder.Callback methods:  
[code lang=&#8221;java&#8221;]  
package com.fuzzion.argine.viewer;  
import android.content.Context;  
import android.hardware.Camera;  
import android.hardware.Camera.PreviewCallback;  
import android.view.SurfaceHolder;  
import android.view.SurfaceView;  
public class CameraView extends SurfaceView implements SurfaceHolder.Callback, PreviewCallback {  
private Camera camera;  
private boolean running = false;  
public CameraView(Context context) {  
super(context);  
SurfaceHolder holder = getHolder();  
holder.addCallback(this);  
holder.setType(SurfaceHolder.SURFACE\_TYPE\_PUSH_BUFFERS);  
setFocusable(true);  
requestFocus();  
}  
@Override  
public void surfaceChanged(SurfaceHolder holder, int format, int width, int height) {  
if(running) {  
camera.stopPreview();  
}  
try {  
camera.setPreviewDisplay(holder);  
} catch(Exception e) {}  
camera.startPreview();  
running = true;  
camera.setPreviewCallback(this);  
}  
@Override  
public void surfaceCreated(SurfaceHolder holder) {  
camera = Camera.open();  
}  
@Override  
public void surfaceDestroyed(SurfaceHolder holder) {  
try {  
if(camera != null) {  
camera.stopPreview();  
running = false;  
camera.release();  
camera = null;  
}  
} catch(Exception e) {}  
}  
@Override  
public void onPreviewFrame(byte[] data, Camera camera) {  
}  
}  
[/code]  
It&#8217;s really simple, just using the startPreview() / stopPreview methods. To overlay our previous view with the camera preview we have to do that in our Activity main class:  
[code lang=&#8221;java&#8221;]  
cv = new CameraView(this);  
tv = new TestView(this);  
setContentView(cv);  
addContentView(tv, new LayoutParams(LayoutParams.FILL\_PARENT, LayoutParams.FILL\_PARENT));  
[/code]  
We&#8217;re adding the camera preview and our previous viewer (removing the background, and changing the arrow color to white) so we can see both views at the same time.  
I&#8217;ve also implemented the PreviewCallback class to receive callbacks with the camera raw data (byte[]). It might be useful some day..  
![](http://labs.rafols.org/img.php?id=ar2-post)