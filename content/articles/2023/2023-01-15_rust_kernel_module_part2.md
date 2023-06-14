Title: Building an out-of-tree Rust Kernel Module Part Two
Tags: rust, kernel, Linux, ArchLinux
Language: en
Summary: Trying to build a hello world out-of-tree Rust kernel module for Linux 6.1. Part two.

In my last [blog post](/building-an-out-of-tree-rust-kernel-module.html), I
tried to build an out-of-tree kernel module in Rust for the 6.1 stock ArchLinux kernel.

During this process I noticed that while `CONFIG_HAVE_RUST=y` is set in the
[ArchLinux kernel config](https://github.com/archlinux/svntogit-packages/blob/706494f4555dca158c9f02932717550e7b66b534/trunk/config),
Rust support get's silently disabled since conflicting options are set.

## Building our own ArchLinux kernel

So I went ahead and started to build my own kernel with a custom config:
<https://aur.archlinux.org/packages/linux-rust>.

I based it off the official ArchLinux kernel package, disabled the conflicting
options, ran `make menuconfig` and enabled Rust support.

A few iterations on the configuration and `PKGBUILD` later, I had a successfully
building kernel with Rust support enabled!

After installing the custom kernel and booting into it, we should finally be
ready!

## Building the out-of-tree module

So back to the
[Rust-for-Linux/rust-out-of-tree-module](https://github.com/rnestler/rust-out-of-tree-module/tree/fix-build-for-linux-6.1):
```bash
make KDIR=~/projects/archpkg/linux-rust/src/archlinux-linux LLVM=1
sudo insmod rust_out_of_tree.ko
```

And we see in the `dmesg` output that the module successfully loaded! 🎉
```
[  451.297415] rust_out_of_tree: loading out-of-tree module taints kernel.
[  451.297460] rust_out_of_tree: module verification failed: signature and/or required key missing - tainting kernel
[  451.297724] rust_out_of_tree: Rust out-of-tree sample (init)
```

If we unload it with `sudo rmmod rust_out_of_tree.ko` we get the following in
`dmesg`:

```
[  461.206748] rust_out_of_tree: My numbers are [72, 108, 200]
[  461.206753] rust_out_of_tree: Rust out-of-tree sample (exit)
```

## Packing the correct metadata

You may have noticed, that we passed
`KDIR=~/projects/archpkg/linux-rust/src/archlinux-linux` to the make call. This
is because the build requires the Rust metadata which is generated during the
kernel build. To make it easier for other people installing the `linux-rust`
kernel, we should package it in `linux-rust-headers` ([^1]).

If we don't pass the `KDIR` option, we get the following build error:
```text
..Rust-for-Linux/rust-out-of-tree-module (git)-[fix-build-for-linux-6.1] % make LLVM=1
make -C /lib/modules/`uname -r`/build M=$PWD
make[1]: Entering directory '/usr/lib/modules/6.1.5-arch2-1-rust/build'
  RUSTC [M] /home/roughl/projects/github/Rust-for-Linux/rust-out-of-tree-module/rust_out_of_tree.o
error: target file "./rust/target.json" does not exist

make[2]: *** [scripts/Makefile.build:307: /home/roughl/projects/github/Rust-for-Linux/rust-out-of-tree-module/rust_out_of_tree.o] Error 1
make[1]: *** [Makefile:1992: /home/roughl/projects/github/Rust-for-Linux/rust-out-of-tree-module] Error 2
make[1]: Leaving directory '/usr/lib/modules/6.1.5-arch2-1-rust/build'
make: *** [Makefile:6: default] Error 2
```

So let's add this file to the package and try to rebuild:

```text
..Rust-for-Linux/rust-out-of-tree-module (git)-[fix-build-for-linux-6.1] % make LLVM=1
make -C /lib/modules/`uname -r`/build M=$PWD
make[1]: Entering directory '/usr/lib/modules/6.1.5-arch2-1-rust/build'
  RUSTC [M] /home/roughl/projects/github/Rust-for-Linux/rust-out-of-tree-module/rust_out_of_tree.o
error[E0463]: can't find crate for `core`
  |
  = note: the `target` target may not be installed
  = help: consider downloading the target with `rustup target add target`
  = help: consider building the standard library from source with `cargo build -Zbuild-std`
```

<details>
<summary>Rest of the terminal output ...</summary>
```text
error[E0463]: can't find crate for `compiler_builtins`

error[E0463]: can't find crate for `kernel`
 --> /home/roughl/projects/github/Rust-for-Linux/rust-out-of-tree-module/rust_out_of_tree.rs:5:5
  |
5 | use kernel::prelude::*;
  |     ^^^^^^ can't find crate

error: cannot find macro `pr_info` in this scope
  --> /home/roughl/projects/github/Rust-for-Linux/rust-out-of-tree-module/rust_out_of_tree.rs:35:9
   |
35 |         pr_info!("Rust out-of-tree sample (exit)\n");
   |         ^^^^^^^

error: cannot find macro `pr_info` in this scope
  --> /home/roughl/projects/github/Rust-for-Linux/rust-out-of-tree-module/rust_out_of_tree.rs:34:9
   |
34 |         pr_info!("My numbers are {:?}\n", self.numbers);
   |         ^^^^^^^

error: cannot find macro `pr_info` in this scope
  --> /home/roughl/projects/github/Rust-for-Linux/rust-out-of-tree-module/rust_out_of_tree.rs:21:9
   |
21 |         pr_info!("Rust out-of-tree sample (init)\n");
   |         ^^^^^^^

error: cannot find macro `module` in this scope
 --> /home/roughl/projects/github/Rust-for-Linux/rust-out-of-tree-module/rust_out_of_tree.rs:7:1
  |
7 | module! {
  | ^^^^^^

error[E0463]: can't find crate for `kernel`
  --> /home/roughl/projects/github/Rust-for-Linux/rust-out-of-tree-module/rust_out_of_tree.rs:19:6
   |
19 | impl kernel::Module for RustOutOfTree {
   |      ^^^^^^ can't find crate

error[E0433]: failed to resolve: use of undeclared type `Vec`
  --> /home/roughl/projects/github/Rust-for-Linux/rust-out-of-tree-module/rust_out_of_tree.rs:23:27
   |
23 |         let mut numbers = Vec::new();
   |                           ^^^ use of undeclared type `Vec`

error[E0412]: cannot find type `Vec` in this scope
  --> /home/roughl/projects/github/Rust-for-Linux/rust-out-of-tree-module/rust_out_of_tree.rs:16:14
   |
16 |     numbers: Vec<i32>,
   |              ^^^ not found in this scope

error[E0412]: cannot find type `ThisModule` in this scope
  --> /home/roughl/projects/github/Rust-for-Linux/rust-out-of-tree-module/rust_out_of_tree.rs:20:31
   |
20 |     fn init(_module: &'static ThisModule) -> Result<Self> {
   |                               ^^^^^^^^^^ not found in this scope

error[E0412]: cannot find type `Result` in this scope
  --> /home/roughl/projects/github/Rust-for-Linux/rust-out-of-tree-module/rust_out_of_tree.rs:20:46
   |
20 |     fn init(_module: &'static ThisModule) -> Result<Self> {
   |                                              ^^^^^^ not found in this scope

error[E0405]: cannot find trait `Drop` in this scope
  --> /home/roughl/projects/github/Rust-for-Linux/rust-out-of-tree-module/rust_out_of_tree.rs:32:6
   |
32 | impl Drop for RustOutOfTree {
   |      ^^^^ not found in this scope

error[E0635]: unknown feature `core_ffi_c`
 --> <crate attribute>:1:9
  |
1 | feature(core_ffi_c)
  |         ^^^^^^^^^^

error[E0425]: cannot find function, tuple struct or tuple variant `Ok` in this scope
  --> /home/roughl/projects/github/Rust-for-Linux/rust-out-of-tree-module/rust_out_of_tree.rs:28:9
   |
28 |         Ok(RustOutOfTree { numbers })
   |         ^^ not found in this scope

error: aborting due to 15 previous errors

Some errors have detailed explanations: E0405, E0412, E0425, E0433, E0463, E0635.
For more information about an error, try `rustc --explain E0405`.
make[2]: *** [scripts/Makefile.build:307: /home/roughl/projects/github/Rust-for-Linux/rust-out-of-tree-module/rust_out_of_tree.o] Error 1
make[1]: *** [Makefile:1992: /home/roughl/projects/github/Rust-for-Linux/rust-out-of-tree-module] Error 2
make[1]: Leaving directory '/usr/lib/modules/6.1.5-arch2-1-rust/build'
make: *** [Makefile:6: default] Error 2
```
</details>

Ok, so the build system is unable to find all the required crates that got
built as part of the kernel.

So to keep it simple, we just add everything form the `rust/` folder into the
package.

```text
..Rust-for-Linux/rust-out-of-tree-module (git)-[fix-build-for-linux-6.1] % make LLVM=1
make -C /lib/modules/`uname -r`/build M=$PWD
make[1]: Entering directory '/usr/lib/modules/6.1.5-arch2-1-rust/build'
  RUSTC [M] /home/roughl/projects/github/Rust-for-Linux/rust-out-of-tree-module/rust_out_of_tree.o
error[E0514]: found crate `core` compiled by an incompatible version of rustc
  |
  = note: the following crate versions were found:
          crate `core` compiled by rustc 1.62.0 (a8314ef7d 2022-06-27): /usr/lib/modules/6.1.5-arch2-1-rust/build/rust/libcore.rmeta
  = help: please recompile that crate using this compiler (rustc 1.66.0 (69f9c33d7 2022-12-12)) (consider running `cargo clean` first)
```
<details>
<summary>Rest of the terminal output ...</summary>
```text
error[E0514]: found crate `kernel` compiled by an incompatible version of rustc
 --> /home/roughl/projects/github/Rust-for-Linux/rust-out-of-tree-module/rust_out_of_tree.rs:5:5
  |
5 | use kernel::prelude::*;
  |     ^^^^^^
  |
  = note: the following crate versions were found:
          crate `kernel` compiled by rustc 1.62.0 (a8314ef7d 2022-06-27): /usr/lib/modules/6.1.5-arch2-1-rust/build/rust/libkernel.rmeta
  = help: please recompile that crate using this compiler (rustc 1.66.0 (69f9c33d7 2022-12-12)) (consider running `cargo clean` first)

error: cannot find macro `pr_info` in this scope
  --> /home/roughl/projects/github/Rust-for-Linux/rust-out-of-tree-module/rust_out_of_tree.rs:35:9
   |
35 |         pr_info!("Rust out-of-tree sample (exit)\n");
   |         ^^^^^^^

error: cannot find macro `pr_info` in this scope
  --> /home/roughl/projects/github/Rust-for-Linux/rust-out-of-tree-module/rust_out_of_tree.rs:34:9
   |
34 |         pr_info!("My numbers are {:?}\n", self.numbers);
   |         ^^^^^^^

error: cannot find macro `pr_info` in this scope
  --> /home/roughl/projects/github/Rust-for-Linux/rust-out-of-tree-module/rust_out_of_tree.rs:21:9
   |
21 |         pr_info!("Rust out-of-tree sample (init)\n");
   |         ^^^^^^^

error: cannot find macro `module` in this scope
 --> /home/roughl/projects/github/Rust-for-Linux/rust-out-of-tree-module/rust_out_of_tree.rs:7:1
  |
7 | module! {
  | ^^^^^^

error[E0514]: found crate `kernel` compiled by an incompatible version of rustc
  --> /home/roughl/projects/github/Rust-for-Linux/rust-out-of-tree-module/rust_out_of_tree.rs:19:6
   |
19 | impl kernel::Module for RustOutOfTree {
   |      ^^^^^^
   |
   = note: the following crate versions were found:
           crate `kernel` compiled by rustc 1.62.0 (a8314ef7d 2022-06-27): /usr/lib/modules/6.1.5-arch2-1-rust/build/rust/libkernel.rmeta
   = help: please recompile that crate using this compiler (rustc 1.66.0 (69f9c33d7 2022-12-12)) (consider running `cargo clean` first)

error[E0433]: failed to resolve: use of undeclared type `Vec`
  --> /home/roughl/projects/github/Rust-for-Linux/rust-out-of-tree-module/rust_out_of_tree.rs:23:27
   |
23 |         let mut numbers = Vec::new();
   |                           ^^^ use of undeclared type `Vec`

error[E0412]: cannot find type `Vec` in this scope
  --> /home/roughl/projects/github/Rust-for-Linux/rust-out-of-tree-module/rust_out_of_tree.rs:16:14
   |
16 |     numbers: Vec<i32>,
   |              ^^^ not found in this scope

error[E0412]: cannot find type `ThisModule` in this scope
  --> /home/roughl/projects/github/Rust-for-Linux/rust-out-of-tree-module/rust_out_of_tree.rs:20:31
   |
20 |     fn init(_module: &'static ThisModule) -> Result<Self> {
   |                               ^^^^^^^^^^ not found in this scope

error[E0412]: cannot find type `Result` in this scope
  --> /home/roughl/projects/github/Rust-for-Linux/rust-out-of-tree-module/rust_out_of_tree.rs:20:46
   |
20 |     fn init(_module: &'static ThisModule) -> Result<Self> {
   |                                              ^^^^^^ not found in this scope

error[E0405]: cannot find trait `Drop` in this scope
  --> /home/roughl/projects/github/Rust-for-Linux/rust-out-of-tree-module/rust_out_of_tree.rs:32:6
   |
32 | impl Drop for RustOutOfTree {
   |      ^^^^ not found in this scope

error[E0635]: unknown feature `core_ffi_c`
 --> <crate attribute>:1:9
  |
1 | feature(core_ffi_c)
  |         ^^^^^^^^^^

error[E0425]: cannot find function, tuple struct or tuple variant `Ok` in this scope
  --> /home/roughl/projects/github/Rust-for-Linux/rust-out-of-tree-module/rust_out_of_tree.rs:28:9
   |
28 |         Ok(RustOutOfTree { numbers })
   |         ^^ not found in this scope

error: aborting due to 14 previous errors

Some errors have detailed explanations: E0405, E0412, E0425, E0433, E0635.
For more information about an error, try `rustc --explain E0405`.
make[2]: *** [scripts/Makefile.build:307: /home/roughl/projects/github/Rust-for-Linux/rust-out-of-tree-module/rust_out_of_tree.o] Error 1
make[1]: *** [Makefile:1992: /home/roughl/projects/github/Rust-for-Linux/rust-out-of-tree-module] Error 2
make[1]: Leaving directory '/usr/lib/modules/6.1.5-arch2-1-rust/build'
make: *** [Makefile:6: default] Error 2
```
</details>

Alright, so it finds the crates, but the `rustc` versions do not match.
Apparently, the build is executed in
`/usr/lib/modules/6.1.5-arch2-1-rust/build`, so we need to tell `rustup` that we
want the 1.62.0 toolchain for that folder.

For that, we just add the `rust-toolchain` to the package as well.

```text
..Rust-for-Linux/rust-out-of-tree-module (git)-[fix-build-for-linux-6.1] % make  LLVM=1
make -C /lib/modules/`uname -r`/build M=$PWD
make[1]: Entering directory '/usr/lib/modules/6.1.5-arch2-1-rust/build'
  RUSTC [M] /home/roughl/projects/github/Rust-for-Linux/rust-out-of-tree-module/rust_out_of_tree.o
error[E0461]: couldn't find crate `core` with expected target triple target-12809083303779448358
  |
  = note: the following crate versions were found:
          crate `core`, target triple target-3911737072772191946: /usr/lib/modules/6.1.5-arch2-1-rust/build/rust/libcore.rmeta
```
<details>
<summary>Rest of the terminal output ...</summary>
```text
error[E0461]: couldn't find crate `compiler_builtins` with expected target triple target-12809083303779448358
  |
  = note: the following crate versions were found:
          crate `compiler_builtins`, target triple target-3911737072772191946: /usr/lib/modules/6.1.5-arch2-1-rust/build/rust/libcompiler_builtins.rmeta

error[E0461]: couldn't find crate `kernel` with expected target triple target-12809083303779448358
 --> /home/roughl/projects/github/Rust-for-Linux/rust-out-of-tree-module/rust_out_of_tree.rs:5:5
  |
5 | use kernel::prelude::*;
  |     ^^^^^^
  |
  = note: the following crate versions were found:
          crate `kernel`, target triple target-3911737072772191946: /usr/lib/modules/6.1.5-arch2-1-rust/build/rust/libkernel.rmeta

error: cannot find macro `pr_info` in this scope
  --> /home/roughl/projects/github/Rust-for-Linux/rust-out-of-tree-module/rust_out_of_tree.rs:35:9
   |
35 |         pr_info!("Rust out-of-tree sample (exit)\n");
   |         ^^^^^^^

error: cannot find macro `pr_info` in this scope
  --> /home/roughl/projects/github/Rust-for-Linux/rust-out-of-tree-module/rust_out_of_tree.rs:34:9
   |
34 |         pr_info!("My numbers are {:?}\n", self.numbers);
   |         ^^^^^^^

error: cannot find macro `pr_info` in this scope
  --> /home/roughl/projects/github/Rust-for-Linux/rust-out-of-tree-module/rust_out_of_tree.rs:21:9
   |
21 |         pr_info!("Rust out-of-tree sample (init)\n");
   |         ^^^^^^^

error: cannot find macro `module` in this scope
 --> /home/roughl/projects/github/Rust-for-Linux/rust-out-of-tree-module/rust_out_of_tree.rs:7:1
  |
7 | module! {
  | ^^^^^^

error[E0461]: couldn't find crate `kernel` with expected target triple target-12809083303779448358
  --> /home/roughl/projects/github/Rust-for-Linux/rust-out-of-tree-module/rust_out_of_tree.rs:19:6
   |
19 | impl kernel::Module for RustOutOfTree {
   |      ^^^^^^
   |
   = note: the following crate versions were found:
           crate `kernel`, target triple target-3911737072772191946: /usr/lib/modules/6.1.5-arch2-1-rust/build/rust/libkernel.rmeta

error[E0433]: failed to resolve: use of undeclared type `Vec`
  --> /home/roughl/projects/github/Rust-for-Linux/rust-out-of-tree-module/rust_out_of_tree.rs:23:27
   |
23 |         let mut numbers = Vec::new();
   |                           ^^^ use of undeclared type `Vec`

error[E0412]: cannot find type `Vec` in this scope
  --> /home/roughl/projects/github/Rust-for-Linux/rust-out-of-tree-module/rust_out_of_tree.rs:16:14
   |
16 |     numbers: Vec<i32>,
   |              ^^^ not found in this scope

error[E0412]: cannot find type `ThisModule` in this scope
  --> /home/roughl/projects/github/Rust-for-Linux/rust-out-of-tree-module/rust_out_of_tree.rs:20:31
   |
20 |     fn init(_module: &'static ThisModule) -> Result<Self> {
   |                               ^^^^^^^^^^ not found in this scope

error[E0412]: cannot find type `Result` in this scope
  --> /home/roughl/projects/github/Rust-for-Linux/rust-out-of-tree-module/rust_out_of_tree.rs:20:46
   |
20 |     fn init(_module: &'static ThisModule) -> Result<Self> {
   |                                              ^^^^^^ not found in this scope

error[E0425]: cannot find function, tuple struct or tuple variant `Ok` in this scope
  --> /home/roughl/projects/github/Rust-for-Linux/rust-out-of-tree-module/rust_out_of_tree.rs:28:9
   |
28 |         Ok(RustOutOfTree { numbers })
   |         ^^ not found in this scope

error[E0405]: cannot find trait `Drop` in this scope
  --> /home/roughl/projects/github/Rust-for-Linux/rust-out-of-tree-module/rust_out_of_tree.rs:32:6
   |
32 | impl Drop for RustOutOfTree {
   |      ^^^^ not found in this scope

error[E0635]: unknown feature `core_ffi_c`
 --> <crate attribute>:1:9
  |
1 | feature(core_ffi_c)
  |         ^^^^^^^^^^

error: aborting due to 15 previous errors

Some errors have detailed explanations: E0405, E0412, E0425, E0433, E0635.
For more information about an error, try `rustc --explain E0405`.
make[2]: *** [scripts/Makefile.build:307: /home/roughl/projects/github/Rust-for-Linux/rust-out-of-tree-module/rust_out_of_tree.o] Error 1
make[1]: *** [Makefile:1992: /home/roughl/projects/github/Rust-for-Linux/rust-out-of-tree-module] Error 2
make[1]: Leaving directory '/usr/lib/modules/6.1.5-arch2-1-rust/build'
make: *** [Makefile:6: default] Error 2
```
</details>

I didn't really understand what the strange target triple meant. Usually a
target triple looks something like `x86_64-unknown-linux-gnu` ([^2]). But in
our case we are using a `target.json` to define the target, so I've no clue
where the difference comes from, since we are using the same JSON file.

## Summing it up

While we managed to compile and run our out-of-tree kernel module in Rust, we
couldn't get the build metadata packaged in a way to do it without a local
build of the kernel.

Well, let's see if anything changes with the 6.2 Linux kernel which should get
released in February. In the meantime, this kernel provides an easy way to start
playing with Rust kernel modules on ArchLinux.

[^1]: The name of the package may be a bit misleading, since the package not
  only bundles the headers but also build scripts and other metadata required
  to build modules.
[^2]: See also <https://doc.rust-lang.org/rustc/targets/index.html>
