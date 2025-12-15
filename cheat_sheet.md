# Network Commands & Formulas Cheat Sheet (BMIT3084)

## Subnetting & Wildcards
- Hosts per subnet: `2^(32 - prefix) - 2` (usable hosts).
- Subnets from borrowed bits: `2^borrowed`.
- Broadcast: last address in subnet; first usable = network+1.
- Wildcard (ACL/NAT): `wildcard = 255.255.255.255 - subnet mask` (e.g., /24 -> 0.0.0.255).

## Static Routing
```
ip route <dest> <mask> <next-hop | exit-intf> [distance]
ip route 0.0.0.0 0.0.0.0 10.0.0.1           ! default
ip route 192.168.50.0 255.255.255.0 g0/0 5  ! floating backup
```
- Verify: `show ip route static`, `show ip route`, `traceroute <ip>`.

## Single-Area OSPFv2
```
router ospf 10
 router-id 1.1.1.1
 network 192.168.10.0 0.0.0.255 area 0
 passive-interface g0/1
```
- Point interface method: `int g0/0` → `ip ospf 10 area 0`.
- Verify: `show ip ospf neighbor`, `show ip ospf interface brief`, `show ip route ospf`.

## Standard ACLs (IPv4)
```
ip access-list standard MGT
 10 permit 10.10.10.0 0.0.0.255
 20 deny any
interface g0/0
 ip access-group MGT in
```
- VTY restrict: `line vty 0 4` → `access-class MGT in`.
- Place near destination; order matters; implicit deny at end.
- Verify: `show access-lists`, `show ip interface g0/0`.

## Extended ACLs (IPv4)
```
ip access-list extended EDGE-IN
 10 remark Allow HQ HTTPS
 20 permit tcp 10.1.1.0 0.0.0.255 host 198.51.100.10 eq 443
 40 permit icmp any any echo
 900 deny ip any any log
interface g0/1
 ip access-group EDGE-IN in
```
- Place near source; match protocol + ports (`eq`, `range`, `lt/gt/neq`, `host`).
- Verify hits: `show access-lists`, watch counters/logs.

## NAT for IPv4
- Inside/outside tagging:
  ```
  int g0/1
   ip address 192.168.10.1 255.255.255.0
   ip nat inside
  int g0/0
   ip address 203.0.113.2 255.255.255.0
   ip nat outside
  ```
- Static 1:1: `ip nat inside source static 192.168.10.10 203.0.113.10`
- Static PAT (port forward): `ip nat inside source static tcp 192.168.10.10 80 203.0.113.10 8080`
- Dynamic pool:  
  ```
  ip access-list standard NAT-INSIDE
   permit 192.168.10.0 0.0.0.255
  ip nat pool PUB 203.0.113.100 203.0.113.110 netmask 255.255.255.0
  ip nat inside source list NAT-INSIDE pool PUB
  ```
- PAT overload (common Internet): `ip nat inside source list NAT-INSIDE interface g0/0 overload`
- Verify: `show ip nat translations [verbose]`, `show ip nat statistics`; clear: `clear ip nat translation *`.

## DHCPv4 on Cisco IOS
- Exclude statics: `ip dhcp excluded-address 192.168.10.1 192.168.10.20`
- Server pool:
  ```
  ip dhcp pool BRANCH
   network 192.168.10.0 255.255.255.0
   default-router 192.168.10.1
   dns-server 8.8.8.8 1.1.1.1
   domain-name branch.local
   lease 7
  ```
- Relay: on client VLAN interface → `ip helper-address <dhcp-server-ip>`
- Router as client: `int g0/0` → `ip address dhcp`
- Verify: `show ip dhcp binding`, `show ip dhcp pool`, `show dhcp lease`.

## Secure Management (VTY/SSH)
```
hostname R1
ip domain-name lab.local
crypto key generate rsa modulus 2048
username admin secret <pw>
line vty 0 4
 transport input ssh
 login local
 ip access-class MGT in
```
- Verify: `show ip ssh`, `show users`.

## Useful Show/Debug
- Routing/NH: `show ip route`, `show cdp neighbors detail`
- Interface: `show ip interface brief`, `show interfaces g0/0`, `show ip interface g0/0`
- ACL hits: `show access-lists`
- NAT: `show ip nat translations`, `show ip nat statistics`
- DHCP: `show ip dhcp binding`, `debug ip dhcp server events` (turn off after use)
- OSPF: `show ip ospf neighbor`, `show ip ospf interface`, `debug ip ospf adj` (use carefully)
