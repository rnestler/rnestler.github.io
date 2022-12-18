Title: A Rust Kernel Module
Tags: rust, kernel
Language: en
Summary: Creating a hello world Rust kernel module
Status: draft

A Rust Kernel Module
====================

With the release of [Linux 6.1](https://lwn.net/Articles/917504/) minimal Rust
support landed and it should be possible to easily build a hello world kernel
module in Rust.

## Installing Linux 6.1

At the time of writing Linux 6.1 was still in testing on ArchLinux, so I
enabled the testing repositories:

```diff
--- pacman.conf	2022-12-18 15:25:10.742819293 +0100
+++ /etc/pacman.conf	2022-12-18 00:32:56.404801925 +0100
@@ -69,8 +69,8 @@
 # repo name header and Include lines. You can add preferred servers immediately
 # after the header, and they will be used before the default mirrors.

-#[testing]
-#Include = /etc/pacman.d/mirrorlist
+[testing]
+Include = /etc/pacman.d/mirrorlist

 [core]
 Include = /etc/pacman.d/mirrorlist
@@ -78,8 +78,8 @@
 [extra]
 Include = /etc/pacman.d/mirrorlist

-#[community-testing]
-#Include = /etc/pacman.d/mirrorlist
+[community-testing]
+Include = /etc/pacman.d/mirrorlist

 [community]
 Include = /etc/pacman.d/mirrorlist
@@ -87,8 +87,8 @@
 # If you want to run 32 bit applications on your x86_64 system,
 # enable the multilib repositories as required here.

-#[multilib-testing]
-#Include = /etc/pacman.d/mirrorlist
+[multilib-testing]
+Include = /etc/pacman.d/mirrorlist

 [multilib]
 Include = /etc/pacman.d/mirrorlist
```


## A first start

Since I just want to play around I decided to build an out-of-tree Rust module.
Luckily there is an example module available for that already:
https://github.com/Rust-for-Linux/rust-out-of-tree-module.

So I went ahead, cloned the repo and tried to just execute make:
```raw
rust-out-of-tree-module (git)-[main] % make
make -C /lib/modules/`uname -r`/build M=$PWD
make[1]: Entering directory '/usr/lib/modules/6.1.0-arch1-1/build'
  RUSTC [M] /home/roughl/projects/github/Rust-for-Linux/rust-out-of-tree-module/rust_out_of_tree.o
error: target file "./rust/target.json" does not exist

make[2]: *** [scripts/Makefile.build:307: /home/roughl/projects/github/Rust-for-Linux/rust-out-of-tree-module/rust_out_of_tree.o] Error 1
make[1]: *** [Makefile:1992: /home/roughl/projects/github/Rust-for-Linux/rust-out-of-tree-module] Error 2
make[1]: Leaving directory '/usr/lib/modules/6.1.0-arch1-1/build'
make: *** [Makefile:6: default] Error 2
```

Well...

Reading through
https://www.kernel.org/doc/html/latest/rust/quick-start.html
it seemed that I have everything I need to get started.

## Trying an in-kernel build

To check if it is just a tooling issue I cloned the
https://github.com/Rust-for-Linux/linux repo and executed the check according
to the documentation:

```
Rust-for-Linux/linux (git)-[rust] % make LLVM=1 rustavailable
***
*** Rust compiler 'rustc' is too new. This may or may not work.
***   Your version:     1.66.0
***   Expected version: 1.62.0
***
***
*** Rust bindings generator 'bindgen' is too new. This may or may not work.
***   Your version:     0.63.0
***   Expected version: 0.56.0
***
Rust is available!
```

Since my toolchain seems to new I installed the older versions with
```
rustup override set $(scripts/min-tool-version.sh rustc)
rustup component add rust-src
cargo install --locked --version $(scripts/min-tool-version.sh bindgen) bindgen
```

Then I enabled Rust support with `make menuconfig` in the *General setup* menu.
The option is only shown if the toolchain is installed and in my case I also
had to disable the `GCC_PLUGINS` option.

