---
tags: [networking, deep-in-net, exercise1, subnetting, point-to-point, cabling]
---

# Exercise 1 — PC-to-PC Links & Point-to-Point Subnetting (/30)

> [!important] Core Concept
> A direct link between exactly two devices only ever needs 2 usable IP addresses — using a full /24 for that would waste 252 addresses. The correctly-sized subnet for any point-to-point link is a **/30**, and sizing a subnet to fit an exact number of hosts (rather than always defaulting to /24) is the actual skill Exercise 1 is testing, even though the instructions don't say "subnet" directly.

## What Exercise 1 actually requires

Three **isolated** PC-to-PC pairs:
- PC0 ↔ PC1
- PC2 ↔ PC3
- PC4 ↔ PC5

Each pair connects directly — no switch, no router — and just needs to ping the other device in its pair.

## Step 1 — Cable type

Connecting two devices of the **same type** (PC to PC) requires a **crossover cable**, not straight-through. See `cable-crossover.md` for the full pin-swap explanation. Using the wrong cable here means no link light at all, regardless of correct IP configuration.

## Step 2 — Why /30 is the right subnet size

A /30 subnet mask (`255.255.255.252`) leaves 2 host bits:

```
Usable hosts = 2^host_bits − 2 = 2^2 − 2 = 2
```

That's exactly enough for a 2-device point-to-point link, with zero addresses wasted. This is a standard real-world convention — router-to-router WAN links almost always use /30 for the same reason.

## Step 3 — Manual calculation method (block size)

For any mask, block size = 256 − (last relevant octet of the mask).

For /30 → mask octet = 252 → block size = 256 − 252 = **4**

Subnet boundaries in the last octet: `0, 4, 8, 12, 16...`

For each boundary block:
- Network address = the boundary itself
- Broadcast address = next boundary − 1
- Usable range = network + 1 to broadcast − 1

## Step 4 — Worked example: PC0 ↔ PC1

Using `192.168.1.0/30`:
- Network address: `192.168.1.0`
- Broadcast address: `192.168.1.3`
- Usable hosts: `192.168.1.1` and `192.168.1.2`

| Device | IP Address | Subnet Mask |
|---|---|---|
| PC0 | 192.168.1.1 | 255.255.255.252 |
| PC1 | 192.168.1.2 | 255.255.255.252 |

## Step 5 — Extending the pattern to all three pairs

Since the pairs are physically isolated (no shared device connects them), they could technically reuse the same range without conflict — but carving each pair its own non-overlapping /30 out of one block is the clean, audit-ready way to demonstrate you understand the subnetting logic, not just memorized one example.

Each subsequent subnet's network address jumps by the block size (4):

| Pair | Subnet | Device A | Device B |
|---|---|---|---|
| PC0 – PC1 | 192.168.1.0/30 | 192.168.1.1 | 192.168.1.2 |
| PC2 – PC3 | 192.168.1.4/30 | 192.168.1.5 | 192.168.1.6 |
| PC4 – PC5 | 192.168.1.8/30 | 192.168.1.9 | 192.168.1.10 |

## Step 6 — Configuring it in Packet Tracer

1. Cable each pair with a **crossover** cable.
2. On each PC: Desktop tab → IP Configuration → Static → enter the IP and subnet mask (`255.255.255.252`) from the table above.
3. **No default gateway needed** — there's no router in this exercise, and nothing exists outside the pair to reach.
4. Test with `ping <other PC's IP>` from the PC's command prompt.

## The skill to carry forward

This "how many host bits do I need to cover exactly N devices with the least waste" logic is the same reasoning used in reverse for Exercises 5–8, where you'll be given a target number of PCs per subnet and need to pick the smallest mask that fits them cleanly. Common sizes worth memorizing alongside /30:

| Hosts needed | Smallest mask that fits | Usable hosts |
|---|---|---|
| 2 | /30 | 2 |
| 6 | /29 | 6 |
| 14 | /28 | 14 |
| 30 | /27 | 30 |
| 62 | /26 | 62 |
| 126 | /25 | 126 |
| 254 | /24 | 254 |

---

## Related notes
- `cable-crossover.md`
- `05-ip-addressing-and-subnetting.md`
- `02-physical-layer-cabling.md`
