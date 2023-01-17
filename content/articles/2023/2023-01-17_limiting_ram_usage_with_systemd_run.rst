Limiting process RAM usage with systemd-run
===========================================

:tags: Linux, ArchLinux, systemd, kernel
:language: en
:summary: Limiting RAM usage of a build process with systemd-run
:status: draft

While building the Linux kernel with `Rust support for ArchLinux
</building-an-out-of-tree-rust-kernel-module-part-two.html>`_, I ran into a
problem where sphinx-build started using excessive amounts of RAM while
building the HTML documentation.

This caused the system to become unresponsive before the OOM killer decided to
just kill my user session.

.. figure:: {static}/images/limiting_ram_usage/sphinx-build-excessive-ram-usage.png
    :target: {static}/images/limiting_ram_usage/sphinx-build-excessive-ram-usage.png
    :alt: htop just shortly before killing the build process
    :align: center
    :width: 80%
    :figwidth: 100%

    htop output just before killing the build process

Looking for a way to not have the OOM killer kill my session, but kill the
build process before it uses up all my available RAM, I found
https://unix.stackexchange.com/a/536046.

So, spawning the build with the following command will kill the build process
as soon as it uses more than 20GB of RAM:

.. sourcecode:: console

    $ systemd-run --scope -p MemoryMax=20G --user makepkg
    Running scope as unit: run-r44087c0ea5d445fda135dc0db42b36ab.scope
    [1]    426653 killed     systemd-run --scope -p MemoryMax=20G --user makepkg
