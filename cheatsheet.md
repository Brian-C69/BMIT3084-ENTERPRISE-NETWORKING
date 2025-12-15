# BMIT3084 Enterprise Networking - Commands Cheatsheet

> Cisco IOS/IOS-XE style syntax (Packet Tracer/GNS3). Replace interface names/IPs to match your topology.

## Subnetting & Wildcards (used in ACL/OSPF)

- Hosts per subnet (usable): `2^(32 - prefix) - 2`
- Subnets from borrowed bits: `2^borrowed`
- Broadcast address: last address in subnet (first usable = network + 1)
- Network address (formula): `network = IP AND subnet-mask`
- Broadcast address (formula): `broadcast = network OR wildcard` (where `wildcard = NOT subnet-mask`)
- Wildcard mask (ACL/OSPF): `wildcard = 255.255.255.255 - subnet mask`
  - `/24` mask `255.255.255.0` -> wildcard `0.0.0.255`
  - Host match -> wildcard `0.0.0.0` (or keyword `host`)
  - Any -> `0.0.0.0 255.255.255.255` (or keyword `any`)

---

## Chapter 1 - Static Routing (IPv4/IPv6)

### Command formats (syntax)
```txt
ip route <DEST-NET> <MASK> {<NEXT-HOP-IP> | <EXIT-INTF> | <NEXT-HOP-IP> <EXIT-INTF>} [ADMIN_DISTANCE]
ip route 0.0.0.0 0.0.0.0 {<NEXT-HOP-IP> | <EXIT-INTF>} [ADMIN_DISTANCE]
ipv6 unicast-routing
ipv6 route <PREFIX/LEN> {<NEXT-HOP-IPv6> | <EXIT-INTF> | <NEXT-HOP-IPv6> <EXIT-INTF>} [ADMIN_DISTANCE]
ipv6 route ::/0 {<NEXT-HOP-IPv6> | <EXIT-INTF>} [ADMIN_DISTANCE]
```

### IPv4 static routes

**Standard static (specific prefix)**
```txt
ip route <DEST-NET> <MASK> <NEXT-HOP>
ip route 10.10.20.0 255.255.255.0 192.0.2.2
```

**Exit-interface only (best on point-to-point)**
```txt
ip route 10.10.30.0 255.255.255.0 s0/0/0
```

**Fully specified (next-hop + exit interface; good on Ethernet)**
```txt
ip route 10.10.40.0 255.255.255.0 192.0.2.2 g0/0
```

**Default route**
```txt
ip route 0.0.0.0 0.0.0.0 198.51.100.1
```

**Floating static (backup; higher AD than primary)**
```txt
ip route 0.0.0.0 0.0.0.0 203.0.113.1 120
```

### IPv6 static routes

**Enable routing**
```txt
ipv6 unicast-routing
```

**Standard static**
```txt
ipv6 route 2001:db8:10:20::/64 2001:db8:12:1::2
```

**Link-local next-hop (must include exit interface)**
```txt
ipv6 route 2001:db8:10:30::/64 fe80::2 g0/0
```

**Default route**
```txt
ipv6 route ::/0 2001:db8:12:1::2
```

### Verify / troubleshoot static & default routes
```txt
show ip route
show ip route static
show ipv6 route
show ipv6 route static
show ip cef <PREFIX>
show ipv6 cef <PREFIX>
show ip arp
show ipv6 neighbors
show ip interface brief
show interface <INTF>
ping <IP>
traceroute <IP>
```

---

## Chapter 2 - Single-Area OSPFv2 (IPv4)

### Command formats (syntax)
```txt
router ospf <PROCESS-ID>
 router-id <A.B.C.D>
 network <IP> <WILDCARD> area <AREA-ID>
 passive-interface <INTF>
!
interface <INTF>
 ip ospf <PROCESS-ID> area <AREA-ID>
```

### OSPF cost + bandwidth (auto-cost) quick notes
- Default OSPF interface cost is derived from bandwidth:
  - `cost = reference_bandwidth / interface_bandwidth`
  - IOS default reference bandwidth is often `100 Mbps` (so fast links may all look like cost 1 unless you change it).
- Two common ways to influence cost:
  - Set explicit cost on an interface (overrides auto): `ip ospf cost <COST>`
  - Set correct interface bandwidth and adjust reference bandwidth:
    - Interface: `bandwidth <Kbps>` (affects OSPF/EIGRP metric calculations; does not change actual physical speed)
    - OSPF global: `auto-cost reference-bandwidth <Mbps>`

### Commands (cost/bandwidth tuning)
```txt
interface <INTF>
 bandwidth <Kbps>
 ip ospf cost <COST>
!
router ospf <PROCESS-ID>
 auto-cost reference-bandwidth <Mbps>
```

