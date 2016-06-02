---
layout:     post
title:      Setting development environment for Mesa
date:       2016-05-30 15:31:19
excerpt_separator: <!--more-->
categories: programming 
tags: [mesa, linux, graphics]
comments:   true

---
While I was working, I was not able to find a good post regarding development environment for mesa, so I decided to write a post myself. Mesa is 
written in C for most part and uses git for version control and autotools to build.
<!--more-->
It will be a familiar environment for most people.

### Download source code

Mesa hosts its mater git repository on [freedestop.org](https://www.freedesktop.org/wiki/).

To get a local copy of source code do

{% highlight bash %}
cd ~
git clone git://anongit.freedesktop.org/git/mesa/mesa
{% endhighlight %}

If you also want the Mesa demos/tests repository:

{% highlight bash %}
git clone git://anongit.freedesktop.org/git/mesa/demos
{% endhighlight %}

Later, you can update your tree from the master repository with:
{% highlight bash %}
git pull origin
{% endhighlight %}

Now do

{% highlight bash %}
cd mesa
{% endhighlight %}

### Set Configure Options

Mesa contains code for many drivers and we will be working on on specific ones, so we need to set the configure options to only build those drivers. configure parameters:

**--enable-debug:**  It enables debug support and build all files with "-g -O0" flags.

**--with-dri-drivers:**  This the list of classic mesa-drivers you want to build. eg. --with-dri-drivers="i965,radeon,nouveau". Building only the drivers on which
you will work will reduce the build time significantly.

**--with-gallium-drivers:** This is the list of gallium drivers that you want to build, you will need this option if you are working on AMD or NVIDIA card. 
if you using this option you will need to use --with-egl-platform with is.

**--with-egl-paltform:** This is the platforms for which we want to build the drivers like drm or X11 or Wayland.

**others:** There are other options for other components like we need --enable-vdpau for vdpau or --enable-omx for omx. If you are installing any other component like vdpau 
you also need to specify its installation directory yourself using --with-vdpau-libdir='/usr/lib64/vdpau'. 

Now run the autogen.sh with the above options:

{% highlight bash %}
./autogen.sh --enable-debug --with-dri-drivers="i965,radeon" --with-gallium-drivers="radeonsi" --with-egl-platforms="drm" --enable-vdpau --with-vdpau-libdir="/usr/lib64/vdpau"
{% endhighlight %}

then run:

{% highlight bash %}
make
{% endhighlight %}

### Using the drivers

There are two ways to use the drivers, either we install the drivers in our system or we specify the location of drivers to individual application.
I prefer the latter one as it won't break your X server in case you mess with source code.

To install the drivers you just need to run:

{% highlight bash %}
sudo make install
{% endhighlight %}

 To use the drivers without installing them:

 If you have not changed the --prefix option in autotools then you will a folder named lib in mesa directory, it contains the drivers that we compiled in the previous step. To instruct OpenGL programs to use this drivers we need
 to change some environment variables, so to run any program we need to run:

{% highlight bash %}
LIBGL_DRIVERS_PATH="~/mesa/lib" LD_LIBRARY_PATH="~/mesa/lib" program_name
{% endhighlight %}

replace program_name with the name of the program. 

I hope you were able to follow the post. If you have any difficulty following the procedure or if you are getting any error, feel free to let me know 
in the comments section I will try to help you as much as I can.

