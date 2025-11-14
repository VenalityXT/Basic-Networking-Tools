Basic Networking Command List  
Applied Networking Concepts – Windows and Ubuntu

This report documents hands-on interaction with the Windows and Ubuntu networking stacks using common administrative utilities. All tasks were performed inside VirtualBox virtual machines, with screenshots captured throughout. The work includes analyzing IP configuration, validating DHCP behavior, resolving DNS, inspecting ARP tables, tracing routes, and reviewing foundational networking theory. Troubleshooting is also incorporated, including resolving DHCP failures through Windows' automated repair tools.

  
**Windows Networking – Command Usage and Analysis**

```PowerShell
ipconfig
```  
Used to display core network details such as IPv4 address, subnet mask, and default gateway. This provides a quick view of the system’s active interface configuration.  
[Screenshot – ipconfig]

```PowerShell
ipconfig /all
```  
Shows full adapter configuration including hostname, MAC address, DHCP status, DNS servers, lease information, and detailed network parameters.  
[Screenshot – ipconfig /all]

```PowerShell
ipconfig /release
```  
Attempts to drop the current DHCP lease. The command initially failed with “The operation failed as no adapter is in the state permissible for this operation,” which indicated the Ethernet interface was not using DHCP. The issue was resolved by running the Windows Network Troubleshooter, which re-enabled DHCP for the adapter. After DHCP was restored, the command successfully released the address.  
[Screenshot – release (error)]  
[Screenshot – release (fixed)]

```PowerShell
ipconfig /renew
```
Requests a new DHCP lease. Prior to fixing DHCP, the command returned an error. Once DHCP was re-enabled via troubleshooting, Windows successfully obtained a valid 192.168.x.x address.  
[Screenshot – renew (error)]  
[Screenshot – renew (fixed)]

```PowerShell
ping 8.8.8.8
```  
Used to verify connectivity and measure round-trip time to Google’s public DNS server. Confirms ICMP operation and basic external reachability.  
[Screenshot – ping]

```PowerShell
tracert 8.8.8.8
```
Displays the route traffic takes from the local machine to the destination, showing each routing hop along the path.  
[Screenshot – tracert]

```PowerShell
arp -a
```
Displays the ARP table, which contains mappings between IPv4 addresses and their corresponding MAC addresses.  
[Screenshot – ARP Table]

  
**Windows – Additional Networking Tasks**

Obtaining the MAC Address  
`getmac` was used to retrieve the system’s physical (MAC) address.  
[Screenshot – getmac]

Obtaining the IP Address  
Identified using `ipconfig`.  
[Screenshot – ipconfig]

Obtaining ARP Table Mappings  
Performed using `arp -a`.  
[Screenshot – arp]

DNS Server Query  
`nslookup google.com` was used to identify the DNS resolver and confirm successful name resolution.  
[Screenshot – nslookup]

Explanation of the ARP Table  
The ARP table stores IPv4-to-MAC address mappings that allow communication on the local network. Windows dynamically updates these entries as devices are discovered. The table enables Layer 3 to Layer 2 translation so that Ethernet frames can be delivered to the correct physical device.

---
  
## Ubuntu Linux Networking – Command Usage and Analysis

```Bash
ifconfig
```  
Displays interface information such as IPv4 address, broadcast address, netmask, and MAC address.  
[Screenshot – ifconfig]

```Bash
ping -c 4 8.8.8.8
```
Tests connectivity to an external host with a fixed number of ICMP requests.  
[Screenshot – ping (Ubuntu)]

```Bash
traceroute 8.8.8.8
```  
Shows the path each packet takes across intermediary routers to reach the destination.  
[Screenshot – traceroute]

```Bash
arp -a
```
Shows IP-to-MAC address mappings known by the local Linux system.  
[Screenshot – arp (Ubuntu)]

  
**Ubuntu – Additional Networking Tasks**

MAC Address  
Identified using ```ip link```.  
[Screenshot – ip link]

IP Address  
Identified using ```ip addr show```.  
[Screenshot – ip addr show]

ARP Table Mapping  
Displayed using ```arp -a```.  
[Screenshot – arp]

DHCP Verification  
The Xnmcli device showX command confirms DHCP-provided IPv4 configuration and shows DHCP options delivered by the server.  
[Screenshot – nmcli]

DHCP Client Request  
Xsudo dhclient -vX was used to demonstrate a DHCP discovery and request sequence from the Linux VM.  
[Screenshot – dhclient]

  
**Networking Devices – Hub, Switch, Router**

Hub  
A hub operates at the Physical Layer (Layer 1). It repeats incoming signals out all other ports without inspecting traffic. It does not maintain MAC tables, does not filter frames, and broadcasts all traffic. Common form factors include small desktop devices with at least four ports.  
[Insert Hub Image]

Switch  
A switch operates at the Data Link Layer (Layer 2). It maintains a MAC address table and forwards frames only to the correct destination port. This reduces collisions and improves efficiency. Switches typically start at four to eight ports and come in desktop or rack-mount form factors.  
[Insert Switch Image]

Router  
A router operates at the Network Layer (Layer 3). It makes forwarding decisions based on IP addresses, performs routing between networks, and may implement NAT and firewall functions. Routers generally include at least two interfaces (LAN/WAN) and are available as home devices or enterprise rack-mounted units.  
[Insert Router Image]

  
**Networking Theory – MAC, IP, DNS, DHCP, DORA**

What Makes Up a MAC Address  
A MAC address is a 48-bit identifier composed of an Organizationally Unique Identifier (first 24 bits) and a device-specific identifier (last 24 bits). It uniquely identifies a network interface at Layer 2.  
[Screenshot – MAC address example]

What Makes Up an IP Address  
An IPv4 address consists of a network portion and a host portion, defined by the subnet mask. The network portion identifies the network, while the host portion identifies devices within that network.  
[Screenshot – IP address example]

Explanation of DNS  
DNS resolves human-readable domain names into numerical IP addresses. It is essential for locating servers, web services, and network resources.

Explanation of DHCP  
DHCP automatically assigns IP addresses and related configuration (gateway, DNS, subnet mask). This allows systems to join networks without manual configuration.  
[Screenshot – DHCP lease information]

Explanation of the DORA Process  
DHCP uses a four-step sequence:  
• Discover – The client broadcasts a request for configuration  
• Offer – The DHCP server responds with an available address  
• Request – The client formally requests that address  
• Acknowledge – The server approves the lease and provides configuration  
[Screenshot – DHCP client request]


