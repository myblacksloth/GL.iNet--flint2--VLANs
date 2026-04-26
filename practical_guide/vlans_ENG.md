
> [!NOTE]  
> The original guide is in Italian. This translation was produced by AI

- [Why this guide](#why-this-guide)
- [Steps covered](#steps-covered)
- [Prerequisites](#prerequisites)
  - [VLAN connection](#vlan-connection)
  - [Changing the default network address](#changing-the-default-network-address)
- [Configuring VLANs for physical devices](#configuring-vlans-for-physical-devices)
  - [Remove the main LAN from the default br-lan and associate it with the main VLAN](#remove-the-main-lan-from-the-default-br-lan-and-associate-it-with-the-main-vlan)
  - [Define interfaces for all VLANs](#define-interfaces-for-all-vlans)
    - [Defining all interfaces](#defining-all-interfaces)
  - [Allow router management from the home interface (via firewall)](#allow-router-management-from-the-home-interface-via-firewall)
- [Configuring wireless networks](#configuring-wireless-networks)
- [Firewall rules](#firewall-rules)
  - [All zones must be able to reach the WAN (internet access)](#all-zones-must-be-able-to-reach-the-wan-internet-access)
  - [Enable DNS for all zones](#enable-dns-for-all-zones)
  - [Enable DHCP for zones](#enable-dhcp-for-zones)
  - [Intra-zone forwarding rules](#intra-zone-forwarding-rules)
  - [Custom firewall rules](#custom-firewall-rules)
- [Configuring Avahi for intra-zone mDNS](#configuring-avahi-for-intra-zone-mdns)
  - [Avahi configuration](#avahi-configuration)
- [Troubleshooting](#troubleshooting)
  - [Firewall](#firewall)
  - [Old OpenWRT (or new too???) traffic not working (to be confirmed)](#old-openwrt-or-new-too-traffic-not-working-to-be-confirmed)
  - [(broken luci) Add rule for server-\>iot](#broken-luci-add-rule-for-server-iot)
  - [Corrupted LuCI](#corrupted-luci)
  - [Avahi not working (still to be fixed)](#avahi-not-working-still-to-be-fixed)
- [Network testing](#network-testing)
  - [Manual MAC address to IP assignment](#manual-mac-address-to-ip-assignment)
  - [Updating firewall rules for a realistic environment](#updating-firewall-rules-for-a-realistic-environment)
  - [Useful commands](#useful-commands)
    - [DHCP](#dhcp)
    - [Check firewall rules](#check-firewall-rules)
    - [Adding rules via SSH](#adding-rules-via-ssh)
    - [Fixing rules via SSH](#fixing-rules-via-ssh)
  - [Path testing](#path-testing)
  - [Avahi testing](#avahi-testing)
- [Explanations for nOObs](#explanations-for-noobs)
  - [What is a local network (LAN)?](#what-is-a-local-network-lan)
  - [What is a switch?](#what-is-a-switch)
  - [What is a VLAN?](#what-is-a-vlan)
  - [What are OpenWRT and LuCI?](#what-are-openwrt-and-luci)
  - [Why do the guide steps need to be done in this order?](#why-do-the-guide-steps-need-to-be-done-in-this-order)
    - [1. First configure the physical switch (VLANs on physical devices)](#1-first-configure-the-physical-switch-vlans-on-physical-devices)
    - [2. Then create the logical interfaces](#2-then-create-the-logical-interfaces)
    - [3. Then configure the wireless networks](#3-then-configure-the-wireless-networks)
    - [4. Then configure the firewall](#4-then-configure-the-firewall)
    - [5. Finally configure Avahi for mDNS](#5-finally-configure-avahi-for-mdns)
  - [Visual summary](#visual-summary)
  - [Quick glossary](#quick-glossary)

# Why this guide

This guide was born from the need to have a practical-technical reference for setting up VLANs on GL.iNet routers via the LuCI interface (directly through OpenWRT).
The main material available is on YouTube, but there are currently no practical written guides in Italian.

> [!NOTE]  
> In the age of AI, this guide was NOT written with the help of AI.
>
> Features not marked as *not working*, *incomplete* or *questionable* have been manually tested.
>
> Should any paragraphs be produced with AI assistance in the future, this will be explicitly stated.

# Steps covered

1. Defining VLANs for physical interfaces
2. Defining VLANs via interfaces
3. Firewall rules for intra-zone traffic
4. Configuring Avahi for intra-zone mDNS (did not work in testing)
5. Troubleshooting
6. Testing

# Prerequisites

> [!IMPORTANT]  
> Save without applying — only apply changes in bulk at the end of each block.
>
> The management network used at this stage is the default 5GHz network.

## VLAN connection

SCR-20260420-ksqv

## Changing the default network address

SCR-20260420-kthu

# Configuring VLANs for physical devices

|  Old interface | New interface  |
| ------------ | ------------ |
|  ![](stuff/i/SCR-20260420-ktwh.png) |   |
| ![](stuff/i/SCR-20260420-kvuw.png)  |   |
| ![](stuff/i/SCR-20260420-kwgs.png)  |   |

## Remove the main LAN from the default br-lan and associate it with the main VLAN

> [!IMPORTANT]  
> This is a fundamental step to avoid connection issues at this stage.

|  Old interface | New interface  |
| ------------ | ------------ |
| ![](stuff/i/SCR-20260420-kygj.png)  | ![](stuff/i/.png)  |

## Define interfaces for all VLANs

|  Old interface | New interface  |
| ------------ | ------------ |
| ![](stuff/i/SCR-20260420-kyut.png)  | ![](stuff/i/.png)  |
| ![](stuff/i/.png)  | ![](stuff/i/.png)  |


### Defining all interfaces

**Repeat the procedure for each VLAN**

| Details  | Screenshot  |
| ------------ | ------------ |
| Creating the home interface | ![](stuff/i/SCR-20260420-kzof.png)  |
| Setting up DHCP for the interface (do this first because the GUI has a bug on older versions) | ![](stuff/i/SCR-20260420-kzyw.png)  |
| Assigning a static IP for the VLAN | ![](stuff/i/SCR-20260420-laje.png)  |
| Creating a new firewall zone for the VLAN | ![](stuff/i/SCR-20260420-lapt.png)  |
| Result | ![](stuff/i/SCR-20260420-layo.png)  |

## Allow router management from the home interface (via firewall)

> [!NOTE]  
> Ignore the pre-configured firewall rules and consider only the rule in question.

| Details  | Screenshot  |
| ------------ | ------------ |
|  Allow input rule from home zone |  ![](stuff/i/SCR-20260420-lcgy.png) |
|   |  ![](stuff/i/.png) |

> [!TIP]
> At this point you can apply the changes.

# Configuring wireless networks

> [!TIP]
> The router will be managed from the home VLAN, so I won't create a WLAN for the management VLAN.

| Details  | Screenshot  |
| ------------ | ------------ |
|  Associate the default 5GHz network with the home VLAN |  ![](stuff/i/SCR-20260420-ldar.png) |
| Delete the default 2.4GHz network to avoid interface startup issues  |  ![](stuff/i/SCR-20260420-ldlv.png) |
| Creating wireless networks for the VLANs  |  ![](stuff/i/SCR-20260420-ldzg.png) |
| Setting the network password and security type  |  ![](stuff/i/SCR-20260420-lefk.png) |

> [!NOTE]  
> If needed, also configure the 5GHz networks.

# Firewall rules

> [!TIP]
> Keep in mind the relationship: ZONE > zone settings (input, output, reject) > Traffic rules
>
> Additional info below.

> [!IMPORTANT]  
> In the section "*Old OpenWRT (or new too???) traffic not working (to be confirmed)*" there is information about zone configuration found in the official documentation and forums.
>
> The content shown there is pending confirmation.
>
> Check that section as well for a clearer overview.

> [!NOTE]  
> input, output, forward refer to the router itself.
>
> input = from the zone toward the router
> output = from the router toward the zone
> forward = from one zone to another passing through the router

## All zones must be able to reach the WAN (internet access)

| Details  | Screenshot  |
| ------------ | ------------ |
|  Preliminary note: the basic configuration works per zone (zone + standard rules) — initially think in terms of columns |  ![](stuff/i/SCR-20260420-lfdm.png) |
|  Allow forwarding from all zones to the WAN zone |  ![](stuff/i/SCR-20260420-lfkl.png) |

## Enable DNS for all zones

| Details  | Screenshot  |
| ------------ | ------------ |
| Traffic rules tab  |  ![](stuff/i/SCR-20260420-lgdh.png) |
| Where to create new rules  |  ![](stuff/i/SCR-20260420-lghx.png) |
|  Enable DNS rules for all zones (otherwise DNS won't work) |  ![](stuff/i/SCR-20260420-lgsp.png) |
| Defined DNS rules |  ![](stuff/i/SCR-20260420-lhej.png) |

## Enable DHCP for zones

> [!IMPORTANT]  
> Configure for all VLAN zones.

| Details  | Screenshot  |
| ------------ | ------------ |
|  Rule configuration |  ![](stuff/i/SCR-20260420-lzcm.png) |


## Intra-zone forwarding rules

> [!TIP]
> Always keep in mind that OpenWRT's firewall works per zone!
>
> So make changes per individual zone! For example, from iot choose who you can forward to.
>
> Repeat the configuration for all zones until you achieve the desired result.

| Details  | Screenshot  |
| ------------ | ------------ |
|  Rules between zones |  ![](stuff/i/SCR-20260420-lifd.png) |
| Example of how to make changes  |  ![](stuff/i/SCR-20260420-lipx.png) |

## Custom firewall rules

> [!NOTE]  
> Configure according to your personal needs.

> [!TIP]
> If configuration is done by IP address rather than MAC address, make sure the involved IP addresses are configured statically (static IP-to-MAC binding).

> [!IMPORTANT]  
> For devices that need to communicate intra-zone via mDNS, a dedicated service must be configured (preferably via Avahi).

| Details  | Screenshot  |
| ------------ | ------------ |
|  IoT device that needs to communicate with a server |  ![](stuff/i/SCR-20260420-ljsx.png) |
| Multimedia devices in IoT that need to communicate with home (mDNS) ... this alone is not enough!  |  ![](stuff/i/SCR-20260420-lkky.png) |


# Configuring Avahi for intra-zone mDNS

> [!TIP]
> If the SSH connection doesn't work, try the following command:

```bash
ssh -o HostKeyAlgorithms=+ssh-rsa -o PubkeyAcceptedKeyTypes=+ssh-rsa root@192.168.10.1
```

| Details  | Screenshot  |
| ------------ | ------------ |
| Enable SSH access from the home VLAN  |  ![](stuff/i/SCR-20260420-llvs.png) |
|  Test SSH connection |  ![](stuff/i/SCR-20260420-lmsf.png) |
|  Search for Avahi packages |  ![](stuff/i/SCR-20260420-loat.png) |
|  Installing Avahi |  ![](stuff/i/SCR-20260420-logg.png) |
|  Installed packages |  ![](stuff/i/SCR-20260420-lpav.png) |

## Avahi configuration

Configuration file:

`/etc/avahi/avahi-daemon.conf`

Modify according to your zones.

```ini
[server]
use-ipv4=yes
use-ipv6=no
allow-interfaces=br-home,br-iot   # your VLANs/bridges
ratelimit-interval-usec=1000000
ratelimit-burst=1000

[reflector]
enable-reflector=yes
reflect-ipv=no
```

Restart the service:

```bash
/etc/init.d/avahi-daemon enable
/etc/init.d/avahi-daemon restart
#
# check
ps | grep avahi
logread | grep -i avahi
```

> [!WARNING]  
> As noted in the firewall section, for mDNS to work correctly you also need to configure firewall rules!

# Troubleshooting

## Firewall

> [!CAUTION]
> Rules are interpreted in order, from first to last. So a specific *allow* must come before a generic *deny*.

| Details  | Screenshot  |
| ------------ | ------------ |
|  The 2 VLANs are not communicating --> adding manual rules |  ![](stuff/i/SCR-20260420-mdsk.png) |
|   |  ![](stuff/i/SCR-20260420-mdxz.png) |
|   |  ![](stuff/i/SCR-20260420-mgkr.png) |
|   |  ![](stuff/i/.png) |
|   |  ![](stuff/i/.png) |

## Old OpenWRT (or new too???) traffic not working (to be confirmed)

If traffic between zones is not working, then (example):

1. forwarding from server to iot  (ENABLE ZONE-TO-ZONE FORWARD in the zone rules)
2. rule ACCEPT from 192.168.8.30 to 192.168.30.21  (allows the connection between the two devices via traffic rules)
3. rule ACCEPT from 192.168.30.21 to 192.168.8.30  (allows return packets via traffic rules)
4. rule REJECT from server to iot  (blocks everything else via traffic rules)
5. rule REJECT from iot to server  (blocks everything else via traffic rules)

Sources used to reach this conclusion:

- [https://oldwiki.archive.openwrt.org/doc/uci/firewall](https://oldwiki.archive.openwrt.org/doc/uci/firewall "https://oldwiki.archive.openwrt.org/doc/uci/firewall")
- [https://openwrt.org/docs/guide-user/firewall/fw3_configurations/fw3_config_examples](http://https://openwrt.org/docs/guide-user/firewall/fw3_configurations/fw3_config_examples "https://openwrt.org/docs/guide-user/firewall/fw3_configurations/fw3_config_examples")
- [https://forum.openwrt.org/t/firewall-zones-forwards-and-rules/25197](http://https://forum.openwrt.org/t/firewall-zones-forwards-and-rules/25197 "https://forum.openwrt.org/t/firewall-zones-forwards-and-rules/25197")
[https://forum.openwrt.org/t/firewall-traffic-rules-override-zone-settings/150238](http://https://forum.openwrt.org/t/firewall-traffic-rules-override-zone-settings/150238 "https://forum.openwrt.org/t/firewall-traffic-rules-override-zone-settings/150238")
[https://forum.openwrt.org/t/how-to-set-up-firewall-rules-zones-correctly-for-vlans/105686](http://https://forum.openwrt.org/t/how-to-set-up-firewall-rules-zones-correctly-for-vlans/105686 "https://forum.openwrt.org/t/how-to-set-up-firewall-rules-zones-correctly-for-vlans/105686")
[https://forum.archive.openwrt.org/viewtopic.php?id=72338](https://forum.archive.openwrt.org/viewtopic.php?id=72338http:// "https://forum.archive.openwrt.org/viewtopic.php?id=72338")

## (broken luci) Add rule for server->iot

```bash
uci add firewall forwarding
uci set firewall.@forwarding[-1].src='server'
uci set firewall.@forwarding[-1].dest='iot'
uci commit firewall
/etc/init.d/firewall restart
```

## Corrupted LuCI

```
/usr/lib/lua/luci/dispatcher.lua:230: /etc/config/luci seems to be corrupt, unable to find section 'main'
stack traceback:
	[C]: in function 'assert'
	/usr/lib/lua/luci/dispatcher.lua:230: in function 'dispatch'
	/usr/lib/lua/luci/dispatcher.lua:127: in function </usr/lib/lua/luci/dispatcher.lua:126>
```

Solution (via SSH on the router):

```bash
ls /etc/init.d/ | grep -E "nginx|httpd|luci"
rm -f /etc/config/luci
# /etc/init.d/uhttpd restart
/etc/init.d/nginx restart
```
```bash
cp /etc/config/luci /etc/config/luci.bak 2>/dev/null

cat > /etc/config/luci <<'EOF'
config core main
	option lang auto
	option mediaurlbase /luci-static/bootstrap
	option resourcebase /luci-static/resources
	option ubuspath /ubus/

config extern flash_keep
	option uci "/etc/config/"
	option dropbear "/etc/dropbear/"
	option openvpn "/etc/openvpn/"
	option passwd "/etc/passwd"
	option opkg "/etc/opkg.conf"
	option firewall "/etc/firewall.user"
	option uploads "/lib/uci/upload/"

config internal languages

config internal sauth
	option sessionpath "/tmp/luci-sessions"
	option sessiontime 3600

config internal ccache
	option enable 1

config internal themes

config internal apply
	option rollback 90
	option holdoff 4
	option timeout 5
	option display 1.5
EOF
```

```bash
/etc/init.d/uhttpd restart
rm -f /tmp/luci-indexcache /tmp/luci-modulecache/*

rm -f /tmp/luci-indexcache
rm -rf /tmp/luci-modulecache/*
```

```bash
uci show luci
uci show luci.main
ubus call uci get '{"config":"luci","section":"main"}'
```

```bash
/etc/init.d/ubus restart
/etc/init.d/rpcd restart
/etc/init.d/uhttpd restart
```

## Avahi not working (still to be fixed)

```bash
# Enable multicast on br-iot if not present
ip link set br-iot multicast on

# Restart avahi
/etc/init.d/avahi-daemon restart
```

# Network testing

## Manual MAC address to IP assignment

| Details  | Screenshot  |
| ------------ | ------------ |
|  Manual assignment |  ![](stuff/i/SCR-20260420-mbvj.png) |

## Updating firewall rules for a realistic environment

| Details  | Screenshot  |
| ------------ | ------------ |
|  Server 2 cam |  ![](stuff/i/SCR-20260420-mcvf.png) |

## Useful commands

### DHCP

```bash
# View all active DHCP leases
cat /tmp/dhcp.leases

# Or with more details (MAC, IP, hostname)
cat /tmp/dhcp.leases | awk '{print $3, $4, $2}'

# Also view devices responding on the network
arp -a
```

### Check firewall rules

```bash
# View all firewall rules with names
uci show firewall | grep -E "name|src_ip|dest_ip|target"
```

View only forwarding rules:

```bash
uci show firewall | grep -n "forwarding"
```

### Adding rules via SSH

```bash
# From server (Pi5) to iot (cam)
uci add firewall rule
uci set firewall.@rule[-1].name='server-to-iot'
uci set firewall.@rule[-1].src='server'
uci set firewall.@rule[-1].dest='iot'
uci set firewall.@rule[-1].src_ip='192.168.8.30'
uci set firewall.@rule[-1].dest_ip='192.168.30.21'
uci set firewall.@rule[-1].target='ACCEPT'
uci set firewall.@rule[-1].proto='any'

# From iot (cam) to server (Pi5)
uci add firewall rule
uci set firewall.@rule[-1].name='iot-to-server'
uci set firewall.@rule[-1].src='iot'
uci set firewall.@rule[-1].dest='server'
uci set firewall.@rule[-1].src_ip='192.168.30.21'
uci set firewall.@rule[-1].dest_ip='192.168.8.30'
uci set firewall.@rule[-1].target='ACCEPT'
uci set firewall.@rule[-1].proto='any'

# Save and apply
uci commit firewall
/etc/init.d/firewall restart
```

### Fixing rules via SSH

```bash
uci set firewall.@rule[21].dest_ip='192.168.30.21'
uci commit firewall
/etc/init.d/firewall restart
```

## Path testing

| Details  | Screenshot  |
| ------------ | ------------ |
|  from home to server |  ![](stuff/i/SCR-20260420-mtnf.png) |
|  from server to iot |  ![](stuff/i/SCR-20260420-mtlm.png) |
<!-- |   |  ![](stuff/i/.png) |
|   |  ![](stuff/i/.png) |
|   |  ![](stuff/i/.png) |
|   |  ![](stuff/i/.png) | -->

## Avahi testing

| Details  | Screenshot  |
| ------------ | ------------ |
|  Firewall rule for forwarding |  ![](stuff/i/SCR-20260420-mvpn.png) |
| Specific firewall rules  |  ![](stuff/i/SCR-20260420-mver.png) |
|  Test |  ![](stuff/i/.png) |

<!--
| ![](stuff/i/.png)  | ![](stuff/i/.png)  |
|   | ![](stuff/i/.png)  |
-->

<!--
|  Old interface | New interface  |
| ------------ | ------------ |
| ![](stuff/i/.png)  | ![](stuff/i/.png)  |


| Details  | Screenshot  |
| ------------ | ------------ |
|   |  ![](stuff/i/.png) |
|   |  ![](stuff/i/.png) |


-->


# Explanations for nOObs

> [!WARNING]  
> This section was created with the help of AI and has NOT been verified by the author.

## What is a local network (LAN)?

When you connect multiple devices (PC, phone, smart TV, cameras) to the same router, they all end up on the same **local network (LAN)**. By default, they all "see" each other: your PC can talk to the IoT camera, the smart TV can attempt connections to your NAS, etc.

This is fine for a simple network, but it's a security problem: if an IoT device (typically poorly secured) is compromised, the attacker gains access to your entire network.

## What is a switch?

A **switch** is the physical "hub" of your network: it's the component (often integrated into the router) that receives data from one port and routes it to the correct destination port. Every device connected via ethernet cable is connected to a port on the switch.

Without VLANs, a switch sends broadcast traffic (network announcements, DHCP requests, mDNS, etc.) to **all** ports: every device receives everything.

## What is a VLAN?

A **VLAN (Virtual LAN)** is a virtual local network: it allows you to logically divide a physical switch into multiple isolated "virtual" switches, without needing separate hardware.

```
Without VLANs:
[PC] [Phone] [IoT Camera] [NAS]  <-- all on the same network, all visible to each other

With VLANs:
VLAN home:    [PC] [Phone]
VLAN iot:     [IoT Camera]
VLAN server:  [NAS]
              ^--- isolated from each other, the firewall decides who can talk to whom
```

Each VLAN has a numeric identifier (VLAN ID) that is "tagged" on ethernet packets, so the switch knows which virtual network each packet belongs to.

## What are OpenWRT and LuCI?

**OpenWRT** is an alternative Linux operating system for routers, far more flexible than factory firmware. GL.iNet routers support it natively.

**LuCI** is OpenWRT's graphical web interface: it lets you configure everything from a browser instead of using the command line.

## Why do the guide steps need to be done in this order?

### 1. First configure the physical switch (VLANs on physical devices)

The switch integrated in the router needs to know which ports belong to which VLAN. This is the lowest level: hardware. Without this step, packets are not correctly routed at the physical level.

> If you skip this step: devices connected via cable won't be assigned to the correct VLAN.

### 2. Then create the logical interfaces

OpenWRT needs a **software interface** for each VLAN: this is like telling the operating system "this VLAN exists, I'm assigning it IP address X.X.X.1, I'm managing its DHCP from here". Each interface corresponds to a bridge (e.g. `br-home`, `br-iot`) that acts as the router's "entry point" into that VLAN.

> If you skip this step: the router doesn't know how to handle traffic for that VLAN, and devices won't receive an IP address.

### 3. Then configure the wireless networks

Wi-Fi networks (SSIDs) must be **associated** with a specific VLAN/interface. Otherwise, all wireless devices end up on the same network regardless of which Wi-Fi they connect to.

> If you skip this step: creating separate SSIDs is pointless — all wireless traffic still goes to the same network.

### 4. Then configure the firewall

By default in OpenWRT, once VLANs are created, they are **completely isolated**: no zone can communicate with the others. The firewall must be explicitly configured to:

- **Allow internet access (WAN)** from all zones — without this rule, IoT devices have no internet
- **Enable DNS** — DNS is the service that translates `google.com` into an IP address. If not explicitly enabled for each zone, websites won't load even if connectivity is present
- **Enable DHCP** — DHCP is the service that automatically assigns an IP address to devices when they connect. Without it, devices remain without an IP
- **Define intra-zone forwarding** — if you want your PC (in `home`) to be able to see the NAS (in `server`), you must tell the firewall explicitly

> If you skip this step: internet won't work on the new VLANs, or some VLANs won't be able to communicate with the ones they should.

> [!CAUTION]
> Firewall rules are read from top to bottom. A specific `ALLOW` rule must come **before** a generic `DENY`, otherwise it is ignored and the generic block takes precedence.

### 5. Finally configure Avahi for mDNS

**mDNS (multicast DNS)** is the protocol used by Apple devices (AirPlay, AirPrint), Chromecast, network printers, etc. to "announce" themselves on the local network without needing a central DNS server.

The problem: mDNS uses **multicast** packets that by definition do not cross network boundaries. If your Apple TV is on the `iot` VLAN and your Mac is on the `home` VLAN, they can't see each other.

**Avahi** is a service that acts as a "translator": it listens for mDNS announcements on one VLAN and "reflects" them onto the others. It must be configured after the firewall because it needs the interfaces to already exist and the multicast firewall rules to be active.

> If you skip this step: Chromecast, AirPlay, network printers won't work across different VLANs.

## Visual summary

```
[Physical device / Wi-Fi]
        |
        v
[Physical switch with VLAN tag]   <-- Step 1: physical VLAN configuration
        |
        v
[OpenWRT logical interface]       <-- Step 2: interfaces + DHCP + IP
        |
        v
[Associated wireless network]     <-- Step 3: SSID -> VLAN
        |
        v
[Firewall / Zones]                <-- Step 4: who can talk to whom
        |
        v
[Avahi for mDNS]                  <-- Step 5: cross-VLAN local services
```

## Quick glossary

| Term | Simple explanation |
|------|--------------------|
| **LAN** | Your home network |
| **WAN** | The internet (the "external" network) |
| **VLAN** | A logically isolated virtual subnetwork |
| **Switch** | The physical "hub" that routes ethernet cables |
| **Bridge** | A software component that joins multiple network interfaces |
| **DHCP** | The service that automatically assigns an IP to devices |
| **DNS** | The service that translates names (google.com) into IP addresses |
| **Firewall** | The "gatekeeper" that decides which connections are allowed |
| **Zone (firewall)** | A group of interfaces with the same security rules |
| **Forward** | Allowing traffic to pass from one zone to another |
| **mDNS** | Protocol for discovering local devices (Chromecast, AirPlay) |
| **Avahi** | Service that propagates mDNS across different VLANs |
| **LuCI** | OpenWRT's graphical web interface |
| **OpenWRT** | Linux operating system for routers |


<!--

> [!NOTE]  
> Highlights information that users should take into account, even when skimming.

> [!TIP]
> Optional information to help a user be more successful.

> [!IMPORTANT]  
> Crucial information necessary for users to succeed.

> [!WARNING]  
> Critical content demanding immediate user attention due to potential risks.

> [!CAUTION]
> Negative potential consequences of an action.
> 
-->
