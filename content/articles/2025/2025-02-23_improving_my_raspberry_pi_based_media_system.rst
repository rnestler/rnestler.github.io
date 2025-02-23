Adding mopidy to my Raspberry Pi Based Media System
===================================================

:tags: Raspberry Pi, Linux, ArchLinux, ARM, snapcast, mopidy
:language: en
:summary: Setting up a media system on a Raspberry Pi 3 based on ArchLinux ARM, snapcast and mopidy

In `a past blogpost
<https://blog.rnstlr.ch/creating-a-raspberry-pi-based-media-system.html>`_ I
described how I did set up a media system based on `snapcast
<https://github.com/badaix/snapcast>`_ and `librespot
<https://github.com/librespot-org/librespot>`_.

What I didn't get to yet was configuring `mopidy <https://mopidy.com/>`_ to
play music from my NAS.

installing mopidy
-----------------

Installing mopidy is as easy as ``sudo pacman -S mopidy``. Configuring snapcast
and mopidy to play music works as follows:

1. Editing ``/etc/snapserver.conf``

   .. code-block:: ini

      source = pipe:///run/snapserver/snapfifo?name=mopidy

2. Editing ``/etc/mopidy/mopidy.conf`` to configure mopidy's audio output

   .. code-block:: ini

      [audio]
      output = audioresample ! audioconvert ! audio/x-raw,rate=48000,channels=2,format=S16LE ! filesink location=/run/snapserver/snapfifo

To be able to play local files I wanted to use the `Mopidy-Local extension
<https://mopidy.com/ext/local/>`_. For that I needed to install the
following packages from the AUR:

- `python-uritools <https://aur.archlinux.org/packages/python-uritools>`_ (dependency of mopidy-local)
- `mopidy-local <https://aur.archlinux.org/packages/mopidy-local>`_

To get access to the music collection of the NAS I mounted the path system wide
via ``/etc/fstab``.

To not store the credentials in plaintext in ``/etc/fstab`` directly I created
a credentials file in ``/var/lib/private``:

.. code-block:: ini

   # /var/lib/private/nas-cifs-credentials
   username=your_username
   password=your_password
   domain=your_domain  # Optional, if applicable

To give mopidy access to the files, we need to know its user and group ID:

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
   scan_timeout = 5000

And let it scan the directory:

.. code-block:: text

   $ sudo mopidyctl local scan

The rather long ``scan_timeout`` of 5s is needed since my NAS goes into power
save mode when idle and needs some seconds for the initial response.

mopidy-mpd
----------

To control mopidy I chose to use the `mpoidy-mpd
<https://mopidy.com/ext/mpd/>`_ extension, also available from the `AUR
<https://aur.archlinux.org/packages/mopidy-mpd>`_.

By default mopidy-mpd is only available from localhost. To make it usable from
the network we need to configure it to listen on all interfaces for both IPv4
and IPv6:

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

Because of that I anticipated a lot of interoperability issues with newer
clients and decided not to use it and uninstalled it again.

mopidy-iris
-----------

Instead of using an mpd client there is the option to use web interfaces to
controll mopidy. `mopidy-iris <https://github.com/jaedb/Iris/>`_ looked like a
popular and well-maintained web interface for mopidy.

After installing and restarting mopidy we get greeted by it's webinterface when
accessing http://$MOPIDY_HOST/iris/  🎉

.. figure:: {static}/images/mopidy/mopidy-iris-welcome.png
    :target: {static}/images/mopidy/mopidy-iris-welcome.png
    :alt: mopidy-iris welcome screen
    :align: center
    :width: 60%
    :figwidth: 100%

    mopidy-iris welcome screen

The interface is straightforward to use and allows to search and add music to
the queue which is everything I want from it.

.. figure:: {static}/images/mopidy/mopidy-playback.png
    :target: {static}/images/mopidy/mopidy-playback.png
    :alt: mopidy-iris playback
    :align: center
    :width: 60%
    :figwidth: 100%

    mopidy-iris playing an album

mopidy snapcast integration
---------------------------

Snapcast supports reporting and controlling the player state of sources via
controllscripts on the `source configuration
<https://github.com/badaix/snapcast/blob/develop/doc/configuration.md#sources>`_

.. code-block:: ini

   # /etc/snapserver.conf
   source = pipe:///run/snapserver/snapfifo?name=mopidy&controlscript=meta_mopidy.py&controlscriptparams=--mopidy-host=muzikskatolo.home

