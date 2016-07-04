---
layout:     post
title:      Creating a Video Player with VDPAU 
date:       2016-06-18 15:31:19
excerpt_separator: <!--more-->
categories: Programming 
tags: [mesa, linux, graphics]
comments:   true

---
I have been working on VDPAU lately and need a test application to test the changes, so instead of using mplayer I decided to make one of my own.
I was not able to find any documentation to do it, although I was able to implement by getting some inspiration(read copying) from mplayer code. I will dicuss about it in this
post. 
<!--more-->

For tl;dr people [here]({{ site.baseurl }}/assets/test.tar.gz) is the source code for the application. 

After going through the post you will be able to write your own harware accelerated video player. I will assume that you read my earlier [post](), if you
haven't give it a try.

## Makefile

We will be using this Makefile to compile our programs, it inlcudes Xlib and vdpau libraries in the compile path. Xlib comes installed with your 
distribution. You need to install libvdpau-dev and mesa-vdpau-drivers package to get vdpau headers. You can install them by using your's distribution's
package manager e.g.: apt-get. The name of our main file is test.c. 

{% highlight bash %}

CC=gcc
CFLAGS=-lX11 -lvdpau

test: test.c
	$(CC) -o test test.c  $(CFLAGS)

{% endhighlight %}

## Coding

I will divide this section into functions performing specific tasks that need to be called in the given order.

### initialize XWindow

We need to write a simple application in Xlib to generate a window which can be used by VDPAU. This code generates a windows sized 300x300 with VDPAU player
as heading.

```c
void init_x() {
	unsigned long black, white;

	dis=XOpenDisplay((char *)0);
   	screen=DefaultScreen(dis);
	black=BlackPixel(dis, screen),
	white=WhitePixel(dis, screen);
        win=XCreateSimpleWindow(dis, DefaultRootWindow(dis), 0, 0,
                300, 300, 5, black, white);
        XSetStandardProperties(dis, win, "VDPAU Player", "VDPAU",
                None, NULL, 0, NULL);
        XSelectInput(dis, win, ExposureMask|ButtonPressMask
                |KeyPressMask);
        gc=XCreateGC(dis, win, 0, NULL);
        XSetBackground(dis, gc, white);
        XSetBackground(dis, gc, black);
        XClearWindow(dis, win);
}
```

### initialize VDPAU

In VDPAU we don't have functions defined globally but are implemented as pointers and we need to retrieve this pointers using get_proc_address function
which itself is retieved as pointer when VdpDevice is created. So first we need to initalize VdpDevice. All the functions in VDPAU return a VdpStatus
which indicates if the function is executed successfully. We will use vdp_st as VdpStatus variable.

```c
vdp_st = vdp_device_create_x11(dis, screen,
                            &vdp_device, &vdp_get_proc_address);
```
the vdp_get_proc_address now contains the address to the pointer to the function VdpGetProcAddress, through which we get pointer to other functions.
First we declare pointer to functions.

```c
static VdpGetProcAddress                 *vdp_get_proc_address;

static VdpDeviceDestroy                  *vdp_device_destroy;

static VdpVideoSurfaceCreate             *vdp_video_surface_create;
static VdpVideoSurfaceDestroy            *vdp_video_surface_destroy;
static VdpVideoSurfacePutBitsYCbCr       *vdp_video_surface_put_bits_y_cb_cr;
```

Now we declare a vdp_function structure which contains pointer to a function and the ID of that function.

```c
struct vdp_function {
    const int id;
    void *pointer;
};
```

We need to pass VdpDevice, function id and function pointer as arguments to vdp_get_proc_address.

```c
struct vdp_function dsc = {VDP_FUNC_ID_DEVICE_DESTROY,
                                &vdp_device_destroy},
vdp_st = vdp_get_proc_address(vdp_device, dsc->id, dsc->pointer);
```


### Create Video and Output Surface 

