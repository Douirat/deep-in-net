---
tags: [networking, deep-in-net, layer3, router, gateway]
---

# Layer 3 Devices — Router & Default Gateway

> [!important] Core Concept
> Switches connect devices *within* a network. Routers connect *different* networks together. A router's whole job is to look at a packet's destination IP, decide which network it belongs to, and forward it one hop closer — over and over, across potentially many routers, until it arrives.

## What is a router?

A router is a device that forwards data packets between different networks (different subnets), operating at **OSI Layer 3 (Network)**.

- Each router interface sits on a *different* subnet, and typically holds the **first usable IP address** of that subnet (a common convention, not a strict rule).
- Routers read the destination **IP address** of a packet, consult their **routing table**, and forward the packet out the correct interface toward its destination network.
- Unlike switches (which forward based on learned MAC addresses within one network), routers make forwarding decisions based on network addresses, which lets them connect completely separate networks (even with different media, like an Ethernet LAN talking to a fiber WAN link).

## Router vs Switch

| | Switch | Router |
|---|---|---|
| OSI Layer | 2 | 3 |
| Connects | Devices within the *same* network | Multiple *different* networks |
| Forwarding decision based on | MAC address | IP address |
| Broadcast traffic | Passes broadcasts through | Blocks broadcasts by default (each interface is its own broadcast domain) |
| Typical project use | Ex2, connecting PCs to a LAN segment | Ex4+, connecting separate subnets |

## Default Gateway

Every host on a network needs a **default gateway** — the IP address of the router interface it sends traffic to whenever the destination is *outside* its own local subnet.

**How a PC decides whether to use the gateway:**
1. PC wants to send a packet to some destination IP.
2. PC compares the destination IP against its own subnet mask.
3. If the destination is on the *same* subnet → PC sends directly (via ARP + MAC address, no router needed).
4. If the destination is on a *different* subnet → PC sends the packet to its configured default gateway, and lets the router handle it from there.

**Common misconfiguration to watch for:** if a PC's default gateway is left blank or set to the wrong IP, it will be able to talk to devices on its own subnet just fine, but any attempt to reach a different subnet will silently fail — this is one of the most common "why can't PC0 reach PC1 across the router" bugs in Packet Tracer.

## How a router makes forwarding decisions (intro-level)

For every incoming packet, the router checks its **routing table** for the destination network:
- If a matching route exists → forward out the associated interface/next hop.
- If no match exists and there's no default route → the packet is dropped, and the sender may get a "Destination unreachable" message.

This is the foundational concept for Exercise 6, where you'll actually build and inspect routing tables — see [[06-routing-tables]] for detail.

---

## Related notes
- [[01-networking-foundations]]
- [[03-switch-and-hub]]
- [[05-ip-addressing-and-subnetting]]
- [[06-routing-tables]]
