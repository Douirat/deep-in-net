---
tags: [networking, deep-in-net, exercise6, router, routing-table, static-route, subnetting, cabling]
---

# Exercise 6 — Two Separate Routers & Static Routes

> [!important] Core Concept
> Exercises 4 and 5 both used a **single router** with one interface per subnet, so the router automatically knew about both subnets the moment its interfaces came up — no manual routing configuration needed. Exercise 6 breaks that shortcut on purpose: now there are **two separate routers**, each directly connected to only one PC subnet, and neither router has any built-in knowledge of the subnet on the *other* router's side. You have to manually teach each router how to reach the far subnet — this is what a **static route** is for, and it's the first exercise where the routing table itself becomes something you actively configure rather than something that just populates itself.

## What Exercise 6 actually requires

- One PC in subnet 1, one PC in subnet 2.
- The two subnets are connected through **two separate routers linked to each other**, not one router with two interfaces.
- Both directions must work: subnet 1 → subnet 2, and subnet 2 → subnet 1.

Typical topology shape:
```
PC0 --- Router1 --- Router2 --- PC1
     (subnet 1)  (transit)  (subnet 2)
```

There are actually **three** subnets here, not two — the link directly between Router1 and Router2 is its own small subnet too, separate from the PC subnets on either end.

> [!note] Check your diagram
> Confirm from `ex06.jpg` whether the topology matches this shape (two routers in series) — this is what the Knowledge section's routing-table question implies, since a single-router setup (like Ex4/5) wouldn't need a routing table explanation at all.

## Step 1 — Why this needs three subnets, not two

- **Subnet 1**: Router1's interface + PC0
- **Transit subnet**: Router1's other interface + Router2's interface (the link between the two routers)
- **Subnet 2**: Router2's other interface + PC1

Each of these is a distinct point-to-point-style relationship needing only 2 usable addresses — so every one of them is sized as a **/30**, same logic as Exercise 1 and Exercise 4.

## Step 2 — Addressing plan

| Subnet | Range | Mask |
|---|---|---|
| Subnet 1 | 192.168.1.0/30 | 255.255.255.252 |
| Transit (Router1 ↔ Router2) | 192.168.2.0/30 | 255.255.255.252 |
| Subnet 2 | 192.168.3.0/30 | 255.255.255.252 |

| Device | IP Address | Mask | Default Gateway |
|---|---|---|---|
| PC0 | 192.168.1.2 | 255.255.255.252 | 192.168.1.1 |
| Router1 Gig0/0 (toward PC0) | 192.168.1.1 | 255.255.255.252 | — |
| Router1 Serial0/0/0 (toward Router2, DCE) | 192.168.2.1 | 255.255.255.252 | — |
| Router2 Serial0/0/0 (toward Router1, DTE) | 192.168.2.2 | 255.255.255.252 | — |
| Router2 Gig0/0 (toward PC1) | 192.168.3.1 | 255.255.255.252 | — |
| PC1 | 192.168.3.2 | 255.255.255.252 | 192.168.3.1 |

## Step 3 — Cable types

| Connection | Cable |
|---|---|
| PC → Router | Straight-through |
| Router → Router | Serial (DTE/DCE) |

The router-to-router link in this exercise uses a **serial cable**, not crossover Ethernet — this models a WAN-style link between two separate router "sites" rather than two routers sitting on the same local segment. See `cable-serial.md` for the full DTE/DCE explanation. The key thing to know now: serial links require **one end to be manually configured as the clock source (DCE)** — this doesn't happen automatically the way Ethernet links do, and it's covered in Step 4 below.

## Step 3.5 — Make sure the routers actually have the right physical ports

This is the step that's easy to skip and causes confusing "interface doesn't exist" errors later. Each router needs an Ethernet-capable port toward its PC **and** a serial port toward the other router — and **serial ports are almost never built into the default router models in Packet Tracer**. Unlike the Ethernet-to-Ethernet topologies in Ex4/Ex5, this exercise specifically requires you to add a serial WAN interface module before the router-to-router link can be cabled at all.

**How to check/add a module in Packet Tracer:**
1. Click the router → **Physical** tab.
2. **Power it off first** — click the power switch/button on the router graphic. Modules cannot be added while the device is powered on.
3. Drag a serial module — typically `WIC-2T` (2-port serial WAN Interface Card) — from the module list on the left into an empty slot on the router. Do this on **both** routers.
4. **Power the router back on.**
5. Only now will a serial interface (e.g. `Serial0/0/0`) appear and accept a cable / be configurable via `interface` in the CLI.

