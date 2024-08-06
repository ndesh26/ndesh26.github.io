---
layout:     post
title:      Surface Go 2 - Alternate Operating Systems
date:       2023-07-22 18:31:19
excerpt_separator: <!--more-->
categories: Programming
tags: [programming,linux]
comments:   true

---

<img class="center-image" src="{{ site.baseurl }}/assets/images/surface-go-2.jpg" alt="drawing" style="width: 65%;"/>

I’ve been exploring alternative operating systems for the Surface Go 2 (4GB RAM, 64GB storage) with an Intel Pentium Gold 4425Y processor. Lately, I've been using the Surface Go more as a tablet and found that Windows 11, despite improvements over Windows 10, still falls short as a tablet OS. Additionally, Windows 11 feels somewhat clunky on the Surface Go 2's minimal specs. I’ve tested several operating systems, including Linux distributions, Android x86, and Chrome OS. Here’s a brief review of each:

<!--more-->

## Bliss OS

Version: 16.9.6

Bliss OS allows running Android on x86 hardware. The installation process was straighforward and most of the hardware was working right out of the box except for the cameras. It supports a full android tablet experience along with play store. The tablet experience was just like an android tablet. And for the desktop mode specific launchers are provided to emulate it. The desktop mode was a bit disappointing with the launchers not baked into the system. They always felt like an additional piece glued on top of a complete system. At times I ended up with 2 taskbars appearing simultaneously. The system was running smoothly as compared Windows 11 but it was still laggy at times. It was also not able to detect the attachment of type cover and change to desktop mode automatically. 

## Chrome OS Flex

Version: 126 (15886.69.0)

ChromeOS Flex is a version of ChromeOS provided by google to run on conventional PC hardware which are not chromebooks. The creation of the bootable USB can be done using the chromebook recovery utility (by selecting ChromeOS flex as the model). The installation is straightforward, just be careful in which drive you are installing the OS to (I do recommend using the commandline option for the installation). 

The OS runs very smoothly and was very snappy for most of the tasks that I performed on it. It supports a full fleged linux distribution running as container which allows installing all the linux applications including the GUI ones too. The hardware support is really good and everything work out of the box except for the cameras. It was able to detect the type cover and change between desktop and tablet moe automatically. The main limitation comes from the fact that we are restricted in terms of the applications. The webapps provide good support but they are not as functional as the android apps when it comes to tablet mode.

## Fyde OS

Version: Fyde for You 18.1 

<img class="center-image" src="{{ site.baseurl }}/assets/images/fyde-os-1.png" alt="drawing" style="width: 65%;"/>

Fyde OS is a variant of the ChromeOS and they have a version specifically made for the Surface Go 2, called 'Fyde for You'. The installation is straightforward and everything works like a charm right away even the cameras. You can feel that OS is specifically targeted towards surface go 2 devices.  It ran very smoothly just like the ChromeOS that it is based on. It supports both Android apps and Linux apps in the platform making it ideal for having the right app ecosystem for both tablet and desktop mode. The only additional step I had to do was to to register the Google play service ID on my google account as this device is not a certified device and after that I was able to install apps normally.

The only negative of Fyde OS is the money. They provide the Fyde for You version on a subscription basis with the price of $14.99 per year. The free installation comes with a 90 day trial after which they require you to purchase the subscription. 

## Linux (Fedora)

Version: 40

<img class="center-image" src="{{ site.baseurl }}/assets/images/fedora.png" alt="drawing" style="width: 65%;"/>

I used the fedora workstation image with GNOME3 as that was the Desktop Environment (DE) with the best touchscreen support. The installation was straightforward and hardware support was great too, except for the common culprit, the camera. GNOME 3 has a single interface for both desktop and table mode which offers a somewhat mixed experience, lacking in both areas. The experience was also not as great due to absence of fractional scaling (only 100% and 200% are supported). You are either stuck with title bars which are too big or too small. The system also felt a big sluggish during the limited time that I used it. GNOME3 is one of the heavier DEs in terms of the RAM usage. 

It is possible to get the cameras working by using the linux-surface kernel, but I didn't give it a try.

## Conclusion

For me Fyde OS is the clear winner among the rest. It is too soon to say if it is worth the 15$ per year though. The 90 days trial is good enough to find out. ChromeOS Flex can be a good option for anyone not planning on using the tablet mode as much. Similarly bliss OS can be an good option for someone planning to use the surface go as majorly a tablet device. 