---
title: "Rust Universal Binaries"
description: "Enabling universal binaries on MacOS"
date: 2022-11-14T19:11:45-05:00
draft: false
categories:
- Technical
tags:
- rust
- apple silicon
# image: "img/ferris.png"
image: "img/ferris_but_nord.jpg"

---

During my adventures learning [Rust](https://www.rust-lang.org/), I've been writing an implementation of [coreutils](https://github.com/AdamIsrael/coreutils/) (arch, base64, basename, wc, etc). It's been an interesting exercise, working to recreate these GNU utilities in a new language, but today I got to learn a whole lot about architectures and Universal Binaries.

So. This weekend, I picked up a 2022 Macbook Pro M2.

And while doing some refactoring over the weekend, to use [workspaces](https://doc.rust-lang.org/cargo/reference/workspaces.html) with [cargo](https://doc.rust-lang.org/cargo/index.html), Rust's package manager, I realized that my implementation of `arch` was returning `x86_64`.

At first, I thought there was a bug with the [platform-info](https://github.com/uutils/platform-info) crate I was using to get the architecture. I looked deeper, and suspected the bug was in the underlying [libc](https://github.com/rust-lang/libc) bindings.

Reader, there was no bug.
<!--more-->

TIL a bunch about Apple Silicon, M2, and universal binaries.

To test this, I created a simple Rust application, using `libc` the same way platform-info does:

```rust
// arch/src/main.rs
extern crate libc;
use self::libc::{uname, utsname};

use std::ffi::CStr;
use std::mem::MaybeUninit;

macro_rules! cstr2cow {
    ($v:expr) => {
        CStr::from_ptr($v.as_ref().as_ptr()).to_string_lossy()
    };
}

fn main() {
    unsafe {
        let mut uts = MaybeUninit::<utsname>::uninit();
        if uname(uts.as_mut_ptr()) != -1 {
            let uts = uts.assume_init();
            println!("{}", cstr2cow!(uts.machine));
        }
    }
}
```

The resulting binary still returned `x86_64`. It turns out that's because Rust is compiling to `x86_64`, and the resulting binaries are running through Rosetta (a compatibility layer that allows Intel binaries to run on Apple Silicon).

It is possible, though, to build a native `arm64` binary. Using `rustup`, the Rust toolchain installer, to add new build targets. In this case, we want to add a target for `aarch64-apple-darwin`:

```
rustup target add aarch64-apple-darwin
```

Next, we can tell `cargo` to build an `arm64` binary:

```
cargo build --target aarch64-apple-darwin
```

Running this binary yields `arm64` "as expected", and the binary itself is also arm64:

```
$ file target/aarch64-apple-darwin/release/arch
target/aarch64-apple-darwin/release/arch: Mach-O 64-bit executable arm64
```

There's still one more step, though. How do we create a [Universal Binary](https://en.wikipedia.org/wiki/Universal_binary) -- both x86_64 and arm64? There's some [discussion](https://github.com/rust-lang/cargo/issues/8875) about if and how to implement this via `cargo`, but for the time being we can run this additional step:

```
lipo -create -output arch target/release/arch target/aarch64-apple-darwin/release/arch
```

This creates a Universal Binary containing both architectures:

```bash
$ file arch
arch: Mach-O universal binary with 2 architectures: [x86_64:Mach-O 64-bit executable x86_64Mach-O 64-bit executable x86_64] [arm64:Mach-O 64-bit executable arm64Mach-O 64-bit executable arm64]
arch (for architecture x86_64):	Mach-O 64-bit executable x86_64
arch (for architecture arm64):	Mach-O 64-bit executable arm64
```

Running the Universal Binary results in this, which is the native architecture.

```bash
$ arch
arm64
```

And there we have it. Rust can build `x86_64` and `arm64` binaries on Apple Silicon, and we can link the two to create a Universal Binary.

It also raises some interesting considerations around architecture. The way I've typically used the `arch` command is to determine what architecture I'm currently running on, but in this case, the answer is both. Cargo and rustup make it easy to add and build for different architectures.

**Addendum**

As pointed out by [@kornel@mastadon.social](https://mastodon.social/@kornel), the reason my build was defaulting to x86_64 was because I was running the Intel build of Rust, installed on my previous Intel-based Mac and transferred over when I setup the new laptop. After uninstalling and reinstalling Rust,I now have arm64 builds by default (and the reverse is true, I can add an intel toolchain to build x86_64).