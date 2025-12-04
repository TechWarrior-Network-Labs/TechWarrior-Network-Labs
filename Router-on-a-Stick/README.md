# Tech Warrior Lab – Router-on-a-Stick (Inter-VLAN Routing)
> Built and documented by **Charly "Tech Warrior" Feuille**

---

## Objective
Implement **router-on-a-stick** using:
- Multiple Layer 2 VLANs  
- 802.1Q trunking  
- Sub-interfaces on the router  
- Inter-VLAN routing  
- Layer 2 switches (no ip routing enabled)

This lab demonstrates **Inter-VLAN routing using a single physical router interface**.

---
## Topology
![Topology Diagram](Router%20on%20a%20stick.drawio.png)

---

##  Build Checklist
- [x] Configure basic switch & router settings  
- [x] Create VLANs (3, 4, 8)  
- [x] Configure trunk ports  
- [x] Create router subinterfaces (G0/0.3, G0/0.4, G0/0.8 native)  
- [x] Test inter-VLAN reachability  
- [x] Verify SVI reachability  
- [x] Save configurations  

---

#  Configuration Summary

---

# SW-1 (Layer 2 Switch)

### VLANs
```bash
vlan 3
 name MGMT
vlan 4
 name Operations
vlan 8
 name Native
```

### Management SVI
```bash
interface vlan 3
 ip address 192.168.3.11 255.255.255.0
 description MGMT
 no shutdown
exit
ip default-gateway 192.168.3.1
```

### Access Port
```bash
interface g0/1
 switchport mode access
 switchport access vlan 3
 spanning-tree portfast
 spanning-tree bpduguard enable
 no shutdown
```

### Trunk to SW2
```bash
interface g0/2
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk native vlan 8
 switchport trunk allowed vlan 3,4,8
 no shutdown
```

### Trunk to Router
```bash
interface g0/0
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk native vlan 8
 switchport trunk allowed vlan 3,4,8
 no shutdown
```

---

# SW-2 (Layer 2 Switch)

### VLANs
(same as SW-1)

### SVI
```bash
interface vlan 3
 ip address 192.168.3.12 255.255.255.0
 description MGMT
 no shutdown
```

### Access Port
```bash
interface g0/1
 switchport mode access
 switchport access vlan 4
 spanning-tree portfast
 spanning-tree bpduguard enable
```

### Trunk Port
```bash
interface g0/0
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk native vlan 8
 switchport trunk allowed vlan 3,4,8
 no shutdown
```

---

#  RTR-1 (Router-on-a-Stick)

### Subinterfaces
```bash
interface g0/0.3
 encapsulation dot1q 3
 description MGMT NETWRK
 ip address 192.168.3.1 255.255.255.0

interface g0/0.4
 encapsulation dot1q 4
 description Operations NETWRK
 ip address 192.168.4.1 255.255.255.0

interface g0/0.8
 encapsulation dot1q 8 native
 description Native VLAN
```

---

# 🧪 Verification Commands

```bash
show vlan brief
show ip interface brief
show interface g0/0
show interfaces trunk
```

### Ping tests
```bash
PC-A → 192.168.4.1
PC-B → 192.168.3.1

RTR-1:
ping 192.168.3.11
ping 192.168.4.3
```

---

# Troubleshooting Notes
- If PCs can’t reach each other → trunk allowed VLAN missing  
- If SVI doesn’t respond → SVI shutdown or wrong VLAN  
- If subinterface down/down → router port is shut or switch trunk down  
- Switch must be **pure Layer 2** → `no ip routing`  

---
# 🏁 Lab Status
**Completed** — Inter-VLAN routing fully operational via router subinterfaces.