### Verify cost/bandwidth
```txt
show ip ospf interface <INTF>
show ip ospf
```

### Wildcard for `network` statement (formula)
- `wildcard = 255.255.255.255 - subnet mask`
- Example: `/30` mask `255.255.255.252` -> wildcard `0.0.0.3`

### Basic configuration (router mode)
```txt
router ospf 1
 router-id 1.1.1.1
 network 10.0.0.0 0.0.0.255 area 0
 network 192.168.1.0 0.0.0.3 area 0
 passive-interface g0/2
```

### Router ID (RID) selection (precedence)
- 1) Manually configured `router-id A.B.C.D`
- 2) Highest IP address on any loopback interface
- 3) Highest IP address on any active (up/up) non-loopback interface
- RID is "sticky" after OSPF starts; apply changes with `clear ip ospf process`.

### Alternative configuration (per-interface)
```txt
interface g0/0
 ip address 10.0.0.1 255.255.255.0
 ip ospf 1 area 0
!
interface s0/0/0
 ip address 192.168.1.1 255.255.255.252
 ip ospf 1 area 0
```

### Tuning (common)
```txt
interface g0/0
 ip ospf cost 10
!
interface g0/1
 ip ospf priority 200
!
clear ip ospf process
```

### Verify / troubleshoot OSPF
```txt
show ip ospf neighbor
show ip ospf interface
show ip ospf interface brief
show ip ospf database
show ip route ospf
show ip protocols
```

---

## Chapter 3 - Network Security (Device Hardening & Secure Management)

### Command formats (syntax)
```txt
enable secret <PASSWORD>
username <USER> secret <PASSWORD>
ip domain-name <DOMAIN>
crypto key generate rsa modulus <BITS>
ip ssh version 2
line vty 0 4
 transport input ssh
 login local
```

### Essentials: local auth + SSH-only VTY
```txt
enable secret <STRONG_PASSWORD>
service password-encryption
hostname R1
ip domain-name example.local
username admin secret <STRONG_PASSWORD>
crypto key generate rsa modulus 2048
ip ssh version 2
```

### Console / VTY hygiene
```txt
line con 0
 exec-timeout 5 0
 logging synchronous
 login local
!
line vty 0 4
 transport input ssh
 exec-timeout 5 0
 login local
```

### Brute-force throttling
```txt
login block-for 60 attempts 5 within 60
```

### AAA (basic local)
```txt
aaa new-model
aaa authentication login default local
aaa authorization exec default local
```

### Legal banner
```txt
banner motd ^CAuthorized access only!^C
```

---

## Chapter 4-5 - ACL Concepts & IPv4 ACL Configuration

### ACL processing rules (precedence)
- ACLs are processed top-down; first match wins.
- Implicit `deny` at the end (add an explicit deny with `log` if you want visibility).

### Command formats (syntax)

**Standard (named)**
```txt
ip access-list standard <NAME>
 <SEQ> permit <SRC> <WILDCARD>
 <SEQ> deny <SRC> <WILDCARD> [log]
interface <INTF>
 ip access-group <NAME> in|out
```

**Extended (named)**
```txt
ip access-list extended <NAME>
 <SEQ> permit|deny <PROTO> <SRC> <SRC-WC> <DST> <DST-WC> [eq|range|gt|lt|neq <PORT>] [log]
interface <INTF>
 ip access-group <NAME> in|out
```

**VTY management ACL**
```txt
line vty 0 4
 access-class <STD-ACL-NAME> in
```

### Apply ACL inbound vs outbound (which interface/port)
- Inbound (`ip access-group <ACL> in`): filters packets as they enter the interface (before routing lookup).
- Outbound (`ip access-group <ACL> out`): filters packets after routing decision, before they leave the interface.
- Standard ACLs (source only): usually place near the destination.
- Extended ACLs (src/dst/proto/ports): usually place near the source.
- VTY protection (SSH/Telnet): apply with `line vty ...` and `access-class <ACL> in` (not `ip access-group`).

### Standard ACL (named) + apply to VTY (management restriction)
```txt
ip access-list standard MGT-VTY
 10 permit 10.10.10.0 0.0.0.255
 20 deny any
!
line vty 0 4
 access-class MGT-VTY in
 transport input ssh
```

### Standard ACL (numbered) + apply to interface
```txt
access-list 10 permit 192.168.10.0 0.0.0.255
interface g0/0
 ip access-group 10 in
```

### Extended ACL (named) + apply inbound on an interface
```txt
ip access-list extended EDGE-IN
 10 remark Allow branch HTTPS to HQ server
 20 permit tcp 10.1.1.0 0.0.0.255 host 198.51.100.10 eq 443
 30 remark Allow ICMP for testing
 40 permit icmp any any echo
 50 permit icmp any any echo-reply
 900 deny ip any any log
!
interface g0/1
 ip access-group EDGE-IN in
```

