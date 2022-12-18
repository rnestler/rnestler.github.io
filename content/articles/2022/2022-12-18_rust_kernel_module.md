Title: Building an out-of-tree Rust Kernel Module
Tags: rust, kernel
Language: en
Summary: Building a hello world out-of-tree Rust kernel module for Linux 6.1
Status: draft

With the release of [Linux 6.1](https://lwn.net/Articles/917504/) minimal Rust
support landed and it should be possible to <del>easily</del> build a hello
world kernel module in Rust.

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
```text
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

The README mentions the following though:

> The kernel tree (KDIR) requires the Rust metadata to be available. These are
> generated during the kernel build, but may not be available for
> installed/distributed kernels (the scripts that install/distribute kernel
> headers etc. for the different package systems and Linux distributions are
> not updated to take into account Rust support yet).

## Trying an in-kernel build

To check if it is just a tooling issue I cloned the
https://github.com/Rust-for-Linux/linux repo and executed the check according
to the documentation:

```text
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
```bash
rustup override set $(scripts/min-tool-version.sh rustc)
rustup component add rust-src
cargo install --locked --version $(scripts/min-tool-version.sh bindgen) bindgen
```

Then I enabled Rust support with `make menuconfig` in the *General setup* menu.
The option is only shown if the toolchain is installed and in my case I also
had to disable the `GCC_PLUGINS` option.

I then just compiled the whole kernel to test it:
```
make LLVM=1
```

This went smoothly, so I continued to build the actual ArchLinux kernel which matches my running kernel:
```text
git remote add archlinux git@github.com:archlinux/linux.git
git fetch archlinux v6.1-arch1
git checkout v6.1-arch1
```

To be sure I ran `make menuconfig` again and applied the same changes again.

A few minutes later I had my kernel ready:
```text
Rust-for-Linux/linux (git)-[rust] % make -j8 LLVM=1
...
Kernel: arch/x86/boot/bzImage is ready  (#2)
make -j8 LLVM=1  1557.69s user 133.21s system 745% cpu 3:46.77 total
```

## Back to the out-of-tree module

Having successfully built the whole kernel with Rust support I went back to the
`rust-out-of-tree-module` repository and tried to build it:

```text
Rust-for-Linux/rust-out-of-tree-module (git)-[main] % make KDIR=../linux LLVM=1
make -C ../linux M=$PWD
make[1]: Entering directory '/home/roughl/projects/github/Rust-for-Linux/linux'
  RUSTC [M] /home/roughl/projects/github/Rust-for-Linux/rust-out-of-tree-module/rust_out_of_tree.o
error: proc macro panicked
  --> /home/roughl/projects/github/Rust-for-Linux/rust-out-of-tree-module/rust_out_of_tree.rs:7:1
   |
7  | / module! {
8  | |     type: RustOutOfTree,
9  | |     name: "rust_out_of_tree",
10 | |     author: "Rust for Linux Contributors",
11 | |     description: "Rust out-of-tree sample",
12 | |     license: "GPL",
13 | | }
   | |_^
   |
   = help: message: Expected byte string

error[E0412]: cannot find type `CStr` in this scope
  --> /home/roughl/projects/github/Rust-for-Linux/rust-out-of-tree-module/rust_out_of_tree.rs:20:29
   |
20 |     fn init(_name: &'static CStr, _module: &'static ThisModule) -> Result<Self> {
   |                             ^^^^ not found in this scope
   |
help: consider importing this struct
   |
5  | use core::ffi::CStr;
   |

error[E0425]: cannot find value `__LOG_PREFIX` in the crate root
  --> /home/roughl/projects/github/Rust-for-Linux/rust-out-of-tree-module/rust_out_of_tree.rs:21:9
   |
21 |         pr_info!("Rust out-of-tree sample (init)\n");
   |         ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^ not found in the crate root
   |
   = note: this error originates in the macro `$crate::print_macro` (in Nightly builds, run with -Z macro-backtrace for more info)

error[E0425]: cannot find value `__LOG_PREFIX` in the crate root
  --> /home/roughl/projects/github/Rust-for-Linux/rust-out-of-tree-module/rust_out_of_tree.rs:34:9
   |
34 |         pr_info!("My numbers are {:?}\n", self.numbers);
   |         ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^ not found in the crate root
   |
   = note: this error originates in the macro `$crate::print_macro` (in Nightly builds, run with -Z macro-backtrace for more info)

error[E0425]: cannot find value `__LOG_PREFIX` in the crate root
  --> /home/roughl/projects/github/Rust-for-Linux/rust-out-of-tree-module/rust_out_of_tree.rs:35:9
   |
35 |         pr_info!("Rust out-of-tree sample (exit)\n");
   |         ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^ not found in the crate root
   |
   = note: this error originates in the macro `$crate::print_macro` (in Nightly builds, run with -Z macro-backtrace for more info)

error[E0050]: method `init` has 2 parameters but the declaration in trait `init` has 1
  --> /home/roughl/projects/github/Rust-for-Linux/rust-out-of-tree-module/rust_out_of_tree.rs:20:20
   |
20 |     fn init(_name: &'static CStr, _module: &'static ThisModule) -> Result<Self> {
   |                    ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^ expected 1 parameter, found 2
   |
   = note: `init` from trait: `fn(&'static kernel::ThisModule) -> core::result::Result<Self, kernel::error::Error>`

error: aborting due to 6 previous errors

Some errors have detailed explanations: E0050, E0412, E0425.
For more information about an error, try `rustc --explain E0050`.
make[2]: *** [scripts/Makefile.build:307: /home/roughl/projects/github/Rust-for-Linux/rust-out-of-tree-module/rust_out_of_tree.o] Error 1
make[1]: *** [Makefile:1992: /home/roughl/projects/github/Rust-for-Linux/rust-out-of-tree-module] Error 2
make[1]: Leaving directory '/home/roughl/projects/github/Rust-for-Linux/linux'
make: *** [Makefile:6: default] Error 2
```

Looking into the compile errors it seemed that some the `fn init` signature
changed and that the `module!` marco expects byte strings.

Changing that:
```diff
diff --git a/rust_out_of_tree.rs b/rust_out_of_tree.rs
index e280409..58cc9ba 100644
--- a/rust_out_of_tree.rs
+++ b/rust_out_of_tree.rs
@@ -6,10 +6,10 @@ use kernel::prelude::*;

 module! {
     type: RustOutOfTree,
-    name: "rust_out_of_tree",
-    author: "Rust for Linux Contributors",
-    description: "Rust out-of-tree sample",
-    license: "GPL",
+    name: b"rust_out_of_tree",
+    author: b"Rust for Linux Contributors",
+    description: b"Rust out-of-tree sample",
+    license: b"GPL",
 }

 struct RustOutOfTree {
@@ -17,7 +17,7 @@ struct RustOutOfTree {
 }

 impl kernel::Module for RustOutOfTree {
-    fn init(_name: &'static CStr, _module: &'static ThisModule) -> Result<Self> {
+    fn init(_module: &'static ThisModule) -> Result<Self> {
         pr_info!("Rust out-of-tree sample (init)\n");

         let mut numbers = Vec::new();
```

We get a successful build!
```text
Rust-for-Linux/rust-out-of-tree-module (git)-[main] % make KDIR=../linux LLVM=1
make -C ../linux M=$PWD
make[1]: Entering directory '/home/roughl/projects/github/Rust-for-Linux/linux'
  RUSTC [M] /home/roughl/projects/github/Rust-for-Linux/rust-out-of-tree-module/rust_out_of_tree.o
  MODPOST /home/roughl/projects/github/Rust-for-Linux/rust-out-of-tree-module/Module.symvers
  LD [M]  /home/roughl/projects/github/Rust-for-Linux/rust-out-of-tree-module/rust_out_of_tree.ko
make[1]: Leaving directory '/home/roughl/projects/github/Rust-for-Linux/linux'
```

But trying to load the resulting module fails:
```bash
sudo insmod rust_out_of_tree.ko
insmod: ERROR: could not insert module rust_out_of_tree.ko: Invalid module format
```
We see this in the `dmesg` output as well:
```
[24593.691184] rust_out_of_tree: version magic '6.1.0-arch1 SMP preempt mod_unload ' should be '6.1.0-arch1-1 SMP preempt mod_unload '
```

Of course we aren't setting the version the same as the ArchLinux kernel. I
copied a few steps from
https://github.com/archlinux/svntogit-packages/blob/packages/linux/trunk/PKGBUILD
to set the same version (we could also just copy the config from there, but
lets see what happens first)
```text
Rust-for-Linux/linux (git)-[tags/v6.1-arch1] % echo "-1" > localversion.10-pkgrel
Rust-for-Linux/linux (git)-[tags/v6.1-arch1] % make -s kernelrelease > version
Rust-for-Linux/linux (git)-[tags/v6.1-arch1] % cat version
6.1.0-arch1-1
```

It didn't work again:
```
Rust-for-Linux/rust-out-of-tree-module (git)-[fix-build-for-linux-6.1] % sudo insmod rust_out_of_tree.ko
insmod: ERROR: could not insert module rust_out_of_tree.ko: Unknown symbol in module
```
The `dmesg` output shows:
```
[25300.463334] rust_out_of_tree: Unknown symbol _RNvNtNtCsfATHBUcknU9_6kernel5print14format_strings4INFO (err -2)
[25300.463353] rust_out_of_tree: Unknown symbol _RNvNtCsfATHBUcknU9_6kernel5print11call_printk (err -2)
[25300.463371] rust_out_of_tree: Unknown symbol __rust_dealloc (err -2)
[25300.463386] rust_out_of_tree: Unknown symbol __rust_realloc (err -2)
[25300.463400] rust_out_of_tree: Unknown symbol __rust_alloc (err -2)
[25300.463414] rust_out_of_tree: Unknown symbol _RNvNtCs3yuwAp0waWO_4core9panicking5panic (err -2)
[25300.463428] rust_out_of_tree: Unknown symbol _RNvMs7_NtCs3yuwAp0waWO_4core3fmtNtB5_9Formatter15debug_lower_hex (err -2)
[25300.463442] rust_out_of_tree: Unknown symbol _RNvXsS_NtNtCs3yuwAp0waWO_4core3fmt3numlNtB7_8LowerHex3fmt (err -2)
[25300.463456] rust_out_of_tree: Unknown symbol _RNvMs7_NtCs3yuwAp0waWO_4core3fmtNtB5_9Formatter15debug_upper_hex (err -2)
[25300.463469] rust_out_of_tree: Unknown symbol _RNvXsT_NtNtCs3yuwAp0waWO_4core3fmt3numlNtB7_8UpperHex3fmt (err -2)
[25300.463483] rust_out_of_tree: Unknown symbol _RNvXs2_NtNtNtCs3yuwAp0waWO_4core3fmt3num3implNtB9_7Display3fmt (err -2)
[25300.463497] rust_out_of_tree: Unknown symbol _RNvMs7_NtCs3yuwAp0waWO_4core3fmtNtB5_9Formatter10debug_list (err -2)
[25300.463510] rust_out_of_tree: Unknown symbol _RNvMs5_NtNtCs3yuwAp0waWO_4core3fmt8buildersNtB5_9DebugList5entry (err -2)
[25300.463524] rust_out_of_tree: Unknown symbol _RNvMs5_NtNtCs3yuwAp0waWO_4core3fmt8buildersNtB5_9DebugList6finish (err -2)
[25300.463538] rust_out_of_tree: Unknown symbol _RNvXs_NtCsfATHBUcknU9_6kernel5errorNtB4_5ErrorINtNtCs3yuwAp0waWO_4core7convert4FromNtNtCsdvv6pRyacSq_5alloc11collections15TryReserveErrorE4from (err -2)
[25300.463552] rust_out_of_tree: Unknown symbol _RNvMNtCsfATHBUcknU9_6kernel5errorNtB2_5Error15to_kernel_errno (err -2)
```

So lets try with the ArchLinux config:
```
wget https://raw.githubusercontent.com/archlinux/svntogit-packages/packages/linux/trunk/config
make olddefconfig
diff -u config .config
```
The configuration difference isn't big
```diff
--- config	2022-12-18 20:51:38.269359290 +0100
+++ .config	2022-12-18 20:53:48.896025137 +0100
@@ -11,6 +11,7 @@
 CONFIG_LD_IS_BFD=y
 CONFIG_LD_VERSION=23900
 CONFIG_LLD_VERSION=0
+CONFIG_RUST_IS_AVAILABLE=y
 CONFIG_CC_CAN_LINK=y
 CONFIG_CC_CAN_LINK_STATIC=y
 CONFIG_CC_HAS_ASM_GOTO_OUTPUT=y
```

A bit strange that `CONFIG_RUST_IS_AVAILABLE=y` isn't set, but
`CONFIG_HAVE_RUST=y` is enabled so all should be good.

Quite some time later we built the ArchLinux kernel as well:
```
make -j8 LLVM=1  15202.68s user 1464.06s system 800% cpu 34:42.57 total
```

But this lead to build failures in the kernel module!
```
Rust-for-Linux/rust-out-of-tree-module (git)-[fix-build-for-linux-6.1] % make KDIR=../linux LLVM=1
make -C ../linux M=$PWD
make[1]: Entering directory '/home/roughl/projects/github/Rust-for-Linux/linux'
  MODPOST /home/roughl/projects/github/Rust-for-Linux/rust-out-of-tree-module/Module.symvers
ERROR: modpost: "_RNvNtNtCsfATHBUcknU9_6kernel5print14format_strings4INFO" [/home/roughl/projects/github/Rust-for-Linux/rust-out-of-tree-module/rust_out_of_tree.ko] undefined!
ERROR: modpost: "_RNvNtCsfATHBUcknU9_6kernel5print11call_printk" [/home/roughl/projects/github/Rust-for-Linux/rust-out-of-tree-module/rust_out_of_tree.ko] undefined!
ERROR: modpost: "__rust_dealloc" [/home/roughl/projects/github/Rust-for-Linux/rust-out-of-tree-module/rust_out_of_tree.ko] undefined!
ERROR: modpost: "__rust_realloc" [/home/roughl/projects/github/Rust-for-Linux/rust-out-of-tree-module/rust_out_of_tree.ko] undefined!
ERROR: modpost: "__rust_alloc" [/home/roughl/projects/github/Rust-for-Linux/rust-out-of-tree-module/rust_out_of_tree.ko] undefined!
ERROR: modpost: "_RNvMs7_NtCs3yuwAp0waWO_4core3fmtNtB5_9Formatter15debug_lower_hex" [/home/roughl/projects/github/Rust-for-Linux/rust-out-of-tree-module/rust_out_of_tree.ko] undefined!
ERROR: modpost: "_RNvXsS_NtNtCs3yuwAp0waWO_4core3fmt3numlNtB7_8LowerHex3fmt" [/home/roughl/projects/github/Rust-for-Linux/rust-out-of-tree-module/rust_out_of_tree.ko] undefined!
ERROR: modpost: "_RNvMs7_NtCs3yuwAp0waWO_4core3fmtNtB5_9Formatter15debug_upper_hex" [/home/roughl/projects/github/Rust-for-Linux/rust-out-of-tree-module/rust_out_of_tree.ko] undefined!
ERROR: modpost: "_RNvXsT_NtNtCs3yuwAp0waWO_4core3fmt3numlNtB7_8UpperHex3fmt" [/home/roughl/projects/github/Rust-for-Linux/rust-out-of-tree-module/rust_out_of_tree.ko] undefined!
ERROR: modpost: "_RNvXs2_NtNtNtCs3yuwAp0waWO_4core3fmt3num3implNtB9_7Display3fmt" [/home/roughl/projects/github/Rust-for-Linux/rust-out-of-tree-module/rust_out_of_tree.ko] undefined!
WARNING: modpost: suppressed 5 unresolved symbol warnings because there were too many)
make[2]: *** [scripts/Makefile.modpost:126: /home/roughl/projects/github/Rust-for-Linux/rust-out-of-tree-module/Module.symvers] Error 1
make[1]: *** [Makefile:1944: modpost] Error 2
make[1]: Leaving directory '/home/roughl/projects/github/Rust-for-Linux/linux'
make: *** [Makefile:6: default] Error 2
```

This are now the compile time errors of the runtime errors we got before!

Apparently the ArchLinux kernel *doesn't* have Rust support, despite it beeing
set in the config. Executing  `make menuconfig` also reveals that options which
conflict with `CONFIG_HAVE_RUST=y` are enabled:
```
│ Symbol: RUST [=n]                                                                                                                                  │
│ Type  : bool                                                                                                                                       │
│ Defined at init/Kconfig:1925                                                                                                                       │
│   Prompt: Rust support                                                                                                                             │
│   Depends on: HAVE_RUST [=y] && RUST_IS_AVAILABLE [=y] && !MODVERSIONS [=n] && !GCC_PLUGINS [=y] && !RANDSTRUCT [=n] && !DEBUG_INFO_BTF [=y]       │
```

At this point I'd probably need to compile and run a custom kernel which
actually has Rust support enabled.
