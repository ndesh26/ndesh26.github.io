---
layout:     post
title:      "Installation guide for Antergos" 
date:       2018-11-11 16:31:19
excerpt_separator: <!--more-->
categories: Programming 
tags: [programming, linux]
comments:   true

---

Antergos is a distro based on Arch Linux. In this post, I will share my experience of installing Antergos with some of the problems that I faced
and their solutions. 

![Antergos](https://upload.wikimedia.org/wikipedia/en/9/93/Antergos_logo_github.png){: .center-image }

<!--more-->

I have been using fedora with i3 as Desktop Environment (DE) for about 2 years and did not face any errors during that time. But recently my
system started hanging a lot. It could have have been some package that I installed or just because of the OS being too old, but I was too
lazy to debug the issue and I wanted to try something fresh and hence decided to try a new distro (with a different DE) after a long time. 

I have wanted to install Arch linux for a long time, but the long installation scares me away each time. And the same happened this
time too, but while searching around arch I came to know about Antergos. We can say that it's a more user friendly version of arch and hence
proved an ideal choice for me. 

I have installed around 10-12 different linux distros and I consider myself to have a decent knowledge about the installation process, but
my self image was shattered when I tried installing Antergos. The installation of this distro became the longest one (~2days) that I have done yet.
Some because of the bugs in the Antergos installer and others because of my wrong doings. That's why I decided to document the problems that I
faced, as they might be of help to some lost souls. 

The [official guide](https://antergos.com/wiki/install/how-to-dual-boot-antergos-windows-uefi-expanded-by-linuxhat/) for installation is well
written so I won't be covering the things that are covered there. I will just document the things that are not mentioned there or the
problems that I faced.

**Problem 1**: The installer fails when checking for the package links because certain packages are not available online.

**Solution**: The reason for the problem is that the antergos mirrors are not configured. This can be verified by running `pacman -Syy`. The
output of the command would be:

```bash
[root@dell ndesh]# pacman -Syy
:: Synchronizing package databases...
error: failed to update antergos (no servers configured for repository)
 core                     132.3 KiB   945K/s 00:00 [##########] 100%
 extra                   1650.6 KiB  1399K/s 00:01 [##########] 100%
 community                  4.7 MiB  1690K/s 00:03 [##########] 100%
 multilib                 174.6 KiB   887K/s 00:00 [##########] 100%
error: failed to synchronize all databases
```

This can be fixed by copying the below snippet in the file `/etc/pacman.d/antergos-mirrorlist`:

```
## Antergos Repository Mirrorlist

# Automated Mirror Selection

Server = http://mirrors.antergos.com/$repo/$arch

# Manual Mirror Selection

# Bulgaria
Server = http://mirror.host.ag/antergos/$repo/$arch
Server = https://mirrors.itbox.bg/antergos/$repo/$arch

# China
Server = https://mirrors.ustc.edu.cn/antergos/$repo/$arch
Server = https://mirrors.tuna.tsinghua.edu.cn/antergos/$repo/$arch

# Czech Republic
Server = https://mirrors.nic.cz/antergos/$repo/$arch

# Denmark
Server = https://mirrors.dotsrc.org/antergos/$repo/$arch

# France
Server = http://cinnarch.polymorf.fr/$repo/$arch
Server = https://eu.mirrors.coltondrg.com/antergos/$repo/$arch

# England
Server = https://www.mirrorservice.org/sites/repo.antergos.com/$repo/$arch

# Germany
Server = https://mirror.de.leaseweb.net/antergos/$repo/$arch
Server = https://mirror.alpix.eu/antergos/$repo/$arch

# Greece
Server = https://ftp.cc.uoc.gr/mirrors/linux/antergos/$repo/$arch

# Hungary
Server = https://quantum-mirror.hu/mirrors/pub/antergos/$repo/$arch

# Japan
Server = http://mirror.antergos.jp/$repo/$arch

# Netherlands
Server = https://mirror.nl.leaseweb.net/antergos/$repo/$arch
Server = https://ftp1.nluug.nl/os/Linux/distr/antergos/$repo/$arch
Server = https://ftp2.nluug.nl/os/Linux/distr/antergos/$repo/$arch
Server = https://mirror.neostrada.nl/antergos/$repo/$arch

# Portugal
Server = https://glua.ua.pt/pub/antergos/$repo/$arch

# Russia
Server = https://mirror.yandex.ru/mirrors/cinnarch/$repo/$arch

# Spain
Server = http://softlibre.unizar.es/cinnarch/$repo/$arch

# Sweden
Server = https://ftp.acc.umu.se/mirror/antergos.com/$repo/$arch

# USA
Server = https://mirror.us.leaseweb.net/antergos/$repo/$arch
Server = https://mirror.umd.edu/antergos/$repo/$arch
Server = https://mirrors.acm.wpi.edu/antergos/$repo/$arch
Server = https://mirrors.servercentral.com/antergos/$repo/$arch
Server = https://antergos.mirror.constant.com/$repo/$arch
Server = https://repo.antergos.info/$repo/$arch
```

Running the `sudo pacman -Syy` should fix the error. Now we need to restart the installer.

**Problem 2**: The installer does not show the option for UEFI install. In this case, the installation will not proceed until you have
specified the `/boot` partition (even if you have configured the `/boot/efi` mountpoint).

**Solution**: This was a problem with my PC as I had also
enabled the legacy mode (at some point in the past for unknown reasons) and the installer picked that up and hence was not allowing a UEFI
install. Once you disable the legacy boot, the installer will automatically give the option for a UEFI install.

It took me a while to figure out these problems especially the first one. While there were a couple more problems but I won't be discussing
them as they were caused by my own negligence and hence are not important from the viewpoint of this post. Now I am ready to give Antergos a test run.




