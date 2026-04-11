# Implementing a Linux Router with DHCP and DNS Services


This project demonstrates the transformation of an Ubuntu instance into a central network gateway, serving as a robust bridge between WAN and LAN interfaces by implementing IP forwarding and NAT. It showcases the seamless integration of critical infrastructure services, specifically DHCP for automated client addressing and BIND9 DNS for private name resolution within the akram.lab domain. Furthermore, the project highlights advanced Layer 3 troubleshooting through the successful diagnosis and resolution of a connectivity conflict caused by Open vSwitch bridge interference.

1. DHCP Server (isc-dhcp-server)
Implemented using a Class C network (192.168.10.0/24).


Address Pool: Dynamically allocates IPs from 192.168.10.10 to 192.168.10.100.


Distributed Options: Injects the Router (Gateway) and DNS settings pointing to the Ubuntu server (192.168.10.1).


DORA Process: Successfully validated the Discovery, Offer, Request, and Acknowledgment cycle on the Kali client.

2. DNS Server (BIND9)
Configuration of a private local domain: akram.lab.


Forward Lookup Zone: Translates hostnames (e.g., kali.akram.lab) into IP addresses.


Reverse Lookup Zone: Translates IPs back to names using the in-addr.arpa structure.


Forwarders: Redirects unknown external queries to Google (8.8.8.8) and Cloudflare (1.1.1.1) DNS.

🔍 Diagnostic & Troubleshooting (Key Highlight)
The Open vSwitch (OVS) Conflict
During testing, a major connectivity issue arose where the client received a Destination Host Unreachable error during ping attempts.


Analysis: Systematic checks showed Layer 2 (VirtualBox) and Layer 3 (Routing Table) were correct.


Root Cause: The enp0s8 interface was found to be "enslaved" to an Open vSwitch (OVS) bridge.


Technical Explanation: When an interface is attached to an OVS bridge, it loses its ability to process IP packets directly, becoming a simple transparent port.


Resolution: Removed the OVS bridge using ovs-vsctl del-br bridge1, which immediately restored Layer 3 connectivity and successful ICMP pings.

🚀 Validation Results

DHCP Verification: Successful allocation of IP 192.168.10.10 to the Kali client.


DNS Verification: nslookup tests confirmed the resolution of kali.akram.lab via the 192.168.10.1 server.

📁 Repository Structure

/configs: Contains dhcpd.conf, named.conf.options, and zone files.


/documentation: Includes the Full Project Documentation (PDF).


Created by Akram Khoulid 3rd-year Networks and Telecommunications Engineering Student at ENSA Safi.
