# Chapter 6 – DHCPv4 (ENSA/SRWE v7.0)

## Module Focus
- Implement DHCPv4 across multiple LANs using Cisco IOS (server, relay, client).
- Know lease lifecycle, renewal timers, verification, and common faults.

## DHCPv4 Concepts
- DHCP automates IPv4 addressing and options (mask, gateway, DNS, domain, NTP, option 150 for TFTP/phones).
- Lease model: address is rented; if not renewed it returns to the pool.
- Timers: T1 = 50% lease (unicast renew to original server); T2 = 87.5% (broadcast renew to any server).

## Message Sequence (DORA)
1) DHCPDISCOVER — client broadcast (src 0.0.0.0:68 -> 255.255.255.255:67) to find servers.
2) DHCPOFFER — server offer (unicast or broadcast) with proposed IP + options.
3) DHCPREQUEST — client selects one offer, broadcasts request for that IP.
4) DHCPACK — server confirms lease; client installs config. DHCPNAK forces restart if denied.

## Cisco IOS DHCPv4 Server
1) Exclude static IPs (gateway, servers, printers):
   - `ip dhcp excluded-address 192.168.10.1 192.168.10.20`
2) Create pool:
   ```
   ip dhcp pool BRANCH-LAN
    network 192.168.10.0 255.255.255.0
    default-router 192.168.10.1
    dns-server 8.8.8.8 1.1.1.1
    domain-name branch.local
    lease 7
    option 150 ip 192.168.10.5
   ```
3) Verify: `show ip dhcp binding`, `show ip dhcp pool`, `show ip dhcp server statistics`.
4) Debug sparingly: `debug ip dhcp server events` or `debug ip dhcp server packets`.
5) Clear bindings: `clear ip dhcp binding *` (drops active leases).

## DHCP Relay (Helper)
- Needed when server is off-subnet; router relays broadcasts as unicasts.
- Example:
  ```
  interface g0/1
   ip address 192.168.20.1 255.255.255.0
   ip helper-address 10.10.10.5
  ```
- Helper forwards UDP by default: DHCP/BOOTP, TFTP 69, DNS 53, NetBIOS 137/138, TACACS 49, Time 37. Restrict with `ip forward-protocol udp <port>` or remove with `no ip forward-protocol udp <port>`.

## DHCP Client on Router Interface
```
interface g0/0
 ip address dhcp
 no shutdown
```
- Verify: `show dhcp lease`, `show ip interface g0/0`.

## Common Pitfalls
- Missing `ip helper-address` on access/VLAN interfaces -> clients never get replies.
- Not excluding static addresses -> gateway handed out, causing duplicate IPs.
- Pool exhaustion -> extend pool or shorten lease time.
- Wrong default-router or DNS in pool -> clients cannot reach off-net or resolve names.
- Multiple uncoordinated servers -> conflicting options/addresses; prefer single authoritative per scope.
- Debug left on -> excessive CPU/log noise; disable after use.

## Security Notes
- Rogue DHCP servers can hijack clients; mitigate with DHCP snooping on switches, trust only uplinks, and rate-limit DHCP on untrusted ports.
