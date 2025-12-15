# Chapter 3 – Network Security Concepts (ENSA v7.0)

## Foundations
- **Security goals:** Confidentiality, integrity, availability (CIA); authenticity, non-repudiation.
- **Key terms:** Asset, vulnerability, threat, exploit, risk (likelihood × impact), mitigation.
- **Attack surface & vectors:** Email/social engineering, web/app exploits, misconfig, lost devices, cloud misperms, removable media, rogue/insider.
- **Threat actors:** Script kiddies, vulnerability brokers, hacktivists, cybercriminals, state-sponsored teams.

## Common Attacks
- **Recon/Access:** Password guessing, sniffing/MiTM, spoofing, VLAN hopping, DHCP spoof, ARP poisoning.
- **Disruption/Abuse:** DoS/DDoS, malware (virus/worm/Trojan/ransomware), botnets, data exfiltration.
- **Social engineering:** Phishing/spear-phishing/whaling, pretexting, baiting, tailgating.

## Defense in Depth (Layered Controls)
- **Policies & people:** Acceptable use, least privilege, change control, backups, user awareness, incident response.
- **Physical:** Locked racks/rooms, badges, cameras.
- **Network controls:** Segmentation/VLANs, ACLs, firewalls (stateless/stateful/NGFW), IPS/IDS, DHCP snooping/Dynamic ARP Inspection (where supported), VPN/IPsec for untrusted paths.
- **Endpoint:** Patching, EDR/AV, disk encryption, host firewalls.
- **Visibility:** Syslog to central collector, SNMP (prefer v3), NetFlow, time sync (NTP) for good logs.

## Device Hardening (Router/Switch)
- Disable unused services/ports; set **strong secrets**: `enable secret`, local users with `secret`.
- **SSH-only mgmt:** `ip domain-name`, `crypto key generate rsa`, `ip ssh version 2`, `line vty 0 4` → `transport input ssh`, `login local`.
- **Console/VTY hygiene:** `exec-timeout`, `logging synchronous`, `login block-for` (throttle brute-force), `service password-encryption` (obfuscation only).
- **AAA ready:** `aaa new-model`, `aaa authentication login default local` (or RADIUS/TACACS+), `aaa authorization exec default local`.
- **Banners:** `banner motd` for legal notice.
- **File/IOS care:** Back up configs/images; verify hashes; restrict TFTP/FTP use; secure SNMP (`snmp-server community` with ACL or SNMPv3).

## Cryptography & VPN Basics
- **Primitives:** Hashing (integrity), symmetric ciphers (speed/confidentiality), asymmetric (key exchange/identity), digital signatures (integrity + auth), PKI and certificates.
- **IPsec overview:** IKE SA establishes keys; IPsec SAs provide ESP (enc + integrity) or AH (integrity only). Modes: tunnel vs transport. Used for site-to-site or remote-access VPNs.

## Practical Checks/Troubleshooting
- **Mgmt access fails:** Verify interface up, VTY set to SSH, keys generated, user exists, ACLs permit source; check `show ip ssh`, `debug ip ssh` (with care).
- **Login brute-force:** Apply `login block-for`, ACL to limit mgmt sources, use AAA with lockout.
- **Time/Logs:** Ensure `ntp server <ip>` and `service timestamps`; check syslog reachability.
- **SNMP:** Prefer v3 (`snmp-server group ... v3 priv`); if v2c, bind with ACL and disable write if not needed.

## Quick Hardening Snippet
```
enable secret <STRONG>
service password-encryption
ip domain-name example.local
crypto key generate rsa modulus 2048
ip ssh version 2
username netops secret <STRONG>
line con 0
 exec-timeout 5 0
 logging synchronous
 password <STRONG>
 login local
line vty 0 4
 transport input ssh
 exec-timeout 5 0
 login local
 login block-for 60 attempts 5 within 60
aaa new-model
aaa authentication login default local
banner motd ^CAuthorized access only!^C
```
