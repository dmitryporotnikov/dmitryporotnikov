---
title: "MikroTik hEX: Route Russian GeoIP via PPPoE, Everything Else via WireGuard, with LTE Failover"
summary: "Split routing on a MikroTik hEX: Russian IPs through your normal ISP, everything else through WireGuard, LTE as backup when PPPoE drops."
date: '2026-05-31T00:00:00+00:00'
draft: false
tags:
  - mikrotik
  - wireguard
  - routing
  - failover
  - lte
---

## The goal

I wanted the hEX to do this:

- Russian IP ranges from `GeoIP-ru` go through the regular PPPoE connection
- Everything else exits via WireGuard
- If PPPoE dies, LTE takes over as internet backup
- All of it should run fine on modest hardware — the hEX isn't a powerhouse

This is RouterOS v7.

## Network layout

```
LAN bridge:      bridge
LAN list:        LAN
Main ISP:         pppoe-out1
WireGuard:        wg1
LTE backup:       lte1
GeoIP list:       GeoIP-ru
```

Routing logic:

```
Destination in GeoIP-ru:
  LAN → main routing table → pppoe-out1

Destination NOT in GeoIP-ru:
  LAN → firewall mangle → via-wg routing table → wg1

If pppoe-out1 goes down:
  main routing table → lte1
```

## Step 1: Interface lists

Check what you already have:

```
/interface list print
/interface list member print
```

Chances are it looks like this:

```
LAN  bridge
WAN  ether1
```

That confirms `in-interface-list=LAN` points to your LAN bridge. Now add your PPPoE interface to the WAN list — it's the real internet facing thing:

```
/interface list member
add list=WAN interface=pppoe-out1
```

And LTE:

```
/interface list member
add list=WAN interface=lte1
```

Both PPPoE and LTE are now recognized as WAN-type interfaces.

## Step 2: Separate routing table for WireGuard

Creating a route for every GeoIP entry is a terrible idea on a router like this. Instead, one routing table handles everything that needs to go through WireGuard.

Create the table:

```
/routing table
add name=via-wg fib
```

Add a default route pointing to WireGuard inside it:

```
/ip route
add dst-address=0.0.0.0/0 gateway=wg1 routing-table=via-wg distance=5 comment="PBR default via WireGuard"
```

Distance can be 1 or 5 — doesn't matter much when it's the only route in that table.

Verify:

```
/ip route print where routing-table=via-wg
```

You want to see:

```
A s 0.0.0.0/0 gateway=wg1 routing-table=via-wg
```

The `A` means active.

## Step 3: NAT for WireGuard

Check your existing NAT rules:

```
/ip firewall nat print
```

If you already have:

```
chain=srcnat action=masquerade out-interface-list=WAN
```

then PPPoE and LTE are both covered — they're in the WAN list.

WireGuard isn't in the WAN list though, so add a separate NAT rule for it:

```
/ip firewall nat
add chain=srcnat out-interface=wg1 action=masquerade comment="NAT via WireGuard"
```

If you don't have a general WAN masquerade rule, add individual ones for each interface:

```
/ip firewall nat
add chain=srcnat out-interface=pppoe-out1 action=masquerade comment="NAT via PPPoE"
add chain=srcnat out-interface=lte1 action=masquerade comment="NAT via LTE backup"
add chain=srcnat out-interface=wg1 action=masquerade comment="NAT via WireGuard"
```

Don't duplicate rules if you already have a working `out-interface-list=WAN` setup.

## Step 4: Bypass address list

We don't want private LAN ranges, multicast, or local addresses getting sent into WireGuard. Create a bypass list:

```
/ip firewall address-list
add list=PBR-BYPASS address=0.0.0.0/8
add list=PBR-BYPASS address=10.0.0.0/8
add list=PBR-BYPASS address=100.64.0.0/10
add list=PBR-BYPASS address=127.0.0.0/8
add list=PBR-BYPASS address=169.254.0.0/16
add list=PBR-BYPASS address=172.16.0.0/12
add list=PBR-BYPASS address=192.168.0.0/16
add list=PBR-BYPASS address=224.0.0.0/4
add list=PBR-BYPASS address=255.255.255.255/32
```

If you know your WireGuard server's public IP, add that too:

```
/ip firewall address-list
add list=PBR-BYPASS address=<WG_SERVER_PUBLIC_IP> comment="WireGuard endpoint must not be routed through itself"
```

Replace `<WG_SERVER_PUBLIC_IP>` with the actual public IP.

## Step 5: Mangle rules for policy routing

This is the core part. We only mark new non-RU connections — keeps the load down.

