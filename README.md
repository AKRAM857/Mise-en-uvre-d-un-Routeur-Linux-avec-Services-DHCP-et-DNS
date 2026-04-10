# Mise-en-oeuvre-d-un-Routeur-Linux-avec-Services-DHCP-et-DNS
# Linux Router with DHCP & DNS Services

A VirtualBox lab simulating a professional network infrastructure where an 
Ubuntu VM acts as a router, DHCP server, and DNS resolver for a Kali Linux client.

## Architecture
## Stack

- **Hypervisor**: VirtualBox
- **Router/Server**: Ubuntu (isc-dhcp-server, bind9)
- **Client**: Kali Linux
- **Networking**: Netplan, IP Forwarding,

## Features

- Automatic IP assignment via DHCP (pool: 192.168.10.10–100)
- DNS resolution for local and external queries
- IP Forwarding between WAN (enp0s3) and LAN (enp0s8)
- NAT masquerading for internet access from client

## Quick Setup

### 1. Configure interfaces (Ubuntu)
\```bash
sudo apt update
sudo apt upgrade
sudo nano /etc/netplan/00-installer-config.yaml
sudo netplan try
sudo netplan apply


### 2. Configure DHCP
\```bash
sudo install isc-dhcp-server
sudo nano /etc/dhcp/dhcpd.conf
sudo nano /etc/default/isc-dhcp-server   # set INTERFACES="enp0s8"
sudo systemctl restart isc-dhcp-server
sudo systemctl status isc-dhcp-server


### 3. Enable IP Forwarding
\```bash
echo "net.ipv4.ip_forward=1" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p


### 4. Test from Kali
\```bash
sudo dhcpcd eth0
sudo ip addr show
ping 192.168.10.1


## Troubleshooting

**Issue**: `Destination Host Unreachable` despite correct IP and routing  
**Root cause**: `enp0s8` was enslaved to an OVS bridge, causing OVS to 
intercept ARP frames with no flow rules to forward them  
**Fix**: `ovs-vsctl del-br <bridge>` then `systemctl stop openvswitch-switch`

## Author

**Akram Khoulid** — École Nationale des Sciences Appliquées  
April 2026
