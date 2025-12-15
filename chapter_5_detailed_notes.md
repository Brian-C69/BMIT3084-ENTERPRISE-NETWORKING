# Chapter 5 – IPv4 ACL Configuration (ENSA v7.0)

## Module 5 Focus
- Implement IPv4 ACLs to filter traffic and secure administrative access.
- Topics: configure standard ACLs, edit with sequence numbers, secure VTY, configure extended ACLs.

## Standard IPv4 ACLs
- **Plan first:** draft policy in a text editor; add remarks to document intent.
- **Create (numbered or named):**
  - Numbered: `access-list 10 permit 192.168.10.0 0.0.0.255`
  - Named: `ip access-list standard LAN-FILTER`
    - `10 permit 192.168.10.0 0.0.0.255`
    - `20 deny any`
- **Apply:** `interface g0/0` → `ip access-group 10 in` (or named). Standard ACLs typically placed near destination.
- **VTY control:**  
  ```
  ip access-list standard MGT-VTY
   10 permit 10.10.10.0 0.0.0.255
   20 deny any
  line vty 0 4
   access-class MGT-VTY in
   transport input ssh
  ```
- **Edit with sequence numbers:** Insert/delete specific lines in named ACLs; `ip access-list standard ...` then `15 permit host 192.168.20.5`.

## Extended IPv4 ACLs
- **Create (named preferred):**
  ```
  ip access-list extended EDGE-IN
   10 remark Allow HQ HTTPS
   20 permit tcp 10.1.1.0 0.0.0.255 host 198.51.100.10 eq 443
   30 remark Allow ICMP test
   40 permit icmp any any echo
   50 permit icmp any any echo-reply
   900 deny ip any any log
  interface g0/1
   ip access-group EDGE-IN in
  ```
- **Placement:** Extended ACLs near source to block unwanted traffic early; ensure return traffic allowed or handled by stateful devices.
- **Service matching:** Specify protocol and ports (`tcp/udp/icmp`, `eq`, `range`, `gt/lt/neq`). Use `host` for /32.
- **Logging:** Add `log`/`log-input` to key ACEs; watch CPU.

## Wildcard Refresher
- Inverse mask: `255.255.255.255 - subnet mask`. `/24` → `0.0.0.255`; host → `0.0.0.0`; any → `0.0.0.0 255.255.255.255` or keyword `any`.

## Best Practices
- Order matters: specific permits/denies before general statements; implicit deny at end—add explicit `deny ... log` if you want visibility.
- Avoid double-binding ACLs in+out on same interface unless needed.
- Allow needed control traffic (routing protocols, DHCP, DNS) explicitly.
- Verify: `show access-lists`, `show ip interface <intf>`, ping/traceroute; use hit counters to confirm matches.

## Troubleshooting Cues
- No matches/hits: ACL not applied to interface or wrong direction.
- Legit traffic dropped: wildcard/mask wrong, statement order wrong, missing protocol/port, implicit deny hit.
- Mgmt blocked: VTY ACL too strict; expand permitted subnet or add specific hosts.
