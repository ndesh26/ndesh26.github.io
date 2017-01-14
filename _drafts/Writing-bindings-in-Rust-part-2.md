---
layout:     post
title:      "Writing bindings in Rust: Part 2"
date:       2017-01-14 15:31:19
excerpt_separator: <!--more-->
categories: Programming
tags: [rust, libevdev]
comments:   true

---

In our last post we looked the way to write a crate which allows raw access to C functions. In this post we will write a wrapper for this crate. 
We will be using a part of evdev-sys to write the wrapper functions.

<!--more-->

This is how lib.rs for evdev-sys looks like:

{% highlight rust %}
extern crate libc;

use libc::{c_int, c_char};

pub enum libevdev {}

extern {
    pub fn libevdev_new() -> *mut libevdev;
    pub fn libevdev_new_from_fd(fd: c_int,
                                ctx: *mut *mut libevdev) -> c_int;
    pub fn libevdev_free(ctx: *mut libevdev);
    pub fn libevdev_get_log_priority() -> libevdev_log_priority;
    pub fn libevdev_grab(ctx: *mut libevdev,
                         grab: libevdev_grab_mode) -> c_int;
    pub fn libevdev_get_fd(ctx: *mut libevdev) -> c_int;
    pub fn libevdev_has_event_pending(ctx: *mut libevdev) -> c_int;
    pub fn libevdev_get_name(ctx: *const libevdev) -> *const c_char;
    pub fn libevdev_set_name(ctx: *mut libevdev, name: *const c_char);
} 
{% endhighlight %}
