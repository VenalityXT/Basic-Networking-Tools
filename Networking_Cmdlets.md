# Basic Networking Command List  
### Cross-Platform Analysis of Windows & Ubuntu Networking Tools

This document provides a structured walkthrough of essential networking utilities across **Windows** and **Ubuntu Linux**, blending practical command output with focused analysis of how each tool reveals the behavior of the underlying network stack.  
The goal is not just to *run* these commands, but to interpret what they show—addressing, routing, ARP resolution, DNS behavior, DHCP negotiation, and how virtualized environments impact all of it.

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

Windows exposes much of its TCP/IP stack through simple but powerful CLI utilities.  
These commands reveal everything from address assignment to routing paths, interface behavior, and ARP cache contents.

Each command below includes **context**, **interpretation**, and **relevant details extracted from the screenshot**, not just surface-level descriptions.

---

## IP Configuration

```PowerShell  
ipconfig  
```

This command provides a quick diagnostic snapshot showing:

- The active interface  
- IPv4 address  
- Subnet mask  
- Default gateway  

<img width="598" height="246" alt="ipconfig" src="https://github.com/user-attachments/assets/e535c7be-c7d2-41de-a285-2abe1f001998" />

### What the screenshot reveals:
- The VM received an address in the `192.168.x.x` range, typical of a home LAN.
- The default gateway is also present—meaning routing is available.
- Subnet mask confirms a standard `/24` network.

This output immediately answers the question:  
**“Am I on the network, and who is my router?”**

---

## Full Adapter Details

```PowerShell  
ipconfig /all  
```

Where the basic `ipconfig` shows the essentials, `/all` exposes the **full interface profile**.

<img width="684" height="480" alt="ipconfig all" src="https://github.com/user-attachments/assets/6eb174a9-530c-4329-b3f4-c293ae22ed33" />

### How to read this output:
| Field | Meaning | Why It Matters |
|-------|---------|----------------|
| Host Name | System's network identity | Important in domain environments |
| Physical Address | MAC | Used for ARP, DHCP, network inventory |
| DHCP Enabled | Yes/No | Tells whether IP is dynamic or static |
| DHCP Server | The leasing source | Should match router or DHCP relay |
| DNS Servers | Resolver list | Unexpected entries = DNS hijack? |
| Lease Obtained/Expires | DHCP timers | Helps diagnose lease conflicts |

### In this screenshot:
- DHCP **is enabled**, confirming the interface expects dynamic addressing.
- DNS is pulled from the router, typical for small networks.
- Lease times are visible, confirming normal DHCP negotiation.

> When diagnosing DNS or DHCP problems, this is the first place to check for mismatched servers, expired leases, or improper static configurations.

---

## Connectivity Tests

```PowerShell  
ping 8.8.8.8  
```

ICMP echo requests validate that packets are leaving the machine and returning from a destination.  

<img width="458" height="212" alt="ping" src="https://github.com/user-attachments/assets/50c692d4-e3ef-44ca-8911-58de3fc166a3" />

### Interpreting the output:
| Field | Meaning | Interpretation |
|-------|---------|----------------|
| Reply | Successful ICMP response | The host is reachable |
| Time | Latency | Under ~40ms = healthy home LAN to Google |
| TTL | Time-to-live | Clue about number of hops |
| Packet Loss | Connectivity issues | 0% is ideal |

This confirms end-to-end reachability and shows no packet loss.

---

## Route Tracing

```PowerShell  
tracert 8.8.8.8  
```

Traceroute reveals exactly **which routers** your packet travels through.

<img width="646" height="334" alt="tracert" src="https://github.com/user-attachments/assets/ca729c1f-93df-4637-a811-b8426ec68d3f" />

### How to read traceroute:
- Each line = a hop (router) between you and the destination.
- `*` means a router didn’t reply but still forwarded packets.
- Increasing latency generally indicates physical distance or processing load.

### Common traceroute concepts:
- **Routing loops:**  
  When traceroute repeats the same hops indefinitely (misconfigured routing).
- **Asymmetric routing:**  
  Outbound path ≠ inbound path; debugging firewalls often reveals this.
- **Dropped hops:**  
  Routers may forward but not respond to ICMP TTL-expired packets.

This screenshot shows a healthy route: hops increase sequentially with no anomalies.

---

## ARP Table

```PowerShell  
arp -a  
```

ARP maps IPv4 addresses to MAC addresses on the **local network**.

<img width="435" height="178" alt="arp" src="httpsgithubusercontent/assets/f2231518-fffd-42ae-8886-c2cb8dec9e87" />

