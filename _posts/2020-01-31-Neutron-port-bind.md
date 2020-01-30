---
layout:     post
title:      "Manually binding Neutron port"
date:       2020-01-31 16:31:19
excerpt_separator: <!--more-->
categories: Programming 
tags: [programming, linux, openstack, networking]
comments:   true
published:  false

---
In this post we will see how we can manually bind a [Neutron]() port to a host. In an openstack the nova service is responsible for
binding the port to the host and attaching it to the virtual machine. 

This post is meant for a very niche section of people and I won't be bothering with covering the background required to understand this
post as it will take an entire post series to do so. Let's get started.

# Why is it needed 

I encountered this when I was working with gre tunnels with neutron networking. If you create a neutron network network over a gre
tunnel, your VMs communicate through the tunnel. But you cannot acccess this network from the compute host. After looking for various
ways to allow compute host to communicate to the VMs, I came up with the idea of creating a neutron port and manually binding it to the
compute host. Once the binding is done, an interface will be created on the compute host which will have IP from the gre network and
will have access to the gre network.

# Problems faced

The neutron agent creates two bridges, br-tun and br-int. The br-int bridge is the one on which the tap interfaces from VMs get
attached. And br-tun is the bridge which is responsible for sending and recieving the packets from the gre tunnel. The OVS flows rules
for the br-tun are created by the neutron agent and get updated whenever the neutron agent is restarted. So we want to have an
additional ovs interface to allow host communication we need to have it on br-int as adding it to br-tun will require adding additional
rules to br-tun every time the neutron agent flushes the existing rules. 

So we can have our new port on br-int, well and good. But the neutron agent also adds rule in br-tun to only allows packets that
orginate from one of the neutron ports (it has rules which check source MAC address of packets). So even we were to add a port to
br-int it won't be able to send and recieve packets. 

Hence we need to provision a neutron port and manually bind it on the host to allow for communication to the gre network.


# How to do it 

First we need to create a neutron port on the gre network 

{% highlight bash %}
$ openstack port create --network gre-net
{% endhighlight %}

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
