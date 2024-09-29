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

# Configuring AdGuard Home

For some reason the AdGuard Home add-on only listens on the local interface
instead of all interfaces so it won't be reachable from outside. As instructed by <https://community.home-assistant.io/t/adguard-listening-on-127-0-0-1-instead-of-the-hassio-ip/310137/9> I changed the add-on configuration accordingly:

<center>
![The Home Assistant AdGuard add-on configuration]({static}/images/home_assistant/adguard_home_addon_configuration.png){width=80%}
</center>

# Configuring Router

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

# Statistics

After a few days of usage without any problems the statistics looked like this:

<center>
![AdGuard statistics]({static}/images/home_assistant/adguard_home_statistics.png){width=80%}
</center>

As you can see there are quite some blocked requests and all request originate
from one client, my router.

# Use AdGuard Directly as the DNS Server

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

When using the DNS server directly we see in the statistics which clients
create the most DNS queries and can also get some insight in what domains are
queried by the WiFi switches for example.

<center>
![AdGuard statistics for individual clients]({static}/images/home_assistant/adguard_home_statistics_2.png){width=80%}
</center>

What I noticed as well is that still around 25% of the requests are coming from
the router. Since all clients should use the AdGuard DNS server directly this
confused me. It turns out that there are two reasons for that:

 1. HomeAssistant itself is configured to use the router DNS directly, which is
    probably smart to avoid trouble if the AdGuard DNS server would misbehave.
 2. My router advertises IPv6 DNS servers as well, but doesn't allow to change
    them in the configuration, thus it still promotes itself as the DNS server
    there.

I found out about the IPv6 thing when I looked into the `systemd-resolve`
status:

```text
$ systemd-resolve --status
Global
           Protocols: +LLMNR +mDNS -DNSOverTLS DNSSEC=no/unsupported
    resolv.conf mode: foreign
Fallback DNS Servers: 1.1.1.1#cloudflare-dns.com 9.9.9.9#dns.quad9.net 8.8.8.8#dns.google 2606:4700:4700::1111#cloudflare-dns.com
                      2620:fe::9#dns.quad9.net 2001:4860:4860::8888#dns.google

Link 2 (enp38s0)
    Current Scopes: none
         Protocols: -DefaultRoute +LLMNR mDNS=resolve -DNSOverTLS DNSSEC=no/unsupported

Link 4 (wlan0)
    Current Scopes: DNS LLMNR/IPv4 LLMNR/IPv6 mDNS/IPv4 mDNS/IPv6
         Protocols: +DefaultRoute +LLMNR mDNS=resolve -DNSOverTLS DNSSEC=no/unsupported
Current DNS Server: $ROUTER_IPv6_ADDRESS
       DNS Servers: $AD_GUARD_IPv4_ADDRESS $ROUTER_IPv6_ADDRESS
```

So IPv6 is now available since more then 20 years but there is still very bad
support for it available in network hardware. My options for fixing this include

 * Disabling IPv6 on my router
 * Deactivating DHCP on my router and use HomeAssistant as my DHCP server
 * Override DNS servers on my clients

But for now I'll just live with the slightly worse statistics on AdGuard.
