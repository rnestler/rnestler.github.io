Title: Creating a Raspberry Pi Based Media System
Tags: Raspberry Pi, Linux, ArchLinux, ARM, snapcast, volumio, mopidy
Language: en
Summary: Setting up a media system on a Raspberry Pi 3 based on ArchLinux ARM, snapcast and mopidy

For quite some time I wanted to setup a Raspberry Pi based sound system with
which I could do the following:

 * Directly connect speakers to it
 * Stream audio from various sources
 * Connect a small USB based sound mixer
 * Run spotifyd
 * Play music from my NAS

# Resources

Researching a bit I found the following interesting resources

   * <https://hifiberry.com/>
   * <https://volumio.com/>
   * <https://github.com/badaix/snapcast>
   * <https://mopidy.com/>
   * <http://techroadtrip.com/audio-software/volumio-vs-moode-vs-picoreplayer/>
   * <https://www.runeaudio.com/>
   * <https://github.com/dbrgn/weltempfaenger/>
   * <https://www.home-assistant.io/blog/2016/02/18/multi-room-audio-with-snapcast/>


# Hardware

For the hardware I settled for

 * A Raspberry Pi 3 which I had laying around anyway
 * [HiFiBerry Amp2](https://www.hifiberry.com/shop/boards/hifiberry-amp2/)
   which looked very promising and wasn't too expensive

![My hardware setup]({static}/images/snapcast/raspberry-pi-audio-setup.jpg){width=100%}

# Software

The first thing I tried was [volumio](https://volumio.com/), which directly
offers a convenient Raspberry Pi image which gets you started quickly:
<https://volumio.com/get-started/>.

While it was very convenient to get started and verify that the amplifier
works, it seemed to me that the free version is very limited and locked down.
Also the premium price with 80$ (around 70 CHF) per year seemed a bit expensive
to me.

Since I wanted something which lets me tinker around a bit more I came up with
the following options:

 * [Buildroot](https://buildroot.org/)
 * [ArchLinuxARM](https://archlinuxarm.org/)

While buildroot would probably allow to get a much more minimal system and a
friend of mine had a good experience using it for [his
project](https://github.com/dbrgn/weltempfaenger), the guide I found for
[setting up
snapcast](https://github.com/badaix/snapos/blob/master/buildroot-external/README.md)
seemed out of date and not maintained anymore.
Also I already know ArchLinux and used ArchLinuxARM for my [kodi
system](https://blog.rnstlr.ch/installing-archlinux-arm-on-the-raspberry-pi-3.html).

## Installing ArchLinux ARM


<details>
<summary>So I followed the basic install guide.</summary>

```text
[alarm@alarmpi ~]$ su
Password:
[root@alarmpi alarm]# pacman-key --init
gpg: /etc/pacman.d/gnupg/trustdb.gpg: trustdb created
gpg: no ultimately trusted keys found
gpg: starting migration from earlier GnuPG versions
gpg: porting secret keys from '/etc/pacman.d/gnupg/secring.gpg' to gpg-agent
gpg: migration succeeded
==> Generating pacman master key. This may take some time.
gpg: Generating pacman keyring master key...
gpg: directory '/etc/pacman.d/gnupg/openpgp-revocs.d' created
gpg: revocation certificate stored as '/etc/pacman.d/gnupg/openpgp-revocs.d/8277D1E09B6B70944D92CD089C16C729F5655E65.rev'
gpg: Done
==> Updating trust database...
gpg: marginals needed: 3  completes needed: 1  trust model: pgp
gpg: depth: 0  valid:   1  signed:   0  trust: 0-, 0q, 0n, 0m, 0f, 1u
[root@alarmpi alarm]# pacman-key --populate archlinuxarm
==> Appending keys from archlinuxarm.gpg...
==> Locally signing trusted keys in keyring...
  -> Locally signed 3 keys.
==> Importing owner trust values...
gpg: setting ownertrust to 4
gpg: inserting ownertrust of 4
gpg: setting ownertrust to 4
==> Updating trust database...
gpg: marginals needed: 3  completes needed: 1  trust model: pgp
gpg: depth: 0  valid:   1  signed:   3  trust: 0-, 0q, 0n, 0m, 0f, 1u
gpg: depth: 1  valid:   3  signed:   1  trust: 0-, 0q, 0n, 3m, 0f, 0u
gpg: depth: 2  valid:   1  signed:   0  trust: 1-, 0q, 0n, 0m, 0f, 0u
[root@alarmpi alarm]# pacman -Syu
:: Synchronizing package databases...
 core                            240.3 KiB   215 KiB/s 00:01 [################################] 100%
 extra                             9.0 MiB  5.62 MiB/s 00:02 [################################] 100%
 community                        45.0   B   236   B/s 00:00 [################################] 100%
 alarm                            94.9 KiB   474 KiB/s 00:00 [################################] 100%
 aur                               9.3 KiB  49.1 KiB/s 00:00 [################################] 100%
:: Starting full system upgrade...
:: Replace raspberrypi-firmware with alarm/raspberrypi-utils? [Y/n]
```

</details>

I then changed the hostname to avoid the conflict with my existing archlinuxarm
machine in the network and rebooted:
```text
[root@alarmpi alarm]# hostnamectl hostname muzikskatolo
[root@alarmpi alarm]# reboot
```

## Setting up the network

To get wifi running I installed `iwd`:
```text
[root@muzikskatolo alarm]# pacman -S iwd
[root@muzikskatolo alarm]# systemctl enable iwd.service
[root@muzikskatolo alarm]# systemctl start iwd.service
```

Then connected to my wifi

```text
[root@muzikskatolo alarm]# iwctl
NetworkConfigurationEnabled: disabled
StateDirectory: /var/lib/iwd
Version: 2.19
[iwd]# station wlan0 scan
[iwd]# station wlan0 connect wifi-ssid
Type the network passphrase for wifi-ssid psk.
Passphrase:
```

And configured systemd to start DHCP for the interface by creating
`/etc/systemd/network/wlan0.network` with the following content:

```conf
[Match]
Name=wlan0

[Network]
DHCP=yes
IgnoreCarrierLoss=3s
```

Followed by

```text
[root@muzikskatolo alarm]# systemctl reload systemd-networkd.service
```

Also I disabled waiting for all networks to be online, since the Pi will only
have wifi were I want to place it:

```text
[root@muzikskatolo alarm]# systemctl disable systemd-networkd-wait-online.service
[root@muzikskatolo alarm]# systemctl enable systemd-networkd-wait-online@wlan0.service
```

## Enabling the Audio

To get the amp running I enabled the `hifiberry-dacplus` `dtoverlay` in `/boot/config.txt`:

```ini
# https://www.hifiberry.com/docs/data-sheets/datasheet-amp2/
dtoverlay=hifiberry-dacplus,slave
```

For the next step I powered down and hooked up the Raspberry Pi to the
hardware.

## Configuring Audio

To be able to test I needed to add the `alarm` user to the `audio` group:

```
sudo usermod -G audio -a alarm
```

Then I used `aplay -l` to find out which device got which card index:

```text
$ aplay -l
**** List of PLAYBACK Hardware Devices ****
card 0: vc4hdmi [vc4-hdmi], device 0: MAI PCM i2s-hifi-0 [MAI PCM i2s-hifi-0]
  Subdevices: 1/1
  Subdevice #0: subdevice #0
card 1: sndrpihifiberry [snd_rpi_hifiberry_dacplus], device 0: HiFiBerry DAC+ HiFi pcm512x-hifi-0 [HiFiBerry DAC+ HiFi pcm512x-hifi-0]
  Subdevices: 1/1
  Subdevice #0: subdevice #0
```

To route audio by default to the hifiberry device I configured alsa to use
card-1 by default I created `/etc/alsa/conf.d/default.conf` with the following
content ([^1]):

```
defaults.pcm.card 1
defaults.ctl.card 1
```

Then I used `speaker-test` to test the speakers

```text
$ speaker-test -c 2

speaker-test 1.2.12

Playback device is default
Stream parameters are 48000Hz, S16_LE, 2 channels
Using 16 octaves of pink noise
Rate set to 48000Hz (requested 48000Hz)
Buffer size range from 8 to 131072
Period size range from 4 to 65536
Periods = 4
was set period_size = 12000
was set buffer_size = 48000
 0 - Front Left
 1 - Front Right
Time per period = 5.007955
 0 - Front Left
 1 - Front Right
```

After connecting my USB audio device I noticed, that the indices of the sound
cards changed:

```text
$ aplay -l
**** List of PLAYBACK Hardware Devices ****
card 0: vc4hdmi [vc4-hdmi], device 0: MAI PCM i2s-hifi-0 [MAI PCM i2s-hifi-0]
  Subdevices: 1/1
  Subdevice #0: subdevice #0
card 1: CODEC [USB AUDIO  CODEC], device 0: USB Audio [USB Audio]
  Subdevices: 1/1
  Subdevice #0: subdevice #0
card 2: sndrpihifiberry [snd_rpi_hifiberry_dacplus], device 0: HiFiBerry DAC+ HiFi pcm512x-hifi-0 [HiFiBerry DAC+ HiFi pcm512x-hifi-0]
  Subdevices: 1/1
  Subdevice #0: subdevice #0
```

So to get around that I switched the configuration to use the name of the card.
For that we need to use a slightly different syntax:

```
pcm.!default {
        type hw
        card sndrpihifiberry
}

ctl.!default {
        type hw
        card sndrpihifiberry
}
```

### Playing audio from the USB audio device

To achieve my goal of playing back the sound from the small USB based sound
mixer I used `alsaloop` to redirect the sound from the USB capture device to
the playback device of the HiFiBerry:

```
alsaloop -C hw:1,0 -P hw:2,0
```

## Setting up snapcast

To stream audio from various sources (including spotify) I decided to use
[snapcast](https://github.com/badaix/snapcast). The nice thing is, that it
allows me to play audio synchronized on both my Raspberry Pis.

Building the [snapcast `PKGBUILD`](https://aur.archlinux.org/packages/snapcast)
directly on the Raspberry Pi failed, since the device doesn't have enough RAM.
So I tried to follow the guides from archlinuxarm to do cross compile builds:

 * <https://archlinuxarm.org/wiki/Distcc_Cross-Compiling>
 * <https://archlinuxarm.org/wiki/Distributed_Compiling>

#### Distcc Master System (Raspberry Pi)

For the distributed build, the Raspberry Pi will be the master system. So we
install `distcc` on it and configure `makepkg` to use it:

```bash
sudo pacman -S distcc
sudo vim /etc/makepkg.conf
# Edit / add the following lines
# DISTCC_HOSTS="$IP_OF_SLAVE:3635"
# MAKEFLAGS="-j4"
```

#### Distcc Slave System (Desktop PC)

On the slave system I first tried to install the cross-compile tools manually
like described by the archlinuxarm page, but this failed with cryptic error
messages from the Raspberry Pi while building the package.

So I followed the guide from the [ArchLinux distcc wiki
page](https://wiki.archlinux.org/title/Distcc#Cross_compiling_with_distcc),
which worked perfectly, using the [distccd-alarm-armv7h AUR
package](https://aur.archlinux.org/packages/distccd-alarm-armv7h):

```
yay distccd-alarm-armv7h
sudo systemctl start distccd-armv7h.service
```

This worked out of the box and I could successfully build the snapcast package
and distribute it to my Raspberry Pis.

### Configuring Snapcast

After installing snapcast it showed the following output :

```text
:: The default setup will create a pipe /tmp/snapfifo.
   Due to recent changes in systemd, pipes in /tmp are by default only
   writable by the owning user (here: sysuser snapserver).
   The safest option is to change the location of the fifo to a
   different location, e.g. /run/snapserver.  This can be done by
   editing /etc/snapserver.conf
   Another possible workaround is to disable the sysctl feature by
   running:
     # sysctl fs.protected_fifos=0
:: Snapcast now has a built-in snapweb control client which is enabled
   by default on a new setup.  This functionality enables a webserver
   on port 1780.  This can be controlled by the doc_root variable in
   /etc/snapserver.conf.
Optional dependencies for snapcast
    python-websocket-client: stream plugin script [installed]
    python-mpd2: stream plugin script
    python-musicbrainzngs: stream plugin script [installed]
```

So the first thing I did is set the default source to `source =
pipe:///run/snapserver/snapfifo?name=default` in `/etc/snapserver.conf`.

I then started the `snapserver` service and started playing white noise trough
the snapfifo:

```
sudo systemctl start snapserver.service
cat /dev/urandom > /run/snapserver/snapfifo
```

I could then point my smart phone to http://muzikskatolo:1780 and start
streaming the random white noise to test that everything is working.


### Adding librespot

snapcast has out of the box support to spawn librespot. The [librespot AUR
package](https://aur.archlinux.org/packages/librespot) needed some patching to
be buildable for the Raspberry Pi. While at it I also removed audio backends
and their dependencies which I didn't need:

```diff
diff --git a/PKGBUILD b/PKGBUILD
index 4a10228..61aeef4 100644
--- a/PKGBUILD
+++ b/PKGBUILD
@@ -8,16 +8,17 @@ pkgver=0.4.2
 _commit=22f8aed
 pkgrel=1
 pkgdesc='Open source client library for Spotify'
-arch=('x86_64' 'aarch64')
+arch=('armv7h' 'x86_64' 'aarch64')
 url='https://github.com/librespot-org/librespot'
 license=('MIT')
 depends=(
        'alsa-lib'
-       'gst-plugins-base-libs'
-       'jack'
        'libpulse'
-       'portaudio'
-       'sdl2')
+)
 makedepends=('cargo' 'git')
 source=("$pkgname::git+$url#commit=$_commit?signed")
 sha256sums=('SKIP')
@@ -25,7 +26,7 @@ validpgpkeys=('EC57B7376EAFF1A0BB56BB0187F5FDE8A56219F4') ## Roderick van Domber

 prepare() {
        cd "$pkgname"
-       cargo fetch --locked --target "$CARCH-unknown-linux-gnu"
+       cargo fetch --locked
 }

 build() {
@@ -33,7 +34,7 @@ build() {
        export CARGO_TARGET_DIR=target
        cd "$pkgname"
        cargo build --release --frozen --features \
-               alsa-backend,portaudio-backend,pulseaudio-backend,jackaudio-backend,rodio-backend,rodiojack-backend,sdl-backend,gstreamer-backend
+               alsa-backend,pulseaudio-backend,rodio-backend
 }

 ## 0 tests
```

Due to some random spotify changes [librespot stopped
working](https://github.com/librespot-org/librespot/issues/972#issuecomment-2320943137)
after a few days and I had to use the [librespot-git AUR
package](https://aur.archlinux.org/packages/librespot-git) which continued to
work and also didn't need any changes to the `PKGBUILD`.

Adding librespot to snapcast was as easy as adding the following source to
`/etc/snapserver.conf` ([^2]):
```
source = librespot:///usr/bin/librespot?name=librespot&devicename=Snapcast
```

I then did setup the `snapclient` on my existing kodi Raspberry Pi and on the
Pi where also the snapserver runs:

```bash
# /etc/default/snapclient on kodi system
SNAPCLIENT_OPTS="--host $IP_OF_SNAPSERVER"
```

```bash
# /etc/default/snapclient on muzikskatolo
SNAPCLIENT_OPTS="--host 127.0.0.1 -s 13"
```

On muzikskatolo snapclient didn't choose the correct output device so I looked
at the output of `snapclient -l` and hard-coded it in the options:
```text
# lots of other devices
13: sysdefault:CARD=sndrpihifiberry
snd_rpi_hifiberry_dacplus, HiFiBerry DAC+ HiFi pcm512x-hifi-0
Default Audio Device
```

Another thing I had to change was the `bind_to_address` of the `snapserver`
([^3]) so that it also listens to incoming IPv6 traffic:

```
[http]
bind_to_address = ::
[tcp]
bind_to_address = ::
[stream]
bind_to_address = ::
```

I now enjoy nice multi-room synced audio while being able to easily control
volume from my smart phone:

<center>
![Snapcast web interface]({static}/images/snapcast/snapcast-webinterface.png){width=80%}
</center>


## Outlook

The next things I want to look at is setting up [mopidy](https://mopidy.com/)
to stream music from my NAS and also to integrate snapcast into [my home
assistant
setup](https://blog.rnstlr.ch/setting-up-home-assistant-on-the-raspberry-pi-3.html).

[^1]: See also <https://wiki.archlinux.org/title/Advanced_Linux_Sound_Architecture#Setting_the_default_sound_card_via_defaults_node>
[^2]: See also <https://github.com/badaix/snapcast/blob/develop/doc/configuration.md#librespot>
[^3]: See also <https://github.com/badaix/snapcast/issues/518#issuecomment-569143476>
