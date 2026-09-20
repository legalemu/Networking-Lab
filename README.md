A small, self-contained network lab I built on a mini PC to practice networking fundamentals and hands-on security work. I practiced scanning, enumeration, exploitation
and the defensive side of each. Everything runs as a virtual machines under Proxmox VE, on an isolated virtual network that's seperated from my home LAN; nothing I do in 
the lab touches real devices.

**Overview**
The lab runs three VMS:

- Kali Linux: the attacker machine (nmap, Metasploit, manual tooling)
- Ubuntu: a general linux host, used as a second targt for practicing and hardening
- Metasploitable 2: a deliberately vulnerable VM, full of open ports and services, used as the main target.

The key part of the design is that the two lab targets sit on their own virtual network with no parth to my home LAN or the internet. Scanning + exploitation fully
contained.

**Network Design:**
Proxmox uses two Linux Bridges:

- vmbr0: bridged to the physical NIC, carries home LAN and gives Proxmox host itself internet access and a management IP.
- vmbr1: an internal bridge with no physical port attached (bridge-ports none). Isolated lab network. Only VMs on vmbr1 talk to eachother.

**IP Addressing**
- AT&T gateway: vmbr0, 192.168.1.254, ISP Router (home LAN)
- Proxmox host: vmbr0, 192.168.1.50, Hypervisor/Management
- Kali: vmbr1, 10.10.10.5, Attacker
- Ubuntu: vmbr1, 10.10.10.10, Linux Host/target
- Metasploitable2: vmbr1, 10.10.10.20, Vulnerable target

**Hardware:**
- Mini PC: Lenovo Thinkcentre 5910q, 8GB RAM, 256GB NvME, Intel Core i5-7500t (2.5 Ghz, 4 cores)
- Switch: TP-Link T-SG108 (unmanaged, no VLAN support, why isolation was done)
- Gateway: AT&T ISP router/gateway

**Verifying Isolation**
The whole lab rests on the lab being cut off from the home network, so I tested it rather than assuming. From Kali on vmbr1:
- ping 192.168.1.254

This fails, which means Kali has no route to the Home LAN or AT&T gateway. Although, Kali can reach the other lab hosts
- ping 10.10.10.20 # succeeds
Metasploitable 2, same isolated network

**What I do**
1. Enumeration: Scan the target and read. For every open port, idnetify service, version, and if it has vulnerabilities.
   - nmap -sn 10.10.10.0/24
     - #host discovery
   - nmap -n -sV -p- 10.10.10.20 # full port + service versions
     - -n skips DNS since there is no resolver on isolated network

2. Manual investigation: Before automated tools, connect to services by hand (ftp, telnet, smbclient). Undertand what the automated tools
   are actually doing.
   
3. Exploitation: Confirm and exploit known vulnerabilities manually with the Metasploit framework on Kali
   
4. Remediation: For each finding, document how I'd actually fix it.

**Thats that broke (and how I fixed them):**
Troubleshooting was a lot of the learning, examples:

1. Metasploitable boot-looped into iPXE: The imported disk wasn't attached to the boot order, VM found nothing bootable
   and fell through the network boot. Fixed by attachig the disk as IDE and ticking it in the boot order.
   
2. Bridge vmbr1 doesn't exist: The bridge was defined in the web UI but never applied, Proxmox keeps changes in interfaces.new until you click
   "Apply Configuration"
   
3. Could not find eth0 on Ubuntu: Ubuntu used predictable interfaces names like ens18, not eth0. Kaili is an exception because it ships with
   predictable naming disabled.

**Skills / Objectives covered**
- Type 1 hypervisor set up and VM management (Proxmox VE) to build a segmented, statically-addressed network.
  reinforces subnetting and segmentation concepts found in Network+.
- Network segmentation and isolation using virtual bridges
- Static IP addressing and subnetting on an isolated network
- Linux networking config (/etc/network/interfaces)
- Port scanning, servie enumeration, and vulnerability identifation

_Built as angoing project. Documented as I go rather than after the fact._
