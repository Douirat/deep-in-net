---
tags: [networking, deep-in-net, exercise8, router, switch, subnetting, cabling]
---

# Exercise 8 — Three-Subnet Full Mesh

> [!important] Core Concept
> Exercise 8 scales Exercise 5's pattern from 2 subnets to 3, using **one router with three LAN interfaces** rather than introducing more routers. Because all three subnets connect to interfaces on the *same* router, that router automatically knows about all three the moment each interface comes up — no static routes needed, same shortcut as Exercise 4/5. This is why the README lists no separate "Knowledge" section for this exercise: there's nothing conceptually new here, just applying Ex5's logic to a third subnet.

## What Exercise 8 actually requires

- All devices on the same switch communicate with each other (Layer 2 — same pattern as every prior switch group).
- **All three subnets can reach each other in both directions** — subnet 1 ↔ subnet 2, subnet 1 ↔ subnet 3, subnet 2 ↔ subnet 3 (6 directions total, 3 pairs).

Topology shape:
```
                [PCs] --- Switch1
                            |
[PCs] --- Switch2 --- Router (3 LAN interfaces) 
                            |
                [PCs] --- Switch3
```

> [!note] Check your diagram
> This assumes a single router with three separate LAN-facing interfaces, one per subnet — confirm this matches `ex08.jpg`. If your diagram instead uses multiple separate routers linked together (like Ex6/7's pattern extended to three subnets), tell me and I'll rebuild this note with the static-route mesh that topology would require instead.

## Step 1 — Why no static routes are needed here

Every subnet in this exercise is **directly connected** to the same router, just on a different interface. A router automatically adds a directly-connected network to its routing table the moment that interface has a valid IP and is brought up with `no shutdown` — this is the exact same shortcut Exercises 4 and 5 relied on. Since all three subnets sit on one router rather than being split across separate routers, there's no gap in knowledge to fill with a static route, unlike Exercises 6 and 7.

## Step 2 — Sizing each subnet

Example: 4 PCs per subnet + 1 router interface = 5 addresses needed per side, same sizing logic as Exercise 5:

```
Usable hosts = 2^host_bits − 2 ≥ 5  →  host_bits = 3  →  mask = /29 (6 usable hosts)
```

| Subnet | Range | Mask |
|---|---|---|
| Subnet 1 | 192.168.1.0/29 | 255.255.255.248 |
| Subnet 2 | 192.168.2.0/29 | 255.255.255.248 |
| Subnet 3 | 192.168.3.0/29 | 255.255.255.248 |

## Step 3 — Assigning addresses

**Subnet 1 (192.168.1.0/29)**

| Device | IP Address | Mask | Default Gateway |
|---|---|---|---|
| Router Fa0/0 | 192.168.1.1 | 255.255.255.248 | — |
| PC0 | 192.168.1.2 | 255.255.255.248 | 192.168.1.1 |
| PC1 | 192.168.1.3 | 255.255.255.248 | 192.168.1.1 |
| PC2 | 192.168.1.4 | 255.255.255.248 | 192.168.1.1 |
| PC3 | 192.168.1.5 | 255.255.255.248 | 192.168.1.1 |

**Subnet 2 (192.168.2.0/29)**

| Device | IP Address | Mask | Default Gateway |
|---|---|---|---|
| Router Fa1/0 | 192.168.2.1 | 255.255.255.248 | — |
| PC4 | 192.168.2.2 | 255.255.255.248 | 192.168.2.1 |
| PC5 | 192.168.2.3 | 255.255.255.248 | 192.168.2.1 |
| PC6 | 192.168.2.4 | 255.255.255.248 | 192.168.2.1 |
| PC7 | 192.168.2.5 | 255.255.255.248 | 192.168.2.1 |

**Subnet 3 (192.168.3.0/29)**

| Device | IP Address | Mask | Default Gateway |
|---|---|---|---|
| Router Fa2/0 | 192.168.3.1 | 255.255.255.248 | — |
| PC8 | 192.168.3.2 | 255.255.255.248 | 192.168.3.1 |
| PC9 | 192.168.3.3 | 255.255.255.248 | 192.168.3.1 |
| PC10 | 192.168.3.4 | 255.255.255.248 | 192.168.3.1 |
| PC11 | 192.168.3.5 | 255.255.255.248 | 192.168.3.1 |

## Step 4 — Cable types

| Connection | Cable |
|---|---|
| PC → Switch | Straight-through |
| Switch → Router | Straight-through |

All dissimilar-device connections — straight-through throughout, no serial or crossover needed anywhere in this exercise since there's only one router.

## Step 5 — Physical modules

A router needs **three** usable LAN interfaces here instead of the usual two — if the default model doesn't have three built-in Ethernet ports, add a module (power off → drag module e.g. `NM-1FE-TX` → power back on). See `ex6-two-routers-static-routes.md` Step 3.5 for the full procedure.

## Step 6 — Configuring the router (CLI)

```
Router> enable
Router# configure terminal

Router(config)# interface fastEthernet 0/0
Router(config-if)# ip address 192.168.1.1 255.255.255.248
Router(config-if)# no shutdown
Router(config-if)# exit

Router(config)# interface fastEthernet 1/0
Router(config-if)# ip address 192.168.2.1 255.255.255.248
Router(config-if)# no shutdown
Router(config-if)# exit

Router(config)# interface fastEthernet 2/0
Router(config-if)# ip address 192.168.3.1 255.255.255.248
Router(config-if)# no shutdown
Router(config-if)# exit
```

## Step 7 — Confirming the routing table

```
Router# show ip route
```
All three subnets — `192.168.1.0/29`, `192.168.2.0/29`, `192.168.3.0/29` — should appear as directly connected (`C`), with no static entries needed at all.

## Step 8 — Verifying full-mesh connectivity

Test all three pairs, in both directions where practical:

1. **Within each switch group** — e.g. PC0 → PC1 (same subnet), confirms Layer 2 is fine before testing across the router.
2. **Subnet 1 ↔ Subnet 2** — PC0 (`192.168.1.2`) → PC4 (`192.168.2.2`), and the reverse.
3. **Subnet 1 ↔ Subnet 3** — PC0 (`192.168.1.2`) → PC8 (`192.168.3.2`), and the reverse.
4. **Subnet 2 ↔ Subnet 3** — PC4 (`192.168.2.2`) → PC8 (`192.168.3.2`), and the reverse.

If any single pair fails while the others succeed, the fault is almost always isolated to that one subnet's interface (`no shutdown` status) or that one subnet's PCs' default gateway — it's not a router-wide problem, since the other two pairs already prove the router itself is forwarding correctly.

## Why this is simpler than it looks

Despite having more subnets than any prior exercise, Ex8 is actually **easier** to get right than Ex6/7, because there's no static routing to configure at all — the complexity here is purely in careful, repeatable addressing (three parallel subnets instead of two) rather than any new routing concept.

---

## Related notes
- `ex4-router-default-gateway.md`
- `ex5-two-subnets-switches-router.md`
- `05-ip-addressing-and-subnetting.md`
- `06-routing-tables.md`
