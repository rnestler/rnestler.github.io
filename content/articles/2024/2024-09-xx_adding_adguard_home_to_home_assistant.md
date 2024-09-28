Title: Adding AdGuard Home to my Home Assistant
Tags: Raspberry Pi, Linux, Home Assistant, AdGuard
Language: en
Summary: Setting up a AdGuard Home on my Home Assistant
Status: draft

I recently noticed that home assistant offers an [AdGuard Home
add-on](https://github.com/hassio-addons/addon-adguard-home). While I already
block adds and tracking in the web with [uBlock
Origin](https://ublockorigin.com/) on my laptops and Android phone, I'm still
annoyed and concerned by all the tracking that happens for example by apps.

# Installing the Add-on

Installing is as easy as going to
<http://homeassistant.local:8123/hassio/store> and selecting the AdGuard Home
add-on.

According to the [AdGuard Home add-on install
docs](https://github.com/hassio-addons/addon-adguard-home/blob/main/adguard/DOCS.md)
one should configure the network to use a static IP address:
<http://homeassistant.local:8123/config/network>

<center>
![The Home Assistant network configuration]({static}/images/home_assistant/home_assistant_network_configuration.png)
</center>

## Configuring AdGuard Home

For some reason the AdGuard Home add-on only listens on the local interface
instead of all interfaces so it won't be reachable from outside. As instructed by <https://community.home-assistant.io/t/adguard-listening-on-127-0-0-1-instead-of-the-hassio-ip/310137/9> I changed the add-on configuration accordingly:

<center>
![The Home Assistant AdGuard add-on configuration]({static}/images/home_assistant/adguard_home_addon_configuration.png){width=80%}
</center>

## Configuring Router

Basically there are two ways to configure your router when wanting to use AdGuard Home:

 1. Adding it as the DNS server your router uses
 2. Adding it to the DNS server distributed via DHCP

The first one has the advantage that local names will still be resolved by the
router, while the second one gives better statistics over which clients use the
AdGuard Home DNS server.

Since I didn't want to break stuff in my network I decided to first go with
option one. Also I kept one of the original DNS servers just in case for the
first test run.

<center>
![Router DNS configuration]({static}/images/home_assistant/adguard_home_router_configuration.png){width=80%}
</center>

## Statistics

After a few days of usage without any problems the statistics looked like this:

<center>
![AdGuard statistics]({static}/images/home_assistant/adguard_home_statistics.png){width=80%}
</center>

As you can see there are quite some blocked requests and all request originate
from one client, my router.

## Use AdGuard Directly as the DNS Server

Since the usage went without any problems I decided to configure my router to
directly use AdGuard Home as the DNS server for its clients.

<center>
![Router DHCP configuration]({static}/images/home_assistant/adguard_home_router_configuration_2.png)
</center>

To make sure that the important local names still resolve (NAS, HomeAssistant,
SnapCast, ...) I added them manually as DNS rewrites.

<center>
![Router DHCP configuration]({static}/images/home_assistant/adguard_home_dns_rewrites.png){width=80%}
</center>
