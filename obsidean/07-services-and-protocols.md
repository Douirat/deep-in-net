---
tags: [networking, deep-in-net, layer4, layer7, dhcp, dns, http, https, ftp, tcp, udp, ports]
---

# Core Network Services & Protocols

> [!important] Core Concept
> Below Layer 4, networking is about *getting data somewhere*. At Layer 4 and above, it's about *what that data means and how reliably it needs to arrive*. TCP/UDP decide the delivery guarantee, ports decide which application on the destination host receives it, and the Layer 7 protocols (DHCP, DNS, HTTP/S, FTP) are the actual "conversations" applications have using that foundation.

## What is a server?

A server is any device (or software running on a device) that provides a service to other devices (clients) on request. In Packet Tracer, the "Server" device type can run multiple services simultaneously (DHCP, DNS, HTTP, FTP, etc.) — Exercise 3 has you enable specific services and disable others on purpose, which mirrors real security practice: **a server should only run the services it needs to** (this is literally one of the exercise's bullet points — reducing "attack surface").

## TCP vs UDP — OSI Layer 4 (Transport)

Both are transport protocols — they take data from an application and get it across the network to the right port on the right host. They differ entirely in *how much they guarantee*.

| | TCP | UDP |
|---|---|---|
| Full name | Transmission Control Protocol | User Datagram Protocol |
| Connection | Connection-oriented (handshake required first) | Connectionless (just sends) |
| Reliability | Guaranteed delivery, retransmits lost packets | No guarantee — "fire and forget" |
| Ordering | Packets arrive in order | No ordering guarantee |
| Overhead | Higher (acknowledgments, sequencing) | Lower (minimal headers) |
| Speed | Slower due to reliability overhead | Faster |
| Analogy | Phone call — confirmed both sides connected | Postcard — sent, hope it arrives |
| Used by | HTTP/HTTPS, FTP, DNS zone transfers, email | DNS queries, DHCP, video streaming, VoIP |

### The TCP three-way handshake (why TCP is "connection-oriented")
1. Client → Server: `SYN` ("I want to connect")
2. Server → Client: `SYN-ACK` ("Okay, acknowledged, I want to connect too")
3. Client → Server: `ACK` ("Confirmed, let's talk")

Only after this handshake does actual data transfer begin — this is the overhead that makes TCP more reliable but slightly slower to start than UDP.

## Ports

A port is a 16-bit number (0–65535) that identifies *which application/service* on a host should receive incoming traffic. The IP address gets data to the right **host**; the port gets it to the right **application on that host**.

- **Well-known ports**: 0–1023, reserved for standard services (see table below).
- **Registered ports**: 1024–49151, used by specific applications.
- **Dynamic/private ports**: 49152–65535, typically used as temporary client-side (ephemeral) ports for outgoing connections.

### Ports & OSI layers for this project's protocols

| Protocol | Port(s) | Transport | OSI Layer (protocol itself) |
|---|---|---|---|
| DHCP | 67 (server), 68 (client) | UDP | 7 (Application) |
| DNS | 53 | UDP (queries), TCP (zone transfers) | 7 (Application) |
| HTTP | 80 | TCP | 7 (Application) |
| HTTPS | 443 | TCP | 7 (Application) |
| FTP | 20 (data), 21 (control) | TCP | 7 (Application) |

## DHCP — Dynamic Host Configuration Protocol

Automatically assigns IP configuration (IP address, subnet mask, default gateway, DNS server) to hosts, so you don't have to manually configure every single PC.

### The DORA process
1. **D**iscover — client broadcasts "is there a DHCP server out there?"
2. **O**ffer — a DHCP server responds with a proposed IP configuration
3. **R**equest — client broadcasts back "yes, I'll take that offer" (broadcast so any other DHCP servers know their offer wasn't taken)
4. **A**cknowledge — server confirms and formally leases the address to the client

### In this project
Exercise 3 requires the DHCP server to hand out addresses to *all* PCs — meaning the PCs must be set to "DHCP" mode (not static) in their IP configuration, and the DHCP server needs a correctly scoped pool matching your chosen subnet.

## DNS — Domain Name System

Translates human-readable names (`deep-in-net.com`) into IP addresses, since computers route based on IP, not names.

### Record types (know at least A and CNAME well — you'll configure both)

| Record | Purpose |
|---|---|
| **A** | Maps a hostname directly to an IPv4 address (e.g. `deep-in-net.local → 192.168.1.99`) |
| **CNAME** | Maps a hostname to *another* hostname (an alias), which then resolves further (e.g. `deep-in-net.com → deep-in-net.local`) |
| **MX** | Specifies mail servers for a domain |
| **NS** | Specifies authoritative name servers for a domain |
| **PTR** | Reverse lookup — maps an IP address back to a hostname |

### In this project
- `deep-in-net.local → 192.168.1.99` is an **A record**.
- `deep-in-net.com → deep-in-net.local` is a **CNAME record** (alias pointing to another name, which itself then resolves via the A record).
- The end result: visiting `https://deep-in-net.com` resolves through the CNAME to the A record's IP, and lands on your HTTPS server.

## HTTP vs HTTPS

| | HTTP | HTTPS |
|---|---|---|
| Full name | HyperText Transfer Protocol | HTTP **Secure** |
| Port | 80 | 443 |
| Encryption | None — data sent in plain text | Encrypted via TLS/SSL |
| Security | Vulnerable to eavesdropping/tampering | Protects confidentiality and integrity in transit |

**In this project:** the exercise explicitly requires HTTPS to work and HTTP to be **disabled** on the same server — this models a real security best practice of never leaving an unencrypted service running when a secure equivalent exists.

## FTP — File Transfer Protocol

Used to transfer files between a client and server. Uses **two** ports: 21 (control connection — commands like login, list directory) and 20 (data connection — actual file transfer).

### FTP Permissions (the "RWDNL" in the project)
When creating a user account on an FTP server, Packet Tracer lets you assign granular permissions:

| Letter | Permission | Meaning |
|---|---|---|
| R | Read | Download/view files |
| W | Write | Upload/modify files |
| D | Delete | Remove files |
| N | Rename | Rename files/folders |
| L | List | View directory contents |

Granting all 5 (RWDNL) to the `deepinnet` user gives that account full control over the FTP server's file system — the project asks for full permissions rather than a restricted subset.

---

## Related notes
- [[01-networking-foundations]]
- [[04-router-and-default-gateway]]
- [[08-linux-networking-commands]]
