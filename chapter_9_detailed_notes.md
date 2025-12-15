# Chapter 9 - WAN Concepts (ENSA v7.0, Module 7)

## Purpose of WANs
- **LAN**: small area, owned/managed by the organization, high-speed Ethernet/Wi‑Fi.
- **WAN**: connects remote sites/users over long distance; typically provided by ISPs/providers and paid as a service.

## How WANs operate (big picture)
- Provider networks interconnect customer sites using access circuits and provider backbones.
- WAN bandwidth and service levels vary by technology and contract (SLA).

## Traditional WAN connectivity (declining use)
- **Leased lines**
- **PPP** (less used)
- **HDLC** (less used)
- Serial point-to-point links (fixed capacity, often expensive vs modern broadband/Ethernet WAN)

## Modern WAN connectivity (common today)
- **MPLS**: provider WAN technology using labels; supports multiple access methods (Ethernet, DSL, cable, etc.).
- **Ethernet WAN / Ethernet over MPLS (EoMPLS)**: replacing older serial WANs.
- **Internet-based broadband**: DSL, cable (DOCSIS), fiber (FTTx), cellular, satellite.
- **PPPoE**: common for DSL subscriber access; PPP can authenticate subscribers and assign IP addresses.

## ISP edge topologies (resilience vs cost)
- **Single-homed**
- **Dual-homed**
- **Multihomed**
- **Dual-multihomed** (most resilient, most expensive)

## Quick operational “show” commands (generic)
```txt
show ip interface brief
show interfaces
show interfaces serial 0/0/0
```
