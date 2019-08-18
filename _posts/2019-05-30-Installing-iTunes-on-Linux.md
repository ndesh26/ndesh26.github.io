---
layout:     post
title:      "Installing iTunes on Linux" 
date:       2019-05-31 16:31:19
excerpt_separator: <!--more-->
categories: Programming 
tags: [programming, linux]
comments:   true

---

After struggling with it for about a month, I was finally able to get iTunes working on my Linux system. There are a lot of tutorials explaining how
to install iTunes with wine but none of them mention any specific version of wine and iTunes that work successfull; hence I am writing one
myself. But before we start here is a screenshot to cheer you up.
<img class="center-image" src="{{ site.baseurl }}/assets/images/iTunes.png" style="width:600px;padding:30px"/>

<!--more-->

## The basic installation

1. First, install wine in your system and the version that worked for me was 4.9. You can find a bunch of tutorials to installing it on your distro. 

2. Next we need to create a wineprefix.\\
    ```bash
    winecfg
    ```
    This will ask for the installation of Wine mono which we won't be needing for running iTunes. Now a lot of tutorials stressed on using 32bit version
    of iTunes but I was able to get both 32 and 64 bit to work on my system. In this tutorial I will be focusing on 64bit version. When the winecfg finally 
    opens up you need to select a Windows version >= Windows7. The rest of the default settings should be fine.

3. Now we need to download the iTunes installer. I was able to get 12.9.0 to work on my system. Any higher versions ran into problems with
   wine.

4. Run the installer with wine.

    ```bash
    wine ~/Downloads/iTunes64Setup-12.9.0.exe
    ```
    Just follow all the usual installation steps nothing special to do. 

5. By this point iTunes should be running without any problems on your system. It will create a shortcut on your Desktop from where you can
   access it. If it is not created for some reason you can use this command to run it manually.
   ```bash
    wine "C:\\\\Program Files\\\\iTunes\\\\iTunes.exe"
    ```


## Making iTunes faster

iTunes by default runs slower with wine and gets stuck when scrolling through the album arts. One way to optimize it is to use vulkan with wine.

1. Install winetricks on you distro which should be available with you package manager (apt, yum, pacman)
2. Run winetricks and select the "Select the default wineprefix" and then select the "Install a Windows DLL or component"
3. We need to install `dxvk` package to use vulkan to display the graphics which will improve the performance and press Ok.
4. Next follow [this](https://github.com/lutris/lutris/wiki/How-to:-DXVK) guide to setup vulkan on your system. And we are good to go, this
   should improve the performance while it is handling all those album arts.

Even with this fix, I do experience some delays when using wine on my system. I will be investigating more into the issue and will update
this blog post.

I hope you were able to run iTunes with the instructions in this post. Cheers.

