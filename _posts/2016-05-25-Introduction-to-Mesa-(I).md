---
layout:     post
title:      Introduction to Mesa (I)
date:       2016-05-25 15:31:19
excerpt_separator: <!--more-->
categories: Graphics
comments:   true

---
I have been working on video post processing on mesa for the past two weeks, So I thought I could also write about it so it might help someone who 
wants to contribute to mesa. So I will be doing a series of post regarding mesa, linus grahics stack ,video processing and more.
<!--more-->
This is the first post in a series of posts to come. It will serve as a introduction to mesa and the linux graphics stack. 

To understand the role of mesa and X Windows System in its current state we need to look at history of linux graphics stack and its evolutions 
over the years.

### X Windows System
To display something on the screen all graphics cards generate a “frame buffer”, ie a block of memory with 3 bytes per pixel, being the RGB color to show at that pixel 
on the screen. One frame-buffer known to the graphics card is the “scan-out buffer”, ie the one that will be displayed. X Windows System has a X Server which does the
job of drawing the frame-buffer for different X Clients. The X Clients follow a protocol to give instruction to X Server as to what needs to be displayed. This protocol 
is called the X11 protocol.
<p style="text-align:center">
<img style="width:400px;" src="{{ site.baseurl }}/assets/images/xserver.png"> 
</p>
X Server manages a window hierarchy:

1. root window = desktop wallpaper
2. top-level windows = application windows
3. subwindows = controls (buttons etc.)

X Clients don’t implement the X11 protocol directly, but use libraries like Xlib or XCB. Toolkits like Gtk, Qt use Xlib or XCB internally to display applications.
A Windows Manager is a special that manages the positions of the top-level windows and draws frame around them. X Server also manages the input  from keyboard, 
mouse and other input devices.

### OpenGL
The above covers 2D graphics as that is what the X server used to be all about. However, the arrival of 3D graphics hardware changed the scenario significantly, 
as we will see now. It led to creation of a standard API. Open Graphics Library (OpenGL) is a application programming interface (API) for rendering 2D and 3D vector graphics. The API is typically used to 
interact with a graphics processing unit (GPU), to achieve hardware-accelerated rendering.

However, in a system where only the X server was allowed to access the graphics hardware we could not have a openGL implementation that talked 
directly to the 3D hardware. Instead, the solution was to provide an implementation of OpenGL that would send OpenGL commands to the X server 
through an extension of the X11 protocol and let the X server translate these into actual hardware commands as it had been doing for 2D commands 
before.
We call this Indirect Rendering, since applications do not send rendering commands directly to the graphics hardware, and instead, render 
indirectly through the X server.
<p style="text-align:center">
<img src="{{ site.baseurl }}/assets/images/indirect.png">
</p>
But developers would soon realize that this solution was not sufficient for intensive 3D applications, such as games, that required to render 
large amounts of 3D primitives while maintaining high frame rates. To achieve good performance using 3D hardware, we need to allow direct access
to hardware. 

The solution to this problem was direct rendring infrastructure(DRI) which we will cover in the next post and also get to see how mesa fits in.
