---
tags: [networking, deep-in-net, exercise4, router, default-gateway, subnetting, cabling]
---

# Exercise 4 — Router-Connected PCs & Default Gateway

> [!important] Core Concept
> This is the first exercise where a device other than the PCs themselves needs an IP address to make communication possible — each router interface sits on its own subnet, and every PC needs a **default gateway** pointing at that interface to reach anything beyond its own local subnet. Without a router in the middle, these two PCs would be as isolated from each other as if they were on completely separate networks — because they are.

## What Exercise 4 actually requires

Two PCs, each connected to a **different interface of the same router**, must be able to communicate with each other:
```
PC0 --- Router (Gig0/0) ... Router (Gig0/1) --- PC1
```

> [!note] Check your diagram
> This assumes exactly one router with one PC on each side (the simplest possible router topology). If your `ex04.jpg` shows more devices per side, treat each side as its own group and size the subnet using the host-count table from `ex2-switch-hub-addressing.md`.

## Step 1 — Why PC0 and PC1 can't just share one subnet

Since PC0 and PC1 connect to *different* router interfaces, they are — by definition — on two separate networks. A router's whole purpose (see `04-router-and-default-gateway.md`) is connecting different networks together, so each side of the router needs its own distinct subnet, not a shared one.

## Step 2 — Sizing each subnet

Each side has exactly 2 devices needing an address: the PC and the router's interface on that side. Same logic as Exercise 1's point-to-point pairs — a **/30** fits perfectly (2 usable hosts, zero waste).

| Side | Subnet | Router interface | PC |
|---|---|---|---|
| PC0 side | 192.168.1.0/30 | 192.168.1.1 | 192.168.1.2 |
| PC1 side | 192.168.2.0/30 | 192.168.2.1 | 192.168.2.2 |

Using two entirely different network numbers (`.1.x` and `.2.x`) — rather than two /30 blocks carved from the same /24 — makes the topology easier to read at a glance in your documentation, though either approach is technically valid.

## Step 3 — Cable types

| Connection | Cable |
|---|---|
| PC0 → Router | Straight-through |
| PC1 → Router | Straight-through |

PC-to-router is a **dissimilar device** connection, same rule as PC-to-switch — straight-through cable. See `cable-straight-through.md`.

## Step 4 — Configuring the router (CLI)

```
Router> enable
Router# configure terminal

Router(config)# interface gigabitEthernet 0/0
Router(config-if)# ip address 192.168.1.1 255.255.255.252
Router(config-if)# no shutdown
Router(config-if)# exit

Router(config)# interface gigabitEthernet 0/1
Router(config-if)# ip address 192.168.2.1 255.255.255.252
Router(config-if)# no shutdown
Router(config-if)# exit
```

`no shutdown` is required on every interface — router interfaces are administratively down by default, so a correctly-assigned IP with no `no shutdown` will still show the link as down.

## Step 5 — Configuring the PCs

| Device | IP Address | Subnet Mask | Default Gateway |
|---|---|---|---|
| PC0 | 192.168.1.2 | 255.255.255.252 | 192.168.1.1 |
| PC1 | 192.168.2.2 | 255.255.255.252 | 192.168.2.1 |

This is the first exercise where the **default gateway field actually matters** — unlike Exercises 1–3, PC0 now needs to reach an address (PC1) that is *outside* its own subnet, so it must know where to send that traffic. See `04-router-and-default-gateway.md` for exactly how a PC decides whether to use its gateway.

## Step 6 — Why no manual routing table entry is needed here

Because both subnets are **directly connected** to the same router (one on each interface), the router automatically adds both networks to its routing table the moment each interface is configured with `no shutdown` — no static route configuration is required in this exercise. This is different from Exercise 6, where multiple *separate* routers will need manually configured static routes to learn about subnets they aren't directly attached to. Verify this with:
```
Router# show ip route
```
You should see both `192.168.1.0/30` and `192.168.2.0/30` listed as directly connected (marked `C`).

## Step 7 — Verifying connectivity

From PC0's command line:
```
ipconfig                → confirm IP, mask, gateway are set correctly
ping 192.168.2.2        → should succeed, confirming PC1 is reachable through the router
```
If the ping fails, check in this order (see `08-linux-networking-commands.md` for the full troubleshooting sequence):
1. Cable type and link status (Layer 1)
2. Each interface's IP config and `no shutdown` status on the router
3. Each PC's default gateway field — a missing or wrong gateway is the most common cause of "same-subnet works, cross-subnet fails" in this exact scenario

## The Knowledge section, mapped to what you just built

| Knowledge question | Where you just saw it |
|---|---|
| What is a router and its role | Step 1 — connecting two otherwise-isolated subnets |
| Router vs switch | Router makes IP-based decisions between networks; a switch never would have let PC0 reach PC1 across two different subnets |
| OSI layer of a router | Layer 3 — see `device-router.md` |
| Default gateway | Step 5 — the field that made cross-subnet communication possible at all |

---

## Related notes
- `ex1-point-to-point-subnetting.md`
- `ex2-switch-hub-addressing.md`
- `04-router-and-default-gateway.md`
- `06-routing-tables.md`
- `device-router.md`