Now we need to create a VdpVideoSurface on which we will upload data later. We need to provide a chroma type as argument, I don't have much idea about this one
but I used VDP_CHROMA_TYPE_420. You can read about it [here](https://en.wikipedia.org/wiki/Chroma_subsampling#4:2:0). We will feed the video width and 
height manually according to the data we input.

```c
vdp_st = vdp_video_surface_create(vdp_device, VDP_CHROMA_TYPE_420, 
                                        vid_width, vid_height,
                                        &video_surface);

```

We need to provide similar argument to initialize VdpOutputSurface, except chroma type is replaced by the format in which it stores the pixel data 
we will use B8G8R8A8 which the color information for each pixel is of 32 bits the 8 least significant bits store the blue color the next 8 store green, followed
by red and alpha.

```c

vdp_st = vdp_output_surface_create(vdp_device, VDP_RGBA_FORMAT_B8G8R8A8,
                                        vid_width, vid_height,
                                        &output_surface);
```

### Put data in Video Surface

We now need to put values on the video surface to diplayed on screen. One way to do is to do it through VdpDecoder by decoding a video file but it 
will complicate things, so we will manually put values on the surface. We will use vdp_video_surface_put_bits_y_cb_cr to put data onto video surface.
We can similarly use vdp_output_surface_get_bits_native to get the data from output surface.

```c
vdp_pixel_format = VDP_YCBCR_FORMAT_Y8U8V8A8;

vdp_st = vdp_video_surface_put_bits_y_cb_cr(video_surface, vdp_pixel_format,
                                                (void const*const*)data,
                                                 image_stride);

vdp_st = vdp_output_surface_get_bits_native(output_surface, NULL,
                                                (void * const*)data,
                                                 image_stride);
```

Here data is array of video width x video height size and NULL argument is the rectangle on which we need to put data on. The image stride is the total size
of the data incase a rectangle is used. The pixel format represents the format of input data which in our case is YCbCr where each one is represented by
8 bits each.

### Intializing and using VdpVideoMixer

VdpVideoMixer renders output surface from the video surface and converts YCbCr values to RGB. First we need to initialize the mixer. 

```c
    
VdpVideoMixerFeature features[MAX_NUM_FEATURES];        
VdpBool feature_enables[MAX_NUM_FEATURES];

static const VdpVideoMixerParameter parameters[VDP_NUM_MIXER_PARAMETER] = {
    VDP_VIDEO_MIXER_PARAMETER_VIDEO_SURFACE_WIDTH,
    VDP_VIDEO_MIXER_PARAMETER_VIDEO_SURFACE_HEIGHT,
    VDP_VIDEO_MIXER_PARAMETER_CHROMA_TYPE
};

const void *const parameter_values[VDP_NUM_MIXER_PARAMETER] = {
    &vid_width,
    &vid_height,
    &vdp_chroma_type
};


vdp_st = vdp_video_mixer_create(vdp_device, feature_count, features,
                                VDP_NUM_MIXER_PARAMETER,
                                parameters, parameter_values,
                                &video_mixer);
```

We need to provide parameters and features to the video mixer which are post prcessing features like noise reduction, sharpness, luma keying etc.
I have been working on this features this whole time. I have implemented luma keying successfully and started working on high quality scaling.
Now we need to render output surface using video mixer.

```c
vdp_st = vdp_video_mixer_render(video_mixer, 
                                VDP_INVALID_HANDLE, 0,
                                field, 0, 
                                (VdpVideoSurface*)VDP_INVALID_HANDLE,
                                video_surface, 0,
                                (VdpVideoSurface*)VDP_INVALID_HANDLE,
                                NULL, output_surface,
                                NULL, NULL,
                                0, NULL);

```

The arguments to the vdp_video_mixer_render are

```c
VdpVideoMixer mixer,
VdpOutputSurface background_surface,
VdpRect const *background_source_rect,
VdpVideoMixerPictureStructure current_picture_structure,
uint32_t video_surface_past_count,
VdpVideoSurface const *video_surface_past,
VdpVideoSurface video_surface_current,
uint32_t video_surface_future_count,
VdpVideoSurface const *video_surface_future,
VdpRect const *video_source_rect,
VdpOutputSurface destination_surface,
VdpRect const *destination_rect,
VdpRect const *destination_video_rect,
uint32_t layer_count,
VdpLayer const *layers
```

In the example above we have only provided minimal argument like video_suface_current, destination_surface and field. field represents how the individual
frames in a video are represented. I have used VDP_VIDEO_MIXER_PICTURE_STRUCTURE_FRAME as the value of the field.

### The Presentation Queue

If you have reached this point then you have a rendered output surface, now we to put it in the presentation queue so it can be displayed. First we 
need to initialize presentation queue.

```c
int init_vdpau_queue()
{
    VdpStatus vdp_st;

    vdp_st = vdp_presentation_queue_target_create_x11(vdp_device, win,
                                                      &vdp_target);
    if(vdp_st != VDP_STATUS_OK) return -1;

    XMapRaised(dis, win);

    vdp_st = vdp_presentation_queue_create(vdp_device, vdp_target,
                                           &vdp_queue);
    if(vdp_st != VDP_STATUS_OK) return -1;

    return 0;
}
```

Now we need to put our output surface in queue.

```c
vdp_st = vdp_presentation_queue_display(vdp_queue,
                                        output_surface, 
                                        0, 0,
                                        0);
```

The first two 0's represent the clip width and height which will be displayed, 0 represents that the entire surface should be displayed. The last 
argument is the time after 1 january 1970(in seconds) at which the surface should be displayed, 0 means that it be displayed at current time. 


## Result 

If you were able to follow my instructions then the code should work fine and should display the data that you gave in a small window. Remember you 
need to give the data in the first 8 bits will be for Cr next for Cr and then Y. I made a file with the values for a sample image you can download it from 
[here]({{ site.baseurl }}/assets/final.txt). After using this image the output will look like

<p style="text-align:center">
<img style="width:100%;" src="{{ site.baseurl }}/assets/images/vdpau_player.png"> 
</p>

You can download the code from [here]({{ site.baseurl }}/assets/test.tar.gz). I have tried to make the code as clean and as readable as possible. I
would love to hear your opinions on this. 
