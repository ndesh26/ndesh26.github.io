---
layout:     post
title:      "Writing bindings in Rust: Part 1"
date:       2017-01-15 02:31:19
excerpt_separator: <!--more-->
categories: Programming
tags: [rust, libevdev]
comments:   true

---
I started learning rust early December, and the best way to learn a language is to use it. So I started looking for a project in rust and
hopefully I was able to find a project which was writing bindings for libevdev in rust. There are a few blogs which talk about bindings but
there were certain aspects that this blogs didn't cover and I had to do some digging myself to get the answers. This aim of this post is to
help anyone who wants to write bindings for a C library in rust.
<!--more-->

To write bindings we need to call the C functions from rust code. To accomplish this rust provides us with foreign function interface(FFI)
to talk with C. Before we call any C function from rust, we first need to declare that function in rust. We use the extern block provided by
FFI for this. We will also use libc crate which provides us with C data types.

### Functions

{% highlight rust %}
extern {
    // the corresponding C function will be
    // void hello_world();
    fn hello_world();
}
{% endhighlight %}

Let's look at some examples to understand how various C functions are translated to their rust counterparts:

{% highlight rust %}
int area(int length, int width);
const char * get_name(unsigned int code);
const char * get_details(void *data);
{% endhighlight %}

{% highlight rust %}
extern {
    fn area(length: libc::c_int, width: libc::int) -> libc::c_int;
    fn get_name(code: libc::c_uint) -> *const libc::c_char;
    // the use of mut with c_void depends on the fact whether the
    // function is going to alter the value pointer by data or not
    fn get_details(data: *mut libc::c_void) -> *const libc::c_char;
}
{% endhighlight %}

### linking with the C lib

The easiest way to link to a C lib is to use `#[link(name = "libname")]` i.e.

{% highlight rust %}
#[link(name = "libname")]
extern {
    fn area(length: libc::c_int, width: libc::int) -> libc::c_int;
}
{% endhighlight %}

But as usual this is not the recommended way, the recommended way is to use a build script to check if the C lib is available and build it
ourselves if not present. A simple build script would check for installed library using pkg-config crate i.e.

{% highlight rust %}
extern crate pkg_config;

fn main() {

   if let Ok(lib) = pkg_config::find_library("libevdev") {
        for path in &lib.include_paths {
            println!("cargo:include={}", path.display());
        }
        return
    }
}
{% endhighlight %}

I won't dwell into building of packages as of now as that would deviate us from the topic at hand, however the crates.io has a good [documentation](http://doc.crates.io/build-script.html) regarding build script.

### Structs

Structs in C and Rust are pretty similar, the difference being the types used.

{% highlight c %}
struct input_absinfo {
	int value;
	int minimum;
	int maximum;
	int fuzz;
	int flat;
	int resolution;
};

void set_abs_info(unsigned int code,
                  const struct input_absinfo *abs);
{% endhighlight %}

translates to

{% highlight rust %}
#[repr(C)]
pub struct input_absinfo {
    pub value: libc::c_int,
    pub minimum: libc::c_int,
    pub maximum: libc::c_int,
    pub fuzz: libc::c_int,
    pub flat: libc::c_int,
    pub resolution: libc::c_int,
}

extern {
    pub fn libevdev_set_abs_info(code: c_uint,
                                 abs: *const input_absinfo);
}
{% endhighlight %}

For opaque structs the rust documentation suggests use of enums, which is explained really nicely [here](https://doc.rust-lang.org/book/ffi.html#representing-opaque-structs).

### Enums

There is no nice way to import enums from C, therefore we need to use c_int in place of enums for functions. Though we can declare the enum as
constants i.e.
{% highlight c %}
enum libevdev_grab_mode {
	LIBEVDEV_GRAB = 3,
	LIBEVDEV_UNGRAB = 4
};

int libevdev_grab(struct libevdev *dev, enum libevdev_grab_mode grab);
{% endhighlight %}
will be used as
{% highlight rust %}
pub enum libevdev {} // libevdev is a opaque struct

pub type libevdev_grab_mode = libc::c_int;

pub const LIBEVDEV_GRAB: libevdev_grab_mode = 3;
pub const LIBEVDEV_UNGRAB: libevdev_grab_mode = 4;

extern {
    pub fn libevdev_grab(ctx: *mut libevdev,
                         grab: libevdev_grab_mode) -> c_int;
}
{% endhighlight %}

### Putting things together

Now we are ready to use the C functions in rust, but since C is "unsafe" as it lacks memory safety we need to call C functions in an `unsafe`
block.

{% highlight rust %}
extern {
    fn area(length: libc::c_int, width: libc::int) -> libc::c_int;
}

fn main() {
    let width: i32 = 5;
    let height: i32 = 10;

    let area = unsafe {
        area(width as libc::c_int, height as libc::c_int)
    };
}
{% endhighlight  %}

We can now declare all the C functions and structs from our library in a crate. This crate would allow us to access raw C functions of the library.
According to the conventions suggested by [crates.io](https://crates.io/) the name of this crate should be libname-sys. But our work is not over
yet as using raw C functions in rust directly is not a good practice, it is also tedious for the end user of the crate as he needs to take care of memory safety and type conversion.
Therefore we need to write a wrapper around this functions which will allow us to make them easier to use and rust-y too.
It will be easier to explain writing wrappers using an example, so we will write a wrapper for a part of the libevdev library in the next post.

Until then Stay Tuned!!