If your router model doesn't already have enough Ethernet ports for the PC-facing side either (older models like the 1841 can ship with none), add an Ethernet module the same way — power off, drag the module (e.g. `NM-1FE-TX`), power on.

If you try to cable or configure an interface that doesn't physically exist yet, Packet Tracer will simply refuse the connection or the CLI won't recognize the interface name — this is a hardware problem, not a configuration mistake, so no amount of correct IP addressing will fix it until the module is actually installed.

> [!tip] Same rule applies to switches
> Switches almost always ship with enough ports for these exercises' scale, but if you ever need more ports than the default model provides, the same power-off → add module → power-on sequence applies.

## Step 4 — Configuring the router interfaces (CLI)

The PC-facing interfaces are still regular Ethernet, but the router-to-router link is now **serial**, which needs one extra thing Ethernet never required: a **clock rate**, set on whichever end is acting as the DCE (the "provider" side in a real WAN link — in Packet Tracer, check which end shows as DCE in the Physical view, or just pick one end consistently and set it there).

**Router1** (PC-facing Ethernet + serial toward Router2 — assume Router1's serial port is the DCE end):
```
Router1(config)# interface gigabitEthernet 0/0
Router1(config-if)# ip address 192.168.1.1 255.255.255.252
Router1(config-if)# no shutdown
Router1(config-if)# exit

Router1(config)# interface serial 0/0/0
Router1(config-if)# ip address 192.168.2.1 255.255.255.252
Router1(config-if)# clock rate 64000
Router1(config-if)# no shutdown
Router1(config-if)# exit
```

**Router2** (PC-facing Ethernet + serial toward Router1 — the DTE end, no clock rate needed here):
```
Router2(config)# interface gigabitEthernet 0/0
Router2(config-if)# ip address 192.168.3.1 255.255.255.252
Router2(config-if)# no shutdown
Router2(config-if)# exit

Router2(config)# interface serial 0/0/0
Router2(config-if)# ip address 192.168.2.2 255.255.255.252
Router2(config-if)# no shutdown
Router2(config-if)# exit
```

> [!important] Only the DCE end gets `clock rate`
> Setting `clock rate` on the DTE end has no effect and isn't needed — but forgetting it entirely on the DCE end is a very common reason a serial link between two correctly-addressed routers still shows as down. If you're not sure which end is DCE, run `show controllers serial 0/0/0` on each router — it will explicitly report "DCE cable" or "DTE cable."

At this point, run `show ip route` on each router — Router1 only knows about `192.168.1.0/30` and `192.168.2.0/30` (directly connected). It has **no idea** `192.168.3.0/30` exists. Router2 is the mirror image. This is exactly the gap a static route fixes.

## Step 5 — Configuring static routes

Each router needs a route to the subnet it isn't directly connected to, pointing at the *other* router's transit-link interface as the next hop:

**On Router1** — teach it how to reach subnet 2:
```
Router1(config)# ip route 192.168.3.0 255.255.255.252 192.168.2.2
```

**On Router2** — teach it how to reach subnet 1:
```
Router2(config)# ip route 192.168.1.0 255.255.255.252 192.168.2.1
```

Both directions are required — configuring only one side means replies can never make it back, and a ping will appear to fail even though the request technically reached its destination. See `06-routing-tables.md` for the full explanation of why both directions matter.

## Step 6 — Verifying the routing table

```
Router1# show ip route
```
You should now see `192.168.3.0/30` listed with an `S` (static) marker, alongside the `C` (directly connected) entries for the other two subnets. Same check on Router2 for `192.168.1.0/30`.

## Step 7 — Verifying end-to-end connectivity

From PC0:
```
ping 192.168.3.2     → should succeed once both static routes are in place
```
From PC1:
```
ping 192.168.1.2     → should also succeed
```

**Debugging order if it fails:** confirm Step 4 (interfaces up, correct IPs) before Step 5 (static routes) — a missing `no shutdown` will make even correctly-written static routes useless, since the interface the route depends on isn't actually active.

## The Knowledge section, mapped to what you just built

| Knowledge question | Where you just saw it |
|---|---|
| What is a routing table | Step 4's `show ip route` output — the list of known networks and how to reach each |
| Its role in routing traffic | Step 5 — without the static route entries, each router would drop packets destined for the subnet it doesn't directly know about |

---

## Related notes
- `ex4-router-default-gateway.md`
- `ex5-two-subnets-switches-router.md`
- `cable-serial.md`
- `06-routing-tables.md`
- `device-router.md`