I ran into a few issues:

- The controlscript in ``/usr/share/snapserver/plug-ins/meta_mopidy.py``
  wasn't marked as executable. This was fixed by the AUR package maintainer
  after `I reported it
  <https://aur.archlinux.org/packages/snapcast#comment-1008816>`_

- I didn't install the optional ``python-websocket-client`` dependency for
  ``snapserver`` which lead to a crash:

  .. code-block:: python

     /usr/share/snapserver/plug-ins/meta_mopidy.py
     Traceback (most recent call last):
       File "/usr/share/snapserver/plug-ins/meta_mopidy.py", line 25, in <module>
         import websocket
     ModuleNotFoundError: No module named 'websocket'

  Installing it with ``sudo pacman -S --asdeps python-websocket-client`` fixed it.

- I needed to explicitly set the ``mopidy-host`` on the command line, since
  otherwise the generated links to the album art won't work since they would
  point to `localhost`.


Accessing the snapcast webinterface then allows us to see the metadata of the
playing song:

.. figure:: {static}/images/mopidy/mopidy-snapcast-integration.png
    :target: {static}/images/mopidy/mopidy-snapcast-integration.png
    :alt: mopidy metadata of the playing song displayed in the snapcast web interface
    :align: center
    :width: 60%
    :figwidth: 100%

snapcast meta sources
---------------------

Snapcast has a nice feature called `meta sources
<https://github.com/badaix/snapcast/blob/develop/doc/configuration.md#meta>`_.

It allows to just play the audio from the first active source:

.. code-block:: ini

   source = pipe:///run/snapserver/snapfifo?name=mopidy&controlscript=meta_mopidy.py&controlscriptparams=--mopidy-host=muzikskatolo.home
   source = librespot:///usr/bin/librespot>?name=librespot&devicename=Snapcast
   source = meta:///librespot/mopidy?name=any

Here I configured a meta source named "any" which plays audio from librespot or
mopidy.

scanning the local library regularly
------------------------------------

Running ``sudo mopidyctl local scan`` manually get's tedious over time when I
add new music. So I created a systemd unit and a timer to run it daily:


.. code-block:: ini

   # /etc/systemd/system/mopidy-local-scan.service
   [Unit]
   Description=Mopidy music server
   After=remote-fs.target

   [Service]
   Type=oneshot
   User=mopidy
   ExecStart=/usr/bin/mopidy --config /usr/share/mopidy/conf.d:/etc/mopidy/mopidy.conf local scan

.. code-block:: ini

   # /etc/systemd/system/daily@.timer
   [Unit]
   Description=Daily timer for %i service

   [Timer]
   OnCalendar=*-*-* 02:00:00
   AccuracySec=6h
   RandomizedDelaySec=1h
   Unit=%i.service
   Persistent=true
   [Install]
   WantedBy=timers.target


After creating the files we can reload ``systemd`` and enable the timer:

.. code-block:: text

   $ sudo systemctl daemon-reload
   $ sudo systemctl enable daily@mopidy-local-scan.timer

   Created symlink '/etc/systemd/system/timers.target.wants/daily@mopidy-local-scan.timer' -> '/etc/systemd/system/daily@.timer'.
   $ sudo systemctl list-timers  --all
   NEXT                            LEFT LAST                              PASSED UNIT                             ACTIVATES
   Sun 2025-02-23 00:00:00 UTC 3h 30min Sat 2025-02-22 09:42:48 UTC      10h ago shadow.timer                     shadow.service
   Sun 2025-02-23 02:56:10 UTC       6h Sat 2025-02-22 18:38:44 UTC 1h 50min ago daily@mopidy-local-scan.timer    mopidy-local-scan.service
   Sun 2025-02-23 09:57:33 UTC      13h Sat 2025-02-22 09:57:33 UTC      10h ago systemd-tmpfiles-clean.timer     systemd-tmpfiles-clean.service
   Fri 2025-02-28 16:49:57 UTC   5 days Mon 2025-02-10 09:45:18 UTC            - archlinux-keyring-wkd-sync.timer archlinux-keyring-wkd-sync.service

   4 timers listed.

With mopidy, librespot and snapcast I have a nice solution to listen to the
music I bought from `Bandcamp <https://bandcamp.com/>`_ and stream from Spotify
in my home.

