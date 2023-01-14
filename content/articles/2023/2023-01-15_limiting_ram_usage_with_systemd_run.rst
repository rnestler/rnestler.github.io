Limiting process RAM usage with systemd-run
===========================================

:tags: Linux, ArchLinux, systemd, kernel
:language: en
:summary: Limiting RAM usage of a build process with systemd-run
:status: draft

While building the Linux kernel with Rust support for ArchLinux I run into an
issue, that building the HTML documentation sphinx-build starts to use
excessive amounts of RAM.

This resulted in the system being un-responsive before the OOM killer decided
to just kill my user session.

.. figure:: {static}/images/limiting_ram_usage/sphinx-build-excessive-ram-usage.png
    :target: {static}/images/limiting_ram_usage/sphinx-build-excessive-ram-usage.png
    :alt: htop output shortly before killing the build process
    :align: center
    :width: 80%
    :figwidth: 100%

    htop output shortly before killing the build process

Searching for a way to not have the OOM killer kill my session, but kill the
build process before it uses all my available RAM I found
https://unix.stackexchange.com/a/536046.

So spawning the build with the following command will kill the build process
once it uses more than 20GB of RAM:

.. sourcecode:: console

    systemd-run --scope -p MemoryMax=20G --user makepkg

