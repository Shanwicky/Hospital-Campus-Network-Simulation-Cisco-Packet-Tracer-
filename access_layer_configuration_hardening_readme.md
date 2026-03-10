# 🏥 Hospital Campus Network – Access Layer Configuration

## 📌 Overview
This repository documents the **Access Layer configuration and hardening** performed in a multi-department hospital campus network designed and implemented using **Cisco Packet Tracer**.


---

## 🎯 Objectives

- Implement proper **VLAN-based segmentation** per department
- Configure **secure access ports** for end devices
- Establish **stable trunk uplinks** to Distribution switches
- Prevent Layer 2 loops using **Spanning Tree Protocol (STP)**
- Apply **basic access-layer security controls**
- Troubleshoot and resolve real-world configuration errors

---

## 🧱 Access Switches Covered

- **Acc-Emerg** – Emergency Department
- **Acc-Rad-1 / Acc-Rad-2** – Radiology
- **Acc-Ward-1 / Acc-Ward-2** – Wards
- **Acc-Pharm** – Pharmacy
- **Acc-Admin** – Administration

Each access switch connects upstream to its respective **Distribution switch** using a single 802.1Q trunk.

---

## 🧩 VLAN Design (Access Layer)

| VLAN ID | Name        | Purpose |
|-------|-------------|---------|
| 20    | EMERGENCY   | Emergency Department |
| 30    | RADIOLOGY   | Radiology |
| 40    | WARDS       | Patient Wards |
| 50    | PHARMACY    | Pharmacy |
| 60    | ADMIN       | Administration |
| 99    | NATIVE      | Native / Infrastructure |

Only required VLANs were created on each access switch to avoid unnecessary VLAN propagation.

---

## ⚙️ Configuration Steps

### 1️⃣ VLAN Creation (Per Access Switch)
```bash
vlan <DEPARTMENT_VLAN>
vlan 70
vlan 80
vlan 99
```

---

### 2️⃣ Access Port Configuration & Hardening
```bash
interface range fa0/1 - 20
 switchport mode access
 switchport access vlan <DEPARTMENT_VLAN>
 spanning-tree portfast
 spanning-tree bpduguard enable
 switchport port-security
 switchport port-security maximum 2
 switchport port-security violation restrict
 switchport port-security aging time 5
 ip dhcp snooping limit rate 10
```

**Why this matters:**
- Prevents rogue switches (BPDU Guard)
- Limits MAC flooding attacks
- Protects against rogue DHCP servers
- Enables fast host connectivity

---

### 3️⃣ Uplink (Trunk) Configuration
```bash
interface fa0/24
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan <DEPARTMENT>,70,80,99
 ip dhcp snooping trust
 no shutdown
```

Key design choices:
- **Native VLAN 99** used consistently
- Only required VLANs allowed on trunks
- DHCP Snooping trust enabled only on uplinks

---

### 4️⃣ DHCP Snooping (Global)
```bash
ip dhcp snooping
ip dhcp snooping vlan <DEPARTMENT>,70,80
no ip dhcp snooping information option
```

---

## 🔍 Verification Commands Used

```bash
show interfaces trunk
show interfaces fa0/24 switchport
show vlan brief
show spanning-tree vlan <ID>
show cdp neighbors
show port-security
show ip dhcp snooping
```

Verification was always performed from **both access and distribution switches**.

---

## ❌ Problems Encountered & Resolutions

### 🔴 Native VLAN Mismatch
**Error:**
```text
%CDP-4-NATIVE_VLAN_MISMATCH
```

**Cause:**
- One side using VLAN 1
- Other side using VLAN 99

**Fix:**
```bash
switchport trunk native vlan 99
```
Configured on **both ends of the link**.

---

### 🔴 STP Showing No Forwarding VLANs

**Causes:**
- VLAN not created on access switch
- VLAN inactive or pruned
- STP convergence delay

**Resolution:**
- Manually created VLANs
- Re-verified trunk allowed VLANs
- Checked STP state after convergence

---

### 🔴 PortFast Warnings on Trunks

**Message:**
```text
Portfast will only have effect when the interface is in non-trunking mode
```

**Explanation:**
- PortFast was applied via interface range
- Trunk ports correctly ignore PortFast

**Action:**
- No change required

---

## ⚠️ Cisco Packet Tracer Limitations

| Limitation | Impact |
|----------|-------|
| Slow STP convergence | Temporary misleading outputs |
| Partial Port Security support | Some aging modes unavailable |
| DHCP Snooping inconsistencies | Required manual verification |
| Missing CLI commands | Used alternative show commands |

These limitations encouraged **real troubleshooting habits** instead of blind command trust.

---

## 🧠 Key Lessons Learned

- Enterprise networks are **non-linear and interconnected**
- Fixing one issue can expose another
- Always verify from **both sides of the link**
- STP behavior becomes intuitive with practice
- Access-layer security must be intentional

---

## ✅ Outcome

✔ Secure and stable access layer
✔ Proper VLAN segmentation
✔ Loop-free Layer 2 topology
✔ Hardened user-facing ports
✔ Real-world troubleshooting experience

---

## 💼 Portfolio Value

This project demonstrates:
- Practical Layer 2 troubleshooting
- Enterprise-style switch configuration
- Security-aware access design
- Understanding beyond certification labs

---

