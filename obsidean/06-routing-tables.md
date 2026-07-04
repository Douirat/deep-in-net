---
tags: [networking, deep-in-net, layer3, routing-table, static-routes]
---

# Routing Tables

> [!important] Core Concept
> A routing table is a router's "map" — a list of known destination networks and which direction (next hop / exit interface) to send packets toward that destination. No entry, no delivery: if a router doesn't have a route to a destination network, it drops the packet. Everything about multi-subnet communication (Ex5–8) comes down to making sure every router involved has a complete map.

## What's in a routing table

Each entry (route) typically contains:

| Field | Meaning |
|---|---|
| **Destination network** | The network address this route applies to (e.g. `192.168.2.0`) |
| **Subnet mask** | Defines the size/boundary of that destination network |
| **Next hop** | The IP address of the next router to forward toward (or "directly connected" if it's a local interface) |
| **Exit interface** | Which physical interface on this router to send the packet out of |
| **Metric / Administrative Distance** | Used to choose between multiple possible routes to the same destination (lower is generally preferred) — more relevant once you have multiple routers/paths |

## How a router picks a route: Longest Prefix Match

If a router has multiple entries that could match a destination IP, it always chooses the **most specific** match — the one with the longest subnet mask (highest CIDR number).

Example: a router has both `192.168.0.0/16` and `192.168.1.0/24` in its table. A packet destined for `192.168.1.50` matches both, but the router uses the `/24` route because it's more specific.

## Types of routes

- **Directly connected routes** — automatically added for any subnet the router has an interface physically configured on. You don't type these in; they appear the moment you set an IP on an interface and bring it up (`no shutdown`).
- **Static routes** — manually configured by an administrator, telling the router "to reach network X, send packets to next-hop Y." This is what Exercise 6 has you configure.
- **Default route** — a catch-all static route (`0.0.0.0/0`) used when no more specific route matches — "if you don't know where else to send it, send it here." Not required for this project's small topologies but good to know conceptually.

## Configuring a static route (Packet Tracer / Cisco IOS)

Basic syntax, run from global configuration mode on the router:

```
Router(config)# ip route <destination-network> <subnet-mask> <next-hop-ip>
```

Example: a router needs to reach the `192.168.2.0/24` network, which sits behind another router at `10.0.0.2`:

```
Router(config)# ip route 192.168.2.0 255.255.255.0 10.0.0.2
```

## Why both directions matter (the recurring project requirement)

Every exercise from 5 onward requires "subnet 1 can reach subnet 2 **and** subnet 2 can reach subnet 1." This is not automatic — routing is not inherently bidirectional. If Router A knows how to reach Subnet 2 but Router B (or Router A itself, depending on topology) doesn't have a route back to Subnet 1, replies never make it back and the connection will look "half broken" (e.g. ping shows "Request timed out" even though the request technically arrived).

**Checklist for any subnet-to-subnet requirement:**
- [ ] Does the router have a directly-connected or static route to the destination subnet?
- [ ] Does the router have a route (or is directly connected) back to the source subnet?
- [ ] Do all hosts involved have the correct default gateway configured?
- [ ] If there are multiple routers in the path, does *every* router in the path have both directions covered?

## Verifying routes in Packet Tracer

On a router CLI:
```
Router# show ip route
```
This lists all known routes — directly connected (marked `C`), static (marked `S`), etc. This is your primary debugging tool once cabling and interface IPs are confirmed correct — always check this before assuming a service-level (DNS/DHCP/etc.) problem.

---

## Related notes
- [[04-router-and-default-gateway]]
- [[05-ip-addressing-and-subnetting]]
- [[08-linux-networking-commands]]
