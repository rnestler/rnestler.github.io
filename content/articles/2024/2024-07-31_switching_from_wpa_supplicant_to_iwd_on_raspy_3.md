Title: Switching from wpa_supplicant to iwd on my Raspberry Pi
Tags: Raspberry Pi, Linux, ArchLinux, ARM, Kodi
Language: en
State: draft
Summary: Switching from the broken `wpa_supplicant` to `iwd` for my Kodi installation on a Raspberry Pi 3

After upgrading my archlinuxarm based Kodi media center as usual with just
running `pacman -Syu` over ssh I didn't suspect anything. But on the next
reboot the system blocked with waiting for wlan0 to get online.

At first I suspected the kernel or linux-firmware upgrade, since a quick search
found that this caused trouble in the past: [firmware update breaks wifi on
Rpi3](https://archlinuxarm.org/forum/viewtopic.php?t=15833)

# Downgrading `linux-firmware`

Since remote connection was broken I sat with the keyboard in front of the TV
screen and tried to downgrade the `linux-firmware` package from the pkg cache:

```
sudo pacman -U /var/cache/pacman/pkg/linux-firmware-20240610.9c10a208-1-any.pkg.tar.xz
```

My hope for this easy fix were crushed, when the network still didn't start
correctly after the next reboot. So I started to look into system logs with
`sudo systemctl status --all` and realized that `wpa_supplicant@wlan0.service`
couldn't start.

# Investigating Further

Searching again I found <https://bbs.archlinux.org/viewtopic.php?id=298025>
from 2024-07-25 indicating that WiFi on Broadcom devices seems broken after the
upgrade to `wpa_supplicant` 2.11.

Looking again in `/var/cache/pacman/pkg/` I realized that the old
`wpa_supplicant` package was missing. Probably since it 2.10 was the current
version when I installed the whole system.

Moving forward I decided to switch to `iwd`, which seems better maintained and
I also use it on my Laptop.

# Switching to `iwd`

So I headed to <https://archlinuxarm.org/packages/armv7h/iwd>, downloaded the
package and its dependency `ell` manually, copied it together with the WiFi
configuration from my laptop from `/var/lib/iwd/` to a USB flash drive and
installed them:
```
sudo mount /dev/sda1 /mnt
sudo pacman -U /mnt/iwd-2.19-1-armv7h.pkg.tar.xz /mnt/ell-0.67-1-armv7h.pkg.tar.xz
```

Then I replaced `wpa_supplicant` with `iwd`

```
sudo systemctl stop wpa_supplicant@wlan0.service
sudo systemctl disable wpa_supplicant@wlan0.service
sudo pacman -Rs wpa_supplicant
sudo systemctl enable iwd.service
sudo systemctl start iwd.service
```

And 🎉 working WiFi again on my Kodi system!
