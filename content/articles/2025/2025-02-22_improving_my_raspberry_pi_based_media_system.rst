Improving my Raspberry Pi Based Media System
============================================

:tags: Raspberry Pi, Linux, ArchLinux, ARM, snapcast, volumio, mopidy
:language: en
:summary: Setting up a media system on a Raspberry Pi 3 based on ArchLinux ARM, snapcast and mopidy
:status: draft

In `a past blogpost
<https://blog.rnstlr.ch/creating-a-raspberry-pi-based-media-system.html>`_ I
described how I did set up a media system based on `snapcast
<https://github.com/badaix/snapcast>`_ and `librespot
<https://github.com/librespot-org/librespot>`_.

What I didn't get to yet was configuring mopidy to play music from my NAS.

mopidy
------

Installing mopidy is as easy as ``sudo pacman -S mopidy``. Configuring snapcast
and mopidy to play music works as follows:

1. Editing ``/etc/snapserver.conf``

   .. code-block:: ini

      source = pipe:///run/snapserver/snapfifo?name=mopidy

2. Editing ``/etc/mopidy/mopidy.conf`` to configure mopidy's audio output

   .. code-block:: ini

      [audio]
      output = audioresample ! audioconvert ! audio/x-raw,rate=48000,channels=2,format=S16LE ! filesink location=/run/snapserver/snapfifo

To be able to play local files I wanted to use the `Mopidy-Local
<https://mopidy.com/ext/local/>`_ extension. For that I needed to install the
following packages:

- `python-uritools (AUR) <https://aur.archlinux.org/packages/python-uritools>`_ (dependency of mopidy-local)
- `mopidy-local (AUR) <https://aur.archlinux.org/packages/mopidy-local>`_

To get access to the music collection of the NAS I mounted the path system wide
via ``/etc/fstab``.

To not store the credentials in plaintext in ``/etc/fstab`` directly I create a
credentials file in ``/var/lib/private``:

.. code-block:: ini

   # /var/lib/private/nas-cifs-credentials
   username=your_username
   password=your_password
   domain=your_domain  # Optional, if applicable

To give mopidy access to the files I looked for its user and group ID:

.. code-block:: text

   $ cat /etc/passwd|grep mopidy
   mopidy:x:46:46:Mopidy User:/var/lib/mopidy:/usr/bin/nologin

and put it together like this:

.. code-block:: text

   #/etc/fstab
   //$IP_OF_NAS/$PATH_TO_MUSIC  /mnt/music  cifs  credentials=/var/lib/private/nas-cifs-credentials,uid=46,gid=46  0  0

Then we need to tell the mopidy extension where our music is:

.. code-block:: ini

   #/etc/mopidy/mopidy.conf
   [local]
   media_dir = /mnt/music

And let it scan the directory:

.. code-block:: text

   $ sudo mopidyctl local scan

mopidy-mpd
^^^^^^^^^^

To control mopidy I chose to use the `mpoidy-mpd
<https://mopidy.com/ext/mpd/>`_ extension, also available from the `AUR
<https://aur.archlinux.org/packages/mopidy-mpd>`_.

To make it usable we need to make it listen on all interfaces, both IPv4 and IPv6:

.. code-block:: ini

   [mpd]
   hostname = ::

Trying to use it quickly revealed that the plugin is quite outdated and won't
work with newer ``mpd`` clients:

.. code-block:: text

   $ mpc
   warning: MPD 0.21 required
   mpd version: 0.19.0

Looking at GitHub I found `mopidy-mpd issue 68
<https://github.com/mopidy/mopidy-mpd/issues/68>`_ and `mopidy-mpd issue 47
<https://github.com/mopidy/mopidy-mpd/issues/47>`_ which confirmed that.

mopidy-iris
^^^^^^^^^^^

`mopidy-iris <https://github.com/jaedb/Iris/>`_ looked like a popular and well-maintained web interface for mopidy.

mopidy snapcast integration
^^^^^^^^^^^^^^^^^^^^^^^^^^^

- Not marked executable -> ``chmod +x ...``
- Deps not installed:

  .. code-block:: text

     /usr/share/snapserver/plug-ins/meta_mopidy.py
     Traceback (most recent call last):
       File "/usr/share/snapserver/plug-ins/meta_mopidy.py", line 25, in <module>
         import websocket
     ModuleNotFoundError: No module named 'websocket'

  -> ``sudo pacman -S --asdeps python-websockets``
  
  -> ``sudo pacman -S --asdeps extra/python-websocket-client``

Notes
-----

- Adding the snapcast dashboard to home assistant
- Adding the snapcast integration

New stuff
---------

- `music-assistant/server <https://github.com/music-assistant/server>`_

