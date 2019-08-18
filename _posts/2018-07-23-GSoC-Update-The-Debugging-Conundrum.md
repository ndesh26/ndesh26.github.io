---
layout:     post
title:      "GSoC Update: The Debugging Conundrum" 
date:       2018-07-25 16:31:19
excerpt_separator: <!--more-->
categories: GSoC 
tags: [gsoc, programming]
comments:   true

---

<p style="text-align:center">
<img class="center-image" src="{{ site.baseurl }}/assets/images/debug_done.jpg" alt="drawing" align="center" style="width:400px"/>
</p>
<!--more-->
At last, I was able to get my code to work!! 😀 😀 😀

In my last [update]({{ site.baseurl }}{% post_url 2018-07-06-GSoC-Update-The-Implementation-Ahead %}), I covered the plan that I was
going to follow to implement dynamic scheduling. In this post, I will talk about the problems that I faced during the implementation and

how I solved some of them. 

Coding the implementation was not all that difficult, and it took me a couple of days to get it done. The major hurdle was getting my code to
be functionally same as earlier code. Most of the time was spent in debugging. It was my first time modifying kernel modules, and hence I was not
used to the general flow of debugging for kernel modules. It took me a while to get used to it. One major hurdle was that in my system I
could not unload the amdgpu module once it is loaded. This was slightly bad as anytime I made a change I had to restart my PC to see the
difference.

A little investigation showed that Xserver was binding the amdgpu module. I didn't look any further into the issue and started working without
the Xserver whenever I had a series of changes to test. I was not used to working without a GUI (even though I use i3wm :P), It took me a
while to get adjusted to it. One major drawback of no GUI is the inability to use the browser and the internet thereof. 

I had written a load test in libdrm which I could use to test my patches. I haven't talked about it in any of the previous posts. But I
wrote the test code even before touching the scheduler code as Christian suggested that its best to have a usecase first before tweaking the
scheduler. And he was right; the usecase came in handy during debugging. The usecase that I wrote was a simple one which issued jobs to the
ring sdma0 ring only. My patch should distribute the load that was issued to sdma0 between the other SDMA engines. 

With the usecase and my code ready I set out to test out my code, but I quickly ran into a bunch of segfaults. The major difficulty with segfaults
for me was that I had no idea how to debug kernel
modules with GDB. Then Christian helped me with the debugging. Whenever a segfault happens, debug information is printed in the dmesg log.
Christian explained to me how to use that information to get the line which is leading to segfault. In his own words:

> you can load the kernel module into gdb and then let gdb resolve the
> address, e.g. something like the following:
>
> > (gdb) /lib/modules/4.18.0-rc1+/kernel/drivers/gpu/drm/amd/amdgpu/amdgpu.ko
> ...
> 
> Then in the gdb prompt use the l (list) command:
> > (gdb) l *(dce100_set_bandwidth+0x94)
> 
> The result is:
>```
>0x2d1cd4 is in dce100_set_bandwidth
>(drivers/gpu/drm/amd/amdgpu/../display/dc/dce100/dce100_hw_sequencer.c:166).
>161            struct dc_state *context,
>162            bool decrease_allowed)
>163    {
>164        struct dc_clocks req_clks;
>165
>166        req_clks.dispclk_khz = context->bw.dce.dispclk_khz * 115 / 100;
>167
>168  dce110_set_safe_displaymarks(&context->res_ctx, dc->res_pool);
>169
>```
> To exit gdb, just type q (quit):
> > (gdb) q 

It took me a while before I got rid of all the segfaults. The reasons behind segfaults turned out to be pretty trivial like accessing a NULL
pointer, accessing an area which was not allocated and so on. After that, I was able to run the module successfully. However, the libdrm
test still had a deadlock.

The deadlock was the last hurdle in my successful implementation. Finding the reason for the deadlock was not an
easy task I went through the code a countless number of times, printed different log messages to get an idea what was going on. Finally, Christian
suggested me to cleanup the code a bit and remove the redundancy from the scheduler API. Removing the redundancy worked wonders for me as
that was the hidden reason for the deadlock. A lot of structures and functions calls had extra fields, and whenever I tried rescheduling
jobs, I had to ensure that all the fields were synchronized. Removing this redundant field made my job easier and my code was
finally running. The meme in the beginning describes my joy of that time pretty well.

I also found a couple of memes that accurately define the state I was in those days. My state of mind kept shifting from meme 1 to meme 2 and vice
versa. 

<p style="text-align:center;">
<img src="{{ site.baseurl }}/assets/images/debug_no_error.jpg" alt="drawing" style="margin:2%;width: 51%"/>
<img src="{{ site.baseurl }}/assets/images/debugging.jpg" alt="drawing" style="width: 29%;margin: 2%"/>
</p>

Now I have a working code the only thing that remains for my project is to get the code committed before the last phase of the GSoC ends
:D

I have started sending the patches to the list already; you can see live action by following the list directly. 

Note: All the memes used in this post are taken from Google Images.


