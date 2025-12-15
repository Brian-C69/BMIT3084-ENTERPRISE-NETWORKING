# Chapter 4 – ACL Concepts & IPv4 ACL Configuration (ENSA v7.0)

## Module 4: ACL Concepts
- **Purpose:** Filter traffic based on L3/L4 header info; enforce policy, limit management access, reduce attack surface. ACLs evaluated top-down, first match wins, implicit deny at end.
- **ACE basics:** Each line = permit/deny statement; order matters; unnamed (numbered) or named ACLs (preferred for readability/edits).
- **Wildcard masks:** Inverse of subnet mask. `0` = must match bit; `1` = “don’t care”. Examples: `/24` mask `255.255.255.0` → wildcard `0.0.0.255`; host match `0.0.0.0`; any = `0.0.0.0 255.255.255.255` or keyword `any`.
- **Types:** Standard (source-only; numbers 1-99/1300-1999) and Extended (source/dest/protocol/ports; numbers 100-199/2000-2699). Named ACLs can be std/ext with `ip access-list {standard|extended} <NAME>`.
- **Placement rule-of-thumb:** Standard near destination (affects entire source). Extended near source (precise match, stop unwanted traffic early).
- **Direction:** Applied inbound or outbound per interface: `ip access-group <ACL> in|out`. For VTY, use `ip access-class <ACL> in`.
- **Logging:** Add `log` (or `log-input`) to ACEs for counters/messages; beware CPU load on busy devices.
- **Editing:** Named ACLs support sequence numbers (`10 permit ...`, `20 deny ...`); numbered ACLs can be resequenced or rebuilt.

## Module 5: ACLs for IPv4 Configuration
### Standard ACL (example)
```
ip access-list standard MGT-VTY
 10 permit 10.10.10.0 0.0.0.255
 20 deny any
line vty 0 4
 access-class MGT-VTY in
 transport input ssh
```

### Extended ACL (example)
```
ip access-list extended BRANCH-FW
 10 permit tcp 10.1.1.0 0.0.0.255 host 192.0.2.10 eq 443
 20 deny ip any any log
interface g0/1
 ip access-group BRANCH-FW in
```

### Common protocols/keywords
- `tcp`, `udp`, `icmp`, `ip` (any L3), `ospf`, `gre`, `eigrp`, `igmp`.
- Ports: `eq 80`, `eq 443`, `range 10000 20000`, `gt`, `lt`, `neq`.
- `host <ip>` shortcut for `/32`; `any` for 0.0.0.0/0; `object-group` not covered in basic IOS here.

### Guidelines and pitfalls
- Explicitly end with `deny ip any any log` (extended) or `deny any log` (standard) if you want visibility; otherwise implicit drop is silent.
- Avoid applying ACLs in both directions on same interface unless required; ensure return traffic is allowed or handled by stateful device upstream.
- Verify before/after with `show access-lists`, `show ip interface <intf>`, and ping/traceroute.
- Ensure ACLs permit required control traffic (routing protocols, DHCP, DNS) as needed.
- Sequence order carefully; place specific permits/denies before general statements.

### Verification & troubleshooting
- `show access-lists` (hit counters), `show ip access-lists <name>`.
- `show run interface <intf>` to confirm binding/direction.
- `debug ip packet` (use caution) or `log` keyword for selective insight.

### Quick build pattern (extended, named)
```
ip access-list extended EDGE-IN
 10 remark Permit branch web to HQ
 20 permit tcp 10.10.0.0 0.0.255.255 host 198.51.100.10 eq 443
 30 remark Allow ICMP echo/echo-reply for testing
 40 permit icmp any any echo
 50 permit icmp any any echo-reply
 900 deny ip any any log
interface g0/0
 ip access-group EDGE-IN in
```
