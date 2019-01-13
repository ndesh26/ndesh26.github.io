---
layout:     post
title:      "VideoCast: Update" 
date:       2019-01-13 16:31:19
excerpt_separator: <!--more-->
categories: Programming 
tags: [Programming]
comments:   true

---

The initial project plan went through a total makeover when I found some new things in my research. The [Smart View SDK](https://developer.samsung.com/smart-view)
being the most prominent finding.

<!--more-->

The Smart View SDK allows us to broadcast content from the mobile app to the Smart TV without requiring a software at the television end. It
allows us to play files that are present in the phone as well as stream videos from the web. It was ideal for me as I needed http streaming
for my application.

In addition to this, I found this [github repo](https://github.com/SamsungDForum/SmartViewSDKDefaultMediaPlayer2.0/tree/master/DMP_Adroid/DefaultMediaPlayer2.0)
which has a sample app which streams video from pre-encoded links. I just had to modify the app a bit to add another fragment with webview
and extract the video links from the webpage that the is opened in the webview. 

With the basic features working the only thing that remained was perfecting the UI bugs and handling corner cases, but before I could do
that I found an app which does all this with minimal ads. I really love the [app](https://play.google.com/store/apps/details?id=cast.video.screenmirroring.casttotv&hl=en)
and it is exactly what I had to accomplish (maybe even better
:P). And with my project died a silent death even before it could mature. 
