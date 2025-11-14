# Basic Networking Command List  
Applied Networking Concepts – Windows and Ubuntu

This lab explores how operating systems expose networking functionality through command-line tools. By working inside isolated Windows and Ubuntu virtual machines, I used foundational cmdlets and utilities to analyze live network behavior, inspect interface configuration, validate DHCP and DNS operations, map ARP tables, and observe routing paths in action.  

More importantly, this project demonstrates the practical skill of interpreting what the system is *doing* beneath the surface—how packets flow, where addresses come from, and how the OS responds when things go wrong. Issues encountered during testing, such as DHCP failures on Windows, were intentionally documented and resolved using built-in troubleshooting methods to reflect real troubleshooting workflow.

---

# Windows Networking – Command Usage and Analysis

### IP Configuration (ipconfig)
XPowerShell  
ipconfig  
X  

This command delivers a concise snapshot of the system’s current network state—IPv4 address, subnet mask, and default gateway. It’s the fastest way to verify whether a host is configured correctly and actively connected.

<img width="598" height="246" alt="ipconfig" src="https://github.com/user-attachments/assets/e535c7be-c7d2-41de-a285-2abe1f001998" />

### Full Adapter Details (ipconfig /all)
XPowerShell  
ipconfig /all  
X  

This expanded view includes everything the OS knows about each interface: MAC addresses, DHCP lease times, DNS servers, interface states, and extended network parameters. It’s the command you use when you need the *why* behind connectivity issues.

<img width="684" height="480" alt="ipconfig all" src="https://github.com/user-attachments/assets/6eb174a9-530c-4329-b3f4-c293ae22ed33" />

### Releasing the DHCP Lease (ipconfig /release)
XPowerShell  
ipconfig /release  
X  

Initially, the release attempt failed. Windows displayed:  
**“The operation failed as no adapter is in the state permissible for this operation.”**

This error revealed that DHCP was disabled on the Ethernet interface. Using Windows’ built-in Network Troubleshooter successfully re-enabled DHCP, after which the release operation worked normally—an excellent real-world example of diagnosing and fixing environmental configuration issues.

<img width="539" height="113" alt="ipconfig release" src="https://github.com/user-attachments/assets/65d1c3bd-81e6-4871-ba94-9518480f7d87" />

<img width="521" height="186" alt="ipconfig release FIXED" src="https://github.com/user-attachments/assets/704cc8b4-7f26-439f-84f6-d84e91a6a2ad" />

### Requesting a New Lease (ipconfig /renew)
XPowerShell  
ipconfig /renew  
X  

After DHCP was restored, the interface immediately pulled a valid 192.168.x.x address from the router. This command is commonly used to refresh stale or incorrect addressing without requiring a reboot.

<img width="541" height="110" alt="ipconfig renew" src="https://github.com/user-attachments/assets/429c7ae6-919c-4594-adae-34fa28f007ee" />

<img width="521" height="209" alt="ipconfig renew FIXED" src="https://github.com/user-attachments/assets/ee33e651-63eb-4043-9f63-643873ca0764" />

### Connectivity Test (ping)
XPowerShell  
ping 8.8.8.8  
X  

ICMP echo requests provide a quick and reliable way to test reachability and latency. Pinging 8.8.8.8 (Google’s DNS server) is a standard troubleshooting step.

<img width="458" height="212" alt="ping" src="https://github.com/user-attachments/assets/50c692d4-e3ef-44ca-8911-58de3fc166a3" />

### Route Tracing (tracert)
XPowerShell  
tracert 8.8.8.8  
X  

This command reveals each hop between the host and the destination, showing where delays or failures occur along the path.

<img width="646" height="334" alt="tracert" src="https://github.com/user-attachments/assets/ca729c1f-93df-4637-a811-b8426ec68d3f" />

### ARP Table Inspection (arp -a)
XPowerShell  
arp -a  
X  

ARP translates IPv4 addresses into MAC addresses. Viewing this table shows which devices the system has communicated with on the local network.

<img width="435" height="178" alt="arp" src="https://github.com/user-attachments/assets/f2231518-fffd-42ae-8886-c2cb8dec9e87" />

---

# Windows – Additional Networking Tasks

### MAC Address Identification
`getmac` retrieves the system’s physical MAC addresses—useful for ARP verification, filtering, inventory, or troubleshooting.

<img width="636" height="103" alt="getmac" src="https://github.com/user-attachments/assets/5a074a78-c05b-482d-8386-74092323aa2d" />

### DNS Query (nslookup)
This verifies DNS server availability and confirms that domain resolution is working properly.

<img width="347" height="145" alt="nslookup" src="https://github.com/user-attachments/assets/bcac0146-b7aa-4429-9f8b-dffbf730774a" />

