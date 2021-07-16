---
id: 88
title: 'Augmented Reality &#8211; I'
date: 2010-07-26T00:43:05+00:00
author: raimon
layout: default
guid: http://blog.rafols.org/?p=88
permalink: /index.php/2010/07/26/augmented-reality-i/
restapi_import_id:
  - 599477c8431d4
original_post_id:
  - "88"
categories:
  - Android
  - Augmented Reality
tags:
  - mobile
---
Since I had my first android phone I got curiosity into augmented reality. I have to say, compared to j2me, android is a lot more powerful and you get surprised about how easy is to achieve some things than in j2me are near impossible or rather complicated. Also seems that AR is some kind of teenager fashion, now it&#8217;s really cool to do things in AR instead of just showing a map with POIs. Let&#8217;s have the POIs floating around the user, even if it&#8217;s more confusing than showing a map, but, hey! it&#8217;s cooler.  
Anyway, I had to try it by myself, otherwise I couldn&#8217;t apply the cliché of having an android phone and did some AR tests. I&#8217;ll introduce the engine I developed with different articles, explaining few parts of it, and if anyone is interested I can provide access to the svn server where the code is stored (send me an email to raimon.rafols [a] gmail.com). You can use this code for any non-commercial project you want but you need the written permission of the author for commercial usage.  
One of the basic parts needed for an AR engine is to know the exact location and orientation of the device. Otherwise it won&#8217;t be possible to show the correct POIs and in the correct position in the screen. Here is a code snippet of the orientation provider.  
[code lang=&#8221;java&#8221;]  
package com.fuzzion.argine.hal.android;  
import java.util.ArrayList;  
import java.util.Collections;  
import java.util.List;  
import android.app.Activity;  
import android.content.Context;  
import android.hardware.Sensor;  
import android.hardware.SensorEvent;  
import android.hardware.SensorEventListener;  
import android.hardware.SensorManager;  
import android.util.Log;  
import com.fuzzion.argine.engine.OrientationListener;  
import com.fuzzion.argine.hal.OrientationProvider;  
public class AndroidOrientationProvider implements OrientationProvider, SensorEventListener {  
private SensorManager sManager;  
private List listeners;  
public AndroidOrientationProvider() {  
listeners = Collections.synchronizedList(new ArrayList());  
}  
@Override  
public void onAccuracyChanged(Sensor sensor, int accuracy) {}  
@Override  
public void onSensorChanged(SensorEvent event) {  
float values[] = event.values;  
switch(event.sensor.getType()) {  
case Sensor.TYPE_ACCELEROMETER:  
break;  
case Sensor.TYPE_ORIENTATION:  
for(OrientationListener listener : listeners) {  
listener.directionChanged(values);  
}  
break;  
}  
}  
@Override  
public void registerOrientationListener(OrientationListener listener) {  
listeners.add(listener);  
}  
@Override  
public void removeOrientationListener(OrientationListener listener) {  
listeners.remove(listener);  
}  
public void start() {  
Activity act = AndroidPlatform.getActivity();  
sManager = (SensorManager) act.getSystemService(Context.SENSOR_SERVICE);  
sManager.registerListener(this, sManager.getDefaultSensor(Sensor.TYPE\_ORIENTATION), SensorManager.SENSOR\_DELAY_FASTEST);  
}  
public void stop() {  
if(sManager != null) {  
sManager.unregisterListener(this);  
}  
}  
}  
[/code]  
In the next entry I&#8217;ll explain the integration of all the parts (camera, orientation and position). Meanwhile you can check the code in svn.  
![](http://labs.rafols.org/img.php?id=ar1-post)