### Port/operator keywords (extended ACL)
```txt
eq 80
eq 443
range 10000 20000
gt 1023
lt 1024
neq 53
```

### Numbered extended ACL examples (common services)

**DHCP (client/server ports)**
- `bootps` = UDP/67 (server), `bootpc` = UDP/68 (client)
```txt
access-list 188 permit udp any eq bootpc any eq bootps
access-list 188 permit udp any eq bootps any eq bootpc
```

**Other common permits (examples)**
```txt
access-list 188 permit udp any any eq domain     ! DNS (53)
access-list 188 permit udp any any eq ntp        ! NTP (123)
access-list 188 permit tcp any any eq ssh        ! SSH (22)
access-list 188 permit tcp any any eq www        ! HTTP (80)
access-list 188 permit tcp any any eq 443        ! HTTPS (443)
access-list 188 permit icmp any any              ! ICMP
```

### Verify / troubleshoot ACLs
```txt
show access-lists
show ip access-lists <NAME>
show ip interface <INTF>
show run interface <INTF>
debug ip packet
```

---

## Chapter 6 - DHCPv4 (Server, Relay, Client)

### Command formats (syntax)
```txt
ip dhcp excluded-address <START-IP> [END-IP]
ip dhcp pool <NAME>
 network <NETWORK> <MASK>
 default-router <GATEWAY-IP>
 dns-server <DNS1> [DNS2]
 domain-name <DOMAIN>
 lease <DAYS> [HOURS] [MINUTES]
!
interface <CLIENT-LAN-INTF>
 ip helper-address <DHCP-SERVER-IP>
!
interface <INTF>
 ip address dhcp
```

### DHCP server on Cisco IOS
```txt
ip dhcp excluded-address 192.168.10.1 192.168.10.20
ip dhcp pool BRANCH-LAN
 network 192.168.10.0 255.255.255.0
 default-router 192.168.10.1
 dns-server 8.8.8.8 1.1.1.1
 domain-name branch.local
 lease 7
 option 150 ip 192.168.10.5
```

### DHCP relay (helper address)
```txt
interface g0/1
 ip address 192.168.20.1 255.255.255.0
 ip helper-address 10.10.10.5
```

### Router interface as DHCP client
```txt
interface g0/0
 ip address dhcp
 no shutdown
```

### Verify / debug / clear
```txt
show ip dhcp binding
show ip dhcp pool
show ip dhcp server statistics
debug ip dhcp server events
debug ip dhcp server packets
clear ip dhcp binding *
show dhcp lease
```

---

## Chapter 7 - NAT for IPv4

### Command formats (syntax)
```txt
interface <INSIDE-INTF>
 ip nat inside
interface <OUTSIDE-INTF>
 ip nat outside
!
ip nat inside source static <INSIDE-LOCAL-IP> <INSIDE-GLOBAL-IP>
ip nat inside source static tcp <INSIDE-LOCAL-IP> <LOCAL-PORT> <INSIDE-GLOBAL-IP> <GLOBAL-PORT>
ip nat pool <POOL-NAME> <START-IP> <END-IP> netmask <MASK>
ip nat inside source list <ACL|ACL-NUM> pool <POOL-NAME> [overload]
ip nat inside source list <ACL|ACL-NUM> interface <OUTSIDE-INTF> overload
ip nat translation timeout <SECONDS>
```

### Tag inside/outside
```txt
interface g0/1
 ip nat inside
!
interface s0/1/0
 ip nat outside
```

### NAT "interesting traffic" ACL (example)
```txt
access-list 1 permit 192.168.10.0 0.0.0.255
```

### Static NAT (1:1)
```txt
ip nat inside source static 192.168.10.254 209.165.201.5
```

### Static PAT (port forwarding)
```txt
ip nat inside source static tcp 192.168.10.10 80 209.165.200.10 8080
```

### Dynamic NAT pool
```txt
ip nat pool NAT-POOL1 209.165.200.226 209.165.200.240 netmask 255.255.255.224
ip nat inside source list 1 pool NAT-POOL1
```

### PAT / NAT overload (many-to-one)
**Overload using a public pool**
```txt
ip nat inside source list 1 pool NAT-POOL1 overload
```

**Overload using the outside interface (most common Internet edge)**
```txt
ip nat inside source list 1 interface serial 0/1/0 overload
```

### Verify / tune / clear NAT
```txt
show ip nat translations
show ip nat translations verbose
show ip nat statistics
ip nat translation timeout <SECONDS>
clear ip nat translation *
clear ip nat statistics
```