### ARP Table Explanation
The ARP table maps IPv4 addresses to MAC addresses. Without it, local-network communication would not function. ARP dynamically updates based on broadcasts and replies, allowing the system to determine where to send Ethernet frames at Layer 2.

---

# Ubuntu Linux Networking – Command Usage and Analysis

### Interface Listing (ifconfig)
XBash  
ifconfig  
X  

Displays the system’s network interfaces, IPv4 configuration, and MAC addresses.

<img width="859" height="629" alt="ubuntu ifconfig" src="https://github.com/user-attachments/assets/c33bf531-7eaf-459a-a671-fa5fc6c0b0b4" />

### Connectivity Test (ping)
XBash  
ping -c 4 8.8.8.8  
X  

Sends four ICMP echo requests to verify basic network connectivity.

<img width="661" height="227" alt="ubuntu ping" src="https://github.com/user-attachments/assets/b52f8124-6cff-4f57-bcbe-c1cbc6cfecd8" />

### Route Tracing (traceroute)
XBash  
traceroute 8.8.8.8  
X  

Shows the routers a packet passes through on its way to the target.

<img width="673" height="732" alt="ubuntu traceroute" src="https://github.com/user-attachments/assets/9850b1e5-ee60-4a69-b820-3fe95db9a974" />

### ARP Table (arp -a)
XBash  
arp -a  
X  

Displays the ARP mappings Linux has learned from local network communication.

<img width="586" height="50" alt="ubuntu arp" src="https://github.com/user-attachments/assets/9ea33134-118f-4497-b455-fa7fee71a89f" />

---

# Ubuntu – Additional Networking Tasks

### MAC Address (ip link)
XBash  
ip link  
X  

<img width="1152" height="146" alt="ubuntu ip link" src="https://github.com/user-attachments/assets/a21fc817-5c78-40f2-aab7-dd7cf41d075b" />

### IP Address (ip addr show)
XBash  
ip addr show  
X  

<img width="1019" height="432" alt="ubuntu ip addr show" src="https://github.com/user-attachments/assets/5bb1c38f-0ced-4fff-80f4-e3aaf06583ec" />

### DHCP Verification (nmcli device show)
Linux’s NetworkManager provides in-depth DHCP details including lease options, gateway, DNS, and addressing information.

<img width="912" height="923" alt="ubuntu nmcli device show" src="https://github.com/user-attachments/assets/9033bfc6-f0c3-45bb-b764-3a3170273e69" />

### DHCP Client Request (dhclient)
XBash  
sudo dhclient -v  
X  

[Insert Screenshot – dhclient]

---

# Networking Devices – Hub, Switch, Router

### Hub
A basic Layer 1 repeater. Hubs forward electrical signals to all ports, creating large collision domains. Simple but inefficient.

<img width="800" height="450" alt="image" src="https://github.com/user-attachments/assets/9c50db83-cfbc-4a14-83c2-a11b9a5d13b8" />

### Switch
Operates at Layer 2. Maintains a MAC address table to intelligently forward frames only to the correct port, significantly reducing collisions.

<img width="800" height="450" alt="image" src="https://github.com/user-attachments/assets/432c2fd3-098a-4616-b254-9c06390ad337" />

### Router
Layer 3 device responsible for forwarding packets across networks, enforcing boundaries, performing NAT, and managing routes.

<img width="800" height="233" alt="image" src="https://github.com/user-attachments/assets/f80b6ad2-1c6b-44dd-87e7-411da775e897" />

---

# Networking Theory – MAC, IP, DNS, DHCP, DORA

### MAC Addresses
A MAC address is a globally unique 48-bit identifier consisting of a manufacturer prefix (OUI) and a device-specific suffix.

<img width="800" height="674" alt="image" src="https://github.com/user-attachments/assets/ec6a6d58-5d62-42b7-a4ae-fa7c8223d502" />

### IP Addresses
An IPv4 address contains a network portion and a host portion, defined by the subnet mask. This structure enables routing and device identification across networks.

<img width="823" height="402" alt="image" src="https://github.com/user-attachments/assets/86a06bdc-9ab9-4bb2-9cb1-c161bc1ba8e8" />

### DNS Overview
DNS resolves domain names (like google.com) into IP addresses. Without DNS, users would be manually typing numerical IPv4 addresses into every service they access.

### DHCP Overview
DHCP dynamically assigns IP addressing information to clients—IP, subnet mask, router, DNS—and maintains timed leases.  
[Insert Screenshot – DHCP lease info]

### DORA Process
Discover → Offer → Request → Acknowledge  
This is the four-step handshake a DHCP client uses to obtain an address.  
[Insert Screenshot – DHCP client request]

