---
layout:     post
title:      "Introducing evdev-rs"
date:       2017-09-02 15:31:19
excerpt_separator: <!--more-->
categories: Programming
tags: [rust, libevdev]
comments:   true

---

Today I published my first library on [crates.io](https://crates.io), evdev-rs. evdev-rs is a wrapper around libevdev in Rust. It provides us
the facility to deal with input devices in a safe way. This post will serve as a tutorial to get started with evdev-rs.

<!--more-->

### What is libevdev?

libevdev is a wrapper library for evdev devices. it moves the common tasks when dealing with evdev devices into a library and provides a
library interface to the callers, thus avoiding erroneous ioctls, etc.

The eventual goal is that libevdev wraps all ioctls available to evdev devices, thus making direct access unnecessary.

### Why a libevdev wrapper? 

The evdev protocol is simple, but quirky, with a couple of behaviors that are non-obvious. libevdev transparently handles some of those quirks.

The [evdev](https://github.com/cmr/evdev/blob/master/src/lib.rs) crate is an implementation of evdev in Rust. Nothing wrong with that, but it
will miss out on any more complex handling that libevdev provides.

### Let's begin

We will write a simple rust program which uses evdev-rs to get input events from a generic input device and prints them to the stdout.
Let's create a simple project using cargo.

{% highlight terminal %}
$ cargo new print_events --bin
     Created binary (application) `print_events` project
{% endhighlight %}

This will create a basic rust project with a binary. We need to add the evdev-rs dependency to the Cargo.toml at this point and cargo will take 
care of the rest. The Cargo.toml will look like this at this point.

{% highlight rust %}
[package]
name = "print_events"
version = "0.1.0"
authors = ["Nayan Deshmukh <nayan26deshmukh@gmail.com>"]

[dependencies]
evdev-rs = "0.0.1"
{% endhighlight %}

We can try running our application now we will cargo for this, it will first download the evdev-rs library and then compile it and then compile
the project and run it.

{% highlight terminal %}
$ cargo run
    Updating registry `https://github.com/rust-lang/crates.io-index`
   Compiling bitflags v0.7.0
   Compiling log v0.3.8
   Compiling cfg-if v0.1.2
   Compiling gcc v0.3.53
   Compiling void v1.0.2
   Compiling libc v0.2.30
   Compiling bitflags v0.4.0
   Compiling pkg-config v0.3.9
   Compiling semver v0.1.20
   Compiling rustc_version v0.1.7
   Compiling nix v0.7.0
   Compiling evdev-sys v0.0.1
   Compiling evdev-rs v0.0.1
   Compiling print_events v0.1.0 (file:///home/ndesh/print_events)
    Finished debug [unoptimized + debuginfo] target(s) in 25.2 secs
     Running `target/debug/print_events`
Hello, world!
{% endhighlight %}

Let's start with some basic evdev code. First, we will open an input device as a `File`(any file from /dev/input) and then initialize the `evdev::Device` with this file.
Then we will print the name of the device (I choose my keyboard for this). The src/main.rs look like this.

{% highlight rust %}
extern crate evdev_rs as evdev;

use evdev::*;
use std::fs::File;

fn main() {
    let path = "/dev/input/event4"; 
    let f = File::open(path).unwrap();

    let d = Device::new_from_fd(&f).unwrap();

    println!("Input device name: \"{}\"", d.name().unwrap_or(""));
}
{% endhighlight %}

Running this print the device name. Notice that we need to run this as a sudo user because a regular user is not allowed access to /dev/input/event
files.

{% highlight terminal %}
$ sudo cargo run
   Compiling print_events v0.1.0 (file:///home/ndesh/print_events)
    Finished debug [unoptimized + debuginfo] target(s) in 0.38 secs
     Running `target/debug/print_events`
Input device name: "AT Translated Set 2 keyboard"
{% endhighlight %}

evdev-rs supports most of the capabilities of libevdev(except for logging), you can have a look at a more complex [example](https://github.com/ndesh26/evdev-rs/blob/master/examples/evtest.rs)
which exploits more capabilites of evdev-rs. You can also have a look at the [Documentation](https://ndesh26.github.io/evdev-rs/evdev_rs/index.html) 
which provides more details about each function.

