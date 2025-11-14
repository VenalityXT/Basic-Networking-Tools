# Basic Networking Command List  
### Cross-Platform Analysis of Windows & Ubuntu Networking Tools

This document captures a structured, high-fidelity walkthrough of essential networking cmdlets and utilities across **Windows** and **Ubuntu Linux**. Each tool is demonstrated, contextualized, and interpreted with a focus on how real systems behave—both when everything works as intended, and when it absolutely does not.

While the commands themselves are foundational, the value lies in observing how the OS exposes its networking stack at Layers 1–4, how DHCP/DNS/ARP behave in practical environments, and how virtualization settings can quietly break everything behind the scenes.  
This file is written not as a school exercise, but as a **technical resource** someone else could learn from.

---

## Table of Contents
- [Windows Networking Essentials](#windows-networking-essentials)
  - [IP Configuration](#ip-configuration)
  - [Full Adapter Details](#full-adapter-details)
  - [Connectivity Tests](#connectivity-tests)
  - [Route Tracing](#route-tracing)
  - [ARP Table](#arp-table)
- [DHCP Failure Case Study (ipconfig /release & /renew)](#dhcp-failure-case-study-ipconfig-release--renew)
- [Additional Windows Networking Tasks](#additional-windows-networking-tasks)
- [Ubuntu Networking Essentials](#ubuntu-networking-essentials)
  - [Interface Listing](#interface-listing)
  - [Connectivity Tests (Linux)](#connectivity-tests-linux)
  - [Route Tracing (Linux)](#route-tracing-linux)
  - [ARP Table (Linux)](#arp-table-linux)
- [Additional Ubuntu Tasks](#additional-ubuntu-tasks)
- [Networking Devices Overview](#networking-devices-overview)
- [Networking Theory](#networking-theory)
- [Real-World Applications](#real-world-applications)

---

# Windows Networking Essentials

Windows provides a collection of CLI utilities that surface real-time information from the TCP/IP stack. These commands make it possible to inspect addressing, routing, ARP behavior, DNS resolution, and DHCP operations—all without external tools.

Below is a structured breakdown of each major cmdlet.

---

## IP Configuration

```PowerShell  
ipconfig  
```

Displays a concise summary of active interfaces, including IPv4, subnet mask, and the default gateway. Ideal for quick health checks or verifying whether a host is participating on the network as expected.

<img width="598" height="246" alt="ipconfig" src="https://github.com/user-attachments/assets/e535c7be-c7d2-41de-a285-2abe1f001998" />

> **Key Point:**  
> `ipconfig` is the “first responder” of Windows networking commands—fast, simple, and tells you whether you even *have* a network before digging deeper.

---

## Full Adapter Details

```PowerShell  
ipconfig /all  
```

Provides the full interface profile: MAC address, DHCP lease, DNS servers, adapter state, and every parameter pulled from the network.  
This is where DHCP and DNS issues normally reveal themselves.

<img width="684" height="480" alt="ipconfig all" src="https://github.com/user-attachments/assets/6eb174a9-530c-4329-b3f4-c293ae22ed33" />

> **Key Point:**  
> `ipconfig /all` is effectively the “network resume” of your machine—everything the OS believes about its interfaces is here.

---

## Connectivity Tests

```PowerShell  
ping 8.8.8.8  
```

Sends ICMP echo requests to measure latency and confirm whether packets are making it past the local network.

<img width="458" height="212" alt="ping" src="https://github.com/user-attachments/assets/50c692d4-e3ef-44ca-8911-58de3fc166a3" />

---

## Route Tracing

```PowerShell  
tracert 8.8.8.8  
```

Shows the sequence of hops between the host and destination. Useful for routing loops, dropped hops, or asymmetric paths.

<img width="646" height="334" alt="tracert" src="https://github.com/user-attachments/assets/ca729c1f-93df-4637-a811-b8426ec68d3f" />

---

## ARP Table

```PowerShell  
arp -a  
```

Retrieves ARP cache entries, mapping IPv4 → MAC addresses discovered on the LAN.

<img width="435" height="178" alt="arp" src="https://github.com/user-attachments/assets/f2231518-fffd-42ae-8886-c2cb8dec9e87" />

---

# DHCP Failure Case Study (ipconfig /release & /renew)

This section highlights a real-world troubleshooting scenario where DHCP operations failed entirely—demonstrating not only how `ipconfig` works, but how virtualization and OS components interact.

---

## Observing the Failure

Attempting to refresh the VM’s network configuration yielded the same error for both release and renew:

```PowerShell  
ipconfig /release  
ipconfig /renew  
```

**“The operation failed as no adapter is in the state permissible for this operation.”**

Release failure:

<img width="539" height="113" alt="ipconfig release" src="https://github.com/user-attachments/assets/65d1c3bd-81e6-4871-ba94-9518480f7d87" />

Renew failure:

<img width="541" height="110" alt="ipconfig renew" src="https://github.com/user-attachments/assets/429c7ae6-919c-4594-adae-34fa28f007ee" />

> **Key Point:**  
> This was *not* a Windows permission issue. The OS was simply telling the truth:  
> “You can’t renew a DHCP lease you don’t have.”

---

## Finding the Root Cause

The real culprit wasn’t Windows—it was VirtualBox.

The VM was still attached to a **NAT** network. In NAT mode, the hypervisor—not the guest OS—manages DHCP.  
Meaning: the Windows VM had no DHCP relationship with the LAN at all.

Once recognized, the fix was straightforward:

### **1. Switch VirtualBox from NAT → Bridged Adapter**  
This placed the VM on the physical LAN, allowing it to interact with the real DHCP server.

### **2. Restart the VM**  
Refreshing the virtual NIC state.

### **3. Run Windows Network Troubleshooter**  
The troubleshooter re-enabled the DHCP client on the Ethernet adapter.

<img width="988" height="474" alt="fixing ipconfig release and renew commands" src="https://github.com/user-attachments/assets/3eb4ca23-a7d3-42bd-b5a5-80640f551b7a" />

This changed everything.

---

## Validating the Fix

### Release now worked:

<img width="521" height="186" alt="ipconfig release FI```ED" src="https://github.com/user-attachments/assets/704cc8b4-7f26-439f-84f6-d84e91a6a2ad" />

### Renew successfully pulled a fresh 192.168.x.x address:

<img width="521" height="209" alt="ipconfig renew FI```ED" src="https://github.com/user-attachments/assets/ee33e651-63eb-4043-9f63-643873ca0764" />

> **Key Point:**  
> Virtualization layers matter. NAT mode acts like protective glass—great for isolation, terrible for testing real DHCP behavior.

---

# Additional Windows Networking Tasks

## MAC Address Identification

```PowerShell  
getmac  
```

Grabs the system’s physical MAC addresses.

<img width="636" height="103" alt="getmac" src="https://github.com/user-attachments/assets/5a074a78-c05b-482d-8386-74092323aa2d" />

---

## DNS Query

```PowerShell  
nslookup google.com  
```

Confirms active DNS resolver and shows name resolution behavior.

<img width="347" height="145" alt="nslookup" src="https://github.com/user-attachments/assets/bcac0146-b7aa-4429-9f8b-dffbf730774a" />

---

# Ubuntu Networking Essentials

Ubuntu provides parallel tools for inspecting and validating network configuration, but exposes more detail through the `ip` suite and NetworkManager.

---

## Interface Listing

```Bash  
ifconfig  
```

Displays interfaces, MAC addresses, IPv4 config, and link state.

<img width="859" height="629" alt="ubuntu ifconfig" src="https://github.com/user-attachments/assets/c33bf531-7eaf-459a-a671-fa5fc6c0b0b4" />

---

## Connectivity Tests (Linux)

```Bash  
ping -c 4 8.8.8.8  
```

<img width="661" height="227" alt="ubuntu ping" src="https://github.com/user-attachments/assets/b52f8124-6cff-4f57-bcbe-c1cbc6cfecd8" />

---

## Route Tracing (Linux)

```Bash  
traceroute 8.8.8.8  
```

<img width="673" height="732" alt="ubuntu traceroute" src="httpsgithubusercontent/assets/9850b1e5-ee60-4a69-b820-3fe95db9a974" />

---

## ARP Table (Linux)

```Bash  
arp -a  
```

<img width="586" height="50" alt="ubuntu arp" src="https://github.com/user-attachments/assets/9ea33134-118f-4497-b455-fa7fee71a89f" />

---

# Additional Ubuntu Tasks

## MAC Address (ip link)

```Bash  
ip link  
```

<img width="1152" height="146" alt="ubuntu ip link" src="https://github.com/user-attachments/assets/a21fc817-5c78-40f2-aab7-dd7cf41d075b" />

---

## IP Address (ip addr show)

```Bash  
ip addr show  
```

<img width="1019" height="432" alt="ubuntu ip addr show" src="https://githubgithubusercontent/assets/5bb1c38f-0ced-4fff-80f4-e3aaf06583ec" />

---

## DHCP Verification (nmcli device show)

NetworkManager reveals DHCP-provided metadata including lease times, DNS, gateway, and assigned IPv4 info.

<img width="912" height="923" alt="ubuntu nmcli device show" src="https://githubusercontent/assets/9033bfc6-f0c3-45bb-b764-3a3170273e69" />

---

## DHCP Client Request (dhclient)

```Bash  
sudo dhclient -v  
```

[Insert Screenshot – dhclient]

---

# Networking Devices Overview

This section briefly distinguishes the three foundational networking devices at Layers 1–3.

---

## Hub (Layer 1)

- Operates as a repeater  
- Forwards incoming signals to all ports  
- No awareness of MAC addresses  
- Large collision domains  

<img src="https://githubusercontent/assets/9c50db83-cfbc-4a14-83c2-a11b9a5d13b8">

---

## Switch (Layer 2)

- Learns MAC addresses  
- Forwards frames intelligently  
- Reduces collisions  
- Backbone of modern LANs  

<img src="https://githubusercontent/assets/432c2fd3-098a-4616-b254-9c06390ad337">

---

## Router (Layer 3)

- Makes forwarding decisions using IP  
- Routes between networks  
- Often handles NAT, ACLs, and firewall tasks  

<img src="https://githubusercontent/assets/f80b6ad2-1c6b-44dd-87e7-411da775e897">

---

# Networking Theory

## MAC Addressing

48-bit globally unique identifier. First half: manufacturer OUI. Second half: interface-specific value.

<img src="https://githubusercontent/assets/ec6a6d58-5d62-42b7-a4ae-fa7c8223d502">

---

## IP Addressing

IPv4 consists of network + host portions defined by the subnet mask.

<img src="https://githubusercontent/assets/86a06bdc-9ab9-4bb2-9cb1-c161bc1ba8e8">

---

## DNS

Translates human-readable domains into IP addresses. Critical for navigation on internal and external networks.

---

## DHCP

Dynamically provides addressing and routing configuration.  
[Insert Screenshot – DHCP Lease Info]

---

## DORA (Discover → Offer → Request → Acknowledge)

The handshake used for DHCP address acquisition.  
[Insert Screenshot – DHCP Client Request]

---

# Real-World Applications

The commands and concepts in this document are fundamental tools for:

- Diagnosing addressing conflicts across VLANs  
- Validating DHCP relay behavior  
- Investigating ARP poisoning or unexpected MAC mappings  
- Debugging asymmetric routing  
- Inspecting DNS failures or misconfigured resolvers  
- Confirming host reachability during outages  
- Auditing interface states and link-layer connectivity  
- Understanding virtualization network modes and their side effects  

Professionals across IT support, systems administration, networking, and SOC analysis rely on these exact workflows daily.  
While simple at first glance, these commands reveal precisely how machines participate in modern networks—and where things break when they don’t.

