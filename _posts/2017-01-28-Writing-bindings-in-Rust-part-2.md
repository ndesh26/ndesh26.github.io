---
layout:     post
title:      "Writing bindings in Rust: Part 2"
date:       2017-01-28 15:31:19
excerpt_separator: <!--more-->
categories: Programming
tags: [rust, libevdev]
comments:   true

---

In the last [post]({{ site.baseurl }}/programming/2017/01/15/Writing-bindings-in-Rust-part-1/), we looked at how we can write a crate which allows raw access to C functions. In this post, we will write a safe abstraction for this crate.
We will be using a part of evdev-sys as an example to see how to write the wrapper functions.

<!--more-->

### Why do we need a wrapper in the first place?

The wrapper is a way of writing a safe abstraction around an unsafe C function which would allow us to use rust specific constructs like
ownership and borrowing to ensure memory safety. This abstraction also allows us to hide details of the bindings and the end user can use it
like any other normal rust function. We can also introduce an API which is more object oriented than the one which provided by C library.

### Let's begin

This is the lib.rs that we are going to use for the example:

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

We not only need to write safe abstractions but we also need to make the API object-oriented. Since most of the function take a libevdev device
pointer we can declare a `Device` class and make all these functions its methods. We will have the `Device` struct which has the raw pointer to
the libevdev device.

{% highlight rust %}
pub struct Device {
    raw: *mut raw::libevdev,
}
{% endhighlight %}

Notice that we have not made the raw pointer public as we want to avoid unsafe access to C pointer by the user.
In rust, the methods for a struct are written in an impl block.

{% highlight rust %}
impl Device {
}
{% endhighlight %}

Let's start with the `libevdev_new` function. We need to call the function and then return a Device struct with the libevdev pointer. We will remove libevdev
libevdev from `libevdev_new` according to rust's naming conventions. Also notice that we are returning `Option<Device>` instead of `Device`
because it is good rust code. Rust provides us with `Option<T>` to deal with function in which we might not always return the required value.

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

`libevdev_free` needs to be executed once the use of the libevdev device is over. Rust provides us the `drop` trait for this. We don't need
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

`as` comes in handy while converting between rust and c types. However, we need to put more effort to convert an `str` to `char *` and vice
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

We have almost completed our wrapper except for one last function which shows the use of enums in rust.

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

We have written a wrapper for evdev-sys. You can have a look at the [repo](https://github.com/ndesh26/evdev-rs) for the complete bindings.
Here are some references which I found useful:

* [Rust Once, Run Everywhere](https://blog.rust-lang.org/2015/04/24/Rust-Once-Run-Everywhere.html)
* [Rust-Book](https://doc.rust-lang.org/book/ffi.html)
- [FFI in Rust - writing bindings for libcpuid](http://siciarz.net/ffi-rust-writing-bindings-libcpuid/)
