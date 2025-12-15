# Chapter 7 - NAT for IPv4 (ENSA v7.0, Module 6)

## Why NAT is used
- **Conserve public IPv4** by allowing private IPv4 internally and translating at the edge.
- Common at the **border/stub router** between an internal LAN and an ISP/WAN.

## NAT vocabulary (important for exam)
- **Inside local**: the private address of an inside host (e.g., `192.168.10.254`).
- **Inside global**: the public address representing that inside host on the internet (e.g., `209.165.201.5`).
- **Outside local/global**: addresses of outside hosts (often the same unless translating on the outside too).

## NAT types (operation)
- **Static NAT (1:1)**: permanent mapping of one inside local to one inside global.
- **Dynamic NAT (pool)**: inside hosts temporarily get a public address from a pool.
- **PAT / NAT overload (many:1)**: multiple inside hosts share one (or a few) public IPs using different **ports**.

## Configuration checklist (typical lab workflow)
1) Identify **inside** and **outside** interfaces.
2) Mark interfaces with `ip nat inside` / `ip nat outside`.
3) Decide translation type:
   - Static NAT, or
   - Dynamic NAT using a pool, or
   - PAT (overload) using a pool or an outside interface.
4) Verify translations and statistics.

## Static NAT (example)
```txt
interface g0/1
 ip nat inside
!
interface s0/1/0
 ip nat outside
!
ip nat inside source static 192.168.10.254 209.165.201.5
```

## Dynamic NAT with a pool (example)
```txt
ip nat pool NAT-POOL1 209.165.200.226 209.165.200.240 netmask 255.255.255.224
ip nat inside source list 1 pool NAT-POOL1
```

## PAT (NAT overload) examples

**Overload using the outside interface (common ISP edge)**
```txt
ip nat inside source list 1 interface serial 0/1/0 overload
```

**Overload using a public pool**
```txt
ip nat inside source list 1 pool NAT-POOL1 overload
```

## Verification / troubleshooting
```txt
show ip nat translations
show ip nat translations verbose
show ip nat statistics
```

## Clearing translations / stats (use with care)
```txt
clear ip nat translation *
clear ip nat translation
clear ip nat statistics
```

## Common issues & notes
- **Wrong inside/outside tagging** → no translations or translations on the wrong interface.
- **ACL/pool mismatch** → translations never created (no “interesting” traffic matches).
- **Routing problems** → NAT can translate correctly but traffic still fails due to missing routes.
- **NAT and IPsec**: NAT modifies IP headers and can **complicate IPsec** (conceptual exam point).