### How to interpret ARP entries:
| Column | Meaning |
|--------|---------|
| Internet Address | IPv4 address on LAN |
| Physical Address | MAC address of discovered device |
| Type | Dynamic vs Static |

### Observations:
- The router’s IP appears with a MAC address—confirming layer-2 visibility.
- Entries are **dynamic**, meaning learned through ARP broadcasts.
- No suspicious duplicate MACs (which can indicate ARP spoofing).

ARP helps answer:  
**“Who is actually on my network, and what hardware do they claim to be?”**

---

# DHCP Failure Case Study (ipconfig /release & /renew)

This was the most instructive part of the project—demonstrating how virtualization impacts network behavior and how DHCP negotiation fails when misconfigured.

---

## The Failure

Attempting:

```PowerShell  
ipconfig /release  
ipconfig /renew  
```

produced the same message both times:

**“The operation failed as no adapter is in the state permissible for this operation.”**

Release attempt:

<img width="539" height="113" alt="ipconfig release" src="httpsgithubusercontent/assets/65d1c3bd-81e6-4871-ba94-9518480f7d87" />

Renew attempt:

<img width="541" height="110" alt="ipconfig renew" src="httpsgithubusercontent/assets/429c7ae6-919c-4594-adae-34fa28f007ee" />

This wasn’t Windows being difficult.  
It simply didn’t *have* a DHCP lease to release or renew.

---

## The Real Root Cause

The VM was running in **NAT mode**.  
In NAT mode:

- VirtualBox is the DHCP server  
- Windows never talks to the real LAN  
- Windows cannot request/release external DHCP leases  
- ipconfig /release & /renew become meaningless

Switching VirtualBox to **Bridged Adapter** puts the VM directly on the LAN, enabling real DHCP interaction.

<img width="988" height="474" alt="fixing ipconfig release and renew commands" src="httpsgithubusercontent/assets/3eb4ca23-a7d3-42bd-b5a5-80640f551b7a" />

After switching modes and restarting the VM, Windows Network Troubleshooter re-enabled DHCP client functionality on the Ethernet adapter.

> The moment bridged mode was enabled, Windows went from  
> “I don’t know who my DHCP server is…”  
> to  
> “Okay, *now* we’re talking to the network.”

---

## After the Fix

Release now worked:

<img width="521" height="186" alt="ipconfig release FI```ED" src="httpsgithubusercontent/assets/704cc8b4-7f26-439f-84f6-d84e91a6a2ad" />

Renew succeeded, acquiring a valid `192.168.x.x` lease:

<img width="521" height="209" alt="ipconfig renew FI```ED" src="httpsgithubusercontent/assets/ee33e651-63eb-4043-9f63-643873ca0764" />

This sequence demonstrates:

- How virtualization settings impact real networking  
- How Windows behaves without DHCP authority  
- How release/renew reflect underlying DHCP states  
- How the Network Troubleshooter resets DHCP client configuration

---

# Additional Windows Networking Tasks

## MAC Address (getmac)

Shows physical MAC addresses bound to Windows interfaces.

```PowerShell  
getmac  
```

<img width="636" height="103" alt="getmac" src="httpsgithubusercontent/assets/5a074a78-c05b-482d-8386-74092323aa2d" />

Use cases:  
- ARP validation  
- Network inventory  
- Troubleshooting mismatched VLAN assignments  
- Detecting MAC spoofing  

---

## DNS Query (nslookup)

```PowerShell  
nslookup google.com  
```

<img width="347" height="145" alt="nslookup" src="httpsgithubusercontent/assets/bcac0146-b7aa-4429-9f8b-dffbf730774a" />

### What this confirms:
- DNS server responding  
- Domain resolution functioning  
- No DNS hijack or redirect  
- Network path supports UDP/TCP queries  

---

# Ubuntu Networking Core Commands

Linux provides deeper, more granular networking tools.  
While Windows focuses on convenience, Linux surfaces the raw details of interfaces, routes, and neighbor caches.

---

## Interface Listing

```Bash  
ifconfig  
```

<img width="859" height="629" alt="ubuntu ifconfig" src="httpsgithubusercontent/assets/c33bf531-7eaf-459a-a671-fa5fc6c0b0b4" />

### What we see:
- Interface `ens33` is up  
- IPv4: in the same subnet as Windows  
- MAC address clearly visible  
- Broadcast & netmask information present  

---

## Connectivity Tests (Linux)

```Bash  
ping -c 4 8.8.8.8  
```

<img width="661" height="227" alt="ubuntu ping" src="httpsgithubusercontent/assets/b52f8124-6cff-4f57-bcbe-c1cbc6cfecd8" />

