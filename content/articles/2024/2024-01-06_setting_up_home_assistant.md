Title: Setting up Home Assistant on the Raspberry Pi 3
Tags: Raspberry Pi, Linux, Home Assistant
Language: en
Status: draft
Summary: Setting up Home Assistant to automate stuff at home

# Motivation

From my time working at Sensirion I still have a lot of sensors like the [SCD4x
CO2 Gadget](https://www.sensirion.com/products/catalog/SCD4x-CO2-Gadget) and
[SHT4x Smart
Gadget](https://sensirion.com/de/produkte/katalog/SHT4x-Smart-Gadget) running.
Also I decided that I want to use smart plugs to power on my coffee machine in
the morning. 

While one can see the sensor data perfectly fine with the [Sensirion MyAmbience
App](https://play.google.com/store/apps/details?id=com.sensirion.myam) and also
use the specific apps for the smart plugs, I wanted a bit more integration for
everything and thus decided to try out [Home
Assistant](https://www.home-assistant.io).

# Installing Home Assistant

Installing Home Assistant on a Raspberry Pi is very easy: Just follow the guide
at <https://www.home-assistant.io/installation/raspberrypi>. Since I didn't
want to use the GUI tools to flash the SD card I used the command line
directly:

```bash
$ wget https://github.com/home-assistant/operating-system/releases/download/11.3/haos_rpi3-64-11.3.img.xz
$ xz --decompress --stdout haos_rpi3-64-11.3.img.xz| sudo dd of=/dev/sda bs=64k oflag=dsync status=progress
```

After that I inserted the SD card into the Raspberry Pi and plugged in power.

# Setting it up

I mostly followed the guide at
<https://www.home-assistant.io/getting-started/onboarding/>


When connecting to <http://homeassistant.local:8123/> I was greeted by the
setup screen:

![Home Assistant Setup Screen]({static}/images/home_assistant/preparing-home-assistant.png)

This setup screen seemed stuck at the following step in the logs:
```
[supervisor.host.manager] Host information reload completed
```

<details>
<summary>Full logs</summary>
```text
s6-rc: info: service s6rc-oneshot-runner: starting
s6-rc: info: service s6rc-oneshot-runner successfully started
s6-rc: info: service fix-attrs: starting
s6-rc: info: service fix-attrs successfully started
s6-rc: info: service legacy-cont-init: starting
cont-init: info: running /etc/cont-init.d/udev.sh
INFO: Using udev information from host
cont-init: info: /etc/cont-init.d/udev.sh exited 0
s6-rc: info: service legacy-cont-init successfully started
s6-rc: info: service legacy-services: starting
services-up: info: copying legacy longrun supervisor (no readiness notification)
services-up: info: copying legacy longrun watchdog (no readiness notification)
s6-rc: info: service legacy-services successfully started
INFO: Starting local supervisor watchdog...
[__main__] Initializing Supervisor setup
[supervisor.docker.network] Can't find Supervisor network, creating a new network
[supervisor.bootstrap] Seting up coresys for machine: raspberrypi3-64
[supervisor.docker.supervisor] Attaching to Supervisor ghcr.io/home-assistant/aarch64-hassio-supervisor with version 2023.12.0
[supervisor.docker.supervisor] Connecting Supervisor to hassio-network
[supervisor.resolution.evaluate] Starting system evaluation with state initialize
[supervisor.resolution.evaluate] System evaluation complete
[__main__] Setting up Supervisor
[supervisor.api] Starting API on 172.30.32.2
[supervisor.hardware.monitor] Started Supervisor hardware monitor
[supervisor.dbus.manager] Connected to system D-Bus.
[supervisor.dbus.agent] Load dbus interface io.hass.os
[supervisor.dbus.hostname] Load dbus interface org.freedesktop.hostname1
[supervisor.dbus.logind] Load dbus interface org.freedesktop.login1
[supervisor.dbus.network] Load dbus interface org.freedesktop.NetworkManager
[supervisor.dbus.rauc] Load dbus interface de.pengutronix.rauc
[supervisor.dbus.resolved] Load dbus interface org.freedesktop.resolve1
[supervisor.dbus.systemd] Load dbus interface org.freedesktop.systemd1
[supervisor.dbus.timedate] Load dbus interface org.freedesktop.timedate1
[supervisor.host.services] Updating service information
[supervisor.host.sound] Updating PulseAudio information
[supervisor.host.sound] Can't update PulseAudio data: Failed to connect to pulseaudio server
[supervisor.host.network] Updating local network information
[supervisor.host.apparmor] Loading AppArmor Profiles: {'hassio-supervisor'}
[supervisor.docker.monitor] Started docker events monitor
[supervisor.updater] Fetching update data from https://version.home-assistant.io/stable.json
[supervisor.docker.interface] Found ghcr.io/home-assistant/aarch64-hassio-cli versions: []
[supervisor.docker.interface] Attaching to ghcr.io/home-assistant/aarch64-hassio-cli with version 2023.11.0
[supervisor.plugins.cli] Starting CLI plugin
[supervisor.docker.cli] Starting CLI ghcr.io/home-assistant/aarch64-hassio-cli with version 2023.11.0 - 172.30.32.5
[supervisor.docker.interface] Found ghcr.io/home-assistant/aarch64-hassio-dns versions: []
[supervisor.docker.interface] Attaching to ghcr.io/home-assistant/aarch64-hassio-dns with version 2023.06.2
[supervisor.plugins.dns] Starting CoreDNS plugin
[supervisor.docker.dns] Starting DNS ghcr.io/home-assistant/aarch64-hassio-dns with version 2023.06.2 - 172.30.32.3
[supervisor.plugins.dns] Updated /etc/resolv.conf
[supervisor.docker.interface] Found ghcr.io/home-assistant/aarch64-hassio-audio versions: []
[supervisor.docker.interface] Attaching to ghcr.io/home-assistant/aarch64-hassio-audio with version 2023.12.0
[supervisor.plugins.audio] Starting Audio plugin
[supervisor.docker.audio] Starting Audio ghcr.io/home-assistant/aarch64-hassio-audio with version 2023.12.0 - 172.30.32.4
[supervisor.docker.interface] Found ghcr.io/home-assistant/aarch64-hassio-observer versions: []
[supervisor.docker.interface] Attaching to ghcr.io/home-assistant/aarch64-hassio-observer with version 2023.06.0
[supervisor.plugins.observer] Starting observer plugin
[supervisor.docker.observer] Starting Observer ghcr.io/home-assistant/aarch64-hassio-observer with version 2023.06.0 - 172.30.32.6
[supervisor.docker.interface] Found ghcr.io/home-assistant/aarch64-hassio-multicast versions: []
[supervisor.docker.interface] Attaching to ghcr.io/home-assistant/aarch64-hassio-multicast with version 2023.06.2
[supervisor.plugins.multicast] Starting Multicast plugin
[supervisor.docker.multicast] Starting Multicast ghcr.io/home-assistant/aarch64-hassio-multicast with version 2023.06.2 - Host
[supervisor.homeassistant.secrets] Loaded 0 Home Assistant secrets
[supervisor.docker.interface] No version found for ghcr.io/home-assistant/raspberrypi3-64-homeassistant
[supervisor.homeassistant.core] No Home Assistant Docker image ghcr.io/home-assistant/raspberrypi3-64-homeassistant found.
[supervisor.docker.interface] Attaching to ghcr.io/home-assistant/raspberrypi3-64-homeassistant with version landingpage
[supervisor.homeassistant.core] Using preinstalled landingpage
[supervisor.homeassistant.core] Starting HomeAssistant landingpage
[supervisor.homeassistant.module] Update pulse/client.config: /data/tmp/homeassistant_pulse
[supervisor.docker.homeassistant] Starting Home Assistant ghcr.io/home-assistant/raspberrypi3-64-homeassistant with version landingpage
[supervisor.os.manager] Detect Home Assistant Operating System 11.3 / BootSlot A
[supervisor.store.git] Cloning add-on https://github.com/esphome/home-assistant-addon repository
[supervisor.store.git] Cloning add-on https://github.com/hassio-addons/repository repository
[supervisor.store.git] Cloning add-on https://github.com/home-assistant/addons repository
[supervisor.store] Loading add-ons from store: 72 all - 72 new - 0 remove
[supervisor.addons.manager] Found 0 installed add-ons
[supervisor.backups.manager] Found 0 backup files
[supervisor.discovery] Loaded 0 messages
[supervisor.ingress] Loaded 0 ingress sessions
[supervisor.resolution.check] Starting system checks with state setup
[supervisor.resolution.check] System checks complete
[supervisor.resolution.evaluate] Starting system evaluation with state setup
[supervisor.resolution.evaluate] System evaluation complete
[supervisor.jobs] 'ResolutionFixup.run_autofix' blocked from execution, system is not running - setup
[supervisor.resolution.evaluate] Starting system evaluation with state setup
[supervisor.resolution.evaluate] System evaluation complete
[__main__] Running Supervisor
[supervisor.os.manager] Rauc: A - marked slot kernel.0 as good
[supervisor.addons.manager] Phase 'initialize' starting 0 add-ons
[supervisor.addons.manager] Phase 'system' starting 0 add-ons
[supervisor.addons.manager] Phase 'services' starting 0 add-ons
[supervisor.core] Skipping start of Home Assistant
[supervisor.addons.manager] Phase 'application' starting 0 add-ons
[supervisor.misc.tasks] All core tasks are scheduled
[supervisor.core] Supervisor is up and running
[supervisor.homeassistant.core] Home Assistant setup
[supervisor.docker.interface] Updating image ghcr.io/home-assistant/raspberrypi3-64-homeassistant:landingpage to ghcr.io/home-assistant/raspberrypi3-64-homeassistant:2024.1.1
[supervisor.docker.interface] Downloading docker image ghcr.io/home-assistant/raspberrypi3-64-homeassistant with tag 2024.1.1.
[supervisor.host.info] Updating local host information
[supervisor.resolution.check] Starting system checks with state running
[supervisor.resolution.checks.base] Run check for dns_server_failed/dns_server
[supervisor.resolution.checks.base] Run check for pwned/addon
[supervisor.resolution.checks.base] Run check for ipv4_connection_problem/system
[supervisor.resolution.checks.base] Run check for dns_server_ipv6_error/dns_server
[supervisor.resolution.checks.base] Run check for security/core
[supervisor.resolution.checks.base] Run check for docker_config/system
[supervisor.resolution.checks.base] Run check for trust/supervisor
[supervisor.resolution.checks.base] Run check for no_current_backup/system
[supervisor.resolution.module] Create new suggestion create_full_backup - system / None
[supervisor.resolution.module] Create new issue no_current_backup - system / None
[supervisor.resolution.checks.base] Run check for free_space/system
[supervisor.resolution.checks.base] Run check for multiple_data_disks/system
[supervisor.resolution.check] System checks complete
[supervisor.resolution.evaluate] Starting system evaluation with state running
[supervisor.resolution.evaluate] System evaluation complete
[supervisor.resolution.fixup] Starting system autofix at state running
[supervisor.resolution.fixup] System autofix complete
[supervisor.host.services] Updating service information
[supervisor.host.network] Updating local network information
[supervisor.host.sound] Updating PulseAudio information
[supervisor.host.manager] Host information reload completed
```
</details>

But after like 10 minutes (which the screen tells you it can take) it
continued.

# Configuring

One of the first things I noticed after setting up the basic system is that:

 * I needed to replace the USB power cable, since the RPI Power status reported a problem
 * Bluetooth seems broken...

While I could easily fix the power supply issue with a different USB cable, I
couldn't figure out what the issue with bluetooth is. Looking at the
bugtrackers of Home Assistant did reveal quite some issues that people are having with bluetooth:

 * <https://github.com/home-assistant/core/issues?q=is%3Aissue+is%3Aopen+bluetooth>
 * <https://github.com/home-assistant/operating-system/issues?q=bluetooth+is%3Aopen+label%3Aboard%2Fraspberrypi>
