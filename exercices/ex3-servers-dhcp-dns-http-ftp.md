---
tags: [networking, deep-in-net, exercise3, dhcp, dns, http, https, ftp, servers, subnetting]
---

# Exercise 3 — DHCP, DNS, HTTPS & FTP Server Configuration

> [!important] Core Concept
> This exercise is the first one where the subnet isn't something you freely choose — the DNS requirement pins one exact address (`192.168.1.99`), so the whole addressing plan has to be built *around* that fixed point rather than calculated from a host count like Exercises 1–2. Everything else (DHCP scope, other server IPs, PC addressing) gets planned to fit around that one non-negotiable address.

## What Exercise 3 actually requires

- All **servers** get static IPs (never DHCP-assigned) — a server needs a predictable, unchanging address since clients and DNS records need to reliably find it.
- A **DHCP server** hands out addresses to all PCs automatically.
- An **HTTPS server** shows a hello page; **HTTP must be disabled** on that same server.
- An **FTP server** has a user `deepinnet` with full `RWDNL` permissions.
- A **DNS server** with two records:
  - `deep-in-net.local` → `192.168.1.99` (A record)
  - `deep-in-net.com` → `deep-in-net.local` (CNAME record)
- Visiting `https://deep-in-net.com` must resolve through to the HTTPS server.

> [!note] Check your diagram
> The exact number of servers and PCs, and whether services are combined onto one server or split across several, depends on your `ex03.jpg` topology. This note assumes 3 separate server devices (DNS+HTTP/HTTPS combined on one, DHCP on another, FTP on a third) and a handful of DHCP-assigned PCs — adjust the addressing table if your diagram differs.

## Step 1 — Build the addressing plan around the fixed IP

Since `192.168.1.99` is explicitly required, the simplest and most conventional choice is a standard `/24` network: `192.168.1.0/24` (mask `255.255.255.0`).

Reserve address ranges by purpose so the plan stays organized and collision-free:

| Range | Purpose |
|---|---|
| 192.168.1.1 | Reserved for a future router/gateway interface (not required this exercise, but good practice) |
| 192.168.1.90 – 192.168.1.99 | Static server addresses |
| 192.168.1.100 – 192.168.1.150 | DHCP pool for PCs |

## Step 2 — Assign static server IPs

| Server | Service(s) | IP Address | Subnet Mask |
|---|---|---|---|
| Server0 | DNS, HTTP, HTTPS | 192.168.1.99 | 255.255.255.0 |
| Server1 | DHCP | 192.168.1.98 | 255.255.255.0 |
| Server2 | FTP | 192.168.1.97 | 255.255.255.0 |

`192.168.1.99` is not arbitrary here — it's the exact address the DNS A record must point to, so this server's static IP and the DNS record have to match precisely.

Configure each in Packet Tracer: Desktop tab → IP Configuration → Static → enter the IP and mask above. Servers are always static, never DHCP — see the Core Concept note in this file.

## Step 3 — Configure DHCP

On the DHCP server's Services tab → DHCP:
- Turn the service **ON**.
- Default Gateway: `192.168.1.1` (reserved above, even with no router present yet — keeps the config consistent if one is added later).
- DNS Server: `192.168.1.99` (points PCs at your DNS server).
- Starting IP address: `192.168.1.100`.
- Subnet Mask: `255.255.255.0`.
- Max users: set to comfortably cover your PC count.

On each **PC**: Desktop tab → IP Configuration → select **DHCP** instead of Static, then confirm (via `ipconfig` in the PC's command line) that it actually received a lease in the `192.168.1.100+` range — not a `169.254.x.x` self-assigned address, which would indicate the DHCP server isn't reachable or isn't configured correctly.

## Step 4 — Configure HTTPS (and disable HTTP)

On the HTTP/HTTPS server's Services tab → HTTP:
- Toggle **HTTPS ON**.
- Toggle **HTTP OFF**.
- Edit the served page (usually `index.html`) to display your hello message.

This directly reflects the security principle from your `07-services-and-protocols.md` note: a server should only run the service it actually needs — HTTPS, not both HTTP and HTTPS side by side.

**Verify:** from a PC, run `netstat` conceptually (or just test in-browser) — port 443 should respond, port 80 should not.

## Step 5 — Configure FTP

On the FTP server's Services tab → FTP:
- Add a new user: username `deepinnet`, set a password.
- Check **all five** permission boxes: Read, Write, Delete, Rename, List (RWDNL) — see `07-services-and-protocols.md` for what each letter grants.

## Step 6 — Configure DNS

On the DNS server's Services tab → DNS:
- Add an **A Record**: name `deep-in-net.local`, IP `192.168.1.99`.
- Add a **CNAME Record**: alias `deep-in-net.com`, points to `deep-in-net.local`.

### Why this resolves correctly without any extra "redirect" step
When a browser looks up `deep-in-net.com`:
1. DNS returns the CNAME → tells the browser the real name is `deep-in-net.local`.
2. DNS resolves `deep-in-net.local` via the A record → `192.168.1.99`.
3. Browser connects directly to `192.168.1.99` on port 443 (HTTPS).

This chain **is** the "redirect" the exercise describes — there's no separate HTTP redirect rule to configure; it's purely a result of the CNAME → A record resolution chain. The verification is functional: open a browser on any PC and navigate to `https://deep-in-net.com` — if it loads your hello page, every piece (DNS, static IP, HTTPS-only config) is working together correctly.

## Step 7 — Verifying everything end-to-end

From a PC command line:
```
ipconfig          → confirm DHCP-assigned IP, mask, gateway, DNS server
nslookup deep-in-net.com     → confirm it resolves through to 192.168.1.99
ping 192.168.1.99            → confirm basic reachability to the server
```
From a PC browser:
```
https://deep-in-net.com      → should load the hello page
http://deep-in-net.com       → should fail/refuse, confirming HTTP is disabled
```

## The Knowledge section, mapped to what you just built

| Knowledge question | Where you just saw it |
|---|---|
| What is a server / DHCP / DNS / HTTP / HTTPS / FTP | Steps 2–6 above |
| TCP vs UDP, ports, OSI layers | DHCP=UDP/67-68, DNS=UDP/53, HTTP=TCP/80, HTTPS=TCP/443, FTP=TCP/20-21 — see `07-services-and-protocols.md` |
| DNS record types | A record vs CNAME record, Step 6 |

---

## Related notes
- `ex1-point-to-point-subnetting.md`
- `ex2-switch-hub-addressing.md`
- `07-services-and-protocols.md`
- `05-ip-addressing-and-subnetting.md`
- `08-linux-networking-commands.md`
