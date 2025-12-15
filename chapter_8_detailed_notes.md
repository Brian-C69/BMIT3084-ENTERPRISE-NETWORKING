# Chapter 8 - VPN and IPsec Concepts (ENSA v7.0, Module 8)

## VPN basics
- A VPN is **virtual** (runs over a public network) and **private** (traffic is encrypted).
- Used to secure traffic across the internet for:
  - **Site-to-site VPNs** (gateway-to-gateway)
  - **Remote-access VPNs** (client-to-gateway)

## VPN types (high level)
- **Site-to-site IPsec VPN**: gateways encrypt/decrypt; internal hosts are usually unaware of the VPN.
- **Remote-access VPN**: created dynamically when users connect; commonly **IPsec** or **SSL VPN**.
- **Enterprise / service provider VPNs**: provider-based VPN services (conceptual).

## GRE over IPsec (why it exists)
- **Standard IPsec (non-GRE)** secures **unicast** traffic.
- **GRE** can carry multicast/broadcast and routing protocol updates (e.g., OSPF), but **is not encrypted by default**.
- **GRE over IPsec**: GRE provides tunneling for “non-unicast/routing” needs; IPsec provides security.

## DMVPN (concept)
- Scalable hub-and-spoke VPN design:
  - Hub and spokes use **mGRE** so one tunnel interface can support multiple IPsec tunnels.
  - Can support spoke-to-spoke tunnels (dynamic).

## IPsec Virtual Tunnel Interface (VTI) (concept)
- Applies IPsec to a **virtual interface** rather than static crypto maps on a physical interface.
- Supports **unicast and multicast encrypted traffic**, so routing protocols can work without GRE.

## IPsec framework essentials
- IPsec security goals: **confidentiality**, **integrity**, **origin authentication**, key exchange via **Diffie-Hellman**.
- **Protocols**:
  - **AH**: integrity/authentication (no encryption)
  - **ESP**: encryption + integrity (common)
- **IKE**: negotiates SAs/keys between peers.
- **SA (Security Association)**: agreed set of algorithms/keys/parameters.
- **Transport vs tunnel mode**: tunnel mode commonly used for site-to-site VPNs.

## NAT and IPsec (important note)
- NAT can interfere with tunneling/security because it changes IP headers (common conceptual question).

## Common verification commands (Cisco IOS)
> Exact commands depend on platform/features; these are common “show” checks.
```txt
show crypto session
show crypto isakmp sa
show crypto ipsec sa
```
