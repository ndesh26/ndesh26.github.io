---
layout:     post
title:      Writing Radio Physical Layer from scratch - Part 1
date:       2020-12-06 15:31:19
excerpt_separator: <!--more-->
categories: Programming
tags: [programming, wireless, arduino]
comments:   true

---
In this blog series we will write the code for the physical layer for a simple radio device to understand how physical layer works.

<!--more-->

For this series we will use the 433MHz RF wireless module. Its a very simple radio transmitter and you can get the a set of Tx and Rx for less 2$. They use Amplitude Shift Keying (ASK), which means they transmit a high signal for value 1 and a no signal for bit 0. They use a carrier wave of 433MHz (and hence the name) to transmit data. You can find a more detailed description of the hardware [here](https://lastminuteengineers.com/433mhz-rf-wireless-arduino-tutorial/). I borrowed a fews pics from the website which are self-explanatory.


# Let's start with the code

