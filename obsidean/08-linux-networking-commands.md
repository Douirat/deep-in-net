---
tags: [networking, deep-in-net, linux, cli, troubleshooting]
---

# Linux Networking Commands

> [!important] Core Concept
> These commands are your diagnostic toolkit — each one answers a different "where is the problem" question, roughly in the order you should reach for them: *Can I even reach this host? What's my own config? What path is my traffic taking? Is name resolution working? What connections are currently open?* Troubleshooting in this project (and in the audit) should always follow this bottom-up-then-across order rather than guesswork.

## `ping`

Tests basic reachability between two hosts using ICMP Echo Request/Reply messages.

```
ping 192.168.1.10
```

- Success = Layer 3 connectivity exists (routing, IP config, and often Layer 1/2 are all fine).
- Failure tells you *something* is wrong but not *what* — treat it as the first sanity check, not the final diagnosis.
- In Packet Tracer, PCs and servers both support `ping` from their command-line/desktop interface.

## `ipconfig` / `ifconfig`

Displays (and can configure) a host's own network interface settings — IP address, subnet mask, default gateway.

- Windows-style: `ipconfig` (Packet Tracer PCs use this style in their CLI)
- Linux-style: `ifconfig` or the newer `ip addr`

Use this first when troubleshooting — confirm the host actually has the IP configuration you expect (especially important right after setting up DHCP, to confirm a PC actually received a lease instead of falling back to an APIPA address like `169.254.x.x`, which indicates DHCP failure).

## `tracert` / `traceroute`

Shows the path (each router hop) a packet takes to reach a destination, and the round-trip time to each hop.

- Windows: `tracert 192.168.2.10`
- Linux: `traceroute 192.168.2.10`

Useful once `ping` fails to a *remote* subnet — traceroute tells you exactly which hop the packet stops progressing at, narrowing down which router is missing a route (see [[06-routing-tables]]).

## `nslookup`

Tests DNS resolution directly — asks "what IP does this hostname resolve to?"

```
nslookup deep-in-net.com
```

This is the exact tool to verify Exercise 3's DNS requirements: confirm `deep-in-net.local` resolves to `192.168.1.99`, and that `deep-in-net.com` correctly resolves through its CNAME to the same address.

## `arp -a`

Displays the ARP (Address Resolution Protocol) table — the local cache mapping IP addresses to MAC addresses for devices on the same subnet.

- Useful to confirm Layer 2 ↔ Layer 3 mapping is happening correctly for local (same-subnet) communication.
- If a device's IP shows up with no valid MAC entry, that's a sign of a Layer 2 (switching/cabling) issue rather than an IP/routing issue.

## `netstat`

Displays active network connections and listening ports on a host.

```
netstat -an
```

Directly useful for verifying Exercise 3's requirement that **HTTP is disabled** while **HTTPS is enabled** — running `netstat` on (or connecting to) the server should show port 443 active/listening and port 80 *not* listening, giving you concrete proof rather than just trusting the service toggle in the GUI.

## Recommended troubleshooting order

1. `ipconfig` / `ifconfig` — confirm your own host has the IP config you expect
2. `ping` — confirm basic reachability
3. `arp -a` — confirm Layer 2 resolution on the local subnet
4. `tracert` — if pinging a remote subnet fails, find which hop breaks
5. `nslookup` — confirm name resolution if the issue is about a hostname/URL rather than a raw IP
6. `netstat` — confirm which services/ports are actually active on a server

---

## Related notes
- [[04-router-and-default-gateway]]
- [[06-routing-tables]]
- [[07-services-and-protocols]]
