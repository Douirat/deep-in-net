---
tags: [networking, deep-in-net, exercise2, switch, hub, subnetting, cabling]
---

# Exercise 2 — Switch & Hub Groups

> [!important] Core Concept
> Unlike Exercise 1's isolated pairs, every PC connected to the same switch (or same hub) needs to be on the **same subnet** — they're sharing one Layer 2 broadcast domain with no router involved, so there's nothing to route between. The only real decision here is picking a subnet mask small enough to fit the number of PCs in each group without wasting addresses, same logic as Exercise 1, just applied to a group instead of a pair.

## What Exercise 2 actually requires

Two **separate, unconnected** groups:
- A set of PCs connected to a **switch** — all must be able to communicate with each other.
- A set of PCs connected to a **hub** — all must be able to communicate with each other.

The switch group and hub group are not connected to one another (no requirement says they need to reach each other), so they can safely reuse addressing patterns or use entirely separate ranges — treat them as two independent local networks.

> [!note] Check your diagram
> The exact number of PCs on the switch and on the hub depends on your `ex02.jpg` topology image. This note uses 4 PCs per group as a worked example — swap in your actual PC count and re-run the sizing table below if it differs.

## Step 1 — Cable types

| Connection | Cable |
|---|---|
| PC → Switch | Straight-through |
| PC → Hub | Straight-through |

Every PC-to-switch or PC-to-hub link uses **straight-through** cable — this differs from Exercise 1, where PC-to-PC direct links needed crossover. The rule is always: same device type = crossover, different device type = straight-through. See `cable-straight-through.md`.

## Step 2 — Why all PCs in one group share a single subnet

A switch (or hub) doesn't route between networks — it only forwards frames at Layer 2. If two PCs on the same switch had different subnets (e.g. one on `192.168.1.x` and another on `192.168.2.x`), they would try to reach each other and fail, because each PC would think the other is "outside its local network" and look for a default gateway that doesn't exist in this exercise. So: **one switch group = one subnet, one hub group = one subnet** (the same subnet range or a different one — your choice, since they don't need to reach each other).

## Step 3 — Sizing the subnet to the group

Same block-size method as Exercise 1, just sized for a group instead of a pair. For **4 PCs**, you need at least 4 usable host addresses:

```
Usable hosts = 2^host_bits − 2 ≥ 4  →  host_bits = 3  →  mask = /29
```

| Hosts needed | Smallest mask that fits | Usable hosts |
|---|---|---|
| 2 | /30 | 2 |
| 4 | /29 | 6 |
| 6 | /29 | 6 |
| 14 | /28 | 14 |
| 30 | /27 | 30 |

(This table extends the one from Exercise 1 — reuse it any time you need to size a subnet to a specific device count.)

## Step 4 — Worked example (4 PCs on the switch, 4 PCs on the hub)

**Switch group** — `192.168.1.0/29` (block size = 256 − 248 = 8)
- Network: `192.168.1.0`, Broadcast: `192.168.1.7`
- Usable range: `192.168.1.1` – `192.168.1.6`

| Device | IP Address | Subnet Mask |
|---|---|---|
| PC0 | 192.168.1.1 | 255.255.255.248 |
| PC1 | 192.168.1.2 | 255.255.255.248 |
| PC2 | 192.168.1.3 | 255.255.255.248 |
| PC3 | 192.168.1.4 | 255.255.255.248 |

**Hub group** — `192.168.2.0/29` (separate range, purely for clarity — not required to differ)

| Device | IP Address | Subnet Mask |
|---|---|---|
| PC4 | 192.168.2.1 | 255.255.255.248 |
| PC5 | 192.168.2.2 | 255.255.255.248 |
| PC6 | 192.168.2.3 | 255.255.255.248 |
| PC7 | 192.168.2.4 | 255.255.255.248 |

## Step 5 — Configuring it in Packet Tracer

1. Cable every PC to its switch/hub with **straight-through** cable.
2. On each PC: Desktop tab → IP Configuration → Static → enter IP and subnet mask from the tables above.
3. **No default gateway needed** — same reasoning as Exercise 1, there's no router, so nothing exists outside each group to reach.
4. Test with `ping` between PCs within the same group.

## Step 6 — The behavioral difference to actually observe

The exercise's real teaching point isn't the IP addressing (which is mechanically identical for both groups) — it's watching **how** the ping travels differently through each device. Switch to **Simulation Mode** in Packet Tracer and send a single PDU (ping) between two PCs on each side:

- **Switch side:** after the first flood/learn, the ping travels directly to the destination port only — you'll see the packet animate along one path.
- **Hub side:** the ping is broadcast out every port every time — you'll see it animate along every cable connected to the hub, even ones not involved in the conversation.

This visual difference is what the Knowledge section is actually asking you to understand: same IP addressing, same cable type, completely different Layer 2 forwarding behavior.

---

## Related notes
- `ex1-point-to-point-subnetting.md`
- `cable-straight-through.md`
- `03-switch-and-hub.md`
- `05-ip-addressing-and-subnetting.md`