Linux pings show similar data:

- packet size  
- RTT (round trip times)  
- min/avg/max/mdev latency summary  

---

## Route Tracing (Linux)

```Bash  
traceroute 8.8.8.8  
```

<img width="673" height="732" alt="ubuntu traceroute" src="httpsgithubusercontent/assets/9850b1e5-ee60-4a69-b820-3fe95db9a974" />

Linux traceroute often uses UDP packets (unlike Windows using ICMP), so path differ slightly—but hop structure remains similar.

---

## ARP Table (Linux)

```Bash  
arp -a  
```

<img width="586" height="50" alt="ubuntu arp" src="httpsgithubusercontent/assets/9ea33134-118f-4497-b455-fa7fee71a89f" />

Same interpretation logic as Windows:

- IP → MAC mappings  
- Mostly dynamic entries  
- Router MAC visible  
- Confirms local-network presence  

---

# Additional Ubuntu Tasks

## MAC Address (ip link)

```Bash  
ip link  
```

<img width="1152" height="146" alt="ubuntu ip link" src="httpsgithubusercontent/assets/a21fc817-5c78-40f2-aab7-dd7cf41d075b" />

---

## IP Address (ip addr show)

```Bash  
ip addr show  
```

<img width="1019" height="432" alt="ubuntu ip addr show" src="httpsgithubusercontent/assets/5bb1c38f-0ced-4fff-80f4-e3aaf06583ec" />

---

## DHCP Verification (nmcli)

<img width="912" height="923" alt="ubuntu nmcli device show" src="httpsgithubusercontent/assets/9033bfc6-f0c3-45bb-b764-3a3170273e69" />

This output reveals:

- DHCP server  
- Lease duration  
- DNS servers  
- IPv4 assignment source  
- Associated device  

---

## DHCP Client Request (dhclient)

```Bash  
sudo dhclient -v  
```

[Insert Screenshot – dhclient]

Shows full DORA exchange in real time.

---

# Networking Devices Overview

A quick overview of foundational Layer 1–3 hardware.

---

## Hub (Layer 1)

<img src="httpsgithubusercontent/assets/9c50db83-cfbc-4a14-83c2-a11b9a5d13b8">

- Traffic is repeated to **all** ports  
- No MAC learning  
- Large collision domains  
- Functionally obsolete except in lab scenarios  

---

## Switch (Layer 2)

<img src="httpsgithubusercontent/assets/432c2fd3-098a-4616-b254-9c06390ad337">

- Learns MAC addresses  
- Reduces collisions  
- Creates separate collision domains  
- Most common LAN device today  

---

## Router (Layer 3)

<img src="httpsgithubusercontent/assets/f80b6ad2-1c6b-44dd-87e7-411da775e897">

- Forwards traffic between networks  
- Makes decisions based on IP addresses  
- Performs NAT, ACLs, firewalling  

---

# Networking Theory (MAC, IP, DNS, DHCP, DORA)

## MAC Address
48-bit hardware identifier.  
OUI = vendor prefix + device-specific suffix.

<img src="httpsgithubusercontent/assets/ec6a6d58-5d62-42b7-a4ae-fa7c8223d502">

---

## IP Address
Divided into **network** and **host** portions based on subnet mask.

<img src="httpsgithubusercontent/assets/86a06bdc-9ab9-4bb2-9cb1-c161bc1ba8e8">

---

## DNS
Resolves domain names to IP addresses.  
Critical for accessing both internal and external services.

---

## DHCP  
Assigns:

- IPv4 address  
- Subnet mask  
- Default gateway  
- DNS servers  
- Lease duration  

[Insert Screenshot – DHCP lease info]

---

## DORA  
DHCP’s four-step negotiation:

1. **Discover** – Client broadcasts request  
2. **Offer** – Server provides an available address  
3. **Request** – Client requests the offered configuration  
4. **Acknowledge** – Server finalizes the lease  

[Insert Screenshot – DHCP client request]

---

# Real-World Applications

These tools are used constantly in enterprise environments to:

- Diagnose VLAN/IP addressing conflicts  
- Verify DHCP relay or scope configuration  
- Investigate ARP poisoning or duplicate MAC addresses  
- Debug DNS failures or slow name resolution  
- Trace routing asymmetry across WAN links  
- Validate host reachability during outages  
- Audit interface configurations during provisioning  
- Understand hypervisor networking impact on guest systems  

This set of commands represents the **baseline operational toolkit** for system administrators, Tier 1 SOC analysts, network technicians, and IT professionals responsible for maintaining reliable connectivity and diagnosing network behavior.

