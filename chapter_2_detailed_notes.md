# Chapter 2 – Single-Area OSPFv2 (ENSA v7.0)

## Module 1: Single-Area OSPFv2 Concepts
- **Purpose & scope:** Link-state IGP for IPv4; fast convergence, hierarchical design. Single-area = all routers in Area 0 (backbone).
- **Cost metric:** Inversely proportional to interface bandwidth (default reference 100 Mbps; modern IOS often 1 Gbps cost 1). Tune with `ip ospf cost` or bandwidth.
- **Router ID (RID):** Chosen in order: manual `router-id`, highest loopback, highest active interface. Sticky until OSPF reset.
- **Neighbor formation:** Hellos on each interface; matching area ID, timers, network type, authentication, stub flags, and MTU. States: Down → Init → 2-Way → ExStart → Exchange → Loading → Full.
- **Network types & DR/BDR:** On broadcast/non-broadcast multi-access, DR/BDR elected by priority (0 = none) then highest RID; adjacencies form fully only to DR/BDR. Point-to-point has no DR/BDR.
- **Link-state database (LSDB):** Routers flood LSAs to describe topology; SPF calculates shortest paths; routing table is derived from LSDB.
- **OSPF packets:** Hello, DBD, LSR, LSU, LSAck. Hellos discover/maintain neighbors; others synchronize LSDBs.
- **Area behavior:** In single-area deployments, all routers share the same LSDB (Area 0), simplifying design and troubleshooting.

## Module 2: Single-Area OSPFv2 Configuration
- **Enable OSPF:** `router ospf <pid>` then advertise interfaces with `network <ip> <wildcard> area 0` or per-interface `ip ospf <pid> area 0`.
- **Set RID:** `router-id <A.B.C.D>`; clear process (`clear ip ospf process`) to apply if already running.
- **Interface tuning:**  
  - **Priority/DR influence:** `ip ospf priority <0-255>` (0 prevents DR election).  
  - **Cost:** `ip ospf cost <value>` (or adjust bandwidth).  
  - **Passive interface:** `passive-interface <intf>` to suppress Hellos on access links while still advertising the network.  
  - **Network type/timers:** adjust if needed to match neighbors; defaults fit Ethernet/serial point-to-point.
- **Stub settings:** In single-area designs usually not used; ensure stub flags match if configured.
- **Verification basics:** `show ip ospf neighbor`, `show ip ospf interface`, `show ip ospf database`, `show ip route ospf`, `show ip protocols`.
- **Ping/trace checks:** Test reachability to remote loopbacks/LANs; if multi-access, confirm DR/BDR roles.
- **Common gotchas:** Missing/wrong wildcard in `network` statements, mismatched timers or area IDs, wrong RID, priority set to 0 unintentionally, interfaces shutdown, or ACLs blocking Hellos.

## Quick Configuration Patterns
- **Minimal single-area (router mode):**
  ```
  router ospf 1
   router-id 1.1.1.1
   network 10.0.0.0 0.0.0.255 area 0
   network 192.168.1.0 0.0.0.255 area 0
  ```
- **Interface-style (preferred for clarity):**
  ```
  interface g0/0
   ip address 10.0.0.1 255.255.255.0
   ip ospf 1 area 0
  interface s0/1/0
   ip address 192.168.1.1 255.255.255.252
   ip ospf 1 area 0
  ```
- **DR control on multi-access:**
  ```
  interface g0/1
   ip ospf priority 200   ! favor this router as DR
  ```
- **Passive access edge:**
  ```
  router ospf 1
   passive-interface g0/2   ! no Hellos; still advertises subnet
  ```

## Troubleshooting Snapshot
- **Neighbors stuck in 2-Way:** DR/BDR election expected on multi-access; full only with DR/BDR. If point-to-point, check network type/timers/auth/MTU.
- **No adjacency:** Verify area ID/timers/auth/stub flags and that interfaces are up; check ACLs for OSPF (IP proto 89).
- **Routes missing:** Ensure networks are matched by `network` statements or interface commands; confirm LSAs present in LSDB; check summarization/filters (if any).
