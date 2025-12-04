# Tech Warrior Lab – Enterprise Static Routing (TX–FL–ATL)
> Built and documented by **Charly "Tech Warrior"**

---

## Objective

Build a **multi-site enterprise network** with three locations:

- **TEXAS** (branch)
- **FLORIDA** (core / hub)
- **ATLANTA** (branch)

Using **static routes + default routes** to provide:

- End-to-end connectivity between all sites  
- Reachability to loopbacks, SVIs, and user VLANs  
- Hub-and-spoke routing (TX/ATL use FL as the core)  
- /30 WAN point-to-point links

---

## Topology


![Topology Diagram](topology.png)

---

## Build Checklist

- [x] Configure basic router and switch settings  
- [x] Build VLANs and subinterfaces in TX & ATL  
- [x] Configure 802.1Q trunks to switches  
- [x] Configure WAN /30 links between TX–FL and FL–ATL  
- [x] Configure static + default routes on all routers  
- [x] Configure mgmt SVIs + default-gateway on switches  
- [x] Verify end-to-end connectivity (PC ↔ PC, loopbacks)  

---

## ⚙️ Configuration Summary

###  Texas-RTR
- Loopback: `10.2.10.1/32`
- WAN to FL-RTR: `10.2.10.69/30` (next-hop `10.2.10.70`)
- Subinterfaces:
  - `G0/3.1101` – MGMT: `10.2.10.9/29`
  - `G0/3.2101` – USER1: `10.2.10.17/29`
- Static routes:
  - Default → `10.2.10.70` (FL-RTR)
  - `10.2.10.192/32` (FL loopback) via `10.2.10.70`
  - `10.2.10.128/32` (ATL loopback) via `10.2.10.70`

---

###  TX-SW
- Pure L2 (`no ip routing`)
- SVI VLAN 1101 (MGMT): `10.2.10.14/29`
- Access:
  - `G0/2` → VLAN 1101 (PC1)
- Trunk:
  - `G0/1` → Texas-RTR (VLANs 1101, 2101)
- Default gateway: `10.2.10.9` (Texas-RTR G0/3.1101)

---

### ATL-RTR
- Loopback: `10.2.10.128/32`
- WAN to FL-RTR: `10.2.10.73/30` (next-hop `10.2.10.74`)
- Subinterfaces:
  - `G0/3.1102` – MGMT: `10.2.10.137/29`
  - `G0/3.2102` – USER1: `10.2.10.145/29`
- Static routes:
  - Default → `10.2.10.74` (FL-RTR)
  - `10.2.10.192/32` (FL loopback) via `10.2.10.74`
  - `10.2.10.1/32` (TX loopback) via `10.2.10.74`

---

### ATL-SW
- Pure L2 (`no ip routing`)
- SVI VLAN 1102 (MGMT): `10.2.10.142/29`
- Access:
  - `G0/2` → VLAN 1102 (PC5)
- Trunk:
  - `G0/1` → ATL-RTR (VLANs 1102, 2102)
- Default gateway: `10.2.10.137` (ATL-RTR G0/3.1102)

---

### FL-CORE-SW (Core L3 Switch)

- Loopback: `10.2.10.194/32`
- SVIs (from previous FL lab):
  - VLAN1100 HR: `10.2.10.209/29`
  - VLAN2100 IT: `10.2.10.217/29`
  - VLAN3100 SALES: `10.2.10.225/27`
- Routed link to FL-RTR:
  - `G0/3` – `10.2.10.201/30`
- Static routes:
  - Default → `10.2.10.202` (FL-RTR)
  - `10.2.10.192/32` via `10.2.10.202` (FL-RTR loopback)
  - `10.2.10.1/32` via `10.2.10.202` (TX loopback)
  - `10.2.10.128/32` via `10.2.10.202` (ATL loopback)

---

### FL-RTR (Core Hub Router)

- WAN to TX-RTR: `10.2.10.70/30` (local IP is `10.2.10.70` or `10.2.10.71` depending on your final config)
- WAN to ATL-RTR: `10.2.10.74/30`
- Routed link to FL-CORE-SW: `10.2.10.202/30`
- Static routes:
  - Default → `10.2.10.201` (FL-CORE-SW)
  - `10.2.10.194/32` → `10.2.10.201` (FL-CORE-SW loopback)
  - `10.2.10.1/32` → `10.2.10.69` (Texas loopback)
  - `10.2.10.128/32` → `10.2.10.73` (Atlanta loopback)
  - Optional host/network routes for TX & ATL subinterfaces (used for troubleshooting)

---

## Verification

From each router:

```bash
show ip interface brief
show ip route
show ip route 10.2.10.1
show ip route 10.2.10.128
show ip route 10.2.10.194


Troubleshooting Notes
If a loopback won’t ping:
Check both directions: forward AND return route
If one site can reach FL but not the other branch:
Missing static route for the remote loopback or subnet
If VLAN hosts can’t reach loopbacks:
Check SVI IPs and default gateways
If TX/ATL can’t reach FL-CORE-SW:
Check FL-RTR ↔ FL-CORE-SW static routes and interface status
