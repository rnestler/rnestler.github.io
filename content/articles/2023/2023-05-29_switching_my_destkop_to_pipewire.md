Title: Switching my Desktop to PipeWire
Tags: PipeWire, PulseAudio
Status: draft
Language: en
Summary: How I switched my desktop from PulseAudio to PipeWire

# Background

While I did setup my new laptop directly with PipeWire back in December 2021,
my existing desktop PC did remain with a PulseAudio setup.

## What is PipeWire?

> [PipeWire] is a new low-level multimedia framework. It aims to offer capture
> and playback for both audio and video with minimal latency and support for
> PulseAudio, JACK, ALSA and GStreamer-based applications. 

> Source: <https://wiki.archlinux.org/title/PipeWire>

The reason I wanted to switch was, that I had some troubles with my bluetooth
headphones (only the SBC codec was available in the A2DP profile and I had some
reliability issues) and I heard that PipeWire should make this better.

## Installation

Most of the pipewire packages were already installed as dependencies for other packages:
<details>
<summary>

<tt>pacman -Qi pipewire</tt>

</summary>

```text
Name            : pipewire
Version         : 1:0.3.71-1
Description     : Low-latency audio/video router and processor
Architecture    : x86_64
URL             : https://pipewire.org
Licenses        : MIT  LGPL
Groups          : None
Provides        : None
Depends On      : libpipewire=1:0.3.71-1  libcamera-base.so=0.0.5-64  libcamera.so=0.0.5-64  libdbus-1.so=3-64  libglib-2.0.so=0-64  libncursesw.so=6-64  libpipewire-0.3.so=0-64  libreadline.so=8-64  libsystemd.so=0-64  libudev.so=1-64
Optional Deps   : gst-plugin-pipewire: GStreamer plugin
                  pipewire-alsa: ALSA configuration
                  pipewire-audio: Audio support
                  pipewire-docs: Documentation
                  pipewire-jack: JACK support
                  pipewire-pulse: PulseAudio replacement
                  pipewire-roc: ROC streaming
                  pipewire-session-manager: Session manager [installed]
                  pipewire-v4l2: V4L2 interceptor
                  pipewire-x11-bell: X11 bell
                  pipewire-zeroconf: Zeroconf support
                  realtime-privileges: realtime privileges with rt module
                  rtkit: realtime privileges with rtkit module [installed]
Required By     : lib32-pipewire  obs-studio  telegram-desktop  wireplumber  xdg-desktop-portal  xdg-desktop-portal-wlr
Optional For    : chromium  electron  electron19  qt5-webengine  qt6-webengine  sdl2
Conflicts With  : None
Replaces        : None
Installed Size  : 3.11 MiB
Packager        : David Runge <dvzrv@archlinux.org>
Build Date      : So 21 Mai 2023 15:17:29
Install Date    : Di 23 Mai 2023 22:53:42
Install Reason  : Installed as a dependency for another package
Install Script  : Yes
Validated By    : Signature
```
</details>

But first we install `pipewire-audio`:
```text
$ sudo pacman -S pipewire-audio
```

Then we remove `pulseaudio-alsa` and install `pipewire-alsa`:
```text
$ sudo pacman -Rs pulseaudio-alsa
$ sudo pacman -S pipewire-alsa
```

Then we install `pipewire-pulse` (which will remove `pulseaudio` and `pulseaudio-bluetooth`)
```text
$ sudo pacman -S pipewire-pulse
resolving dependencies...
looking for conflicting packages...
:: pipewire-pulse and pulseaudio are in conflict. Remove pulseaudio? [y/N] y
:: pipewire-pulse and pulseaudio-bluetooth are in conflict. Remove pulseaudio-bluetooth? [y/N] y
```

When trying to stop the (now uninstalled) pulseaudio service to start pipewire
I got an error:

```
$ sudo systemctl stop pulseaudio.service
Failed to stop pulseaudio.service: Unit pulseaudio.service not loaded.
```

Since I didn't want to invest too much time (maybe I should just have stopped
it before uninstalling it?) I just logged out and back in to restart my
session.[^session]

## Problems

After restarting the session pulseaudio clients seemed to be broken:
```bash
$ pactl stat
Connection failure: Connection refused
pa_context_connect() failed: Connection refused
```

It turns out that for some reason the pipewire-pulse.socket listener wasn't
active:
```bash
$ systemctl --user status pipewire-pulse.socket
○ pipewire-pulse.socket - PipeWire PulseAudio
     Loaded: loaded (/usr/lib/systemd/user/pipewire-pulse.socket; enabled; preset: enabled)
     Active: inactive (dead)
   Triggers: ● pipewire-pulse.service
     Listen: /run/user/1000/pulse/native (Stream)
```

Starting it is as easy as `systemctl --user start pipewire-pulse.socket` and
now `pactl` shows some output:

```
$ pactl stat
Currently in use: 2 blocks containing 8.0 KiB bytes total.
Allocated during whole lifetime: 2 blocks containing 8.0 KiB bytes total.
Sample cache size: 0 B
```

Also pavucontrol now shows me LDAC, AAC, and SBC-XQ in addition to just SBC:

![pavucontrol showing more codecs]({static}/images/pipewire/pavucontrol-bluetooth-profiles.png){width=100%}

[PipeWire]: https://pipewire.org/
[^session]: I should probably have used `systemctl --user stop pulseaudio.service`, since these are user units.
