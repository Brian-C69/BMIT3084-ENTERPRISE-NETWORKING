# Chapter 11 - Network Management (ENSA v7.0, Module 10)

## Device discovery: CDP (Cisco) and LLDP (vendor-neutral)

### CDP (enable/disable)
```txt
cdp run
no cdp run
```

### CDP verification
```txt
show cdp
show cdp interface
show cdp neighbors
show cdp neighbors detail
```

### LLDP (enable/disable)
```txt
lldp run
no lldp run
```

### LLDP verification
```txt
show lldp
show lldp neighbors
show lldp neighbors detail
```

## Time synchronization: manual clock vs NTP

### Manual time set (lab)
```txt
clock set 20:36:00 nov 15 2019
clock set hh:mm:ss mm dd yyyy
```

### NTP configuration and verification
```txt
ntp server 209.165.200.225
ntp server 192.168.1.1
show ntp associations
show ntp status
```

## Syslog basics
- Syslog destinations include: **buffer**, **console**, **terminal**, **syslog server**.
- Use timestamps so log entries include time.

### Timestamps (command from slides)
```txt
service timestamps log datetime
```

## SNMP basics (concepts)
- SNMP manager queries agents (UDP **161**); agents send traps (UDP **162**).
- Versions: SNMPv1 (legacy), SNMPv2c (community strings), SNMPv3 (auth + privacy).

### Common IOS SNMP configuration (reference)
> Slides focus on operation; this is a common lab configuration pattern.
```txt
snmp-server community <COMMUNITY> RO
```
