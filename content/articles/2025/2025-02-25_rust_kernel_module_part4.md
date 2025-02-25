Title: Building an out-of-tree Rust Kernel Module Part Four
Tags: rust, kernel, Linux, ArchLinux
Language: en
Summary: Building the Linux kernel with Rust support for ArchLinux
Status: draft

In my [last blog post], I showed how to build the ArchLinux kernel with Rust
support enabled and also packaging the correct build metadata to support
out-of-tree kernel modules.

In the meantime I gave a presentation about it in the [Rust Zürisee Meetup] and
kept the [AUR package] more or less up to date.

Also some people got interested in enabling it for the official kernel build
for ArchLinux[^1] but hit some issues and the change got reverted again.

## Building with the distro rust-bindgen

With the release of Linux 6.11, instead of a pinned version there should be an
actual minimum version for the Rust toolchain to build the Linux with Rust
support. See the [git pull request for 6.11] and the [rust-version-policy] on
<https://rust-for-linux.com>.

Using the distro packaged Rust toolchain should make it easier to be included
in the official repositories some day, since using rustup to pin the version
leads to issues when building in a clean chroot environment[^2].

So in December 2024 I started with using rust-bindgen from the Arch
repositories: <https://github.com/rnestler/archpkg-linux-rust/pull/11>. This
lead into a roadblock when I noticed, that rust-bindgen 0.71.1 breaks the build
of Linux 6.11 with Rust 1.78, while it worked just fine with rust-bindgen
0.70.1.

In January 2025, when updating to to the 6.12 version of the kernel I upgraded
the PKGBUILD to use Rust 1.83 and the current ArchLinux version of
rust-bindgen: <https://github.com/rnestler/archpkg-linux-rust/pull/12/>. This
seemed to work just fine.

Do to unclarities how to ensure that the Rust version used to build the kernel
is the same that will be used to build out-of-tree modules, I decided to still
pin the Rust version used and only support building the kernel with rustup.

## Building with the whole distro Rust toolchain

In February a nice pull-request hit my archpkg-linux-rust repo:
<https://github.com/rnestler/archpkg-linux-rust/pull/13>. The author of it
fixed a bug in the `PKGBUILD` and adapted it as well to build with `rustc` and
`rust-src` from the repos.

TODO:
 * Show the provides mechanism of pacman and how it allows to support building with rustc and rustup

## Looking forward

Since we can now build the Linux kernel with Rust support using the distro
packages it should be possible to build it in a clean chroot environment and
thus there shouldn't be much in the way of Rust support landing in official
kernel packages.

[^1]: See the discussion in the official package repository: <https://gitlab.archlinux.org/archlinux/packaging/packages/linux/-/issues/63>
[^2]: See the comments for the AUR package <https://aur.archlinux.org/packages/linux-rust#comment-990983>

[last blog post]: https://aur.archlinux.org/packages/linux-rust#comment-990983
[Rust Zürisee Meetup]: https://www.meetup.com/rust-zurich/events/293322905/?eventOrigin=group_events_list
[AUR package]: https://aur.archlinux.org/packages/linux-rust
[git pull request for 6.11]: https://lkml.org/lkml/2024/7/25/53
[rust-version-policy]: https://rust-for-linux.com/rust-version-policy
