---
tags: [networking, deep-in-net, packet-tracer, practical, cli]
---

# Cisco Packet Tracer — Practical Skills

> [!important] Core Concept
> Packet Tracer is where every concept above gets *applied*, not just understood. The audit explicitly checks whether you can build a topology live and configure it via CLI — so the goal isn't to memorize this note, it's to have done each of these actions with your own hands enough times that it's muscle memory.

## Placing and Cabling Devices

- Devices live in the bottom-left category bar: End Devices (PCs, servers), Switches, Hubs, Routers, Connections (cables).
- **Connections** submenu has the cable types — Packet Tracer can auto-pick the right cable type if you select "Automatic," but for learning purposes, manually select straight-through/crossover per [[02-physical-layer-cabling]] so you actually internalize when each is used.
- A cable end shows:
  - Green dot = link up (both interfaces active and compatible)
  - Red dot = link down (wrong cable type, interface administratively down, or duplex/speed mismatch)
  - Amber/blinking = negotiating (normal briefly after connecting)

## Configuring Static IP

**On a PC/Server (GUI):** Desktop tab → IP Configuration → select "Static," enter IP address, subnet mask, default gateway.

**On a Router interface (CLI):**
```
Router> enable
Router# configure terminal
Router(config)# interface gigabitEthernet 0/0
Router(config-if)# ip address 192.168.1.1 255.255.255.0
Router(config-if)# no shutdown
```

`no shutdown` is easy to forget and is one of the most common reasons an interface with a correct IP still shows down — router interfaces are administratively shut down by default.

## Configuring DHCP

**Option A — Server device running the DHCP service:**
Server → Services tab → DHCP → turn Service ON → set:
- Default Gateway
- DNS Server
- Starting IP address
- Subnet Mask
- Max number of users

**Option B — Router acting as DHCP server (CLI):**
```
Router(config)# ip dhcp pool LAN1
Router(dhcp-config)# network 192.168.1.0 255.255.255.0
Router(dhcp-config)# default-router 192.168.1.1
Router(dhcp-config)# dns-server 192.168.1.99
Router(dhcp-config)# exit
Router(config)# ip dhcp excluded-address 192.168.1.1 192.168.1.10
```
(The excluded-address line reserves the low range for static devices like the router interface and servers, so DHCP doesn't hand out an address you're already using.)

**On the client PC:** Desktop → IP Configuration → select "DHCP" (instead of Static) and confirm it receives a lease matching your pool.

## Configuring Router CLI — Core Commands Reference

| Command | Mode | Purpose |
|---|---|---|
| `enable` | User exec → Privileged exec | Enter privileged mode |
| `configure terminal` | Privileged exec → Global config | Enter global configuration |
| `interface <type> <number>` | Global config → Interface config | Select an interface to configure |
| `ip address <ip> <mask>` | Interface config | Assign static IP to interface |
| `no shutdown` | Interface config | Activate the interface |
| `ip route <network> <mask> <next-hop>` | Global config | Add a static route |
| `show ip route` | Privileged exec | View the routing table |
| `show ip interface brief` | Privileged exec | Quick view of all interfaces, their IPs, and up/down status |
| `show running-config` | Privileged exec | View full current device configuration |

## Configuring Services on a Server Device

The Server device in Packet Tracer has a **Services** tab with toggles for each service (HTTP, DHCP, DNS, FTP, etc.):

- **DNS tab:** add resource records — choose type (A Record, CNAME Record), enter name and matching value, click **Add**.
- **HTTP tab:** toggle HTTP and HTTPS independently — this is exactly where Exercise 3's "HTTPS on, HTTP off" requirement is set. You can also edit the actual HTML page served (the "hello message").
- **FTP tab:** add a new user, set username/password, and check the specific permission boxes (Read/Write/Delete/Rename/List = RWDNL).

## Testing Connectivity

**Realtime mode vs Simulation mode** (toggle bottom-right of Packet Tracer):
- **Realtime mode** — network behaves as if time is passing normally; good for general connectivity testing (pings, browsing, CLI commands).
- **Simulation mode** — freezes time and lets you step through a single packet's journey hop-by-hop, showing exactly which layer/protocol is active at each step. This is invaluable for *understanding* (and demonstrating in an audit) exactly how a packet travels through switches/routers, not just confirming that it arrived.

**From a PC's command line, useful test commands:**
```
ping <ip-or-hostname>
ipconfig
tracert <ip>
nslookup <hostname>
```

**From a browser (on a PC's Desktop tab):**
Navigate to `https://deep-in-net.com` to verify the full DNS → redirect → HTTPS chain from Exercise 3 actually works end-to-end, not just each piece in isolation.

---

## Related notes
- [[02-physical-layer-cabling]]
- [[06-routing-tables]]
- [[07-services-and-protocols]]
- [[08-linux-networking-commands]]
