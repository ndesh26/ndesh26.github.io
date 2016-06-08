---
layout:     post
title:      Introduction to VDPAU 
date:       2016-06-06 15:31:19
excerpt_separator: <!--more-->
categories: Programming 
tags: [mesa, linux, graphics]
comments:   true

---
I have been working with VDPAU this whole time and have gained a good insight into its working, So here's a post describing my experience. VDPAU is 
a API similar to OpenGL. It stands for Video Decode and Presentation API for Unix. And as the name suggests it deals with decoding and processing of 
video using the hardware. 
<!--more-->
VDPAU allows applications to use acceleration hardware present in the GPU. VDPAU was developed by nvidia which then open sourced the library. 

### Implementation 

To implement VDPAU we need to provide a userspace API to allows user applications to access hardware acceleration. This is provided by Nvidia as libvdpau
which is available for most of the linux distributions. The other thing that you need is implemetation of VDPAU in your drivers. While propietary 
Nvidia drivers have full VDPAU support, the Open source drivers have not implemented the entire API. Though the open source driver have the entire 
basic API implemented, the extra features are yet to be implemented. 

Now we need applications which can utilize VDPAU, as of now vlc, mplayer, Gstreamer, mpv are some of the software that offer VDPAU support.

### The VDPAU API

The API consists of various objects which, will look at the most important ones.

##### VdpDevice.

A VdpDevice is the root object in VDPAU's object system. From VdpDevice object handle, all other API entry points can be retrieved and invoked.

##### VdpVideoSurface

VdpVideoSurface contains planes on which values for each pixel are loaded in [YCbCr](https://en.wikipedia.org/wiki/YCbCr) format. It is first
object to which pixel data is copied from video file frame by frame. 

##### VdpOutputSurface

VdpOuputSurface contains pixel values which can be rendered on display. As the name suggests it represents the final surface that we see on our screen.

##### VdpBitmapSurface

VdpBitmapSurfce contains read only RGB values to create application UI for displaying the video. You cannot render to a VdpBitmapSurface, just upload 
native data from the memory.

##### VdpDecoder
VdpDecoder reads data from memory buffer and decodes it and stores the result in a VdpVideoSurface. 

##### VdpVideoMixer
VdpVideoMixer is responsible from rendering a VdpOutputSurface from a VdpVideoSurface. VdpVideoMixer converts YCbCr values to RGB values and adds
various overlays implementing features like noise reduction, temporal deinterlacing etc.

##### VdpPresentationQueue
VdpPresentationQueue contains a list of VdpOutputSurface with a time stamp at which they are displayed on screen.

This image represents data flow from memory to the screen through VDPAU API.
<p style="text-align:center">
<img style="width:500px;" src="{{ site.baseurl }}/assets/images/vdpau_data_flow.png"> 
</p>

In the upcoming post we will try using the VDPAU API to make a simple player of our own.
