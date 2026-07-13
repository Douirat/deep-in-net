---
tags: [networking, deep-in-net, exercise5, router, switch, default-gateway, subnetting, cabling]
---

# Exercise 5 — Two Switched Subnets Through a Shared Router

> [!important] Core Concept
> This exercise is Exercise 2 and Exercise 4 combined — each subnet is now a *group* of PCs hanging off its own switch (Layer 2, same as Ex2), and the two groups reach each other through a single router with one interface per subnet (Layer 3, same as Ex4). Nothing new conceptually is introduced; the skill being tested is combining what you already know at a slightly larger scale.

## What Exercise 5 actually requires

- All devices on **the same switch** communicate with each other (Layer 2, no router involved — same requirement pattern as Exercise 2).
- All devices in **subnet 1** communicate with all devices in **subnet 2**, and vice versa (Layer 3, same requirement pattern as Exercise 4).

Typical topology shape:
```
[PCs] --- Switch1 --- Router (Gig0/0) ... Router (Gig0/1) --- Switch2 --- [PCs]
  (subnet 1)                                                    (subnet 2)
```

> [!note] Check your diagram
> This assumes one switch per subnet, both connecting into the same router. Confirm PC count per side from your `ex05.jpg` and adjust the mask in Step 2 if it differs from the 4-PCs-per-subnet example used here.

## Step 1 — Why this is Ex2 + Ex4, not a new concept

- Within **Switch1**, all connected PCs are on one subnet and reach each other exactly like Exercise 2 — pure Layer 2 switching, no gateway involved.
- Reaching **across** to Switch2's subnet requires the router, exactly like Exercise 4 — each PC needs a default gateway pointing at its local router interface.

The only genuinely new skill is sizing a subnet that has to fit **multiple PCs plus the router's interface** in the same range (Exercise 4 only ever needed room for 1 PC + 1 router interface).

## Step 2 — Sizing each subnet

Example: 4 PCs per subnet + 1 router interface = 5 addresses needed per side.

```
Usable hosts = 2^host_bits − 2 ≥ 5  →  host_bits = 3  →  mask = /29 (6 usable hosts)
```

| Subnet | Range | Mask |
|---|---|---|
| Subnet 1 | 192.168.1.0/29 | 255.255.255.248 |
| Subnet 2 | 192.168.2.0/29 | 255.255.255.248 |

## Step 3 — Assigning addresses

By convention, give the router interface the **first usable address** in each subnet, and hand out the rest to PCs.

**Subnet 1 (192.168.1.0/29)** — usable range `.1`–`.6`

| Device | IP Address | Mask | Default Gateway |
|---|---|---|---|
| Router Gig0/0 | 192.168.1.1 | 255.255.255.248 | — |
| PC0 | 192.168.1.2 | 255.255.255.248 | 192.168.1.1 |
| PC1 | 192.168.1.3 | 255.255.255.248 | 192.168.1.1 |
| PC2 | 192.168.1.4 | 255.255.255.248 | 192.168.1.1 |
| PC3 | 192.168.1.5 | 255.255.255.248 | 192.168.1.1 |

**Subnet 2 (192.168.2.0/29)** — usable range `.1`–`.6`

| Device | IP Address | Mask | Default Gateway |
|---|---|---|---|
| Router Gig0/1 | 192.168.2.1 | 255.255.255.248 | — |
| PC4 | 192.168.2.2 | 255.255.255.248 | 192.168.2.1 |
| PC5 | 192.168.2.3 | 255.255.255.248 | 192.168.2.1 |
| PC6 | 192.168.2.4 | 255.255.255.248 | 192.168.2.1 |
| PC7 | 192.168.2.5 | 255.255.255.248 | 192.168.2.1 |

## Step 4 — Cable types

| Connection | Cable |
|---|---|
| PC → Switch | Straight-through |
| Switch → Router | Straight-through |

Both are dissimilar-device connections — same rule used in every exercise so far. See `cable-straight-through.md`.

## Step 5 — Configuring the router (CLI)

```
Router> enable
Router# configure terminal

Router(config)# interface gigabitEthernet 0/0
Router(config-if)# ip address 192.168.1.1 255.255.255.248
Router(config-if)# no shutdown
Router(config-if)# exit

Router(config)# interface gigabitEthernet 0/1
Router(config-if)# ip address 192.168.2.1 255.255.255.248
Router(config-if)# no shutdown
Router(config-if)# exit
```

## Step 6 — Why no static routes are needed here (still)

Same reasoning as Exercise 4: both subnets are **directly connected** to this one router, one per interface, so the router automatically knows how to reach both the moment each interface is up. Confirm with:
```
Router# show ip route
```
Both `192.168.1.0/29` and `192.168.2.0/29` should appear as directly connected (`C`).

## Step 7 — Verifying connectivity

1. **Within a switch group first** — from PC0, `ping 192.168.1.3` (PC1, same subnet). This isolates any Layer 2/switch cabling issues before introducing the router into the test.
2. **Across subnets** — from PC0, `ping 192.168.2.2` (PC4, other subnet). If step 1 works but step 2 fails, the problem is almost always a missing/wrong default gateway on one of the PCs, or the router interface not being `no shutdown`.

## Common mistake to watch for

Forgetting the default gateway on even one PC in the group is the most likely failure — it won't affect that PC's ability to reach others on its own switch (Step 7.1 still passes), but it will silently break only the cross-subnet ping, making it look like a router problem when it's actually a single PC's configuration.

---

## Related notes
- `ex2-switch-hub-addressing.md`
- `ex4-router-default-gateway.md`
- `05-ip-addressing-and-subnetting.md`
- `06-routing-tables.md`
