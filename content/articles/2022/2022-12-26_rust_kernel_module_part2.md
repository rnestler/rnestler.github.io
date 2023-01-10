Title: Building an out-of-tree Rust Kernel Module Part Two
Tags: rust, kernel, Linux, ArchLinux
Language: en
Summary: Trying to build a hello world out-of-tree Rust kernel module for Linux 6.1. Part two.
Status: draft

In my last [blog posts](//building-an-out-of-tree-rust-kernel-module.html) I
tried to build an out-of-tree kernel module in Rust for the 6.1 stock kernel
for ArchLinux.

During this process I noticed, that while `CONFIG_HAVE_RUST=y` is set in the
[ArchLinux kernel config](https://github.com/archlinux/svntogit-packages/blob/706494f4555dca158c9f02932717550e7b66b534/trunk/config)
, Rust support get's silently disabled, since conflicting options are set.

## Building our own ArchLinux kernel

So I went ahead and started to build my own kernel with a custom config:
<https://aur.archlinux.org/packages/linux-rust>.

I based it of the official ArchLinux kernel package, disabled the conflicting
options, ran `make menuconfig` and enabled Rust support. 

A few iterations on the configuration and `PKGBUILD` later I had a successfully
building kernel with Rust support enabled!

After installing the custom kernel and booting into it we should finally be
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
