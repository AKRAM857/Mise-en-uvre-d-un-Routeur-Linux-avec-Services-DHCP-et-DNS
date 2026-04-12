# Linux Router with DHCP, DNS & SDN (Open vSwitch)

A VirtualBox lab simulating a professional network infrastructure where an Ubuntu VM acts as a 
router, DHCP server, DNS resolver, and SDN controller for Kali Linux and Alpine Linux clients, 
enhanced with Open vSwitch for software-defined networking.

---
## Architecture

```
                        UBUNTU (Router / Controller)
                 ┌──────────────────────────────────────┐
                 │          OVS Bridge: bridge12         │
                 │                                      │
                 │  bridge12 (internal) tag:10  192.168.10.1 │
                 │  enp0s8   (cable)    tag:10  ──→ Kali    │
                 │  vlan20   (internal) tag:20  192.168.20.1 │
                 │  enp0s9   (cable)    tag:20  ──→ Alpine  │
                 │  enp0s3   (WAN/NAT)  ──→ Internet        │
                 └──────────────────────────────────────┘
                        |                    |
           ┌────────────┘                    └─────────────┐
           │                                               │
  ┌─────────────────┐                        ┌──────────────────┐
  │   Kali Linux    │                        │  Alpine Linux    │
  │ 192.168.10.20   │                        │ 192.168.20.20    │
  │  VLAN 10 (LAN)  │                        │  VLAN 20 (LAN2)  │
  └─────────────────┘                        └──────────────────┘
```

## Stack

- **Hypervisor**: VirtualBox
- **Router/Server**: Ubuntu (isc-dhcp-server, bind9, Open vSwitch)
- **Clients**: Kali Linux (VLAN 10) / Alpine Linux (VLAN 20)
- **Networking**: Netplan, IP Forwarding, NAT (iptables), OVS, VLANs

---

## Features

- Automatic IP assignment via DHCP for both VLANs
  - VLAN 10 pool: `192.168.10.50 – 192.168.10.200`
  - VLAN 20 pool: `192.168.20.50 – 192.168.20.200`
- Private DNS with BIND9 for `akram.lab` domain
  - Forward zone: `kali.akram.lab` → `192.168.10.20`
  - Forward zone: `alpine.akram.lab` → `192.168.20.10`
  - Reverse zone: `10.168.192.in-addr.arpa`
  - Forwarders: `8.8.8.8` / `1.1.1.1` for external queries
- IP Forwarding between WAN (`enp0s3`) and LAN (`bridge12`, `vlan20`)
- NAT masquerading for internet access from all clients
- SDN with Open vSwitch — VLAN-based traffic isolation

---

## SDN Concept (Open vSwitch)

This project integrates **Software Defined Networking** by separating:

- **Control Plane** → Ubuntu kernel (decides where packets go)
- **Data Plane** → Open vSwitch bridge12 (moves packets between ports)
---

## Troubleshooting & Lessons Learned

### 1. OVS Bridge Blocking ICMP (but not DHCP)

**Symptom**: `Destination Host Unreachable` despite correct IP and routing table.  
**Root Cause**: `enp0s8` was enslaved to OVS but the IP was still assigned to it —
OVS intercepted all frames, and without flow rules, dropped ARP silently.  
**Why DHCP survived**: DHCP is L2 broadcast — OVS forwards broadcasts.
ARP/ICMP are L3 unicast — OVS dropped them with no flow rules configured.  
**Fix**: `ovs-vsctl del-br bridge1` then reassign IP to the internal port.

### 2. Two Internal Ports on Same Tag Broke Ping

**Symptom**: Created `vlan10` alongside `bridge12`, both with tag 10 — ping between 
Kali and Ubuntu failed.  
**Root Cause**: Two internal ports on the same tag creates an ambiguous path — OVS 
doesn't know which port to deliver the reply to.  
**Fix**: Remove `vlan10`, keep only `bridge12` for tag 10.

### 3. DNS Failed Due to a Single Character Typo

**Symptom**: BIND9 zones not loading, DNS completely non-functional.  
**Root Cause**: `in-addr-arpa` instead of `in-addr.arpa` (dash instead of dot).  
**Lesson**: DNS is extremely sensitive to syntax — always validate with  
`named-checkconf` and `named-checkzone` before restarting.

### 4. enp0s8 IP Must Be Removed Before Adding to OVS

**Symptom**: Connectivity issues after adding `enp0s8` to OVS while it still had an IP.  
**Fix**: Always `ip addr flush dev enp0s8` before `ovs-vsctl add-port`.

---

## Validation Results

| Test | Result |
|---|---|
| Kali gets IP via DHCP | ✅ 192.168.10.x |
| Alpine gets IP via DHCP | ✅ 192.168.20.x |
| Kali pings Ubuntu | ✅ |
| Alpine pings Ubuntu | ✅ |
| Kali resolves kali.akram.lab | ✅ |
| Alpine resolves kali.akram.lab | ✅ |
| Kali pings 8.8.8.8 | ✅ |
| Alpine pings 8.8.8.8 | ✅ |

---
---

## Author

**Akram Khoulid** — 3rd year Networks & Telecommunications Engineering  
École Nationale des Sciences Appliquées (ENSA) — Safi  
April 2026
