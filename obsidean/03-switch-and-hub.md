---
tags: [networking, deep-in-net, layer2, switch, hub]
---

# Layer 2 Devices — Switch & Hub

> [!important] Core Concept
> A hub is "dumb" — it repeats everything to everyone. A switch is "smart" — it learns which device sits behind which port and sends data only where it needs to go. This one difference is why switches replaced hubs everywhere except in legacy/teaching scenarios like this project's Exercise 2.

## Hub — OSI Layer 1 (Physical)

A hub is a simple multi-port repeater. It has no intelligence:

- When a signal arrives on one port, the hub **broadcasts** it out to *every other port*, regardless of the intended destination.
- It operates purely at the physical layer — it doesn't read MAC addresses, IP addresses, or anything else. It just repeats electrical signals.
- All devices connected to a hub share **one collision domain** — if two devices transmit at the same time, their signals collide and both have to retransmit (using CSMA/CD, Carrier Sense Multiple Access with Collision Detection).
- This makes hubs inefficient and insecure (every device sees every other device's traffic) — which is exactly why they're obsolete in real networks today.

## Switch — OSI Layer 2 (Data Link)

A switch is an intelligent multi-port bridge:

- It builds and maintains a **MAC address table** (CAM table) — a mapping of which MAC address is reachable through which physical port.
- When a frame arrives, the switch looks at the destination MAC address and forwards the frame **only out the port** where that device lives — not to everyone.
- If the destination MAC isn't yet known (table doesn't have it), the switch **floods** the frame to all ports once, then learns the reply's source port and adds it to the table for next time.
- Each switch port is its own **collision domain**, meaning no waiting/colliding between devices at all (this is a big reason for the throughput improvement over hubs).
- All ports on a switch (with no VLANs configured) are still one **broadcast domain** — broadcast traffic still reaches everyone, but device-to-device unicast traffic does not.

## Switch vs Hub — Side by Side

| | Hub | Switch |
|---|---|---|
| OSI Layer | 1 (Physical) | 2 (Data Link) |
| Intelligence | None — pure repeater | Learns MAC addresses, forwards intelligently |
| Traffic handling | Broadcasts to all ports | Sends only to the destination port |
| Collision domains | 1 shared domain for all ports | 1 domain per port |
| Broadcast domain | 1 shared | 1 shared (unless VLANs used) |
| Security | Poor — everyone sees everyone's traffic | Better — traffic only reaches intended device |
| Speed/efficiency | Poor at scale, more collisions | Much better, minimal collisions |
| Real-world use today | Essentially obsolete | Standard in every LAN |

## Why Exercise 2 uses both

The exercise deliberately puts a switch group and a hub group side by side so you can directly observe the behavior difference in Packet Tracer's **Simulation mode** — watch how a ping travels: through the hub it lights up every cable, through the switch it lights up only the path to the destination (after the first flood/learn).

> [!tip] Practical test in Packet Tracer
> Use Simulation Mode and send a single PDU (ping) between two devices on the hub side, then the same on the switch side. Compare how many links "light up" carrying the packet. This visually confirms flooding vs targeted forwarding.

---

## Related notes
- [[01-networking-foundations]]
- [[02-physical-layer-cabling]]
- [[04-router-and-default-gateway]]