---

## Chapter 8 - VPN and IPsec Concepts (common \"show\" checks)
```txt
show crypto session
show crypto isakmp sa
show crypto ipsec sa
```

---

## Chapter 9 - WAN Concepts (basic operational checks)
```txt
show ip interface brief
show interfaces
show interfaces serial 0/0/0
```

---

## Chapter 10 - QoS Concepts (MQC pattern + verification)
### Command formats (syntax)
```txt
class-map match-any VOICE
 match protocol rtp
policy-map QOS-OUT
 class VOICE
  priority percent 10
 class class-default
  fair-queue
interface g0/0
 service-policy output QOS-OUT
show policy-map interface g0/0
```

---

## Chapter 11 - Network Management (CDP/LLDP/NTP/Syslog/SNMP)

### CDP / LLDP discovery
### Command formats (syntax)
```txt
cdp run
no cdp run
show cdp
show cdp interface
show cdp neighbors
show cdp neighbors detail
lldp run
no lldp run
show lldp
show lldp neighbors
show lldp neighbors detail
```

### Time (clock/NTP)
```txt
clock set hh:mm:ss mm dd yyyy
ntp server <NTP_SERVER_IP>
show ntp associations
show ntp status
```

### Log timestamps
```txt
service timestamps log datetime
```

### SNMP (common lab pattern)
```txt
snmp-server community <COMMUNITY> RO
```

### Save/backup (common lab workflows)
```txt
copy running-config startup-config
```

### Backup config to FTP (copy run to ftp)
```txt
copy running-config ftp:
Address or name of remote host []? <FTP_SERVER_IP>
Destination filename [running-config]? <R1-running-config>
```


---

## Useful Show/Debug (quick list)
- Routing/next-hop: `show ip route`, `traceroute <IP>`
- Interfaces: `show ip interface brief`, `show interfaces g0/0`, `show ip interface g0/0`
- OSPF: `show ip ospf neighbor`, `show ip ospf interface`, `debug ip ospf adj` (use carefully)
- ACL hits: `show access-lists`
- NAT: `show ip nat translations`, `show ip nat statistics`
- DHCP: `show ip dhcp binding`, `debug ip dhcp server events` (turn off after use)

---

## Extra Tips (common exam/lab points)

### Layer 3 switch notes (IOS multilayer switch)
- L3 switches use very similar Cisco IOS syntax, plus L3 features:
  - Enable routing: `ip routing`
  - Inter-VLAN routing via SVIs:
    ```txt
    interface vlan 10
     ip address 192.168.10.1 255.255.255.0
    interface vlan 20
     ip address 192.168.20.1 255.255.255.0
    ```
  - Routed port (no switchport):
    ```txt
    interface g0/1
     no switchport
     ip address 10.0.0.2 255.255.255.252
    ```
- Apply ACLs to the correct L3 interface (SVI `vlan X` or routed port) and correct direction (`in`/`out`).

### Administrative Distance (AD) (definition + quick reference)
- AD is the "trust" of a route source; lower AD is preferred.
- Common defaults (Cisco):
  - Connected: `0`
  - Static: `1` (unless you set a different distance)
  - OSPF: `110`
- Floating static route = static route with a higher AD than the primary route (so it is used only on failure).

### Route selection precedence (how routers pick a route)
1) Longest prefix match (most specific route wins)
2) If tie, lowest AD wins
3) If tie, lowest metric wins (protocol-specific, e.g., OSPF cost)
4) If still tie, equal-cost load balancing may occur

### Routing table codes (Cisco IOS `show ip route`)
- `L` - Local (host route for the interface IP, /32)
- `C` - Connected (directly connected network)
- `S` - Static
- `R` - RIP
- `M` - Mobile
- `B` - BGP
- `D` - EIGRP
- `EX` - EIGRP external
- `O` - OSPF (intra-area)
- `O IA` - OSPF inter-area
- `O E1` - OSPF external type 1
- `O E2` - OSPF external type 2
- `O N1` - OSPF NSSA external type 1
- `O N2` - OSPF NSSA external type 2
- `i` - IS-IS
- `su` - IS-IS summary
- `L1` - IS-IS level-1
- `L2` - IS-IS level-2
- `ia` - IS-IS inter area
- `*` - Candidate default route
- `U` - Per-user static route
- `o` - ODR (On-Demand Routing)
- `P` - Periodic downloaded static route
- `H` - NHRP
- `l` - LISP
- `a` - Application route

### IPv6 routing table codes (Cisco IOS `show ipv6 route`)
- Common ones you will see in labs: `L`, `C`, `S`, `R`, `O`, `O IA`, `O E1/E2`, `O N1/N2`, `B`