```
/ip firewall mangle
add chain=prerouting in-interface-list=LAN dst-address-list=PBR-BYPASS action=accept comment="PBR: bypass local/private destinations"

add chain=prerouting in-interface-list=LAN connection-state=new connection-mark=no-mark dst-address-list=!GeoIP-ru dst-address-type=!local action=mark-connection new-connection-mark=conn-via-wg passthrough=yes comment="PBR: mark new non-RU connections"

add chain=prerouting in-interface-list=LAN connection-mark=conn-via-wg action=mark-routing new-routing-mark=via-wg passthrough=no comment="PBR: route non-RU via WireGuard"
```

What each rule does:

- Rule 1: Leave local/private traffic alone
- Rule 2: New connections NOT going to GeoIP-ru get marked as `conn-via-wg`
- Rule 3: Packets from that marked connection get the `via-wg` routing mark

RouterOS then sends those packets through the `via-wg` table, which points out WireGuard.

## Step 6: Fix FastTrack

FastTrack can skip mangle processing entirely. If that happens, your policy routing breaks.

We want FastTrack only for normal unmarked traffic.

Check your FastTrack rule:

```
/ip firewall filter print detail where action=fasttrack-connection
```

Update it to exclude marked connections:

```
/ip firewall filter
set [find where action=fasttrack-connection] connection-mark=no-mark
```

Now `conn-via-wg` traffic won't be FastTracked.

Clear existing marked connections so they reconnect properly:

```
/ip firewall connection
remove [find connection-mark=conn-via-wg]
```

This briefly interrupts those connections — they reconnect with the correct routing.

## Step 7: LTE backup

PPPoE already injects the primary default route with distance=1. LTE needs to add a backup route with higher distance.

Check your LTE APN settings:

```
/interface lte apn print detail
```

Configure LTE to add a backup default route:

```
/interface lte apn
set [find default=yes] add-default-route=yes default-route-distance=10
```

Now your routes look like:

```
pppoe-out1 distance=1  = primary
lte1 distance=10       = backup
```

Verify:

```
/ip route print where dst-address=0.0.0.0/0
```

Should show:

```
0.0.0.0/0 via pppoe-out1 distance=1
0.0.0.0/0 via lte1 distance=10
```

PPPoE takes priority when it's up. When it drops, LTE kicks in.

## Step 8: Check WireGuard peer

Full tunnel WireGuard routing requires the peer to allow 0.0.0.0/0:

```
/interface wireguard peers print detail
```

If `allowed-address=0.0.0.0/0` isn't set, traffic might get routed to wg1 but never actually enter the tunnel.

## Step 9: Testing

### WireGuard routing table

```
/ip route print where routing-table=via-wg
```

Should show:

```
A s 0.0.0.0/0 gateway=wg1 routing-table=via-wg
```

### Mangle counters

```
/ip firewall mangle print stats
```

Open a non-RU website. These rules should be incrementing:

```
PBR: mark new non-RU connections
PBR: route non-RU via WireGuard
```

### Marked connections

```
/ip firewall connection print where connection-mark=conn-via-wg
```

Good connections show `SACs`. If you see `SACFs`, FastTrack is still interfering.

The `F` flag means FastTrack has it — go back to Step 6.

### WireGuard traffic

```
/interface monitor-traffic wg1 once
```

Open a non-RU site from a LAN device. You should see traffic on wg1.

### Public IP check

Hit a non-RU IP check site from a LAN device. You should see your WireGuard IP.

Then hit a RU-hosted site that's in your GeoIP-ru list. You should see your normal PPPoE IP.

## Step 10: LTE failover test

Temporarily disable PPPoE:

```
/interface pppoe-client disable pppoe-out1
```

Check default routes:

```
/ip route print where dst-address=0.0.0.0/0
```

LTE should be active now. Try pinging:

```
/ping 1.1.1.1
```

Re-enable PPPoE:

```
/interface pppoe-client enable pppoe-out1
```

Routes should go back to PPPoE as primary.

## How it works (plain version)

Every new LAN connection hits the router. If the destination is in GeoIP-ru, the router does nothing special — the traffic follows the main route out through PPPoE.

If the destination isn't in GeoIP-ru, the router marks the connection and shoves it into the `via-wg` routing table. That table has a default route via wg1, so the traffic goes through WireGuard.

LTE sits in the main table as a backup with higher distance. When pppoe-out1 fails, the main table falls back to lte1 automatically.

## Troubleshooting commands

```
/interface list print
/interface list member print

/ip route print where dst-address=0.0.0.0/0
/ip route print where routing-table=via-wg

/ip firewall mangle print stats
/ip firewall connection print where connection-mark=conn-via-wg

/ip firewall filter print detail where action=fasttrack-connection
/ip firewall nat print

/interface wireguard print
/interface wireguard peers print detail
/interface monitor-traffic wg1 once

/interface lte print
/interface lte apn print detail
/interface monitor-traffic lte1 once
```

## Result

```
RU traffic:       LAN → pppoe-out1
Non-RU traffic:   LAN → wg1
PPPoE down:       main table → lte1
```
