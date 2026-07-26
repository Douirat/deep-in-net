---
tags: [networking, deep-in-net, exercise7, router, switch, routing-table, static-route, subnetting, cabling]
---

# Exercise 7 — Switch Groups Across Two Separate Routers

> [!important] Core Concept
> This exercise is Exercise 5 and Exercise 6 combined — each subnet is a *group* of PCs on its own switch (like Ex5), but the two subnets connect through **two separate routers linked by a serial connection** (like Ex6) instead of one router with two interfaces. That means the static route requirement from Ex6 comes back, just with a switch full of PCs on each end instead of a single PC.

## What Exercise 7 actually requires

- All devices on the same switch communicate with each other (Layer 2 — same pattern as Ex2/Ex5).
- All devices in subnet 1 communicate with all devices in subnet 2, and vice versa (Layer 3, across two routers).

Topology shape (confirmed from actual build):
```
[PCs] --- Switch1 --- RouterA (Fa0/0) ===(Serial2/0)=== RouterB (Fa0/0) --- Switch2 --- [PCs]
   (subnet 1)                      (transit)                       (subnet 2)
```

## Step 1 — Why this is Ex5 + Ex6, not a new concept

- Within each switch group, PCs reach each other exactly like Exercise 5 — pure Layer 2, no gateway involved.
- Reaching across to the *other* switch's subnet requires **two** routers instead of one, so each router again only knows about its own directly-connected subnet — exactly the gap Exercise 6 introduced, and the same fix applies: static routes.

## Step 2 — Actual addressing plan used

Unlike Exercises 5 and 6's tightly-sized examples (/29, /30), this build uses a straightforward **/24 for each LAN side** and a **/30 for the transit link** — perfectly valid, since the project's only hard requirement is that netmasks and IPs are respected consistently, not that every subnet is minimally sized. A /24 is simpler to document and leaves headroom to add more PCs later without re-addressing.

| Subnet | Range | Mask |
|---|---|---|
| Subnet 1 (LAN, RouterA side) | 192.168.1.0/24 | 255.255.255.0 |
| Transit (RouterA ↔ RouterB) | 10.10.0.0/30 | 255.255.255.252 |
| Subnet 2 (LAN, RouterB side) | 192.168.2.0/24 | 255.255.255.0 |

## Step 3 — Actual interface assignments

| Device | Interface | IP Address | Mask |
|---|---|---|---|
| RouterA | FastEthernet0/0 (toward Switch1) | 192.168.1.1 | 255.255.255.0 |
| RouterA | Serial2/0 (toward RouterB) | 10.10.0.1 | 255.255.255.252 |
| RouterB | Serial2/0 (toward RouterA) | 10.10.0.2 | 255.255.255.252 |
| RouterB | FastEthernet0/0 (toward Switch2) | 192.168.2.1 | 255.255.255.0 |

PCs on Switch1 take any address in `192.168.1.2`–`192.168.1.254`, mask `255.255.255.0`, gateway `192.168.1.1`.
PCs on Switch2 take any address in `192.168.2.2`–`192.168.2.254`, mask `255.255.255.0`, gateway `192.168.2.1`.

## Step 4 — Cable types

| Connection | Cable |
|---|---|
| PC → Switch | Straight-through |
| Switch → Router | Straight-through |
| Router → Router | Serial DCE |

PC-to-switch and switch-to-router are dissimilar-device connections (straight-through). Router-to-router uses the **Serial DCE** icon from Packet Tracer's Connections palette — see `cable-serial.md` for why there are two serial icons (DCE and DTE) even though only one cable is needed.

## Step 5 — Physical modules

Confirm each router actually has the serial port available (add a `WIC-2T` module if not — power off, add module, power back on) before attempting to cable the transit link. See `ex6-two-routers-static-routes.md` Step 3.5 for the full procedure. Since both `Serial2/0` interfaces are already showing **up/up**, this step and clocking are already handled correctly in this build.

## Step 6 — Confirming the routing gap before fixing it

Immediately after the interfaces come up, each router only knows its own side:

```
Router# show ip route
```

RouterA sees `192.168.1.0/24` and `10.10.0.0/30` as directly connected (`C`), but has no route to `192.168.2.0/24`. RouterB is the mirror image. This is the exact gap a static route fixes.

## Step 7 — Configuring static routes (actual commands used)

**On RouterA** — route to RouterB's LAN, via RouterB's transit-link IP:
```
Router(config)# ip route 192.168.2.0 255.255.255.0 10.10.0.2
```

**On RouterB** — route to RouterA's LAN, via RouterA's transit-link IP:
```
Router(config)# ip route 192.168.1.0 255.255.255.0 10.10.0.1
```

Both directions are required — configuring only one side means replies can never make it back.

## Step 8 — Verified routing table (confirmed working)

```
RouterA# show ip route
...
     10.0.0.0/30 is subnetted, 1 subnets
C       10.10.0.0 is directly connected, Serial2/0
C    192.168.1.0/24 is directly connected, FastEthernet0/0
S    192.168.2.0/24 [1/0] via 10.10.0.2

RouterB# show ip route
...
     10.0.0.0/30 is subnetted, 1 subnets
C       10.10.0.0 is directly connected, Serial2/0
S    192.168.1.0/24 [1/0] via 10.10.0.1
C    192.168.2.0/24 is directly connected, FastEthernet0/0
```

Both static (`S`) entries are present in both directions — routing is complete.

## Step 9 — Verifying end-to-end connectivity

1. **Within a switch group first** — from any PC on Switch1, ping another PC on Switch1. Isolates Layer 2 issues before the router path is involved.
2. **Across subnets** — from a PC on Switch1 (`192.168.1.x`), `ping 192.168.2.x` (a PC on Switch2, through both routers). If Step 1 passes but Step 2 fails, check in order: interface `no shutdown` status, `clock rate` on the DCE end, then the static routes on both routers.

## Common mistakes to watch for (all previously seen individually, now compounding)
- Forgetting a PC's default gateway (Ex5's mistake) — breaks only that PC's cross-subnet reachability.
- Forgetting `clock rate` on the DCE serial end (Ex6's mistake) — breaks the entire transit link, so *nothing* crosses between subnets. (Not an issue in this build — both Serial interfaces already show up/up.)
- Configuring a static route in only one direction (Ex6's mistake) — makes it look like the ping "half works" (request arrives, reply never returns). Confirmed fixed in Step 7/8 above.

---

## Related notes
- `ex5-two-subnets-switches-router.md`
- `ex6-two-routers-static-routes.md`
- `cable-serial.md`
- `06-routing-tables.md`
