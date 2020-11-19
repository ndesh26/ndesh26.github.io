---
layout:     post
title:      "Introduction to Wireless Physical Layer"
date:       2020-11-20 16:31:19
excerpt_separator: <!--more-->
categories: Programming 
tags:       [programming, wireless, arduino]
comments:   true

---

Recently I started working on the physical layer of 5G NR. And whenever any of my friends asked me about my work, I always ended giving
them a primer on wireless communication before I could start talking about my work. And hence this blog to generate awareness about the
area that I have been working on for the past few months.


<!--more-->

Let's start from the beginning.

### Physical Layer

The physical layer (the lowest layer in the OSI model of computer networks) is responsible for transmitting raw data bits on a
physical medium. The physical layer converts the data bits into physical properties like electrical signals, radio waves or even [pigeons](https://en.wikipedia.org/wiki/IP_over_Avian_Carriers).  The receiver then detects these physical changes to enable the transfer of information. In the case of Internet, the most popular physical layer is Ethernet. This refers to the LAN cable through which we get our internet connection. The physical layer is responsible for ensuring the integrity of the data transmitted. It needs to deal with modification/loss of data when the data is travelling through the physical medium. It uses checksums and retransmissions to that end to handle it.

### Wireless Physical Layer

The wireless phyical layer is a totally different ball game than the wired one and comes with its challenges: 

* **Attenuation**: The signal gets weaker as it travels farther through the air (as compared to copper wire).
* **Interference**: There is a possibility of different radio signals interfering with each other.
* **Multipath**: Even the same signal self interferes with itself. The receiver not only receives the signal from the transmitter directly but also the same signal reflected (and delayed) through other places.

![Multipath]({{ site.baseurl }}/assets/images/multipath-reflections.png)

### Cellular Networks

Cellular networks are a special case of wireless communication with its caveats, where we need to deal with many to one communication at a big scale. Each network tower is responsible for receiving/transmitting data to all the mobile devices that come in its area (called a cell). Cellular networks deal with this by using various multiplexing schemes (time, frequency, code) simultaneously. 

With the advent of 5G, the buzz word in the physical layer has been MIMO (Multiple Input Multiple Output). Both receiver and transmitter use multiple antennas to transmit/receive the signal. Even though the antennas are located at the same location, they receive a signal which is slightly different than each other. We exploit this difference to send more data over the same physical channel (in simple terms, at the same frequency).

![MIMO]({{ site.baseurl }}/assets/images/MIMO.png)


### Virtualization

Traditionally physical layer was implemented in the hardware itself. It was largely due to the fact that physical layer was not complicated and required fast processing. But with the arrival of new technologies, the wireless physical layer is becoming increasingly complex. There has been a trend towards shifting processing from hardware to software. This hardware virtualization provides better flexibility, reconfigurability and programmability. Virtualization is cost-effective as compared to its hardware counterpart. This is the reason why software engineers like me are working in this field.

### My work

We have covered enough material where I can start talking about my work. I'm still working as a software engineer. I work at the physical layer of 5G NR (New Radio). The physical layer receives data from the antennas in the form of complex numbers (each wave has a phase and amplitude which could be represented as a complex number). We first need to remove the interference from the environment and then separate the data for each cellphone device. The next step is error correction, validity check and restransmission. A lot of the algorithms that I use for this come from the realm of signal processing. I'm not responsible for designing these algorithms (the algorithms team does that). I take care of optimizing them and ensuring that we can process the data before your youtube video starts buffering. 

We employ different optimization techniques like vectorisation, rearranging memory format, dynamic profiling and analysis, and sometimes optimizing the algorithms themselves. I mostly work in c++, but I sometimes need to look at the assembly code to understand the bottlenecks in the system.  

Now you know a bit about my work and the new trends in wireless communication. 