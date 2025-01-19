Title: Improving my Raspberry Pi Based Media System
Tags: Raspberry Pi, Linux, ArchLinux, ARM, snapcast, volumio, mopidy
Language: en
Summary: Setting up a media system on a Raspberry Pi 3 based on ArchLinux ARM, snapcast and mopidy
Status: draft


In [a past
blogpost](https://blog.rnstlr.ch/creating-a-raspberry-pi-based-media-system.html)
I described how I did set up a media system based on
[snapcast](https://github.com/badaix/snapcast) and
[librespot](https://github.com/librespot-org/librespot)

What I didn't get to yet was configuring mopidy to play music from my NAS.

## mopidy

Installing mopidy is as easy as `sudo pacman -S mopidy`. Configuring snapcast
and mopidy to play music works as follows:

 1. Editing `/etc/snapserver.conf`
    ```toml
    source = pipe:///run/snapserver/snapfifo?name=mopidy
    ```
 2. Editing `/etc/mopidy/mopidy.conf` to configure mopidy's audio output
    ```toml
    [audio]
    output = audioresample ! audioconvert ! audio/x-raw,rate=48000,channels=2,format=S16LE ! filesink location=/run/snapserver/snapfifo
    ```

To be able to play local files I wanted to use the
[Mopidy-Local](https://mopidy.com/ext/local/) extension. For that I needed to
install the following packages from the AUR:

 * [python-uritools](https://aur.archlinux.org/packages/python-uritools) (dependency of mopidy-local)
 * [mopidy-local](https://aur.archlinux.org/packages/mopidy-local)

To get access to the music collection of the NAS I mounted the path system wide
via `/etc/fstab`.

To not store the credentials in plaintext in `/etc/fstab` directly I create a
credentials file in `/var/lib/private`:

```ini
# /var/lib/private/nas-cifs-credentials
username=your_username
password=your_password
domain=your_domain  # Optional, if applicable
```

To give mopidy access to the files I looked for its user and group ID

```passwd
$ cat /etc/passwd|grep mopidy
mopidy:x:46:46:Mopidy User:/var/lib/mopidy:/usr/bin/nologin
```

and put it together like this: 

```fstab
#/etc/fstab
//$IP_OF_NAS/$PATH_TO_MUSIC  /mnt/music  cifs  credentials=/var/lib/private/nas-cifs-credentials,uid=46,gid=46  0  0
```

Then we need to tell the mopidy extension where our music is:

```toml
#/etc/mopidy/mopidy.conf
[local]
media_dir = /mnt/music
```

And let it scan the directory

```
$ sudo mopidyctl local scan
```

To control mopidy I chose to use the [mpoidy-mpd](https://mopidy.com/ext/mpd/)
extension, also available from the
[AUR](https://aur.archlinux.org/packages/mopidy-mpd)

To make it usable we need to make it listen on all interfaces, both IPv4 and
IPv6:
```
[mpd]
hostname = ::
```

TODO: mpd problems:
 * https://github.com/mopidy/mopidy-mpd/issues/68 and https://github.com/mopidy/mopidy-mpd/issues/47
  ```
  $ mpc
  warning: MPD 0.21 required
  mpd version: 0.19.0
  ```

mopidy-iris

## Notes

 * Adding the snapcast dashboard to home assistant
 * Adding the snapcast integration

## New stuff

 * https://github.com/music-assistant/server
