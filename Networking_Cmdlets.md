# Basic Networking Command List  
### Cross-Platform Analysis of Windows & Ubuntu Networking Tools

This document provides a structured walkthrough of essential networking utilities across **Windows** and **Ubuntu Linux**, blending practical command output with focused analysis of how each tool reveals the behavior of the underlying network stack. The goal is not just to *run* these commands, but to interpret what they show—addressing, routing, ARP resolution, DNS behavior, DHCP negotiation, and how virtualized environments impact all of it.

The document is organized for clarity, readability, and technical depth, using varied Markdown formatting to create a resource that is both visually engaging and practically informative.

---

## Table of Contents
- [Windows Networking Core Commands](#windows-networking-core-commands)
- [IP Configuration](#ip-configuration)
- [Full Adapter Details](#full-adapter-details)
- [Connectivity Tests](#connectivity-tests)
- [Route Tracing](#route-tracing)
- [ARP Table](#arp-table)
- [DHCP Failure Case Study (ipconfig /release & /renew)](#dhcp-failure-case-study-ipconfig-release--renew)
- [Additional Windows Networking Tasks](#additional-windows-networking-tasks)
- [Ubuntu Networking Core Commands](#ubuntu-networking-core-commands)
- [Interface Listing](#interface-listing)
- [Connectivity Tests (Linux)](#connectivity-tests-linux)
- [Route Tracing (Linux)](#route-tracing-linux)
- [ARP Table (Linux)](#arp-table-linux)
- [Additional Ubuntu Tasks](#additional-ubuntu-tasks)
- [Networking Devices Overview](#networking-devices-overview)
- [Networking Theory (MAC, IP, DNS, DHCP, DORA)](#networking-theory-mac-ip-dns-dhcp-dora)
- [Real-World Applications](#real-world-applications)

---

# Windows Networking Core Commands

Windows exposes much of its TCP/IP stack through simple but powerful CLI utilities. These commands reveal everything from address assignment to routing paths, interface behavior, and ARP cache contents.

Each command below includes **context**, **interpretation**, and **details extracted from the screenshot**, not just surface-level descriptions.

---

## IP Configuration  

```  
ipconfig  
```

This command provides a quick diagnostic snapshot showing:

- The active interface  
- IPv4 address  
- Subnet mask  
- Default gateway  

<img width="598" height="246" alt="ipconfig" src="https://github.com/user-attachments/assets/e535c7be-c7d2-41de-a285-2abe1f001998" />

### What the screenshot reveals:

- VM received an address in the `192.168.x.x` range.  
- Default gateway is present — routing is active.  
- Mask confirms standard `/24` network.

This output immediately answers:

> **“Am I on the network, and who is my router?”**

---

## Full Adapter Details  

```  
ipconfig /all  
```

The `/all` option exposes the **entire** adapter profile.

<img width="684" height="480" alt="ipconfig all" src="https://github.com/user-attachments/assets/6eb174a9-530c-4329-b3f4-c293ae22ed33" />

### How to read this output:

| Field | Meaning | Why It Matters |
|-------|---------|----------------|
| Host Name | Local machine identity | Important for domain joins |
| Physical Address | MAC address | DHCP, ARP, inventory |
| DHCP Enabled | Yes/No | Static misconfigs cause outages |
| DHCP Server | Lease source | Should be router or DHCP relay |
| DNS Servers | Resolver order | Wrong entries → DNS hijack |
| Lease Times | Renewal intervals | Conflict detection |

This screenshot shows:

- DHCP **enabled**  
- DNS from router  
- Normal lease timings  

Useful when diagnosing DHCP or DNS conflicts.

---

## Connectivity Tests  

```  
ping 8.8.8.8  
```

<img width="458" height="212" alt="ping" src="https://github.com/user-attachments/assets/50c692d4-e3ef-44ca-8911-58de3fc166a3" />

### Interpreting the ICMP output:

| Field | Interpretation |
|-------|----------------|
| Reply | Host is reachable |
| TTL | Rough hop-distance |
| Time | Latency measurement |
| Packet Loss | Connection stability |

Ping validates basic reachability and tests latency on the path.

---

## Route Tracing  

```  
tracert 8.8.8.8  
```

<img width="646" height="334" alt="tracert" src="https://github.com/user-attachments/assets/ca729c1f-93df-4637-a811-b8426ec68d3f" />

### How to read traceroute results:

- Each hop = router between client and destination.  
- `*` = router forwarded packet but didn’t respond.  
- Increasing latency = physical distance or router load.

Common issues traceroute reveals:

- **Routing loops** (same hop repeating)
- **Asymmetric routing** (return path != outbound)
- **Filtered hops** (firewalls blocking TTL-expired ICMP)

---

## ARP Table  

```  
arp -a  
```

<img width="435" height="178" alt="arp" src="https://github.com/user-attachments/assets/93334ba8-8710-4c1b-8f81-0c7e0e827f92" />

### ARP reveals L2 identities:

- Maps **IPv4 → MAC**
- Shows learned devices on LAN
- Detects duplicate MAC anomalies
- Verifies router MAC presence

---

# DHCP Failure Case Study (ipconfig /release & /renew)

This section demonstrates how virtualization settings impact real network behavior and why DHCP negotiation failed at first.

---

## The Failure  

```  
ipconfig /release  
ipconfig /renew  
```

Both commands returned:

**“The operation failed as no adapter is in the state permissible for this operation.”**

Release attempt:

<img width="539" height="113" alt="ipconfig release" src="https://github.com/user-attachments/assets/9668e726-2b14-43e3-8b9d-bba7ddfc3e27" />

Renew attempt:

<img width="541" height="110" alt="ipconfig renew" src="https://github.com/user-attachments/assets/a15bcd84-321d-4ae4-947d-90991c8dd0bc" />

---

## The Real Root Cause

The VM was in **NAT mode**, meaning:

- VirtualBox acted as the DHCP server  
- The VM wasn’t truly on the LAN  
- Windows could not contact the real DHCP server  
- Release/Renew had nothing to operate on  

Switching to **Bridged Adapter** mode fixed this.

<img width="988" height="474" alt="fixing ipconfig release and renew commands" src="https://github.com/user-attachments/assets/60505178-2f87-4ae2-92ae-cf08c41185fe" />

After the change + VM restart, the Windows Network Troubleshooter automatically re-enabled DHCP on the adapter.

---

## After the Fix

Release:

<img width="521" height="186" alt="ipconfig release FI```ED" src="https://github.com/user-attachments/assets/592b4a19-cad5-4ce7-81a5-7fe4eb35a190" />

Renew:

<img width="521" height="209" alt="ipconfig renew FI```ED" src="https://github.com/user-attachments/assets/620bddac-3fbd-4b82-baba-04014b9cd4df" />

This confirmed:

- The VM is now on the real LAN  
- DHCP lease negotiation works  
- Troubleshooter reset DHCP client behavior  
- Bridged mode allows real network interaction  

---

# Additional Windows Networking Tasks

## MAC Address (getmac)

```  
getmac  
```

<img width="636" height="103" alt="getmac" src="https://github.com/user-attachments/assets/a5d351d3-074d-4496-924b-624b8a4c5bc7" />

---

## DNS Query (nslookup)

```  
nslookup google.com  
```

<img width="347" height="145" alt="nslookup" src="https://github.com/user-attachments/assets/c8972386-dc58-4e1f-bb7d-209858ec10fc" />

Confirms:

- DNS server reachable  
- DNS resolution working  
- No hijacking or redirection  

---

# Ubuntu Networking Core Commands

## Interface Listing

```  
ifconfig  
```

<img width="859" height="629" alt="ubuntu ifconfig" src="https://github.com/user-attachments/assets/0ded335b-0055-46c2-b4fa-62f7be68c5fb" />

---

## Connectivity Tests (Linux)

```  
ping -c 4 8.8.8.8  
```

<img width="661" height="227" alt="ubuntu ping" src="https://github.com/user-attachments/assets/384398a3-345a-489e-8c2d-462878433a1b" />

---

## Route Tracing (Linux)

```  
traceroute 8.8.8.8  
```

<img width="673" height="732" alt="ubuntu traceroute" src="https://github.com/user-attachments/assets/9efaa038-0924-4030-8661-6adfbd4aa7fd" />

---

## ARP Table (Linux)

```  
arp -a  
```

<img width="586" height="50" alt="ubuntu arp" src="https://github.com/user-attachments/assets/84739b30-c203-4d20-a3f4-5026c22a8ffb" />

---

# Additional Ubuntu Tasks

## MAC Address

```  
ip link  
```

<img width="1152" height="146" alt="ubuntu ip link" src="https://github.com/user-attachments/assets/25b05bf4-0522-410e-8250-7b9ac3fff7ff" />

---

## IP Address

```  
ip addr show  
```

<img width="1019" height="432" alt="ubuntu ip addr show" src="https://github.com/user-attachments/assets/a9d57c03-f448-45d1-b17e-1c3a107113a3" />

---

## DHCP Verification (nmcli)

<img width="912" height="923" alt="ubuntu nmcli device show" src="https://github.com/user-attachments/assets/2d667bf5-1bb2-46fa-9022-e2a7464eea40" />

Reveals:

- DHCP server  
- Lease time  
- DNS servers  
- Address source  

---

## DHCP Client Request

```  
sudo dhclient -v  
```

> Shows the full DORA process in real time.

---

# Networking Devices Overview

## Hub (Layer 1)

![Hub drawio](https://github.com/user-attachments/assets/ffbeb430-4a2e-443a-9aa7-5c6c10c68fea)

> A hub repeats *everything* to *everyone* — the “shouting in a room” device.

---

## Switch (Layer 2)

![Switch drawio](https://github.com/user-attachments/assets/67ada540-a86c-4723-95cb-6011ff0851ac)

> A switch learns MAC addresses and forwards frames intelligently.

---

## Router (Layer 3)

![Router drawio](https://github.com/user-attachments/assets/2f573cfa-99c1-4df6-a47e-ffecd8755aec)

> The router connects your network to *other* networks — the “world beyond your neighborhood.”

---

# Networking Theory (MAC, IP, DNS, DHCP, DORA)

## MAC Address  

> 48-bit burned-in hardware identifier.

![MAC drawio](https://github.com/user-attachments/assets/a4f43230-9832-4fc3-9d3e-10c051c4c45e)

---

## IP Address  

> Divided into **network** and **host** sections using the subnet mask.

![IP drawio](https://github.com/user-attachments/assets/bd15cd8d-9f64-42e5-8f1f-77f643aeea41)

---

## DNS  

> Translates domain names into IP addresses.

---

## DHCP  

Assigns:

- IPv4  
- Mask  
- Gateway  
- DNS  
- Lease duration  

---

## DORA  

1. Discover  
2. Offer  
3. Request  
4. Acknowledge  

---

# Real-World Applications

These tools are used constantly in enterprise networking work:

- Diagnose VLAN misalignment  
- Validate DHCP scopes  
- Investigate ARP poisoning  
- Debug DNS failures  
- Trace WAN routing asymmetry  
- Confirm host reachability  
- Validate hypervisor networking  
- Inventory networking hardware  

This command set represents the **baseline operational toolkit** for system administration, enterprise networking, and Tier 1/2 SOC analysis.

---






