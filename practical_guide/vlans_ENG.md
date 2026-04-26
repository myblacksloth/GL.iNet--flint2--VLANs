
> [!NOTE]  
> The original guide is in Italian. This translation was produced by AI.

- [Why this guide](#why-this-guide)
- [Steps covered](#steps-covered)
- [Prerequisites](#prerequisites)
  - [VLAN connection](#vlan-connection)
  - [Changing the default network address](#changing-the-default-network-address)
- [Configuring VLANs for physical devices](#configuring-vlans-for-physical-devices)
  - [Remove the main LAN from the default `br-lan` and associate it with the main VLAN](#remove-the-main-lan-from-the-default-br-lan-and-associate-it-with-the-main-vlan)
  - [Define interfaces for all VLANs](#define-interfaces-for-all-vlans)
    - [Defining all interfaces](#defining-all-interfaces)
      - [Older versions](#older-versions)
      - [Newer versions \(e.g. 21...\)](#newer-versions-eg-21)
  - [Allow router management from the home interface \(via firewall\)](#allow-router-management-from-the-home-interface-via-firewall)
- [Configure wireless networks](#configure-wireless-networks)
- [Firewall rules](#firewall-rules)
  - [All zones must be able to reach the WAN \(internet access\)](#all-zones-must-be-able-to-reach-the-wan-internet-access)
  - [Enable DNS for all zones](#enable-dns-for-all-zones)
  - [Enable DHCP for the zones](#enable-dhcp-for-the-zones)
  - [Intra-zone forwarding rules](#intra-zone-forwarding-rules)
  - [Custom firewall rules](#custom-firewall-rules)
- [Configuring Avahi for intra-zone mDNS](#configuring-avahi-for-intra-zone-mdns)
  - [Avahi configuration](#avahi-configuration)
    - [Configuration](#configuration)
    - [Firewall rules required for operation](#firewall-rules-required-for-operation)
    - [Activation](#activation)
- [Basic configuration summary](#basic-configuration-summary)
  - [Possible IoT-\>home rules](#possible-iot-home-rules)
- [Troubleshooting](#troubleshooting)
  - [Firewall](#firewall)
  - [Intra-VLAN traffic does not work](#intra-vlan-traffic-does-not-work)
  - [(broken luci) Add rule for server-\>iot](#broken-luci-add-rule-for-server-iot)
  - [Corrupted LuCI](#corrupted-luci)
- [Useful commands](#useful-commands)
    - [DHCP](#dhcp)
    - [Check firewall rules](#check-firewall-rules)
    - [Add rules via SSH](#add-rules-via-ssh)
    - [Fix rules via SSH](#fix-rules-via-ssh)
- [Network testing](#network-testing)
  - [Path testing](#path-testing)
- [Explanations for nOObs](#explanations-for-noobs)
  - [What is a local network \(LAN\)?](#what-is-a-local-network-lan)
  - [What is a switch?](#what-is-a-switch)
  - [What is a VLAN?](#what-is-a-vlan)
  - [What are OpenWRT and LuCI?](#what-are-openwrt-and-luci)
  - [Why must the guide steps be done in this order?](#why-must-the-guide-steps-be-done-in-this-order)
    - [1. First configure the physical switch \(VLANs on physical devices\)](#1-first-configure-the-physical-switch-vlans-on-physical-devices)
    - [2. Then create the logical interfaces](#2-then-create-the-logical-interfaces)
    - [3. Then configure the wireless networks](#3-then-configure-the-wireless-networks)
    - [4. Then configure the firewall](#4-then-configure-the-firewall)
    - [5. Finally configure Avahi for mDNS](#5-finally-configure-avahi-for-mdns)
  - [Visual summary](#visual-summary)
  - [Quick glossary](#quick-glossary)


> [!CAUTION]
> The author of this guide accepts NO responsibility for damage to property and/or persons resulting from consulting this guide or applying the material shown in it.
> By following the steps below, the user assumes full responsibility for any damage caused to the hardware on which they make changes from the software's original state.

# Why this guide

This guide comes from the need for a practical technical guide for defining VLANs on GL.iNet routers through the LuCI interface (directly via OpenWRT).
Most of the material currently available is on YouTube, but at the moment there are no practical written guides in Italian.

> [!NOTE]  
> In the age of AI, this guide was NOT written using AI.
>
> Features not marked as *not working*, *incomplete*, or *dubious* have been manually tested.
> 
> If, in the future, any paragraphs are produced with the help of AI, this will be explicitly stated.

# Steps covered

1. defining VLANs for physical interfaces
2. defining VLANs through interfaces
3. firewall rules for intra-zone traffic
4. configuring Avahi for intra-zone mDNS (it did not work in the tests carried out)
5. Troubleshooting
6. Testing

> [!TIP]
> If you are unsure about what you are doing, first try configuring a single VLAN
>
> (section [Basic configuration summary](#basic-configuration-summary))
>
> and only then configure all the others.
> 
> The indicated section is a basic working implementation of a complete configuration, but it does not include all firewall rules needed to obtain the desired security level.

# Prerequisites

> [!IMPORTANT]  
> Save without applying, and apply changes in bulk only at the end of each block.
>
> The management network used at this stage is the default 5GHz network.

## VLAN connection

SCR-20260420-ksqv

## Changing the default network address

SCR-20260420-kthu

# Configuring VLANs for physical devices

|  Old interface | New interface  |
| ------------ | ------------ |
|  ![](stuff/i/SCR-20260420-ktwh.png) |  ![](stuff/i/SCR-20260422-rjkv.png) |
| | ![](stuff/i/SCR-20260422-rjwx.png) |
| ![](stuff/i/SCR-20260420-kvuw.png)  |  ![](stuff/i/SCR-20260422-rkde.png) |
| ![](stuff/i/SCR-20260420-kwgs.png)  |   |

<!--
![](stuff/i/.png)
-->

SCR-20260422-rkde

## Remove the main LAN from the default `br-lan` and associate it with the main VLAN

> [!IMPORTANT]  
> Fundamental step to avoid connection issues at this stage.

![](stuff/i/SCR-20260422-rlcy.png)

|  Old interface | New interface  |
| ------------ | ------------ |
| ![](stuff/i/SCR-20260420-kygj.png)  | ![](stuff/i/SCR-20260422-rkqo.png)  |

## Define interfaces for all VLANs

|  Old interface | New interface  |
| ------------ | ------------ |
| ![](stuff/i/SCR-20260420-kyut.png)  | ![](stuff/i/SCR-20260422-rljz.png)  |
<!-- | ![](stuff/i/.png)  |   | -->


### Defining all interfaces

**Repeat the procedure for each VLAN**

#### Older versions

| Details  | Screenshot  |
| ------------ | ------------ |
| Create the `home` interface | ![](stuff/i/SCR-20260420-kzof.png)  |
| Set DHCP for the interface (do this first because the graphical interface has a bug on older versions) | ![](stuff/i/SCR-20260420-kzyw.png)  |
| Assign a static IP to the VLAN | ![](stuff/i/SCR-20260420-laje.png)  |
| Create a new firewall zone for the VLAN | ![](stuff/i/SCR-20260420-lapt.png)  |
| Result | ![](stuff/i/SCR-20260420-layo.png)  |

#### Newer versions (e.g. 21...)

| Details  | Screenshot  |
| ------------ | ------------ |
|  interface definition |  ![](stuff/i/SCR-20260422-ronl.png) |
|   |  ![](stuff/i/SCR-20260422-rmqs.png) |
|   |  ![](stuff/i/SCR-20260422-rnue.png) |
|   |  ![](stuff/i/SCR-20260422-rnzm.png) |
|   |  ![](stuff/i/SCR-20260422-rnqe.png) |
|   |  ![](stuff/i/.png) |

## Allow router management from the home interface (via firewall)

> [!NOTE]  
> Ignore the preconfigured firewall rules and consider only the one being discussed here.

| Details  | Screenshot  |
| ------------ | ------------ |
|  Allow input rule from the `home` zone |  ![](stuff/i/SCR-20260420-lcgy.png) |
|   |  ![](stuff/i/.png) |

> [!TIP]
> At this point it is possible to apply the changes.

# Configure wireless networks

> [!TIP]
> The router will be managed from the `home` VLAN, so I do not create a WLAN for the management VLAN.

| Details  | Screenshot  |
| ------------ | ------------ |
|  Associate the default 5GHz network with the `home` VLAN |  ![](stuff/i/SCR-20260420-ldar.png) |
| Delete the default 2.4GHz network to avoid interface startup problems |  ![](stuff/i/SCR-20260420-ldlv.png) |
| Create the wireless networks for the VLANs |  ![](stuff/i/SCR-20260420-ldzg.png) |
| Set the password and security type of the network |  ![](stuff/i/SCR-20260420-lefk.png) |

> [!NOTE]  
> If needed, also configure the 5GHz networks.

# Firewall rules

> [!TIP]
> Keep in mind the relationship ZONE > zone settings (input, output, reject) > Traffic rules
>
> additional information follows below

> [!IMPORTANT]  
> In the section "*Old OpenWRT (or new too???) traffic not working (to be confirmed)*" there is information about zone configuration gathered from official documentation and forums.
>
> The illustrated contents are still awaiting confirmation.
>
> Review that section as well to get a clearer overview.

> [!NOTE]  
> input, output, forward refer to the router itself
>
> input = from the zone toward the router
> output = from the router toward the zone
> forward = from one zone to another through the router

## All zones must be able to reach the WAN (internet access)

| Details  | Screenshot  |
| ------------ | ------------ |
|  Preliminary note: the basic configuration works per zone (zone + standard rules) - initially think column by column |  ![](stuff/i/SCR-20260420-lfdm.png) |
|  Allow forwarding from all zones to the WAN zone |  ![](stuff/i/SCR-20260420-lfkl.png) |

## Enable DNS for all zones

| Details  | Screenshot  |
| ------------ | ------------ |
| Traffic rules tab |  ![](stuff/i/SCR-20260420-lgdh.png) |
| Where to create new rules |  ![](stuff/i/SCR-20260420-lghx.png) |
|  Enable DNS rules for all zones (otherwise DNS will not work) |  ![](stuff/i/SCR-20260420-lgsp.png) |
| Defined DNS rules |  ![](stuff/i/SCR-20260420-lhej.png) |

## Enable DHCP for the zones

> [!IMPORTANT]  
> Configure it for all VLAN zones.

| Details  | Screenshot  |
| ------------ | ------------ |
|  Rule configuration |  ![](stuff/i/SCR-20260420-lzcm.png) |


## Intra-zone forwarding rules

> [!TIP]
> Always keep in mind that OpenWRT's firewall works per zone!
> 
> So make changes zone by zone. For example, from `iot`, choose which destinations forwarding is allowed to.
>
> Repeat the configuration for all zones until you get the desired result.

| Details  | Screenshot  |
| ------------ | ------------ |
|  rules between zones |  ![](stuff/i/SCR-20260420-lifd.png) |
| Example of how to make the changes |  ![](stuff/i/SCR-20260420-lipx.png) |

## Custom firewall rules

> [!NOTE]  
> Configure according to your own needs.

> [!TIP]
> If the configuration is done by IP address rather than MAC address, make sure the involved IP addresses are configured statically (static binding between IP and MAC).

> [!IMPORTANT]  
> For devices that must communicate across zones via mDNS, you need to configure a dedicated service (preferably via Avahi).

| Details  | Screenshot  |
| ------------ | ------------ |
|  IoT device that needs to communicate with a server |  ![](stuff/i/SCR-20260420-ljsx.png) |
| Multimedia devices in IoT that must communicate with `home` (mDNS) ... this alone is not enough!  |  ![](stuff/i/SCR-20260420-lkky.png) |


# Configuring Avahi for intra-zone mDNS

> [!TIP]
> On some routers it is necessary to use this SSH command:

```bash
ssh -o HostKeyAlgorithms=+ssh-rsa -o PubkeyAcceptedKeyTypes=+ssh-rsa root@192.168.10.1

scp -O -o HostKeyAlgorithms=+ssh-rsa -o PubkeyAcceptedKeyTypes=+ssh-rsa root@192.168.10.1:/root/file ./
```

| Details  | Screenshot  |
| ------------ | ------------ |
| Enable SSH access from the `home` VLAN |  ![](stuff/i/SCR-20260420-llvs.png) |
|  Test SSH connection |  ![](stuff/i/SCR-20260420-lmsf.png) |
|  Search for Avahi packages |  ![](stuff/i/SCR-20260420-loat.png) |
|  Install Avahi (**use the DBUS version**) |  ![](stuff/i/SCR-20260420-logg.png) |
|  Installed packages |  ![](stuff/i/SCR-20260426-lbgo.png) |

## Avahi configuration

> [!WARNING]  
> As indicated in the firewall section, for mDNS to work correctly you must also configure the firewall rules as shown below.

### Configuration

Configuration file:

`/etc/avahi/avahi-daemon.conf`

Modify according to your zones.

```ini
[server]
#host-name=foo
#domain-name=local
use-ipv4=yes
use-ipv6=yes
check-response-ttl=no
use-iff-running=no
allow-interfaces=br-home,br-iot

[publish]
publish-addresses=yes
publish-hinfo=yes
publish-workstation=no
publish-domain=yes
#publish-dns-servers=192.168.1.1
#publish-resolv-conf-dns-servers=yes

[reflector]
enable-reflector=yes
reflect-ipv=no

[rlimits]
#rlimit-as=
rlimit-core=0
rlimit-data=4194304
rlimit-fsize=0
rlimit-nofile=30
rlimit-stack=4194304
rlimit-nproc=3
```

**Pay close attention to**

```ini
allow-interfaces=br-home,br-iot
enable-reflector=yes
```

### Firewall rules required for operation

|  Description | Screenshot  |
| ------------ | ------------ |
| Create a rule to allow mDNS traffic from the `iot` zone to the router |  ![](./stuff/i/SCR-20260426-lcmr.png) |
| Result |  ![](./stuff/i/SCR-20260426-lcwf.png) |
<!-- |   |  ![](./stuff/i/.png) |
|   |  ![](./stuff/i/.png) | -->

> [!TIP]
> It would be advisable to enable mDNS for all zones.

### Activation

Restart the service:

```bash
/etc/init.d/avahi-daemon enable
/etc/init.d/avahi-daemon restart
#
# check
ps | grep avahi
logread | grep -i avahi
```

# Basic configuration summary

> [!NOTE]  
> This section is a basic working configuration and serves as a short summary of what has been shown so far.

> [!IMPORTANT]  
> This is only an example; it does not implement all the security rules that may be necessary.

> [!TIP]  
> It is recommended to configure the firewall rules as follows:
> 
> first enable generic forwarding rules between zones:
> for safety, disable input and forward (which will then be configured, where needed, through traffic rules)
> 
> 1. enable DNS for all zones
> 2. enable DHCP for all zones
> 3. enable specific rules for the desired traffic
> 4. enable mDNS where necessary
> 5. disable all traffic from a given zone toward the others


|  Description | Screenshot  |
| ------------ | ------------ |
| Physical switch configuration |  ![](./stuff/i/r1/SCR-20260426-lemh.png) |
| Interface configuration |  ![](./stuff/i/r1/SCR-20260426-leve.png) |
| Wireless networks |  ![](./stuff/i/r1/SCR-20260426-lfgq.png) |
| Firewall zones |  ![](./stuff/i/r1/SCR-20260426-lftw.png) |
| Firewall traffic rules (`iot->home` ban is technically **SUPERFLUOUS** because the `iot` zone already blocks forwarding in the firewall zone configuration) |  ![](./stuff/i/r1/SCR-20260426-lgoq.png) |
| Avahi for mDNS |  ![](./stuff/i/r1/SCR-20260426-licz.png) |
<!-- |   |  ![](./stuff/i/r1/.png) |
|   |  ![](./stuff/i/r1/.png) |
|   |  ![](./stuff/i/r1/.png) |
|   |  ![](./stuff/i/r1/.png) |
|   |  ![](./stuff/i/r1/.png) |
|   |  ![](./stuff/i/r1/.png) |
|   |  ![](./stuff/i/r1/.png) |
|   |  ![](./stuff/i/r1/.png) |
|   |  ![](./stuff/i/r1/.png) |
|   |  ![](./stuff/i/r1/.png) |
|   |  ![](./stuff/i/r1/.png) |
|   |  ![](./stuff/i/r1/.png) | -->

## Possible IoT->home rules

- Traffic from `home` is enabled by default toward every zone (zone forwarding enabled + intra-VLAN traffic configured through the zone rules)

![](./stuff/i/SCR-20260426-qqiw.png)

# Troubleshooting

## Firewall

> [!CAUTION]
> Rules are interpreted in order, from first to last. Therefore a specific *allow* must be placed before a generic *deny*.

| Details  | Screenshot  |
| ------------ | ------------ |
|  The 2 VLANs are not communicating --> add manual rules |  ![](stuff/i/SCR-20260420-mdsk.png) |
|   |  ![](stuff/i/SCR-20260420-mdxz.png) |
|   |  ![](stuff/i/SCR-20260420-mgkr.png) |
<!-- |   |  ![](stuff/i/.png) |
|   |  ![](stuff/i/.png) | -->

## Intra-VLAN traffic does not work

If traffic between zones does not work, then (example):

1. forwarding from `server` to `iot`  (ENABLE ZONE-TO-ZONE FORWARD in the zone rules)
2. rule ACCEPT from 192.168.8.30 to 192.168.30.21  (allows the connection between the two devices through traffic rules)
3. rule ACCEPT from 192.168.30.21 to 192.168.8.30  (allows return packets through traffic rules)
4. rule REJECT from `server` to `iot`  (blocks everything else through traffic rules)
5. rule REJECT from `iot` to `server`  (blocks everything else through traffic rules)

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

solution (via SSH on the router):

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

# Useful commands

### DHCP

```bash
# Show all active DHCP leases
cat /tmp/dhcp.leases

# Or with more details (MAC, IP, hostname)
cat /tmp/dhcp.leases | awk '{print $3, $4, $2}'

# Also view the devices responding on the network
arp -a
```

### Check firewall rules

```bash
# Show all firewall rules with names
uci show firewall | grep -E "name|src_ip|dest_ip|target"
```

Show only forwarding rules:

```bash
uci show firewall | grep -n "forwarding"
```

### Add rules via SSH

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

### Fix rules via SSH

```bash
uci set firewall.@rule[21].dest_ip='192.168.30.21'
uci commit firewall
/etc/init.d/firewall restart
```

# Network testing

## Path testing

| Details  | Screenshot  |
| ------------ | ------------ |
|  from home to server |  ![](stuff/i/SCR-20260420-mtnf.png) |
|  from server to iot |  ![](stuff/i/SCR-20260420-mtlm.png) |

<!-- |   |  ![](stuff/i/.png) |
|   |  ![](stuff/i/.png) |
|   |  ![](stuff/i/.png) |
|   |  ![](stuff/i/.png) | -->

<!-- |  Test |  ![](stuff/i/.png) | -->

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

This is fine for a simple network, but it is a security problem: if an IoT device (typically not very secure) is compromised, the attacker gains access to the whole network.

## What is a switch?

A **switch** is the physical "switchboard" of your network: it is the component (often integrated into the router) that receives data from one port and forwards it to the correct destination port. Every device connected via Ethernet cable is connected to a port on the switch.

Without VLANs, a switch sends broadcast traffic (network announcements, DHCP requests, mDNS, etc.) to **all** ports: every device receives everything.

## What is a VLAN?

A **VLAN (Virtual LAN)** is a virtual local network: it lets you logically divide a physical switch into multiple isolated "virtual" switches, without needing separate hardware.

```
Without VLAN:
[PC] [Phone] [IoT Camera] [NAS]  <-- all on the same network, all can see each other

With VLAN:
VLAN home:    [PC] [Phone]
VLAN iot:     [IoT Camera]
VLAN server:  [NAS]
              ^--- isolated from each other, the firewall decides who can talk to whom
```

Each VLAN has an identifier number (VLAN ID) that is "tagged" onto Ethernet packets, so the switch knows which virtual network each packet belongs to.

## What are OpenWRT and LuCI?

**OpenWRT** is an alternative Linux operating system for routers, much more flexible than factory firmware. GL.iNet routers support it natively.

**LuCI** is OpenWRT's graphical web interface: it lets you configure everything from the browser instead of using the command line.

OpenWRT also already includes a fundamental concept: the **bridge**. A bridge is an internal "software switch" inside the router that brings together physical ports, VLANs, and logical interfaces into a single management point. This is why names like `br-lan`, `br-home`, and `br-iot` appear in the guide.

These bridges are often **already preconfigured** because OpenWRT, especially on modern DSA devices, creates a default LAN bridge (`br-lan`) so the LAN ports immediately work as a single local network. This is not an optional detail: it is part of how OpenWRT internally organizes networking.

For this reason, bridges generally **should not be created manually at random**: in most cases you should start from the ones that already exist and modify or associate them correctly with your VLANs. Creating unnecessary or duplicate bridges can lead to inconsistent configurations, loss of connectivity, ports ending up in the wrong bridge, or conflicts with the existing `br-lan`.

## Why must the guide steps be done in this order?

### 1. First configure the physical switch (VLANs on physical devices)

The switch integrated into the router must know which ports belong to which VLAN. This is the lowest level: hardware. Without this step, packets are not routed correctly at the physical level.

> If you skip this step: devices connected by cable are not assigned to the correct VLAN.

### 2. Then create the logical interfaces

OpenWRT must have a **software interface** for each VLAN: it is like telling the operating system "this VLAN exists, I assign it IP address X.X.X.1, and I manage its DHCP from here". Each interface corresponds to a bridge (e.g. `br-home`, `br-iot`) that acts as the router's "entry point" into that VLAN.

In practice, you are not "inventing the router network from scratch": you are correctly connecting the VLANs to the bridges that OpenWRT already uses as its internal structure. That is why this guide first removes the LAN from the default bridge and then reassigns everything in an orderly way, instead of creating bridges manually without a clear reason.

> If you skip this step: the router does not know how to manage traffic for that VLAN, and devices do not receive an IP.

### 3. Then configure the wireless networks

The Wi-Fi networks (SSIDs) must be **associated** with a specific VLAN/interface. Otherwise, all wireless devices end up in the same network regardless of which Wi-Fi they connect to.

> If you skip this step: creating separate SSIDs is useless, because all wireless traffic still goes to the same network.

### 4. Then configure the firewall

By default, once VLANs are created, OpenWRT **completely isolates** them: no zone can communicate with the others. The firewall must be configured explicitly to:

- **Allow internet access (WAN)** from all zones — without this rule, IoT devices have no internet
- **Enable DNS** — DNS is the service that translates `google.com` into an IP address. If it is not explicitly enabled for each zone, websites will not open even if connectivity exists
- **Enable DHCP** — DHCP is the service that automatically assigns an IP address to devices when they connect. Without it, devices remain without an IP
- **Define intra-zone forwardings** — if you want your PC (in `home`) to see the NAS (in `server`), you must explicitly tell the firewall

> If you skip this step: internet will not work on the new VLANs, or some VLANs will not be able to communicate with the ones they should.

> [!CAUTION]
> Firewall rules are read from top to bottom. A specific `ALLOW` rule must be placed **before** a generic `DENY`, otherwise it is ignored and the generic block takes precedence.

### 5. Finally configure Avahi for mDNS

**mDNS (multicast DNS)** is the protocol used by Apple devices (AirPlay, AirPrint), Chromecast, network printers, etc. to "announce themselves" on the local network without needing a central DNS server.

The problem: mDNS uses **multicast** packets that, by definition, do not cross network boundaries. If your Apple TV is in the `iot` VLAN and your Mac is in the `home` VLAN, they cannot see each other.

**Avahi** is a service that acts as a "translator": it listens to mDNS announcements on one VLAN and "reflects" them to the others. It must be configured after the firewall because it needs the interfaces to already exist and the multicast firewall rules to be active.

> If you skip this step: Chromecast, AirPlay, and network printers will not work across different VLANs.

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
|---------|----------------------|
| **LAN** | Your home network |
| **WAN** | The internet (the "external" network) |
| **VLAN** | A logically isolated virtual subnet |
| **Switch** | The physical "switchboard" that routes Ethernet cables |
| **Bridge** | A software component that joins multiple network interfaces |
| **DHCP** | The service that automatically gives devices an IP |
| **DNS** | The service that translates names (`google.com`) into IP addresses |
| **Firewall** | The "gatekeeper" that decides which connections are allowed |
| **Zone (firewall)** | A group of interfaces with the same security rules |
| **Forward** | Allow traffic to pass from one zone to another |
| **mDNS** | Protocol for discovering local devices (Chromecast, AirPlay) |
| **Avahi** | Service that propagates mDNS between different VLANs |
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
