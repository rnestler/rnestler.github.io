Title: Running Spotify on ArchLinux ARM on the Raspberry Pi 3
Tags: Raspberry Pi, Linux, ArchLinux, ARM
Language: en
Status: draft
Summary: Getting Spotify to be able to play music on my Raspberry Pi 3

# Trying the Kodi Add-On

The first thing I tried was to look for a Spotify Kodi Add-On. What I found
that looked promising was the following:
<https://github.com/glk1001/glk1001.github.io>

But after installing and getting it to run I encountered a few issues:

 * It was a bit cumbersome to navigate from inside Kodi and a bit laggy
 * Spotify connect wasn't working from my phone
 * My spotify account got blocked after a while and I received the follwoing
   email from spotify:
   > To protect your Spotify account, we have reset your password as we have
   > detected suspicious activity.

Since I have previously used `spotifyd` and it seemed to work great with
Spotify connect, I decided to not bother any longer and try it out on the Pi as
well.

# Trying spotifyd

## Installation

Luckily ArchLinux ARM provides us with a package out of the box:
```bash
$ sudo pacman -S spotifyd
```

## Running as a system service

Since I didn't want to need to log in to run spotifyd as a user service I tried
to run it as a system service:

```bash
$ sudo cp /usr/lib/systemd/user/spotifyd.service  /etc/systemd/system
$ sudo systemctl daemon-reload
# Create /etc/spotifyd.conf with the following content
[global]
use_mpris = false
$ sudo systemctl enable spotifyd.service
$ sudo systemctl start spotifyd.service
```

<details>
<summary>Why do we need to disable mpris?</summary>

When running as a system service, spotifyd can't connect to the user session
dbus service and will crash when starting:

```text
Dec 03 20:42:01 alarmpi spotifyd[6793]: The application panicked (crashed).
Dec 03 20:42:01 alarmpi spotifyd[6793]: Message:  Failed to initialize DBus connection: D-Bus error: Using X11 for dbus-daemon autolaunch was disabled at compile time, set your DBUS_SESSION_BUS_ADDRESS instead (>
Dec 03 20:42:01 alarmpi spotifyd[6793]: Location: src/dbus_mpris.rs:169
Dec 03 20:42:01 alarmpi spotifyd[6793]: Backtrace omitted. Run with RUST_BACKTRACE=1 environment variable to display it.
Dec 03 20:42:01 alarmpi spotifyd[6793]: Run with RUST_BACKTRACE=full to include source snippets.
Dec 03 20:42:01 alarmpi systemd[1]: spotifyd.service: Main process exited, code=exited, status=101/n/a
Dec 03 20:42:01 alarmpi systemd[1]: spotifyd.service: Failed with result 'exit-code'.`
```
</details>

So while ths worked quite well it has two downsides:

 * Running as a system service means we run spotifyd as root. This is not nice
   from a security perspective (we could probably change this by creating a new
   user to run it as)
 * Can't control it via playerctl / MPRIS over dbus

## Running it as user service

So how can we run spotifyd on boot, but still run it in the user session? The
trick is to
[`enable-linger`](https://www.freedesktop.org/software/systemd/man/latest/loginctl.html#enable-linger%20USER%E2%80%A6)
for the user that should run the spotifyd service and then enable it as a user
service. See also
<https://github.com/Spotifyd/spotifyd/wiki/Installing-on-a-Raspberry-Pi#starting-spotifyd-at-boot>

```
# As the user 'alarm'
$ loginctl enable-linger
$ systemctl --user enable spotifyd.service
```

When then trying to play something over spotify I noticed, that spotifyd crashes:
```
$ journalctl --user -u spotifyd.service
...
Dec 05 14:57:50 alarmpi spotifyd[2589]: ALSA lib confmisc.c:855:(parse_card) cannot find card '0'
Dec 05 14:57:50 alarmpi spotifyd[2589]: ALSA lib conf.c:5204:(_snd_config_evaluate) function snd_func_car>
Dec 05 14:57:50 alarmpi spotifyd[2589]: ALSA lib confmisc.c:422:(snd_func_concat) error evaluating strings
Dec 05 14:57:50 alarmpi spotifyd[2589]: ALSA lib conf.c:5204:(_snd_config_evaluate) function snd_func_con>
Dec 05 14:57:50 alarmpi spotifyd[2589]: ALSA lib confmisc.c:1342:(snd_func_refer) error evaluating name
Dec 05 14:57:50 alarmpi spotifyd[2589]: ALSA lib conf.c:5204:(_snd_config_evaluate) function snd_func_ref>
Dec 05 14:57:50 alarmpi spotifyd[2589]: ALSA lib conf.c:5727:(snd_config_expand) Evaluate error: No such >
Dec 05 14:57:50 alarmpi spotifyd[2589]: ALSA lib pcm.c:2675:(snd_pcm_open_noupdate) Unknown PCM default
Dec 05 14:57:50 alarmpi spotifyd[2589]: Audio Sink Error Connection Refused: <AlsaSink> Device default Ma>
Dec 05 14:57:50 alarmpi systemd[458]: spotifyd.service: Main process exited, code=exited, status=1/FAILURE
Dec 05 14:57:50 alarmpi systemd[458]: spotifyd.service: Failed with result 'exit-code'.
```

Probably we need to add our user to the `audio` group:

```bash
$ sudo usermod -a -G audio alarm
```

After that a reboot was necessary so the spotifyd service starts with the new
permissions.

## Integrating With Kodi

I wanted to have at least some integration with Kodi, so I decided to write a
little script to show which song is playing.

```bash
sudo pacman -S kodi-rpi-eventclients playerctl
```

Then a simple script at `~/bin/spotifyd_kodi_notification.sh`:
```bash
#!/usr/bin/bash
song=$(playerctl metadata title)
kodi-send --host 127.0.0.1 --action "Notification(Spotify,$song,5000,DefaultIconInfo.png)"
```

and configure spotifyd to call it in `~/.config/spotifyd/spotifyd.conf`:

```
[global]
on_song_change_hook = '~/bin/spotifyd_kodi_notification.sh'
```

And we get nice little notifiations whenever a song changes.

A bit annoying is, that the script needs almost 3 seconds to execute:
```bash
$ time bin/spotifyd_kodi_notification.sh
Sending: {'type': 'action', 'content': 'Notification(Spotify,Hu Chai,5000,DefaultIconInfo.png)'}

real	0m2.899s
user	0m0.318s
sys	0m0.072s
```
