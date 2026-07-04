---
tags: [networking, deep-in-net, layer3, ip-addressing, subnetting, critical]
---

# IP Addressing & Subnetting

> [!important] Core Concept
> An IP address + subnet mask together answer one question: "which part of this address identifies the network, and which part identifies the specific host on it?" Subnetting is just deliberately shrinking the "host" portion to carve one big network into several smaller, isolated ones. Everything else (routing, DHCP scopes, DNS zones) depends on getting this split right — and the audit requires you to do it **by hand, no tools.**

## IPv4 Address Structure

An IPv4 address is 32 bits, written as 4 decimal numbers (octets) separated by dots, each octet 0–255 (i.e. 8 bits each).

```
192   .   168   .   1    .   10
11000000.10101000.00000001.00001010
```

Every address is really just this stream of 32 bits — the dotted decimal notation is purely for human readability.

## Network Portion vs Host Portion

A subnet mask defines where the split happens between "network" bits and "host" bits.

```
IP:    192.168.1.10       = 11000000.10101000.00000001.00001010
Mask:  255.255.255.0      = 11111111.11111111.11111111.00000000
                             └──────── network ────────┘└─ host ─┘
```

- Bits where the mask is `1` → network portion (must match for two hosts to be on the same subnet).
- Bits where the mask is `0` → host portion (this is the range of addresses available to assign to devices).

## CIDR Notation

CIDR (Classless Inter-Domain Routing) notation is shorthand: `/24` simply means "24 bits are the network portion" — i.e. it's a faster way of writing `255.255.255.0`.

### CIDR ↔ Decimal Mask Conversion Table (memorize this — used constantly)

| CIDR | Subnet Mask | Binary (last relevant octet) | Hosts per subnet |
|---|---|---|---|
| /24 | 255.255.255.0 | 00000000 | 254 |
| /25 | 255.255.255.128 | 10000000 | 126 |
| /26 | 255.255.255.192 | 11000000 | 62 |
| /27 | 255.255.255.224 | 11100000 | 30 |
| /28 | 255.255.255.240 | 11110000 | 14 |
| /29 | 255.255.255.248 | 11111000 | 6 |
| /30 | 255.255.255.252 | 11111100 | 2 |

**How to build this table from memory, on the spot (the actual skill you need):**
Each bit borrowed from the host portion doubles the number of subnets and halves the hosts per subnet.
- /24 → 256 total addresses, 254 usable (minus network + broadcast)
- /25 → 128 total addresses, 126 usable
- /26 → 64 total addresses, 62 usable
- /27 → 32 total addresses, 30 usable

Pattern: **total addresses = 2^(32 − CIDR)**, and the mask octet value is `256 − (2^(32-CIDR) as it applies to that octet)`.

## Manual Calculation Method (the actual audit skill)

Given an IP and a mask, you need to find: **Network address, Broadcast address, First usable host, Last usable host, Number of usable hosts.**

### Worked Example 1: `192.168.1.77 /27`

**Step 1 — Find the "block size" (increment) in the interesting octet.**
/27 = 255.255.255.224. The interesting octet is the 4th (224). Block size = 256 − 224 = **32**.

**Step 2 — List subnet boundaries in that octet using multiples of the block size:**
`0, 32, 64, 96, 128, 160, 192, 224`

**Step 3 — Find which block the IP falls into.**
77 falls between 64 and 96 → this subnet's range is 64–95.

**Step 4 — Fill in the answer:**
- Network address: `192.168.1.64`
- Broadcast address: `192.168.1.95` (one below the next boundary, 96 − 1)
- First usable host: `192.168.1.65` (network + 1)
- Last usable host: `192.168.1.94` (broadcast − 1)
- Usable hosts: `2^(32-27) − 2 = 32 − 2 = 30`

### Worked Example 2: `10.0.5.130 /26`

/26 = 255.255.255.192 → block size = 256 − 192 = **64**
Boundaries: `0, 64, 128, 192`
130 falls between 128 and 192 → subnet range 128–191

- Network address: `10.0.5.128`
- Broadcast address: `10.0.5.191`
- First usable host: `10.0.5.129`
- Last usable host: `10.0.5.190`
- Usable hosts: `2^6 − 2 = 62`

> [!tip] The "block size" method above is the fastest manual technique — it avoids full binary conversion for most problems. Fall back to full binary math only when the subnet boundary falls across an octet you're less sure about.

## Subnetting a Network into N Subnets

When the project says "subnet 1" and "subnet 2" need to communicate, you (or the scenario) has taken one larger network and split it. The process:

1. Decide how many subnets you need (or how many hosts each subnet needs).
2. Figure out how many bits to borrow from the host portion:
   - To get **N subnets**, you need to borrow enough bits so that `2^borrowed_bits ≥ N`.
   - To support **H hosts per subnet**, you need to leave enough host bits so that `2^host_bits − 2 ≥ H`.
3. Apply the new mask, then use the manual method above to lay out each subnet's range.

### Example: Split `192.168.1.0/24` into 4 subnets

Need `2^b ≥ 4` → b = 2 bits borrowed → new mask = /26 (255.255.255.192), block size 64.

| Subnet | Network | Usable range | Broadcast |
|---|---|---|---|
| 1 | 192.168.1.0 | .1 – .62 | 192.168.1.63 |
| 2 | 192.168.1.64 | .65 – .126 | 192.168.1.127 |
| 3 | 192.168.1.128 | .129 – .190 | 192.168.1.191 |
| 4 | 192.168.1.192 | .193 – .254 | 192.168.1.255 |

This is exactly the kind of table you should be able to produce for Exercises 5–8, where each "subnet" in the diagram needs its own clean, non-overlapping range.

## Private IP Ranges (RFC 1918)

These ranges are reserved for internal/private networks and are what you'll use throughout Packet Tracer (they're never routed on the public internet):

| Range | CIDR | Typical use |
|---|---|---|
| 10.0.0.0 – 10.255.255.255 | 10.0.0.0/8 | Large private networks |
| 172.16.0.0 – 172.31.255.255 | 172.16.0.0/12 | Medium private networks |
| 192.168.0.0 – 192.168.255.255 | 192.168.0.0/16 | Small/home private networks (most common in labs) |

## Practice Checklist
Before moving to Exercise 5, do these fully by hand and check your work:
- [ ] Find network/broadcast/usable range for `172.16.4.50/28`
- [ ] Find network/broadcast/usable range for `192.168.10.200/25`
- [ ] Split `10.10.0.0/16` into 8 equal subnets, list all 8 ranges
- [ ] Split `192.168.1.0/24` into subnets that support one with 50 hosts and one with 10 hosts (unequal/VLSM-style split)

---

## Related notes
- [[01-networking-foundations]]
- [[04-router-and-default-gateway]]
- [[06-routing-tables]]
