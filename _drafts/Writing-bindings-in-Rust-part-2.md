---
layout:     post
title:      "Writing bindings in Rust: Part 2"
date:       2017-01-14 15:31:19
excerpt_separator: <!--more-->
categories: Programming
tags: [rust, libevdev]
comments:   true

---

In our last post we looked at how to write a crate which allows raw access to C functions. In this post we will write a wrapper for this crate.
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
    pub fn libevdev_get_name(ctx: *const libevdev) -> *const c_char;
    pub fn libevdev_set_name(ctx: *mut libevdev, name: *const c_char);
}
{% endhighlight %}

We not only need to write safe but we also need to make the API object oriented. Since most of the function take a libevdev device pointer we can
declare a device class and make all these functions its methods. We will have a Device struct which has the raw pointer to libevdev.

{% highlight rust %}
pub struct Device {
    raw: *mut raw::libevdev,
}
{% endhighlight %}

Notice that we have not made the raw pointer public as we like to avoid unsafe access to c pointer by the user.
In rust the methods for a struct are written in a impl block.

{% highlight rust %}
impl Device {
}
{% endhighlight %}

Let's start with the `libevdev_new` function. We need to call the function and then return a Device struct with the libevdev pointer. We will
libevdev from `libevdev_new` according to rust naming conventions. Also notice that we are returning `Option<Device>` instead of a `Device`
because its good code. Rust provides us with `Option<T>` to deal with function in which we might not always return the required value.

{% highlight rust %}
impl Device {
    pub fn new() -> Option<Device> {
        let libevdev = unsafe {
            raw::libevdev_new()
        };

        if libevdev.is_null() {
            None
        } else {
            Some(Device {
                raw: libevdev,
            })
        }
    }
}
{% endhighlight %}

The `libevdev_free` need to be executed once the use of libevdev device is over. Rust provides us with `drop` trait for this. We don't need
to free the device ourselves instead we specify the code that needs to be executed once a value goes out of scope and rust will execute it
for us. We need to implement `libevdev_free` in the drop trait for `Device`.

{% highlight rust %}
impl Drop for Device {
    fn drop(&mut self) {
        unsafe {
            raw::libevdev_free(self.raw);
        }
    }
}
{% endhighlight %}

Now's let write a wrapper for a function with some arguments, most of the arguments types are easier to deal except for the Strings.

{% highlight rust %}
pub fn new_from_fd(fd: &File) -> Result<Device, Errno> {
    let mut libevdev = 0 as *mut _;
    let result = unsafe {
        raw::libevdev_new_from_fd(fd.as_raw_fd(), &mut libevdev)
    };

    match result {
        0 => Ok(Device { raw: libevdev }),
        k => Err(Errno::from_i32(-k)),
    }
}

pub fn grab(&mut self, grab: GrabMode) -> Result<(), Errno> {
    let result = unsafe {
        raw::libevdev_grab(self.raw, grab as c_int)
    };

    match result {
        0 => Ok(()),
        k => Err(Errno::from_i32(-k)),
    }
}
{% endhighlight %}

`as` comes in handy while converting between rust and c types. However we need to put more effort to conver a `str` to `char *` and vice
versa. To get a `char *` from `str` we use `libc::CString` i.e.

{% highlight rust %}
pub fn set_name(&self, name: &str) {
    let name = CString::new(name).unwrap();
    unsafe {
        raw::libevdev_set_name(self.raw, name.as_ptr())
    }
}
{% endhighlight %}

The code for the other way around is slightly bigger so we better declare a function to convert from `char *` to `str`.

{% highlight rust %}
fn ptr_to_str(ptr: *const c_char) -> Option<&'static str> {
    let slice : Option<&CStr> = unsafe {
        if ptr.is_null() {
            return None
        }
        Some(CStr::from_ptr(ptr))
    };

    match slice {
        None => None,
        Some(s) => {
            let buf : &[u8] = s.to_bytes();
            Some(std::str::from_utf8(buf).unwrap())
        }
    }
}

pub fn name(&self) -> Option<&str> {
    ptr_to_str(unsafe {
        raw::libevdev_get_name(self.raw)
    })
}
{% endhighlight %}

We have almost comleted our wrapper except for just one last function which shows the use of enums in rust.

{% highlight rust %}
pub enum LogPriority {
    Error = raw::LIBEVDEV_LOG_ERROR as isize,
    Info = raw::LIBEVDEV_LOG_INFO as isize,
    Debug = raw::LIBEVDEV_LOG_DEBUG as isize,
}

pub fn get_log_priority() -> LogPriority {
    unsafe {
        let priority = raw::libevdev_get_log_priority();
        match priority {
            raw::LIBEVDEV_LOG_ERROR => LogPriority::Error,
            raw::LIBEVDEV_LOG_INFO => LogPriority::Info,
            raw::LIBEVDEV_LOG_DEBUG => LogPriority::Debug,
            c => panic!("unknown log priority: {}", c),
        }
    }
}
{% endhighlight %}

We have completed our wrapper for evdev-sys. You can have a look at the [repo](https://github.com/ndesh26/evdev-rs) for the complete bindings.
