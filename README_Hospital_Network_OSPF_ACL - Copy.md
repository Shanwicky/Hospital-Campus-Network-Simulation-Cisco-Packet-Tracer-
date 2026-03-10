# Hospital Network Project – Routing & Security Enhancements

## Overview
This project simulates a **hospital enterprise network** in **Cisco Packet Tracer**, focusing on **real-world routing, redundancy, and security** rather than isolated labs.

The work documented here covers:
- OSPF (single-area → multi-area)
- OSPF cost engineering
- OSPF MD5 authentication
- Inter-VLAN routing with HSRP
- ACL-based security for a remote clinic
- DHCP troubleshooting after routing & ACL changes

This README documents **what was added, what broke, how it was fixed, and how everything was verified**.

---

## Topology Context (High Level)
- Core Layer: `Core-SW-1`, `Core-SW-2` (Layer 3)
- Distribution / Access: Multiple VLANs (Admin, Pharm, Ward, etc.)
- Remote Site: `RemoteClinic-R1`
- Routing: OSPF
- Redundancy: HSRP per VLAN
- Security: Extended ACL on Core-SW-2

---

## Features Implemented

### 1. OSPF Routing (Backbone)
- OSPF Process ID: `1`
- Backbone Area: `Area 0`
- Point-to-point routed link between Core-SW-2 and RemoteClinic-R1
- Verified FULL adjacency

**Verification**
```bash
show ip ospf neighbor
show ip route ospf
```

---

### 2. OSPF Cost Engineering
Used manual cost to influence routing metrics.

```bash
interface GigabitEthernet0/2
 ip ospf cost 100
```

**Result**
- OSPF routes show expected cost `[110/101]`
- Path selection is predictable

---

### 3. OSPF MD5 Authentication
Secured OSPF adjacency using MD5 message-digest authentication.

```bash
interface GigabitEthernet0/2
 ip ospf authentication message-digest
 ip ospf message-digest-key 1 md5 HOSPITAL_OSPF
```

**Issue Encountered**
- OSPF adjacency dropped when configured on one side only

**Fix**
- Applied identical MD5 config on both devices

---

### 4. Multi-Area OSPF Design
- Backbone: Area 0
- Remote Clinic LAN: Area 10 (Stub)

```bash
router ospf 1
 network 172.16.10.0 0.0.0.255 area 10
 area 10 stub
```

---

### 5. ACL Policy – Remote Clinic
```bash
ip access-list extended REMOTECLINIC_ACL
 deny ip 172.16.10.0 0.0.0.255 10.10.99.0 0.0.0.255
 permit ip 172.16.10.0 0.0.0.255 10.10.90.0 0.0.0.255
 permit ip any any
```

Applied inbound on Core-SW-2.

---

## Major Issues & Fixes

### DHCP Failure After OSPF + ACL
- PCs couldn’t get IPs in Acc-Pharm VLAN
- Root cause: VLAN/trunk/DHCP path impacted by changes
- Fix: Restored correct VLAN & DHCP relay path

---

## Final Validation
- Inter-VLAN ping from Acc-Pharm laptop (10.10.50.12)
- ACL counters incremented
- OSPF stable, authenticated, multi-area

✔ Network fully operational
