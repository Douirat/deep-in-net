---
tags: [networking, deep-in-net, physical-layer, cabling]
---

# Physical Layer & Cabling (RJ-45)

> [!important] Core Concept
> The physical layer is the actual wire carrying electrical signals between devices. The **wiring pattern inside the cable's connector** determines whether two devices can talk — get it wrong and there's no link light at all, regardless of how correctly everything above Layer 1 is configured.

## What is RJ-45?

RJ-45 (**Registered Jack 45**) is the standard connector used for Ethernet cabling — the clear plastic 8-pin plug at the end of a network cable. Inside, it carries 8 individual copper wires (4 twisted pairs), each pair reducing electromagnetic interference through the twisting.

- Physically resembles a slightly larger phone jack (RJ-11).
- Terminates into two common wiring standards: **T568A** and **T568B** (differ only in the order 2 pairs are wired — this is what creates straight-through vs crossover cables).

## Straight-Through vs Crossover Cables

The difference is entirely in how the 8 wires are ordered at each end of the cable.

### Straight-through cable
- Both ends wired identically (both T568A, or both T568B).
- Used to connect **dissimilar devices** — i.e. devices at different layers of typical hierarchy:
  - PC → Switch
  - PC → Hub
  - Router → Switch
  - Switch → Router

### Crossover cable
- One end T568A, the other T568B — the transmit and receive pairs are swapped.
- Used to connect **similar devices** — devices of the same type talking directly:
  - PC → PC
  - Switch → Switch
  - Router → Router
  - Hub → Hub

### Why does it matter?
Each device has a fixed pair of pins it *transmits* on and a fixed pair it *listens* on. If you connect two PCs with a straight-through cable, both devices try to transmit and listen on the same pins — no communication happens. The crossover cable swaps the wires physically so that Device A's "transmit" lines up with Device B's "receive," and vice versa.

| Connection | Cable type |
|---|---|
| PC ↔ PC | Crossover |
| PC ↔ Switch | Straight-through |
| PC ↔ Hub | Straight-through |
| Switch ↔ Switch | Crossover |
| Switch ↔ Router | Straight-through |
| Router ↔ Router | Crossover |

## Auto-MDIX

Modern network equipment (most switches and NICs made in the last ~15 years) includes **Auto-MDIX** (Automatic Medium-Dependent Interface Crossover), which automatically detects the cable and pinout and adjusts internally. This means in real hardware, straight-through and crossover often work interchangeably.

> [!note] Relevance to Packet Tracer
> Packet Tracer still models older-style behavior and may enforce the correct cable type for a link to come up (shown as a green dot vs red dot on the connection). If a link won't come up in Exercise 1, check the cable type before troubleshooting anything else — this is a Layer 1 problem, not a configuration problem, and no amount of IP/DNS/DHCP setup will fix it.

---

## Related notes
- [[01-networking-foundations]]
- [[03-switch-and-hub]]
