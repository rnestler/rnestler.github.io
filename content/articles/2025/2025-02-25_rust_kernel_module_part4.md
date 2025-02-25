Title: Building an out-of-tree Rust Kernel Module Part Four
Tags: rust, kernel, Linux, ArchLinux
Language: en
Summary: Building the Linux kernel with Rust support for ArchLinux
Status: draft

In my last [blog
post](/building-an-out-of-tree-rust-kernel-module-part-three.html), I showed
how to build the ArchLinux kernel with Rust support enabled and also packaging
the correct build metadata to support out-of-tree kernel modules.


In the meantime I gave a presentation about it in the [Rust Zürisee Meetup] and
kept the [AUR package] more or less up to date.

## Building with the distro Rust version

## Looking forward

In the meantime Linux 6.3 got released and 6.4 will be released shortly. But it
doesn't look like they will bump the minimum version of Rust needed to compile:
<https://github.com/torvalds/linux/blob/v6.4-rc6/scripts/min-tool-version.sh#L29>

But reading <https://rust-for-linux.com/rust-version-policy> it looks like
there are plans to start tracking the latest Rust release for Linux:

> It remains to be decided how often the Rust version upgrades will land.
> Ideally we would track the latest Rust release, but it remains to be seen how
> other kernel developers feel about it.

[Rust Zürisee Meetup]: https://www.meetup.com/rust-zurich/events/293322905/?eventOrigin=group_events_list
[AUR package]: https://aur.archlinux.org/packages/linux-rust 
