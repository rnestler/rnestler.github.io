Title: Installing ArchLinux ARM on the Raspberry Pi 3
Tags: rust, kernel, Linux, ArchLinux
Language: en
Status: draft
Summary: Replacing my RetroPie installation with ArchLinux ARM on my Raspberry Pi 3

Today I plugged in my Raspberry Pi 3 which had
[RetroPie](https://retropie.org.uk/) installed on it. It notified me, that the
installation is out of date and not longer supported and I should reinstall the
system.

Since I had to do a reinstall anyway and didn't want to be bothered again in
the future I decided to try out ArchLinux ARM, so I just get the rolling
releases without a need to do re-installs.

# Installation

To install the base image I followed mostly the guide from ArchLinux ARM:
<https://archlinuxarm.org/platforms/armv8/broadcom/raspberry-pi-3>

After that I logged in via ssh. I needed to enforce password authentication,
since otherwise ssh would just try all my ssh keys to authenticate which lead
to a `Received disconnect from 192.168.0.31 port 22:2: Too many authentication
failures`

```
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
```
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
if we can still do that.

## Configuring WiFi

```
wpa_passphrase SSID PSK > /etc/wpa_supplicant/wpa_supplicant@wlan0.service
systemctl start wpa_supplicant@wlan0.service
systemctl enable wpa_supplicant@wlan0.service
```

# Installing Kodi

Since I wanted to run Kodi on the Pi (along side RetroPi / Emulation station) 
I followed this guide: <https://kodi.wiki/view/HOW-TO:Install_Kodi_on_Raspberry_Pi>

```
pacman -S kodi
# selecting kodi-rpi
```

After
```
>>> Remove any manual tweaks made to /boot/config.txt particulary a line such
    as gpu_mem=xxx.  Driver setup for Kodi is stored in /boot/kodi.config.txt

    Manually append the following to /boot/config.txt to make them active:
    [all]
    include kodi.config.txt

    A reboot will be required to activate them if this is a fresh install.`
```

```
sudo systemctl enable kodi.service
```


```
Optional dependencies for kodi-rpi
    afpfs-ng: Apple shares support
    bluez: Blutooth support
    linux-rpi: HW accelerated decoding [installed]
    python-pybluez: Bluetooth support
    pulseaudio: PulseAudio support
    pipewire: PipeWire support
```

