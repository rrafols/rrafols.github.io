---
id: 327
title: 'BlackBerry 10 &#8211; Native algorithm optimization'
date: 2014-08-02T04:57:06+00:00
author: raimon
layout: post
guid: http://blog.rafols.org/?p=327
permalink: /index.php/2014/08/02/blackberry-10-native-algorithm-optimization/
restapi_import_id:
  - 599477c8431d4
original_post_id:
  - "327"
categories:
  - BlackBerry
  - BlackBerry 10
tags:
  - BlackBerry10
---
When looking for performance it&#8217;s important to understand what is going on at the compiler level and how the compiler is optimising our code. For example, enabling the assembly output will allow us to see what is exactly generating and have a greater understanding of why some parts of the code execute faster or the reason why it&#8217;s not performing as it should.  
BlackBerry 10 uses the QCC compiler (more information here: [QCC](https://developer.blackberry.com/native/reference/core/com.qnx.doc.neutrino.utilities/topic/q/qcc.html?f=qcc "qcc")). Enabling the right flags will generate a .s file of the generated assembly:  
[<img src="http://blog.rafols.org/wp-content/uploads/qcc_flags2.png" alt="qcc_flags2" class="alignnone size-medium wp-image-329" />](http://blog.rafols.org/wp-content/uploads/qcc_flags2.png)  
[<img src="http://blog.rafols.org/wp-content/uploads/qcc_flags3.png" alt="qcc_flags3" class="alignnone size-medium wp-image-328" />](http://blog.rafols.org/wp-content/uploads/qcc_flags3.png)  
Once we got the flags enabled we can start optimising the code. Let&#8217;s take as example a YUV2RGB converter and optimise it for BlackBerry 10 devices (used in this app <http://appworld.blackberry.com/webstore/content/21198027/>) . This converter can be used to read from camera preview buffers and convert it to RGB to save it as an image or show it on the screen. [YUV](http://en.wikipedia.org/wiki/YUV) is a format the encodes luma & chroma independently:  
[<img src="http://blog.rafols.org/wp-content/uploads/yuv_format.png" alt="yuv_format" class="alignnone size-medium wp-image-331" />](http://blog.rafols.org/wp-content/uploads/yuv_format.png)  
To make this example simpler, assume the following values (hardcoded from a camera preview buffer size)

<pre>int src_height = 768;
int src_width = 1024;
int src_stride = 1024;
int uvStart = 1024 * 768; // offset to the UV data (after all the Y data)
</pre>

This is our first approach:

<pre>void inline renderFrame(unsigned char *src, char *dst, int rect[4], int frame, int dst_stride) {
    int i, j;
    for (i = 0; i &lt; src_height; i++) {
        for(j = 0; j &lt; src_width; j++) {
            int y_rpos = i * src_stride + j;
            int uv_rpos = uvStart + ((i/2) * src_stride) + (j/2) * 2;
            int wpos = i * dst_stride + j * 4;
            float y = src[y_rpos     ];
            float u = src[uv_rpos    ];
            float v = src[uv_rpos + 1];
            int r = clip((int) ((y - 16) * 1.164                     + 1.596 * (v - 128)));
            int g = clip((int) ((y - 16) * 1.164 - 0.391 * (u - 128) - 0.813 * (v - 128)));
            int b = clip((int) ((y - 16) * 1.164 + 2.018 * (u - 128)));
            dst[wpos    ] = b;
            dst[wpos + 1] = g;
            dst[wpos + 2] = r;
            dst[wpos + 3] = 0xff;
        }
    }
}
</pre>

As with this YUV format a pair of Y values share the same UV values we can generate 2 pixels with just 2 Y and 1 single UV pair, generating 2 pixels per iteration.

<pre>float y0 = src[y_rpos    ];
float y1 = src[y_rpos + 1];
float u = src[uv_rpos    ];
float v = src[uv_rpos + 1];
int r0 = clip((int) ((y0 - 16) * 1.164                     + 1.596 * (v - 128)));
int g0 = clip((int) ((y0 - 16) * 1.164 - 0.391 * (u - 128) - 0.813 * (v - 128)));
int b0 = clip((int) ((y0 - 16) * 1.164 + 2.018 * (u - 128)));
int r1 = clip((int) ((y1 - 16) * 1.164                     + 1.596 * (v - 128)));
int g1 = clip((int) ((y1 - 16) * 1.164 - 0.391 * (u - 128) - 0.813 * (v - 128)));
int b1 = clip((int) ((y1 - 16) * 1.164 + 2.018 * (u - 128)));
dst[wpos    ] = b0;
dst[wpos + 1] = g0;
dst[wpos + 2] = r0;
dst[wpos + 3] = 0xff;
dst[wpos + 4] = b1;
dst[wpos + 5] = g1;
dst[wpos + 6] = r1;
dst[wpos + 7] = 0xff;
</pre>

So far so good, but there are many arithmetic operations that can be shared and there is no need to do them twice

<pre>float y0 = src[y_rpos    ] - 16;
float y1 = src[y_rpos + 1] - 16;
float u = src[uv_rpos    ] - 128;
float v = src[uv_rpos + 1] - 128;
float y0v = y0 * 1.164;
float y1v = y1 * 1.164;
float chromaR = 1.596 * v;
float chromaG = -0.391 * u - 0.813 * v;
float chromaB = 2.018 * u;
int r0 = clip((int) (y0v + chromaR));
int g0 = clip((int) (y0v + chromaG));
int b0 = clip((int) (y0v + chromaB));
int r1 = clip((int) (y1v + chromaR));
int g1 = clip((int) (y1v + chromaG));
int b1 = clip((int) (y1v + chromaB));
...
</pre>

and we&#8217;ve improved 14 multiplications and 20 additions/subtractions to 6 multiplications and 12 additions/substractions. Pretty sure our CPU would like that although we&#8217;re far from done. There are many values that doesn&#8217;t change in every pixel so we can move those to the external loop.

<pre>void inline renderFrame(unsigned char *src, char *dst, int rect[4], int frame, int dst_stride) {
    int i, j;
    for (i = 0; i &lt; src_height; i++) {
        int y_rpos = i * src_stride;
        int uv_rpos = uvStart + ((i/2) * src_stride);
        int wpos = i * dst_stride;
        for(j = 0; j &lt; src_width; j++) {
            float y0 = src[y_rpos    ] - 16;
            float y1 = src[y_rpos + 1] - 16;
            float u = src[uv_rpos    ] - 128;
            float v = src[uv_rpos + 1] - 128;
            float y0v = y0 * 1.164;
            float y1v = y1 * 1.164;
            float chromaR = 1.596 * v;
            float chromaG = -0.391 * u - 0.813 * v;
            float chromaB = 2.018 * u;
            int r0 = clip((int) (y0v + chromaR));
            int g0 = clip((int) (y0v + chromaG));
            int b0 = clip((int) (y0v + chromaB));
            int r1 = clip((int) (y1v + chromaR));
            int g1 = clip((int) (y1v + chromaG));
            int b1 = clip((int) (y1v + chromaB));
            dst[wpos    ] = b0;
            dst[wpos + 1] = g0;
            dst[wpos + 2] = r0;
            dst[wpos + 3] = 0xff;
            dst[wpos + 4] = b1;
            dst[wpos + 5] = g1;
            dst[wpos + 6] = r1;
            dst[wpos + 7] = 0xff;
            wpos += 8;
            y_rpos += 2;
            uv_rpos += 2;
        }
    }
}
</pre>

As some CPU doesn&#8217;t contain hardware accelerated floating point processors, floating point operations are emulated in software. In others, plain integer operations are usually faster. We can simulate floating point arithmetics with plain integers using [Fixed point arithmetics](http://en.wikipedia.org/wiki/Fixed-point_arithmetic)

<pre>int y0v = y0 * 298;   // 1.164 * 256
int y1v = y1 * 298;   // 1.164 * 256
int chromaR = 408 * v;                // 1.596 * 256
int chromaG = -100 * u - 208 * v;     // - 0.391 * 256, 0.813 * 256
int chromaB = 517 * u;                // 2.018 * 256
int r0 = clip((y0v + chromaR) &gt;&gt; 8);  // &gt;&gt; 8 is equivalent to dividing by 256
int g0 = clip((y0v + chromaG) &gt;&gt; 8);
int b0 = clip((y0v + chromaB) &gt;&gt; 8);
int r1 = clip((y1v + chromaR) &gt;&gt; 8);
int g1 = clip((y1v + chromaG) &gt;&gt; 8);
int b1 = clip((y1v + chromaB) &gt;&gt; 8);
</pre>

As YUV values are always 0 <= YUV <= 255 some operations can be already recalculated in small arrays

<pre>void precalc() {
    int i;
    for(i = 0; i &gt; 8;
        factorRV[i] = ( 408 * (i - 128)) &gt;&gt; 8;
        factorGU[i] = (-100 * (i - 128)) &gt;&gt; 8;
        factorGV[i] = (-208 * (i - 128)) &gt;&gt; 8;
        factorBU[i] = ( 517 * (i - 128)) &gt;&gt; 8;
        clipVals[i] = min(max(i - 300, 0), 255);
    }
}
</pre>

We also precalculated the clipping values, so we remove the comparisons in the inner loop.  
No we can use these precalc tables in our code:

<pre>int y0 = factorY[src[y_rpos    ]];
int y1 = factorY[src[y_rpos + 1]];
int chromaR = factorRV[v];
int chromaG = factorGU[u] + factorGV[v];
int chromaB = factorBU[v];
int r0 = clipVals[y0 + chromaR + 300];
int g0 = clipVals[y0 + chromaG + 300];
int b0 = clipVals[y0 + chromaB + 300];
int r1 = clipVals[y1 + chromaR + 300];
int g1 = clipVals[y1 + chromaG + 300];
int b1 = clipVals[y1 + chromaB + 300];
</pre>

We introduced a 300 offset into the array indexes to avoid problems when values are negative. We can avoid that index offset by doing the following:

<pre>int *clipVals_ = &clipVals[300];
...
int r0 = clipVals_[y0 + chromaR];
int g0 = clipVals_[y0 + chromaG];
int b0 = clipVals_[y0 + chromaB];
int r1 = clipVals_[y1 + chromaR];
int g1 = clipVals_[y1 + chromaG];
int b1 = clipVals_[y1 + chromaB];
</pre>

We could write pixels in one single operation instead of using 4 write operations (one per single byte). We&#8217;ll create different precalc tables to store values for the R, G and B components with shift masks and alpha transparency already in place.

<pre>for(i = 0; i &lt; 1024; i++) {
    clipVals[i]  = min(max(i - 300, 0), 255);
    clipValsR[i] = 0xFF000000 | (min(max(i - 300, 0), 255) &lt;&lt; 16);
    clipValsG[i] = min(max(i - 300, 0), 255) &lt;&lt; 8;
    clipValsV[i] = min(max(i - 300, 0), 255);
}
void inline renderFrame(unsigned char *src, char *dst, int rect[4], int frame, int dst_stride) {
    int i, j;
    int *clipValsR_ = &clipValsR[300];
    int *clipValsG_ = &clipValsG[300];
    int *clipValsB_ = &clipValsB[300];
    int *dsti = (int *) dst;
    for (i = 0; i &lt; src_height; i++) {
        int y_rpos = i * src_stride;
        int uv_rpos = uvStart + ((i/2) * src_stride);
        int wpos = i * dst_stride;
        for(j = 0; j &lt; src_width; j++) {
            int u = src[uv_rpos    ];
            int v = src[uv_rpos + 1];
            int y0 = factorY[src[y_rpos    ]];
            int y1 = factorY[src[y_rpos + 1]];
            int chromaR = factorRV[u];
            int chromaG = factorGU[u] + factorGV[v];
            int chromaB = factorBU[u];
            dst[wpos    ] = clipValsR_[y0 + chromaR] | clipValsG_[y0 + chromaG] | clipValsB_[y0 + chromaB];
            dst[wpos + 1] = clipValsR_[y1 + chromaR] | clipValsG_[y1 + chromaG] | clipValsB_[y1 + chromaB];
            wpos += 2;
            y_rpos += 2;
            uv_rpos += 2;
        }
    }
}
</pre>

**Execution time**  
Small graph showing the execution time of the same function with different optimisations. At the end we achieved around 600% performance increase.  
[<img src="http://blog.rafols.org/wp-content/uploads/exec_time.png" alt="exec_time" class="alignnone size-medium wp-image-348" />](http://blog.rafols.org/wp-content/uploads/exec_time.png)  
There are still quite a lot of things than can be done, stay tuned for the second part of this post!