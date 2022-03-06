---
layout:     post
title:      Writing an OS for RPi B+ - Intro 
date:       2022-03-06 18:31:19
excerpt_separator: <!--more-->
categories: Programming
tags: [programming, os, rpi]
comments:   true

---
I got my hands on a second hand [Raspberry Pi 1 B+](https://www.pololu.com/product/2752) for just 4$. For those who don't know it's a credit card sized computer
capable of running linux. It is powered by a ARM processor and has 512MB of RAM. It also had an ethernet port, HDMI output, 3.5mm audio jack and 4 USB ports.
It's a popular hardware among the electronic hobbyists.

<img class="center-image" src="{{ site.baseurl }}/assets/images/rpib+.jpg" alt="drawing" style="width: 50%;"/>

<!--more-->

I have been thinking for quite some time to work on an OS project. But I couldn't either find the time and the right hardware to target. I did start a minor project
targetting a generic x86 VM on qemu. For me personally, I only enjoy tinkering when I can see my code run on a real hardware. The Rpi was perfect for this. It was 
not too complicated to understand and yet complicated enough to require a OS of it's own.

I don't plan on writing a blog series or such on the topic as there are already numerous blogs available on this topic (However I couldn't find one targetting RPi 1).
But I do wish to document the challenges that I face, the progress that I make and the design decisions that I take, mostly for me to reference it later in future.
Before starting anything it's important to define the scope of the project. My initial target is to achieve: 

1. First and foremost, being able to run code on real hardware. This would require performing the boot up sequence and setting up the memory
2. Use the HDMI output to print output on screen
3. Take input from keyboard via usb port

The other things that can be done after the initial milestones are achieved:

4. Separate userspace from kernel space
5. Support basic graphics 
6. I do want to write Network stack here but that's just me being too ambitious

I plan on using C++ for this project as I already use it in my job. I don't have experience of working with the ARM architecture, so I need to study up a bit on that
side as well. The next post will cover more details on the environment setup for developing the project.
