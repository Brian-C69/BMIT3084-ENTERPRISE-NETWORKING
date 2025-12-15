# BMIT3084 Enterprise Networking – Summary Notes

## SRWE: Static Routing & Troubleshooting
- **Static route types:** standard (`ip route <dest> <mask> {next-hop|exit-intf}` / `ipv6 route`), default (`0.0.0.0/0` or `::/0`), floating (higher AD for backup), and host routes (/32, /128).
- **Defaults & recursion:** next-hop-only routes require recursive lookup; exit-interface routes are single lookup. Use fully specified static routes on multi-access links to avoid ARP/ND churn.
- **Verification/troubleshooting:** `show ip route`, `show ipv6 route`, `show ip route static`, `traceroute`, `ping`, check interface status and correct AD; ensure matching masks and reachable next-hop. Default routes must be advertised or redistributed if dynamic protocols are in use.
- **Common issues:** wrong prefix/length, missing outbound interface, next-hop unreachable, asymmetric paths, forgotten `ip route 0.0.0.0/0` for internet edge.

## ENSA: Single-Area OSPFv2 (Concepts & Configuration)
- **Why OSPF:** link-state protocol with fast convergence, hierarchical areas (Area 0 backbone here single-area), uses cost (1 Gbps = cost 1 by default in modern IOS).
- **Operation essentials:** router ID selection (manual > highest loopback > highest active interface), Hello/dead timers build adjacencies, DR/BDR election on multi-access segments, LSAs flood within the area, SPF builds the LSDB and routing table.
- **Config basics:** `router ospf <pid>` + `network <addr> <wildcard> area 0` or interface `ip ospf <pid> area 0`; set `ip ospf priority` for DR/BDR influence; `ip ospf cost` or adjust interface bandwidth to tune metrics; use `passive-interface` for security/no neighbors.
- **Verification:** `show ip ospf neighbor`, `show ip ospf interface`, `show ip ospf database`, `show ip route ospf`.

## ENSA: Network Security Concepts
- **Foundations:** protect assets via CIA triad, least privilege, and layered defenses; understand threats, vulnerabilities, risks, exploits, and mitigations.
- **Threat landscape:** internal and external attack vectors (phishing, malware, misconfig, rogue devices); data-loss vectors (email/social, cloud, removable media, lost devices, weak access control).
- **Actors:** script kiddies, vulnerability brokers, hacktivists, cybercriminals, and state-sponsored groups; motivations range from curiosity to profit or espionage.
- **Mitigations:** strong identity/authN, patching, segmentation, secure configs, backups, logging/monitoring, DLP controls, and user awareness.

## ENSA: ACL Concepts & IPv4 ACL Configuration
- **Purpose:** traffic filtering, hardening, and basic policy enforcement; processed top-down, first match wins, implicit deny at end.
- **Types/placement:** standard ACL (source-only) nearest destination; extended ACL (source/dest/protocol/ports) nearest source. Named ACLs preferred for readability.
- **Wildcards:** inverse masks for matching subnets/hosts (`0.0.0.0` host, `0.0.0.255` /24). Use `any`/`host` shortcuts and sequence numbers for edits.
- **Apply & verify:** `ip access-group <acl> {in|out}` on interfaces; `ip access-class` for VTY; `show access-lists`, `show ip interface`.

## ENSA: NAT for IPv4
- **Drivers:** conserve IPv4, hide internal addressing, enable overlapping migrations.
- **Flavors:** static 1:1, dynamic pool, PAT/overload (many-to-one), and policy NAT. Define inside/outside interfaces, pools, and ACLs for interesting traffic.
- **Key commands:** `ip nat inside source static <in> <out>`, `ip nat inside source list <acl> pool <name> overload`, `ip nat pool <name> <start> <end> netmask <mask>`.
- **Troubleshooting:** `show ip nat translations`, `show ip nat statistics`, correct ACL direction, ensure routes to pools/inside networks.

## ENSA: DHCPv4
- **Process:** DORA (Discover, Offer, Request, Ack); lease timers negotiated.
- **Config on router:** exclude statics (`ip dhcp excluded-address`), define pool (`ip dhcp pool <name>`), `network`, `default-router`, `dns-server`, options (e.g., `option 150` for VoIP), and `lease`.
- **Relays:** use `ip helper-address <server-ip>` on interfaces to forward DHCP (and other UDP services). Verify with `show ip dhcp binding`, `show ip dhcp server statistics`.

## ENSA: VPN & IPsec Concepts
- **Use cases:** secure site-to-site and remote-access connectivity over untrusted networks.
- **Building blocks:** encapsulation + encryption + integrity + authentication. IPsec phases (IKE SA, then IPsec SA), modes (tunnel vs transport), protocols (ESP for confidentiality/integrity, AH for integrity only), hashing (SHA), ciphers (AES), DH for key exchange, PSK/PKI for identity.
- **VPN types:** MPLS/VPN services, GRE+IPsec site-to-site, SSL client VPNs; choose based on scale, mobility, and app needs.

## ENSA: WAN Concepts
- **Purpose/operation:** connect LANs across geography using service-provider infrastructure; customer edge (CE) to provider edge (PE) demarcation.
- **Traditional options:** leased lines/HDLC/PPP, Frame Relay/MPLS, Metro-E. **Modern:** broadband (DSL/cable), fiber, cellular/5G, satellite, and SD-WAN overlays with VPN/IPsec.
- **Design notes:** balance cost vs bandwidth/SLAs; redundancy via dual providers/paths; QoS and security overlays (IPsec) often required on internet circuits.

## ENSA: QoS Concepts
- **Why QoS:** manage limited bandwidth to protect delay/jitter-sensitive voice/video and critical apps from congestion.
- **Key mechanisms:** classification/marking (CoS/DSCP), queuing (FIFO, WFQ, CBWFQ, LLQ for strict priority), congestion avoidance (WRED), policing vs shaping, link-efficiency tools (compression, LFI).
- **Models:** best-effort (no QoS), IntServ/RSVP (per-flow guarantees), DiffServ (per-hop behaviors with DSCP classes) as the scalable default.

## ENSA: Network Management
- **Discovery:** CDP (Cisco) and LLDP (open) to map neighbors; enable/verify, limit scope for security.
- **Time:** NTP client/server for consistent timestamps; prefer authenticated NTP.
- **Monitoring:** SNMP manager/agent/MIB with gets/sets/traps; choose SNMPv3 for auth+privacy. Syslog levels 0–7, central collectors recommended; ensure timestamps and `logging buffered`.
- **File/IOS care:** back up and restore configs/images (`copy run start`, `copy run tftp`, `copy tftp flash`, `verify /md5`, `boot system`), keep version control and golden images.

## Quick Command Reminders
`ip route 0.0.0.0 0.0.0.0 <next-hop>` (default) • `ip route <dest> <mask> <next-hop> <AD>` (floating) • `router ospf 1` + `network ... area 0` • `ip access-group <ACL> in|out` • `ip nat inside/outside` + `ip nat inside source ... overload` • `ip dhcp pool <name>` + `network/default-router` • `ip helper-address <server>` • CDP/LLDP: `show cdp neighbors` / `show lldp neighbors` • Syslog: `logging host <ip>`, `service timestamps log datetime msec` • NTP: `ntp server <ip>`; `show ntp status`.
