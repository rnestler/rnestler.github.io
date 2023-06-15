Title: Building an out-of-tree Rust Kernel Module Part Three
Tags: rust, kernel, Linux, ArchLinux
Language: en
Summary: Trying to build a hello world out-of-tree Rust kernel module for Linux. Part Three.

In my last [blog post](/building-an-out-of-tree-rust-kernel-module-part-two.html),
I managed to build an out-of-tree kernel module in Rust for a custom 6.1 Linux
kernel I packaged for ArchLinux:
<https://aur.archlinux.org/packages/linux-rust>.

While this worked, I couldn't manage to package the correct metadata to build
the out-of-tree kernel module without a local build of the kernel, which kind
of defeats the purpose of the exercise.

In the meantime Linux 6.2 got released so I thought I'd try again.

## Updating our own ArchLinux kernel

So I went ahead and updated my [linux-rust] package to a 6.2 kernel. While
doing this I tried to stay as close as possible to the default ArchLinux kernel
packages [config] file to make it generally usable.

## Building the out-of-tree module

Linux 6.2 changed the module metadata strings from byte arrays to proper
`str`s. For that we needed to update the kernel module. See:
<https://github.com/Rust-for-Linux/rust-out-of-tree-module/pull/5>

## Same error as before

So I went ahead recompiled the kernel, rebooted, tried to compile the kernel
module and got the same error as before:
```text
..Rust-for-Linux/rust-out-of-tree-module (git)-[main] % make LLVM=1
make -C /lib/modules/`uname -r`/build M=$PWD
  RUSTC [M] /home/roughl/projects/github/Rust-for-Linux/rust-out-of-tree-module/rust_out_of_tree.o
error[E0461]: couldn't find crate `core` with expected target triple target-12809083303779448358
  |
  = note: the following crate versions were found:
          crate `core`, target triple target-5559158138856098584: /usr/lib/modules/6.2.10-arch1-1-rust/build/rust/libcore.rmeta
```

## Finding the root cause for the error

But this time I wanted to find out what the cause for this issue is! It turns
out that it is a bug in rustc, where the file *path* instead of the *content*
of the `target.json` where used. See

 * <https://github.com/Rust-for-Linux/linux/issues/792>
 * <https://github.com/rust-lang/rust/pull/98225>

## Compiling with Rust 1.63.0

So since this bug got fix in Rust 1.63.0 I decided to try to compile the kernel
with a newer (and thus unsupported, since the Linux kernel uses unstable Rust
features) version.

This will print a scary warning during the build process:
```
***
*** Rust compiler 'rustc' is too new. This may or may not work.
***   Your version:     1.63.0
***   Expected version: 1.62.0
***
```

Also there were a few warnings from the Rust compiler itself about features
that got now stabilized:

```text
warning: the feature `nll` has been stable since 1.63.0 and no longer requires an attribute to enable
   --> rust/alloc/lib.rs:170:12
    |
170 | #![feature(nll)] // Not necessary, but here to test the `nll` feature.
    |            ^^^
    |
    = note: `#[warn(stable_features)]` on by default
```

## Successfully building an out-of-tree kernel module!

So finally I was able to compile the out-of-tree module without a local build
of the kernel:
```text
..Rust-for-Linux/rust-out-of-tree-module (git)-[main] % make LLVM=1
make -C /lib/modules/`uname -r`/build M=$PWD
  RUSTC [M] /home/roughl/projects/github/Rust-for-Linux/rust-out-of-tree-module/rust_out_of_tree.o
  MODPOST /home/roughl/projects/github/Rust-for-Linux/rust-out-of-tree-module/Module.symvers
  CC [M]  /home/roughl/projects/github/Rust-for-Linux/rust-out-of-tree-module/rust_out_of_tree.mod.o
  LD [M]  /home/roughl/projects/github/Rust-for-Linux/rust-out-of-tree-module/rust_out_of_tree.ko
```

Which also loaded successfully:
```text
[125766.698929] rust_out_of_tree: loading out-of-tree module taints kernel.
[125766.698985] rust_out_of_tree: module verification failed: signature and/or required key missing - tainting kernel
[125766.699611] rust_out_of_tree: Rust out-of-tree sample (init)
[125771.403175] rust_out_of_tree: My numbers are [72, 108, 200]
[125771.403183] rust_out_of_tree: Rust out-of-tree sample (exit)
```

I also spent some time figuring out which parts of the build metadata are
really needed, since previously I just copied the whole `rust/` folder. In the
end we just need:

 * `rust/target.json` the custom target description
 * `rust/*.rmeta` the compiled rust libraries
 * `rust/*.so` the pre-compiled procedural macros
 * `rust-toolchain` such that we use the same Rust compiler version that was
   used to compile the kernel

## Looking forward

In the meantime Linux 6.3 got released and 6.4 will be released shortly. But it
doesn't look like they will bump the minimum version of Rust needed to compile:
<https://github.com/torvalds/linux/blob/v6.4-rc6/scripts/min-tool-version.sh#L29>

But reading <https://rust-for-linux.com/rust-version-policy> it looks like
there are plans to start tracking the latest Rust release for Linux:

> It remains to be decided how often the Rust version upgrades will land.
> Ideally we would track the latest Rust release, but it remains to be seen how
> other kernel developers feel about it.

[linux-rust]: https://aur.archlinux.org/packages/linux-rust
[config]: https://gitlab.archlinux.org/archlinux/packaging/packages/linux/-/blob/6.2.10.arch1-1/config?ref_type=tags
