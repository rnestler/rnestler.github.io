Title: Switching my Raspberry Pi 3 running Kodi to aarch64
Tags: Raspberry Pi, Linux, ArchLinux, ARM, Kodi
Language: en
Status: draft
Summary: Switching my Raspberry Pi 3 B+ which runs Kodi to aarch64

Back in [2023](//installing-archlinux-arm-on-the-raspberry-pi-3.html) I set up
my Raspberry Pi 3 B+ with Kodi and it served me well, even if the hardware is
quite low power.


## Kodi crashing to start

TODO

```
What error was shown exactly?

```

# Seting up snapcast

Following my own instructions from a [previous
blogpost](//creating-a-raspberry-pi-based-media-system.html) I set up the
snapcast client.

## Cross Compiling snapcast

I followed again the guide from the [ArchLinux distcc wiki
page](https://wiki.archlinux.org/title/Distcc#Cross_compiling_with_distcc),
just changing from the armv7 ot the armv8 toolchain:

### Distcc Master System (Raspberry Pi)

```
sudo pacman -S distcc
sudo vim /etc/makepkg.conf
# Edit / add the following lines
BUILDENV=(distcc color !ccache check !sign)
DISTCC_HOSTS="$IP_OF_SLAVE:3636"
MAKEFLAGS="-j4"
```

### Distcc Slave System (Desktop PC)

```
yay distccd-alarm-armv8 
sudo systemctl start distccd-armv8.service
```


## Building my own dashboard

 * Cross compiling for ArchLinuxARM
 * Creating a ArchLinuxARM Docker container
 * Cargo cross

No fonts installed

```
Using FemtoVG OpenGL renderer
Rendering at 1920x1080

thread 'main' (14238) panicked at /cargo/registry/src/index.crates.io-1949cf8c6b5b557f/fontique-0.7.0/src/backend/fontconfig.rs:685:61:
called `Result::unwrap()` on an `Err` value: NoMatch
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```
