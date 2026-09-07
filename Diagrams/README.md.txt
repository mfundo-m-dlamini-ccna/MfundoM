# Sentech Enterprise Lab Architecture | Logical IP Addressing Chart

This document serves as the formal infrastructure addressing plan for the multi-departmental network architecture deployed using the **Cisco 2911 ISR Core Router** and **Cisco Catalyst 2960 Edge Switch**.

---

## 🗺️ Master Subnet Allocation Matrix

| Segment Name | VLAN ID | Network Address / CIDR | Subnet Mask | Usable Host Range | Default Gateway | Purpose / Operational Standard |
| :--- | :---: | :--- | :--- | :--- | :--- | :--- |
| **HR_Department** | `10` | `192.168.10.0/24` | `255.255.255.0` | `192.168.10.11` to `192.168.10.254` | `192.168.10.1` | Corporate HR infrastructure host systems. Enforces a **minimum baseline of 2 active PCs** to validate path traversal. |
| **Finance_Department** | `20` | `192.168.20.0/24` | `255.255.255.0` | `192.168.20.11` to `192.168.20.254` | `192.168.20.1` | Corporate Finance host systems. Enforces a **minimum baseline of 2 active PCs**. Restricted from accessing HR via strict Extended ACLs. |
| **Management_Native** | `99` | `192.168.99.0/24` | `255.255.255.0` | `192.168.99.2` to `192.168.99.254` | `192.168.99.1` | Isolated out-of-band remote administration domain (SSH and Secure SVI). |

---

## 🛠️ Core Infrastructure Node Assignments

| Device Hostname | Interface / Sub-Interface | Assigned IP Address | Operational Status | Connected Infrastructure Endpoint |
| :--- | :--- | :--- | :---: | :--- |
| **Sentech-Core-R1** | `Gi0/0` | *None (Trunk Parent)* | **UP** | Connects directly to Edge Switch `Fa0/24`. |
| **Sentech-Core-R1** | `Gi0/0.10` | `192.168.10.1` | **UP** | Router-on-a-Stick gateway node for VLAN 10. |
| **Sentech-Core-R1** | `Gi0/0.20` | `192.168.20.1` | **UP** | Router-on-a-Stick gateway node for VLAN 20. |
| **Sentech-Core-R1** | `Gi0/0.99` | `192.168.99.1` | **UP** | Router-on-a-Stick gateway node for Native Management VLAN 99. |
| **Sentech-Edge-SW1** | `Vlan99` (SVI) | `192.168.99.10` | **UP** | Remote SSH Management interface for network administrators. |

---

## 🔒 Reserved & Excluded Pools (Static Infrastructure Blocks)

The first **10 IP addresses** in each production subnet are explicitly excluded from dynamic Cisco IOS DHCP allocation pools to protect infrastructure nodes from IP address allocation conflicts:

*   **VLAN 10 Excluded Range:** `192.168.10.1` – `192.168.10.10` *(Reserved for Gateways, Local Network Printers, and Local Server elements).*
*   **VLAN 20 Excluded Range:** `192.168.20.1` – `192.168.20.10` *(Reserved for Gateways and dedicated Accounting storage components).*

---

## 🔌 Switch Edge Port Allocation Map

| Physical Interfaces | Switchport Mode | Operational VLAN Assignment | Security Constraint Enforced | Target Endpoint Deployment |
| :--- | :--- | :---: | :--- | :--- |
| **Fa0/1 — Fa0/5** | `Access` | `VLAN 10` | Port-Security Max: 2 MACs (`Sticky`), Violation: `Shutdown` | HR Department Workstations (Hosts 1 & 2 active). |
| **Fa0/6 — Fa0/10** | `Access` | `VLAN 20` | Port-Security Max: 2 MACs (`Sticky`), Violation: `Shutdown` | Finance Department Workstations (Hosts 1 & 2 active). |
| **Fa0/11 — Fa0/23** | `Access` | *None* | **Administratively Shutdown (`disable`)** | Unused interfaces blocked to prevent malicious on-site breaches. |
| **Fa0/24** | `Trunk` | `10, 20, 99` | Native VLAN explicitly redefined to `99` | Uplink pipe interfacing directly with Cisco 2911 `Gi0/0`. |
| **Gi0/1 — Gi0/2** | `Access` | *None* | **Administratively Shutdown (`disable`)** | Unused high-speed interfaces blocked. |
