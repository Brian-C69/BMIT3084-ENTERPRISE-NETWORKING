# Chapter 10 - QoS Concepts (ENSA v7.0, Module 9)

## Why QoS exists
- QoS matters when links are congested: queueing introduces **delay**, **jitter**, and **drops**.
- Real-time traffic (voice/video) is sensitive to delay/jitter; data is more tolerant.

## Key transmission characteristics
- **Bandwidth**: bits per second available.
- **Delay (latency)**: time to deliver a packet end-to-end.
- **Jitter**: variation in delay.
- **Loss**: packets dropped due to congestion/errors.

## Traffic characteristics (examples)
- **Voice**: predictable, needs priority; requires low delay/jitter.
- **Video**: higher bandwidth; also delay/jitter sensitive.
- **Data**: bursty; can consume available bandwidth (e.g., file transfers).

## Congestion management (queueing/scheduling)
- **FIFO**: first in, first out (no priority).
- **WFQ**: automated fair allocation by flows.
- **CBWFQ**: user-defined traffic classes with guaranteed bandwidth under congestion.
- **LLQ**: strict priority queue added to CBWFQ (commonly reserved for voice).

## Congestion avoidance
- **WRED**: drops lower-priority packets early to avoid tail drop and regulate TCP.

## QoS models (conceptual)
- **Best-effort**: no QoS guarantees.
- **IntServ/RSVP**: per-flow reservations.
- **DiffServ**: scalable per-hop behaviors using DSCP classes.

## Classification & marking (conceptual)
- Mark at Layer 2 (**802.1p CoS**) or Layer 3 (**DSCP**).
- Classification can use interfaces, ACLs, and class maps.

## Common IOS QoS (MQC) command pattern (reference)
> Module is mostly conceptual; this is a common Cisco IOS QoS workflow you’ll see in labs.
```txt
class-map match-any VOICE
 match protocol rtp
!
policy-map QOS-OUT
 class VOICE
  priority percent 10
 class class-default
  fair-queue
!
interface g0/0
 service-policy output QOS-OUT
```

## Verification commands (common)
```txt
show policy-map interface
show policy-map interface g0/0
```
