Title: Building an out-of-tree Rust Kernel Module Part Three
Tags: rust, kernel, Linux, ArchLinux
Status: draft
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

Updating it: https://github.com/Rust-for-Linux/rust-out-of-tree-module/pull/5

Reasons for the previous error:
 * https://github.com/Rust-for-Linux/linux/issues/792
 * https://github.com/rust-lang/rust/pull/98225

[linux-rust]: https://aur.archlinux.org/packages/linux-rust
[config]: TODO
