---
title: 'Harden a Fresh MikroTik Router'
summary: 'Step-by-step guide to secure a MikroTik router after Quick Set configuration, covering firewall rules, OSPF, DNS, and management access restrictions.'
date: '2026-07-13T00:00:00+00:00'
---

# Harden a Fresh MikroTik Router

Before starting, make sure Quick Set has already configured the WAN, LAN bridge, DHCP, and NAT. This guide assumes:

- `ether1` is the WAN interface
- `bridge1` is the trusted LAN bridge
- The router may be exposed through an upstream router's DMZ or have a direct public IP
- (optional, in my setup) OSPF is used to reach additional trusted networks

## 1. Verify interface roles

```routeros
/interface list print
/interface list member print
/interface bridge port print
```

You want to see:

```text
WAN -> ether1
LAN -> bridge1
```

`ether1` must not be a member of `bridge1`. If it is, fix that first:

```routeros
/interface list
add name=WAN
add name=LAN

/interface list member
add list=WAN interface=ether1
add list=LAN interface=bridge1
```

## 2. Define trusted networks

Add whatever networks you need to trust:

```routeros
/ip firewall address-list
add list=TRUSTED-LANS address=192.168.1.0/24 comment="Main LAN"
add list=TRUSTED-LANS address=192.168.2.0/24 comment="OSPF LAN 2"
add list=TRUSTED-LANS address=192.168.3.0/24 comment="OSPF LAN 3"
```

## 3. Install the IPv4 firewall

Apply rules in Safe Mode where possible.

```routeros
/ip firewall filter

add chain=input action=accept \
    connection-state=established,related,untracked \
    comment="INPUT 01 - established related"

add chain=input action=drop \
    connection-state=invalid \
    comment="INPUT 02 - drop invalid"

add chain=input action=accept \
    protocol=icmp \
    comment="INPUT 03 - allow ICMP"

add chain=input action=accept \
    protocol=ospf \
    src-address=192.168.1.252 \
    in-interface=bridge1 \
    comment="INPUT 04 - OSPF peer 192.168.1.252"

add chain=input action=accept \
    protocol=ospf \
    src-address=192.168.1.253 \
    in-interface=bridge1 \
    comment="INPUT 05 - OSPF peer 192.168.1.253"

add chain=input action=accept \
    src-address-list=TRUSTED-LANS \
    in-interface-list=LAN \
    comment="INPUT 06 - trusted networks to router"

add chain=input action=drop \
    in-interface-list=WAN \
    comment="INPUT 07 - drop WAN to router"

add chain=input action=drop \
    comment="INPUT 99 - drop everything else"

add chain=forward action=accept \
    connection-state=established,related,untracked \
    comment="FORWARD 01 - established related"

add chain=forward action=drop \
    connection-state=invalid \
    comment="FORWARD 02 - drop invalid"

add chain=forward action=accept \
    src-address-list=TRUSTED-LANS \
    dst-address-list=TRUSTED-LANS \
    comment="FORWARD 03 - routing between trusted LANs"

add chain=forward action=accept \
    src-address-list=TRUSTED-LANS \
    out-interface-list=WAN \
    comment="FORWARD 04 - trusted LANs to Internet"

add chain=forward action=accept \
    in-interface-list=WAN \
    connection-nat-state=dstnat \
    comment="FORWARD 05 - allow explicit port forwards"

add chain=forward action=drop \
    in-interface-list=WAN \
    comment="FORWARD 06 - drop unsolicited WAN"

add chain=forward action=drop \
    comment="FORWARD 99 - drop everything else"
```

## 4. OSPF notes

If your OSPF peers live on `bridge1`, allow protocol 89 only from the known peer addresses, as shown in the firewall rules above.

Check neighbor status:

```routeros
/routing ospf neighbor print
/ip route print where ospf
```

You should see learned routes like:

```text
192.168.2.0/24 via 192.168.1.253
192.168.3.0/24 via 192.168.1.252
```

The trusted-network forward rule is what lets traffic flow between your main LAN and those OSPF-learned networks.

## 5. Verify NAT

Quick Set normally creates the right masquerade rule, but verify it:

```routeros
/ip firewall nat print
```

You want to see:

```routeros
chain=srcnat action=masquerade out-interface-list=WAN
```

If it's missing, add it:

```routeros
/ip firewall nat
add chain=srcnat action=masquerade \
    out-interface-list=WAN \
    comment="Masquerade internal networks"
```

Don't add destination-NAT rules here unless you are intentionally exposing a service.

## 6. Restrict management services

Turn off everything you do not need:

```routeros
/ip service
disable ftp
disable telnet
disable www-ssl
disable reverse-proxy
disable api
disable api-ssl
```

Now lock down the services you're keeping. Restrict SSH, www, and WinBox to your trusted networks:

```routeros
/ip service
set ssh address=192.168.1.0/24,192.168.2.0/24,192.168.3.0/24
set www address=192.168.1.0/24,192.168.2.0/24,192.168.3.0/24
set winbox address=192.168.1.0/24,192.168.2.0/24,192.168.3.0/24
```

If you do not need WebFig HTTP, disable it:

```routeros
/ip service disable www
```

## 7. Restrict Layer 2 management

```routeros
/tool mac-server
set allowed-interface-list=LAN

/tool mac-server mac-winbox
set allowed-interface-list=LAN

/tool mac-server ping
set enabled=no

/ip neighbor discovery-settings
set discover-interface-list=LAN

/tool bandwidth-server
set enabled=no
```

## 8. DNS

If your LAN devices point to the MikroTik for DNS:

```routeros
/ip dns
set servers=1.1.1.1,1.0.0.1 allow-remote-requests=yes
```

The firewall rules already block WAN access to the router, so the DNS resolver stays internal.

## 9. Remove temporary debug logging (my unique quirk, you may not have those)

Check what logging rules exist:

```routeros
/system logging print detail
```

Remove any temporary PPP/PPPoE debug rules if you see them:

```routeros
/system logging remove [find where topics="pppoe,debug"]
/system logging remove [find where topics="ppp,debug"]
```

Keep the defaults: `info`, `error`, `warning`, and `critical`.

## 10. Verify before enabling upstream DMZ

Run these before you expose the router:

```routeros
/ip firewall filter print detail without-paging
/ip firewall filter print stats
/ip firewall nat print
/ip service print
/interface list member print
/routing ospf neighbor print
/ip route print where ospf
```

After enabling the upstream DMZ, watch the WAN drop counter:

```routeros
/ip firewall filter print stats \
    where comment="INPUT 07 - drop WAN to router"
```

That counter should climb. Good — the MikroTik is dropping unsolicited traffic from the WAN.

## 11. Save the secured configuration

```routeros
/export hide-sensitive file=router-secured
/system backup save name=router-secured
```

For remote administration, use a VPN (WireGuard works well) instead of exposing WinBox, SSH, or WebFig directly.
