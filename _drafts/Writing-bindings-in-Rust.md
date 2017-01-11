---
layout:     post
title:      Writing bindings in Rust
date:       2016-12-07 15:31:19
excerpt_separator: <!--more-->
categories: Programming
tags: [mesa, linux, graphics]
comments:   true

---
I started learning rust early december, and the best way to learn a language is to use it. So I started looking for a project in rust and
hopefully I was able to find a project which was writing bindings for libevdev in rust. There are a few blogs which talk about bindings but
there were certain aspects that this blogs didn't cover and I did do some digging into code to get the answers. This aim of this post is to
help anyone who wants to write bindings for a C library in rust.
<!--more-->

To write bindings we need to call the C functions from rust code. Rust provides us with foreign function interface(FFI) to talk with C.
Before we call any C function from rust, we first need to declare that function in rust. We use the extern block provided by FFI for this.
We will also use libc crates which provides us with c data types.

### Functions

{% highlight rust %}
extern {
    // the function go here
    fn hello_world();
}
{% endhighlight %}

Let's look at some examples to understand how various C functions are translated to their rust counterparts:

{% highlight c %}
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

The easiest way to link to a C lib is to use `#[link(name = "lib_name")]` i.e.

{% highlight rust %}
#[link(name = "lib_name")]
extern {
    fn area(length: libc::c_int, width: libc::int) -> libc::c_int;
}
{% endhighlight %}

But as usual this is not the recommended way, the recommended way is to use a build script to check if the C lib is available and build it
ourselves if not present. I won't dwell into specifics of build script as the crates doc cover them very well, here a [link](http://doc.crates.io/build-script.html)
to the doc.

### Structs
