# Hospital Network Project – OSPF, WAN, and Security Implementation


---

## Features Implemented
- Multi-area OSPF (Area 0 backbone + Area 10 stub)
- OSPF cost engineering
- OSPF MD5 authentication
- WAN point-to-point routing
- ACL-based security for Remote Clinic users
- HSRP, STP, and VTP in the core layer

---

## OSPF Configuration

### WAN OSPF (Point-to-Point)
```cisco
interface GigabitEthernet0/2
 ip address 10.10.200.1 255.255.255.252
 ip ospf network point-to-point
 ip ospf cost 100
```

### Multi-Area Design
```cisco
router ospf 1
 network 10.10.200.0 0.0.0.3 area 0
 network 172.16.10.0 0.0.0.255 area 10
 area 10 stub
```

---

## OSPF MD5 Authentication
```cisco
interface GigabitEthernet0/2
 ip ospf authentication message-digest
 ip ospf message-digest-key 1 md5 HOSPITAL_OSPF
```

Verified with:
```cisco
show ip ospf interface
show ip ospf neighbor
```

---

## ACL Security Policy

```cisco
ip access-list extended REMOTECLINIC_ACL
 deny ip 172.16.10.0 0.0.0.255 10.10.99.0 0.0.0.255
 permit ip 172.16.10.0 0.0.0.255 10.10.90.0 0.0.0.255
 permit ip any any
```

Applied inbound on Core-SW-2 WAN interface.

Verified using:
```cisco
show access-lists REMOTECLINIC_ACL
```

---

## Issues Faced and Fixes

### OSPF adjacency failures
- Fixed by matching network type and MD5 keys

### Routes not appearing
- Fixed by implementing multi-area OSPF

### ACL not enforcing
- Fixed by adding explicit deny rules and verifying counters

### Packet Tracer auto-exit
- Fixed by rebuilding config and saving frequently

---

## Verification
- `show ip ospf neighbor`
- `show ip route ospf`
- `show access-lists`
- End-to-end ping tests

---

## Resume Line
Designed and implemented a secure multi-area OSPF hospital network with WAN cost engineering, MD5 authentication, and ACL-based access control.
