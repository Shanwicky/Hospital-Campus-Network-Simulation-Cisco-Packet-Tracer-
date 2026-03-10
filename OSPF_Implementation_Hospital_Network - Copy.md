# 🧭 OSPF Implementation & Troubleshooting
## Hospital Campus Network + Remote Clinic Expansion

---

## 📌 Overview
This project extends a redundant hospital campus network by implementing **OSPF (Open Shortest Path First)** in the core layer and connecting a **remote clinic site** using a routed point-to-point link.

The work includes **real-world troubleshooting**, especially resolving issues caused by **STP blocking OSPF control traffic**, and validating full end-to-end routing.

This is not a “perfect-path” lab — multiple failures occurred and were intentionally debugged and fixed.

---

## 🏗️ Network Design Summary

### Hospital Campus
- Dual Core Layer (Core-SW-1, Core-SW-2)
- Multiple VLANs (10–90) with HSRP for gateway redundancy
- Layer-2 Distribution switches
- Dedicated **OSPF Transit VLAN (VLAN 100)** between cores

### Remote Clinic
- RemoteClinic-R1 router
- Routed point-to-point link to Core-SW-2
- Clinic LAN: `172.16.10.0/24`

---

## 🚀 Features Added
- OSPF Area 0 in the core layer
- Dedicated transit VLAN for OSPF
- Routed point-to-point WAN-style link
- STP root tuning to fix OSPF adjacency failure
- Full bidirectional route exchange

---

## 🔧 Core OSPF Configuration

### VLAN 100 (Core Transit)
```cisco
vlan 100
 name CORE_TRANSIT
```

### Core Switch SVIs
```cisco
interface vlan 100
 ip address 10.10.100.1 255.255.255.252   ! Core-SW-1
 ip address 10.10.100.2 255.255.255.252   ! Core-SW-2
 no shutdown
```

### Enable OSPF
```cisco
router ospf 1
 router-id 1.1.1.1        ! Core-SW-1
 router-id 2.2.2.2        ! Core-SW-2
 network 10.10.100.0 0.0.0.3 area 0
```

---

## ❌ Failure 1: OSPF Neighbor Not Forming (Core ↔ Core)

### Symptom
```cisco
show ip ospf neighbor
```
Returned **no neighbors**.

### Root Cause
- VLAN 100 was allowed on distribution trunks
- STP detected a loop and **blocked the core trunk**
- OSPF Hello packets never reached the peer

### Fix
- Removed VLAN 100 from all distribution trunks
- Allowed VLAN 100 **only on core-to-core trunk**
- Forced core to be STP root for VLAN 100

```cisco
spanning-tree vlan 100 priority 0      ! Core-SW-1
spanning-tree vlan 100 priority 4096   ! Core-SW-2
```

### Result
```cisco
show ip ospf neighbor
```
```
FULL/DR , FULL/BDR
```

---

## ❌ Failure 2: OSPF Working but No Routes

### Symptom
```cisco
show ip route ospf
```
Returned empty.

### Root Cause
- Both cores already had all VLANs as connected routes
- OSPF had nothing new to advertise

### Fix
- Added a remote site with a unique LAN network

---

## 🌍 Remote Clinic Router Configuration

```cisco
interface g0/0
 ip address 10.10.200.2 255.255.255.252
 no shutdown

interface g0/1
 ip address 172.16.10.1 255.255.255.0
 no shutdown
```

```cisco
router ospf 1
 router-id 3.3.3.3
 network 10.10.200.0 0.0.0.3 area 0
 network 172.16.10.0 0.0.0.255 area 0
```

---

## ❌ Failure 3: Clinic LAN Not Advertised

### Symptom
- OSPF neighbor was FULL
- Clinic LAN route missing

### Root Cause
- LAN interface was **not physically connected**
- OSPF does not advertise down interfaces

### Fix
- Connected switch + PC to clinic LAN
- Interface became up/up
- Route was automatically advertised

---

## 🔗 Core ↔ Clinic Routed Link

### Core-SW-2
```cisco
interface gigabitEthernet0/2
 no switchport
 ip address 10.10.200.1 255.255.255.252
 ip ospf network point-to-point
 no shutdown
```

---

## ✅ Final Verification

### Core-SW-2
```cisco
show ip route ospf
```
```
O 172.16.10.0/24 via 10.10.200.2, GigabitEthernet0/2
```

### RemoteClinic-R1
```cisco
show ip route ospf
```
```
O 10.10.10.0/24
O 10.10.20.0/24
...
O 10.10.90.0/24
```

---

## 🧠 Key Lessons Learned
- STP can silently break routing protocols
- Transit VLANs must be tightly controlled
- Always verify interface state before debugging routing
- OSPF advertises **interfaces**, not ideas
- `show ip interface brief` is the fastest truth

---

## 🏁 Conclusion
This implementation demonstrates enterprise-grade OSPF deployment, real troubleshooting, and multi-site routing validation. The network was intentionally debugged to mirror production behavior rather than ideal lab conditions.
