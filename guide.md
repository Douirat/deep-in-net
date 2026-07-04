# deep-in-net — Study Guide

Study order follows the logical dependency chain of the project, not just the exercise order.
Some topics (like subnetting) need to be learned *before* the exercise that needs them, since
Ex5–8 all depend on it.

Check items off as you study them.

---

## 1. Networking Foundations (do this first — everything else builds on it)

- [ ] What is a network? (LAN, WAN, node, host)
- [ ] The OSI model — all 7 layers, what each one does
  - [ ] Layer 1 – Physical
  - [ ] Layer 2 – Data Link
  - [ ] Layer 3 – Network
  - [ ] Layer 4 – Transport
  - [ ] Layer 5 – Session
  - [ ] Layer 6 – Presentation
  - [ ] Layer 7 – Application
- [ ] The TCP/IP model and how it maps onto OSI (4 layers vs 7)
- [ ] MAC address vs IP address — what each identifies, and at which layer

**Why first:** Every device and protocol in this project gets identified by "what OSI layer does it operate at" — you can't answer any Knowledge question without this.

---

## 2. Physical Layer & Cabling (needed for Ex1)

- [ ] What is an RJ-45 connector/cable
- [ ] Straight-through cable — when it's used (unlike devices, e.g. PC↔Switch)
- [ ] Crossover cable — when it's used (like devices, e.g. PC↔PC, Switch↔Switch)
- [ ] Auto-MDIX (why modern equipment often doesn't care which cable you use)

---

## 3. Layer 2 Devices — Switch & Hub (needed for Ex2)

- [ ] What a hub does (broadcast to all ports, no intelligence) — OSI Layer 1
- [ ] What a switch does (MAC address table, forwards only to the right port) — OSI Layer 2
- [ ] Collision domains vs broadcast domains (hub = one collision domain, switch = separates them)
- [ ] Why switches scale better than hubs

---

## 4. Layer 3 Devices — Router (needed for Ex4)

- [ ] What a router does — OSI Layer 3
- [ ] Router vs switch (which one connects different networks vs same network)
- [ ] What a "default gateway" is and why every host needs one
- [ ] How a router decides where to send a packet (routing table, intro-level for now)

---

## 5. IP Addressing & Subnetting (critical — needed for Ex5, 6, 7, 8, and the audit)

This is the section to spend the most time on. **No calculators allowed in the audit.**

- [ ] IPv4 address structure (32 bits, 4 octets)
- [ ] Network portion vs host portion of an address
- [ ] Subnet mask — what it does, in decimal and binary (e.g. 255.255.255.0)
- [ ] CIDR notation (/24, /25, /26...) and converting CIDR ↔ decimal mask by hand
- [ ] How to calculate, for a given IP + mask, by hand:
  - [ ] Network address
  - [ ] Broadcast address
  - [ ] First usable host / last usable host
  - [ ] Number of usable hosts
- [ ] How to subnet a network into N smaller subnets (borrowing bits)
- [ ] Private IP ranges (10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16)
- [ ] Practice: do at least 5–10 subnetting problems fully by hand, no tools, until it's fast

**Why now, before Ex3:** Ex3 already requires you to respect "defined netmask and IP address," so you need to be comfortable reading/assigning subnets before you even get there.

---

## 6. Routing Tables (needed for Ex6, and useful for Ex5/7/8)

- [ ] What a routing table contains (destination network, mask, next hop, interface)
- [ ] How a router picks which route to use (longest prefix match, basic idea)
- [ ] Static routes — how to configure one in Packet Tracer via CLI
- [ ] How two separate subnets reach each other through a router (both directions)

---

## 7. Core Network Services & Protocols (needed for Ex3)

- [ ] What a "server" is and its general purpose
- [ ] **TCP vs UDP**
  - [ ] Connection-oriented vs connectionless
  - [ ] Which OSI layer they operate at (Layer 4)
  - [ ] Examples of protocols that use each
- [ ] **Ports**
  - [ ] What a port is and why it's needed alongside an IP address
  - [ ] Well-known ports for the protocols below
- [ ] **DHCP**
  - [ ] What it does (automatic IP assignment)
  - [ ] DORA process (Discover, Offer, Request, Acknowledge)
  - [ ] Port used
- [ ] **DNS**
  - [ ] What it does (name → IP resolution)
  - [ ] Record types: A, CNAME, MX, NS, PTR — know at least A and CNAME well (you'll use both)
  - [ ] Port used
- [ ] **HTTP**
  - [ ] What it does, port used, OSI layer
- [ ] **HTTPS**
  - [ ] How it differs from HTTP (TLS/SSL encryption)
  - [ ] Port used
- [ ] **FTP**
  - [ ] What it does, port(s) used
  - [ ] User permissions model (Read, Write, Delete, rename, List — this is your "RWDNL")

---

## 8. Linux Networking Commands (used to test/troubleshoot everything above)

- [ ] `ping` — test reachability
- [ ] `ipconfig` / `ifconfig` (or Packet Tracer's PC CLI equivalent) — view IP config
- [ ] `tracert` / `traceroute` — see the path packets take
- [ ] `nslookup` — test DNS resolution
- [ ] `arp -a` — view MAC-to-IP mappings
- [ ] `netstat` — view active connections/ports (good for verifying which services are open/closed, e.g. confirming HTTP is disabled in Ex3)

---

## 9. Cisco Packet Tracer Practical Skills (apply everything above)

- [ ] Placing and cabling devices (PCs, switches, hubs, routers, servers)
- [ ] Configuring static IP on a server/router interface
- [ ] Configuring DHCP pool on a router or DHCP server
- [ ] Using the router CLI (basic IOS commands: `enable`, `configure terminal`, `interface`, `ip address`, `no shutdown`, `ip route`)
- [ ] Configuring services on a Server device (DHCP, DNS, HTTP, FTP tabs)
- [ ] Testing connectivity in Simulation mode / with CLI commands from PC

---

## Suggested Study → Exercise Mapping

| Study sections done | You're ready for |
|---|---|
| 1, 2 | Ex1 |
| 1, 3 | Ex2 |
| 1, 5, 7, 9 | Ex3 |
| 1, 4 | Ex4 |
| 1, 4, 5, 9 | Ex5 |
| 1, 4, 5, 6, 9 | Ex6 |
| 1, 4, 5, 6, 9 | Ex7 |
| 1, 4, 5, 6, 9 | Ex8 |

---

## Next Steps
Once this study pass is done, I can put together a **per-exercise implementation checklist** (with the exact IPs/subnets you decide on, device configs, and test commands) to track as you actually build the `.pkt` files.