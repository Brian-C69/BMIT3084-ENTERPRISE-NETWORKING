# Chapter 1 – Static Routing (SRWE v7.0)

## Part 1: Module 15 – IP Static Routing
### What static routing is for
- Deterministic paths for small/edge networks, backup to dynamic protocols, and route summarization.
- Works for IPv4 and IPv6; installed with administrative distance (AD) 1 unless overridden.

### Static route types
- **Standard static**: specific prefix via `ip route <net> <mask> <next-hop|exit-intf>` / `ipv6 route <prefix> <next-hop|intf>`.
- **Default static**: catch-all `0.0.0.0/0` or `::/0` to reach “everything else”; typical at WAN edge or stub.
- **Floating static**: backup path with higher AD (e.g., `ip route 0.0.0.0 0.0.0.0 198.51.100.1 5`).
- **Host route**: single host (/32 or /128) for a specific device (often a server/tunnel endpoint).
- **Summary static**: aggregates multiple subnets to shrink the routing table.

### Next-hop options (IPv4/IPv6)
- **Next-hop only**: specify `ip route 10.1.0.0 255.255.0.0 192.0.2.2`; router must resolve outbound interface (recursive lookup).
- **Exit interface only**: `ip route 10.1.0.0 255.255.0.0 s0/0/0`; single lookup; best for point-to-point links (avoid on multi-access).
- **Fully specified**: include both `ip route 10.1.0.0 255.255.0.0 192.0.2.2 s0/0/0`; prevents ARP/ND churn on shared segments.
- IPv6 note: next-hop can be global unicast or link-local, but when using link-local you must also state the exit interface.

### Key commands (IPv4)
- Configure: `ip route <net> <mask> {<next-hop> | <intf> | <next-hop> <intf>} [distance]`.
- Default: `ip route 0.0.0.0 0.0.0.0 <next-hop|intf> [distance]`.
- Verify: `show ip route [static]`, `show ip cef <prefix>`, `ping`, `traceroute`.

### Key commands (IPv6)
- Enable unicast: `ipv6 unicast-routing`.
- Configure: `ipv6 route <prefix/len> {<next-hop> | <intf> | <next-hop> <intf>} [distance]`.
- Default: `ipv6 route ::/0 <next-hop|intf> [distance]`.
- Verify: `show ipv6 route static`, `show ipv6 cef <prefix>`.

### Building the initial table (example highlights)
- Without statics, routers only know directly connected and local (host) routes; inter-LAN pings fail.
- Add next-hop statics from R1 toward R2 for each remote LAN; repeat for IPv6 prefixes.
- Directly connected statics on point-to-point links reduce lookups but are discouraged on multi-access.

### Design and usage notes
- Prefer next-hop or fully specified routes on Ethernet; reserve exit-interface-only for point-to-point.
- Use summary/default statics at edges to minimize route table size.
- Keep floating statics’ AD higher than the primary (dynamic or static) route.
- Remember to redistribute statics if dynamic protocols must learn them.

## Part 2: Module 16 – Troubleshoot Static & Default Routes
### Packet forwarding with statics
- Router decapsulates the frame, looks up the destination IP.
  - If a specific static matches: use its next-hop/exit-interface.
  - Else if a default static exists: forward via default.
  - Else: drop and send ICMP unreachable.
- Downstream routers repeat lookup/forwarding; final hop ARPs/NDs for the host MAC and delivers.

### Common misconfigurations
- Wrong prefix/mask or typo in next-hop/exit interface.
- Next-hop not reachable (missing connected route, shutdown link, or wrong VLAN).
- Default route missing on edge; or present but points to the wrong peer.
- Floating static AD not higher than primary, causing unintended use.
- IPv6 using link-local next-hop without specifying the interface.
- Using exit-interface-only on multi-access Ethernet, causing ARP storms or blackholing.

### Troubleshooting workflow
- **Check table**: `show ip route` / `show ipv6 route` for presence and source codes (S/S*).
- **Resolve next-hop**: ensure connected route to next-hop exists; `show ip arp`, `show ipv6 neighbors`.
- **Interface state**: `show ip interface brief`, `show interface <intf>` for up/up.
- **Path test**: `ping`, `traceroute` to remote LAN and to next-hop; for IPv6 include `source` if needed.
- **CEF/forwarding**: `show ip cef <prefix> detail` or `show ipv6 cef <prefix>` to see resolved adjacency.
- **Logs/ICMP**: look for “% No valid route” or ICMP unreachables from peers; confirm ACLs are not blocking.

### Default route best practices
- Edge to ISP: install static default; ensure return path (ISP static/route-map or NAT).
- Stub routers: single default to upstream, plus specific statics for directly attached LANs.
- In dual-stack, configure both `0.0.0.0/0` and `::/0`.

### Floating static best practices
- Set AD higher than primary: e.g., `distance 5` vs primary dynamic (110) is too low; choose >110 (e.g., 120) if OSPF is primary.
- Ensure backup next-hop/interface is actually reachable when primary fails.

### Quick checklist
- Prefix/mask correct? Next-hop reachable? Interface up? AD intended? Default present where needed? Redistribution required? ARP/ND resolving? No overlapping statics shadowing specifics?
