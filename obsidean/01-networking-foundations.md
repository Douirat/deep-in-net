---
tags: [networking, deep-in-net, osi-model, tcp-ip, fundamentals]
---

# Networking Foundations

> [!important] Core Concept
> A network is just a set of rules (protocols) that let independent devices talk to each other reliably. The **OSI model** is the map of those rules — it breaks "communication" into 7 layers so that each layer only has to worry about one job. Every device and protocol you touch in this project can be pinned to one of these layers, and that's the lens the audit will test you through.

## What is a network?

A network is a group of two or more devices (hosts) connected so they can exchange data. Key vocabulary:

- **Host / node** — any device with an address on the network (PC, server, printer, phone).
- **LAN (Local Area Network)** — devices in one physical/geographical area (a building, a home).
- **WAN (Wide Area Network)** — networks connected across large distances (the internet is the biggest WAN).
- **Topology** — the physical or logical layout of how devices connect (star, bus, mesh, etc.). Packet Tracer exercises are mostly star topologies (devices connecting into a central switch/hub).

## The OSI Model — 7 Layers

OSI = **Open Systems Interconnection**. It's a conceptual (not literally implemented) model that splits network communication into 7 layers, each layer only talking to the layer directly above or below it.

Mnemonic (bottom to top): **P**lease **D**o **N**ot **T**hrow **S**ausage **P**izza **A**way

| # | Layer | Job | Real-world analogy | Examples in this project |
|---|-------|-----|---------------------|---------------------------|
| 1 | **Physical** | Raw bit transmission over a medium (cables, radio, voltage/light signals) | The road itself | RJ-45 cables, hubs |
| 2 | **Data Link** | Node-to-node delivery on the same local network, uses MAC addresses, error detection | Addressing a local street | Switches, MAC addresses |
| 3 | **Network** | Routing data between different networks, uses IP addresses | The postal system routing between cities | Routers, IP addresses, subnetting |
| 4 | **Transport** | End-to-end delivery, reliability, ordering (TCP), or fast/no guarantee (UDP) | Registered mail (TCP) vs postcard (UDP) | TCP, UDP, ports |
| 5 | **Session** | Establishes/manages/terminates sessions between apps | Keeping a phone call connected | Login sessions |
| 6 | **Presentation** | Data translation, encryption, compression | Translating a language | TLS/SSL encryption (part of HTTPS) |
| 7 | **Application** | The actual application-level protocol the user interacts with | The letter's content | HTTP, HTTPS, FTP, DNS, DHCP |

### Why this matters for the project
Nearly every "Knowledge" bullet in the exercises asks you to identify the OSI layer a device or protocol works at. If you can place a device/protocol on this table from memory, you can answer almost all of them.

- Hub → Layer 1
- Switch → Layer 2
- Router → Layer 3
- TCP/UDP → Layer 4
- HTTP/HTTPS/FTP/DNS/DHCP → Layer 7

## The TCP/IP Model

The TCP/IP model is the practical, 4-layer model that the real internet actually runs on. OSI is more of a teaching/reference tool; TCP/IP is what's implemented.

| TCP/IP Layer | Maps to OSI layers |
|---|---|
| Application | 5, 6, 7 |
| Transport | 4 |
| Internet | 3 |
| Network Access (Link) | 1, 2 |

You'll see both models referenced — OSI for granular layer questions, TCP/IP when people talk about "the internet protocol suite" more loosely.

## MAC Address vs IP Address

These are the two addressing systems you'll use constantly, and they operate at different layers:

| | MAC Address | IP Address |
|---|---|---|
| OSI Layer | 2 (Data Link) | 3 (Network) |
| Format | 6 bytes, hex (e.g. `00:1A:2B:3C:4D:5E`) | 4 bytes, decimal (IPv4) e.g. `192.168.1.10` |
| Assigned by | Manufacturer (burned into the NIC) | Network administrator / DHCP |
| Scope | Unique per physical network interface, doesn't change | Can change depending on which network you're connected to |
| Used for | Local delivery within the same network segment | Routing between different networks |

**Why both exist:** IP gets a packet to the right *network* (like getting a letter to the right city). MAC gets it to the right *device* on that final local network (like the mail carrier finding the right house). A router strips/rewrites Layer 2 info at every hop, but the Layer 3 IP destination stays the same until the packet arrives.

---

## Related notes
- [[02-physical-layer-cabling]]
- [[03-switch-and-hub]]
- [[04-router-and-default-gateway]]
- [[05-ip-addressing-and-subnetting]]
