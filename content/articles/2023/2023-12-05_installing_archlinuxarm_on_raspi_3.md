Title: Installing ArchLinux ARM on the Raspberry Pi 3
Tags: Raspberry Pi, Linux, ArchLinux, ARM
Language: en
Summary: Replacing my RetroPie installation with ArchLinux ARM on my Raspberry Pi 3

Today I plugged in my Raspberry Pi 3 which had
[RetroPie](https://retropie.org.uk/) installed on it. It notified me, that the
installation is out of date and not longer supported and I should reinstall the
system.

Since I had to do a reinstall anyway and didn't want to be bothered again in
the future, I decided to try out ArchLinux ARM, so I just get the rolling
releases without a need to do re-installs.

# Installation

To install the base image I followed mostly the guide from ArchLinux ARM:
<https://archlinuxarm.org/platforms/armv8/broadcom/raspberry-pi-3>

For the setup I needed to connect it with an Ethernet cable, since the guide
doesn't explain how to set up WiFi prior to first booting it.

After that I logged in via ssh. I needed to enforce password authentication,
since otherwise ssh would just try all my ssh keys to authenticate which lead
to a `Received disconnect from 192.168.0.31 port 22:2: Too many authentication
failures`

```bash
ssh -o PreferredAuthentications=password alarm@alarmpi
# switch to root user
su
# From the install guide
pacman-key --init
pacman-key --populate archlinuxarm
# Update the whole system
pacman -Syu
```

During the update step I got a scary warning while building the initramfs:
```text
(12/14) Updating linux-rpi initcpios...
==> Building image from preset: /etc/mkinitcpio.d/linux-rpi.preset: 'default'
==> Using configuration file: '/etc/mkinitcpio.conf'
  -> -k 6.1.64-2-rpi-ARCH -c /etc/mkinitcpio.conf -g /boot/initramfs-linux.img
==> Starting build: '6.1.64-2-rpi-ARCH'
  -> Running build hook: [base]
  -> Running build hook: [udev]
  -> Running build hook: [autodetect]
  -> Running build hook: [modconf]
  -> Running build hook: [kms]
  -> Running build hook: [keyboard]
  -> Running build hook: [keymap]
  -> Running build hook: [consolefont]
==> WARNING: consolefont: no font found in configuration
  -> Running build hook: [block]
  -> Running build hook: [filesystems]
  -> Running build hook: [fsck]
==> Generating module dependencies
==> Creating gzip-compressed initcpio image: '/boot/initramfs-linux.img'
==> WARNING: errors were encountered during the build. The image may not be complete.
error: command failed to execute correctly
```

I tried to figure out what went wrong and then decided to just reboot and see
if we can still do that, which worked without problems.

## Configuring WiFi

Since I didn't want to bother with connecting the raspi with an ethernet cable
at the location were I wanted to run it, I needed to configure WiFi before
continuing.

Since it will run stationary, it is enough to just hard-code a single network:

```bash
wpa_passphrase SSID PSK > /etc/wpa_supplicant/wpa_supplicant@wlan0.service
systemctl start wpa_supplicant@wlan0.service
systemctl enable wpa_supplicant@wlan0.service
```

Then we tell systemd-network to enable DHCP for the wlan0 interface by creating
`/etc/systemd/network/wlan0.network` with the following content:

```
[Match]
Name=wlan0

[Network]
DHCP=yes
IgnoreCarrierLoss=3s
```

After reloading with `systemctl reload systemd-networkd.service` WiFi was
connected.

### Update 2023-12-14

After a while I noticed, that the `systemd-networkd-wait-online.service` blocks
the startup of the pi for about 2 minutes. This is due to the fact that it will
wait for all links to be fully configured by default. (See
<https://www.freedesktop.org/software/systemd/man/latest/systemd-networkd-wait-online.service.html>)

This can be changed by only waiting for `wlan0` to be up:

```
sudo systemctl disable systemd-networkd-wait-online.service
sudo systemctl enable systemd-networkd-wait-online@wlan0.service
```

# Installing Kodi

Since I wanted to run Kodi on the Pi (along side RetroPi / Emulation station)
I followed this guide: <https://kodi.wiki/view/HOW-TO:Install_Kodi_on_Raspberry_Pi>

```
pacman -S kodi
# selecting kodi-rpi
```

After installing the output of pacman showed the following:
```text
>>> Remove any manual tweaks made to /boot/config.txt particulary a line such
    as gpu_mem=xxx.  Driver setup for Kodi is stored in /boot/kodi.config.txt

    Manually append the following to /boot/config.txt to make them active:
    [all]
    include kodi.config.txt

    A reboot will be required to activate them if this is a fresh install.`
```

<details>
<summary>Optional kodi dependencies</summary>
Kodi showed the following optional dependencies: Since I currently don't want
to use bluetooth for anything, nor want to use pipewire or pulseaudio but plain
ALSA, I didn't install any of them.
```text
Optional dependencies for kodi-rpi
    afpfs-ng: Apple shares support
    bluez: Blutooth support
    linux-rpi: HW accelerated decoding [installed]
    python-pybluez: Bluetooth support
    pulseaudio: PulseAudio support
    pipewire: PipeWire support
```
</details>

So I edited it accordingly, enabled the kodi service and rebooted:

```
sudo systemctl enable kodi.service
```

So far everything is working quite nicely and one of the next things will be to
get a spotify plugin to run on the Raspberry Pi.
