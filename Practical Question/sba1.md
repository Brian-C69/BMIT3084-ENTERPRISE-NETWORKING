SBA configuration quick commands (paste per device)

Topology reference
- Save the provided topology screenshot as `topology.png` in this folder.
- If the filename differs, adjust the link below accordingly.

`![Topology](topology.png)`

NAT
```
conf t
!
interface g0/0
 ip nat inside
interface g0/1
 ip helper-address 172.16.20.2
 ip nat inside
interface s0/0/1
 ip nat outside
!
interface tunnel1
 ip address 172.16.1.2 255.255.255.252
 mtu 1476
 tunnel source s0/0/1
 tunnel destination 208.165.100.6
interface tunnel2
 ip address 172.16.2.2 255.255.255.252
 tunnel source s0/0/1
 tunnel destination 208.165.100.10
!
router ospf 10
 router-id 1.1.1.1
 auto-cost reference-bandwidth 100000
 network 172.16.20.0 0.0.0.255 area 0
 network 172.16.30.0 0.0.0.255 area 0
 passive-interface g0/1
 default-information originate
!
ip route 0.0.0.0 0.0.0.0 208.165.100.1
ip route 10.10.10.128 255.255.255.128 172.16.1.1
ip route 10.10.10.0   255.255.255.128 172.16.2.1
!
ip nat pool NAT-TARUCPOOL 208.165.102.1 208.165.102.5 netmask 255.255.248
ip access-list extended NAT-ACL
 permit ip 172.16.10.0 0.0.0.255 any
 permit ip 172.16.30.0 0.0.0.255 any
ip nat inside source list NAT-ACL pool NAT-TARUCPOOL overload
ip nat inside source static 172.16.30.254 208.165.102.6
!
ip access-list standard OFFICE-ACL
 permit host 172.16.10.128
line vty 0 4
 access-class OFFICE-ACL in
 password cisco
 login
 transport input telnet
!
ntp server 172.16.30.254
logging 172.16.30.254
end
wr
```

DHCP router
```
conf t
interface g0/0
 ip address 172.16.20.2 255.255.255.0
 no shut
interface g0/1
 ip address 172.16.10.1 255.255.255.0
 ip access-group 188 in
 no shut
!
router ospf 10
 router-id 2.2.2.2
 auto-cost reference-bandwidth 100000
 network 172.16.20.0 0.0.0.255 area 0
 network 172.16.10.0 0.0.0.255 area 0
 passive-interface g0/1
 default-information originate
!
ip dhcp excluded-address 172.16.10.1 172.16.10.5
ip dhcp pool POOL-CK1
 network 172.16.10.0 255.255.255.0
 default-router 172.16.10.1
 dns-server 172.16.30.254
ip dhcp excluded-address 172.16.30.1 172.16.30.5
ip dhcp pool POOL-Kong2
 network 172.16.30.0 255.255.255.0
 default-router 172.16.30.1
 dns-server 172.16.30.254
!
access-list 188 permit udp any eq 68 any eq 67
access-list 188 permit udp 172.16.10.0 0.0.0.255 host 172.16.30.254 eq 53
access-list 188 permit tcp 172.16.10.0 0.0.0.255 host 172.16.30.254 eq 53
access-list 188 permit tcp 172.16.10.128 0.0.0.127 host 172.16.30.254 eq 443
access-list 188 permit tcp host 172.16.10.128 host 172.16.30.254 eq 443
access-list 188 permit tcp 172.16.10.128 0.0.0.127 host 172.16.30.254 eq 21
access-list 188 permit tcp 172.16.10.128 0.0.0.127 host 172.16.30.254 eq 20
access-list 188 permit ip 172.16.10.0 0.0.0.255 208.165.103.0 0.0.0.255
access-list 188 permit tcp host 172.16.10.128 host 172.16.20.1 eq 23
access-list 188 permit ip 172.16.10.0 0.0.0.255 172.16.30.0 0.0.0.255
access-list 188 permit icmp 172.16.10.0 0.0.0.255 host 172.16.10.1
end
wr
```

ISP
```
conf t
interface g0/1
 ip address 208.165.103.1 255.255.255.0
 no shut
interface s0/0/0
 ip address 208.165.100.5 255.255.255.252
 clock rate 64000
 no shut
interface s0/0/1
 ip address 208.165.100.1 255.255.255.252
 clock rate 64000
 no shut
interface s0/1/0
 ip address 208.165.100.9 255.255.255.252
 clock rate 64000
 no shut
ip route 172.16.10.0 255.255.255.0 208.165.100.2
ip route 172.16.30.0 255.255.255.0 208.165.100.2
ip route 10.10.10.0 255.255.255.128 208.165.100.10
ip route 10.10.10.128 255.255.255.128 208.165.100.6
end
wr
```

VPN1
```
conf t
interface s0/0/0
 ip address 208.165.100.6 255.255.255.252
 no shut
interface g0/1
 ip address 10.10.10.1 255.255.255.0
 no shut
interface tunnel1
 ip address 172.16.1.1 255.255.255.252
 tunnel source s0/0/0
 tunnel destination 208.165.100.2
ip route 0.0.0.0 0.0.0.0 208.165.100.5
ip route 172.16.0.0 255.255.0.0 172.16.1.2
end
wr
```

VPN2
```
conf t
interface s0/0/0
 ip address 208.165.100.10 255.255.255.252
 no shut
interface g0/1
 ip address 10.10.10.2 255.255.255.0
 no shut
interface tunnel2
 ip address 172.16.2.1 255.255.255.252
 tunnel source s0/0/0
 tunnel destination 208.165.100.2
ip route 0.0.0.0 0.0.0.0 208.165.100.9
ip route 172.16.0.0 255.255.0.0 172.16.2.2
interface s0/0/0
 no cdp enable
end
wr
```

CDP disable on VPN1
```
conf t
no cdp run
end
wr
```
