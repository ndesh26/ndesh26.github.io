---
layout:     post
title:      "A Personal Guide to Linux kernel's Makefile" 
date:       2018-07-27 16:31:19
excerpt_separator: <!--more-->
categories: Programming 
tags: [GSoC, Programming, linux]
comments:   true

---

I have been working on the linux kernel for the past 2 months as part of my GSoC and during that time I had to deal with the Makefile
in the Linux kernel. The Makefile has some handy targets which are not easy for a beginner to find. In this post I will talk
about some of the targets that I found useful during my work. 
<!--more-->

Here is an incomplete list of targets: 

* **The obvious ones**: These are the commands that are required to install linux kernel in your machine, these are the obvious ones, and you will find that every
    linux tutorial mentions them. I won't go into any more details. 
{% highlight bash %}
$ make -jN
$ make headers_install
$ make modules_install
$ make install
{% endhighlight %}

* **menuconfig**: This target provides a GUI interface for modifying the linux .config file. It is used to enable and disable various features and
    modules for the linux build
{% highlight bash %}
$ make menuconfig 
{% endhighlight %}

* **Building specific modules**: This command allows us to build specific modules/group of modules only. It is useful when you need
    to test a single module and want to avoid recompiling other parts of the linux kernel. This could be useful when testing a module for a
    bug. You need to use the `SUBDIRS` variable to point the module(s) that you want to build.
{% highlight bash %}
$ make -j3 modules SUBDIRS=drivers/gpu/drm/
{% endhighlight %}

* **defconfig**: This command generates a generic .config file to build the linux. You can also use this command to generate .config files for
    different architectures by using the `ARCH` variable. This is specially useful when you need a .config file for cross compiling kernel. 
{% highlight bash %}
$ make ARCH=arm defconfig
{% endhighlight %}

* **cscope**: This command generates the cscope database for the kernel. This makes reading kernel code a breeze. This target should be used
    with the `ARCH` variable only to generate the cscope database corresponding to the architecture with which you are working.
{% highlight bash %}
$ make ARCH=x86_64 cscope 
{% endhighlight %}

* **htmldocs**: As the name suggests it generates the html version of the kernel docs. It comes in handy for understanding code which is
    documented. The documentation for the lastest kernel is also available [online](https://www.kernel.org/doc/html/latest/). 
{% highlight bash %}
$ make htmldocs 
{% endhighlight %}

* **localmodconfig**: This a rather useful but obscure option. It creates a .config which is specific to your system. It only enables the
    modules that are loaded in you system when the command is being called. This allows us to create a .config file with only the minimal modules enabled
    to reduce the compilation time.
{% highlight bash %}
$ make localmodconfig 
{% endhighlight %}

I guess that completes the list that I used during my project. I hope it helps someone in need.
