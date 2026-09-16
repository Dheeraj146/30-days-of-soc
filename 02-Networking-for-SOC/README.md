# Day 2 — Networking for SOC

## 🎯 Objective

The objective of Day 2 is to understand computer networking from a SOC analyst's perspective. A SOC analyst constantly works with IP addresses, MAC addresses, ports, protocols, packets, DNS requests, ARP traffic, routing information, and network logs. Understanding how these components communicate is essential for detecting and investigating suspicious network activity.

This day focuses on understanding not only **what a networking component is**, but also **what happens during communication**, **where each address is used**, and **how the same information becomes useful during security monitoring and incident investigation**.

---

## 1. What Is a Network?

A network is a collection of devices that communicate with each other to exchange data and access resources. Devices can communicate directly or through networking equipment such as switches, routers, firewalls, wireless access points, and gateways.

A simple network can contain only two devices. For example, two computers connected through an Ethernet cable can exchange traffic. Larger networks contain hundreds or thousands of endpoints, servers, printers, security appliances, and other systems.

From a SOC perspective, a network is important because attacks also generate network activity. Scanning, brute-force attempts, command-and-control communication, malware downloads, DNS tunneling, data exfiltration, and denial-of-service attacks can all produce observable network traffic.

---

## 2. The Smallest Practical Network

The simplest communication scenario is two devices connected to the same network.

For example:

```text
Device A                         Device B
192.168.1.10                     192.168.1.20
MAC-A                            MAC-B
   |                                |
   +----------- Network ------------+
```

When Device A communicates with Device B on the same local network, multiple networking concepts are involved simultaneously.

- **IP address** identifies the endpoint at Layer 3.
- **MAC address** identifies the network interface at Layer 2 on the local network.
- **Port number** identifies the application/service at the transport layer.
- **Protocol** determines how the communication is performed.

For local Ethernet communication, the sender ultimately needs the destination MAC address to construct the Ethernet frame. The IP addresses remain part of the IP packet carried inside that frame.

Therefore, IP and MAC addresses do not compete with each other. They operate at different layers and solve different problems.

---

## 3. MAC Address

A MAC address is a Layer 2 hardware/interface address used for communication on a local network. Ethernet frames contain source and destination MAC addresses.

Example:

```text
Source MAC      AA:AA:AA:AA:AA:AA
Destination MAC BB:BB:BB:BB:BB:BB
```

A switch primarily uses MAC addresses to determine where an Ethernet frame should be forwarded.

### Why SOC analysts care about MAC addresses

MAC addresses can help identify devices during local-network investigations. They can appear in ARP information, DHCP records, switch logs, wireless controller logs, and forensic evidence.

However, a MAC address normally has local-network significance. Routers do not simply carry the original Ethernet source and destination MAC addresses across the entire Internet. At each routed hop, the Layer 2 frame is rebuilt for the next network segment.

---

## 4. IP Address

An IP address provides logical addressing at Layer 3. It allows systems to identify the source and destination of IP traffic and enables routing between different networks.

Examples:

```text
IPv4: 192.168.1.10
IPv4: 10.0.0.15
IPv6: 2001:db8::10
```

### Private IP addresses

Common private IPv4 ranges are:

- `10.0.0.0/8`
- `172.16.0.0/12`
- `192.168.0.0/16`

Private IP addresses are normally used inside internal networks and are not directly routable on the public Internet.

### Public IP addresses

A public IP address is globally routable and can be used for communication across the Internet, subject to routing and security controls.

### SOC relevance

SOC analysts frequently investigate source and destination IP addresses. An alert may show an internal source IP connecting to a suspicious external IP, or multiple internal systems communicating with the same external address.

---

## 5. MAC Address vs IP Address

MAC and IP addresses have different responsibilities.

| Component | Layer | Main purpose |
|---|---|---|
| MAC address | Layer 2 | Local network/interface addressing |
| IP address | Layer 3 | Logical addressing and routing |
| Port | Layer 4 | Identifying an application/service |

A useful way to understand this is:

```text
MAC  → Who should receive this frame on this local network?
IP   → Which host/network is the packet trying to reach?
Port → Which service/application should handle the traffic?
```

During communication, all of these can participate in the same connection, but they are used for different purposes.

---

## 6. OSI Model

The OSI model divides network communication into seven conceptual layers.

| Layer | Name | Examples |
|---|---|---|
| 7 | Application | HTTP, DNS, SMTP, SSH |
| 6 | Presentation | Encoding, encryption, compression |
| 5 | Session | Session management |
| 4 | Transport | TCP, UDP |
| 3 | Network | IPv4, IPv6, ICMP, routing |
| 2 | Data Link | Ethernet, MAC, ARP |
| 1 | Physical | Cable, radio, electrical/optical signals |

The OSI model is useful for troubleshooting and security analysis because it helps identify where a particular event occurs.

For example, a MAC-address problem is generally associated with Layer 2, while an incorrect IP route is associated with Layer 3 and a blocked TCP port is associated with Layer 4.

---

## 7. TCP/IP Model

In real-world networking, the TCP/IP model is commonly used as a practical representation of network communication.

```text
Application       → HTTP, DNS, SSH, SMTP
Transport         → TCP, UDP
Internet          → IP, ICMP
Network Access    → Ethernet, Wi-Fi
```

SOC analysts encounter TCP/IP concepts constantly because network security tools inspect traffic generated by these protocols.

---

## 8. Encapsulation

When an application sends data, each networking layer adds information required for delivery.

A simplified representation is:

```text
Application Data
      ↓
TCP/UDP Header + Data
      ↓
IP Header + Segment
      ↓
Ethernet Header + IP Packet + Trailer
      ↓
Physical transmission
```

At the destination, the process is reversed through decapsulation.

This is important during packet analysis because a packet capture may expose information from multiple layers simultaneously.

---

## 9. Ethernet Frame, IP Packet, and TCP Segment

These terms describe data at different networking layers.

```text
Ethernet Frame
└── IP Packet
    └── TCP Segment
        └── Application Data
```

An Ethernet frame contains Layer 2 information such as source and destination MAC addresses.

An IP packet contains Layer 3 information such as source and destination IP addresses.

A TCP segment contains Layer 4 information such as source port, destination port, sequence numbers, acknowledgements, and TCP flags.

This layered structure is fundamental to packet analysis in tools such as Wireshark.

---

## 10. Switch

A switch connects devices within a local network and forwards Ethernet frames based primarily on MAC addresses.

The switch learns which MAC address is reachable through which physical port by observing incoming frames. It maintains this information in a MAC address table.

Example:

```text
MAC-A → Port 1
MAC-B → Port 2
MAC-C → Port 3
```

When a frame arrives for MAC-B, the switch can forward it toward Port 2.

### SOC relevance

Switch infrastructure can provide valuable evidence during investigations, especially for identifying which physical or logical interface is associated with a device.

---

## 11. Router

A router connects different IP networks and makes forwarding decisions based on Layer 3 information.

Example:

```text
LAN A: 192.168.1.0/24
          |
        Router
          |
LAN B: 10.0.0.0/24
```

If a host on `192.168.1.0/24` needs to communicate with `10.0.0.0/24`, the traffic may be sent to the router, which determines where the packet should go next.

Routers are critical security telemetry sources because routing devices can generate logs containing source addresses, destination addresses, interfaces, protocols, and other connection information.

---

## 12. Default Gateway

The default gateway is the device a host uses when it needs to communicate with a destination outside its local subnet.

For example:

```text
Host:           192.168.1.10
Subnet:         255.255.255.0
Gateway:        192.168.1.1
```

If the destination is outside `192.168.1.0/24`, the host normally forwards the traffic to the default gateway.

A SOC analyst should understand this because suspicious external connections from an endpoint generally leave the local network through a gateway, router, firewall, or other security control where telemetry may be generated.

---

## 13. ARP — Address Resolution Protocol

ARP is used in IPv4 local networks to discover the MAC address associated with an IPv4 address.

Suppose a host knows:

```text
Destination IP: 192.168.1.20
```

but does not know the destination MAC address. It can send an ARP request asking which device owns that IP address.

Conceptually:

```text
Who has 192.168.1.20?
        ↓
192.168.1.20 is at BB:BB:BB:BB:BB:BB
```

The sender can then use that MAC address when constructing the local Ethernet frame.

### SOC relevance

ARP is important for detecting and investigating attacks such as ARP spoofing/poisoning. An attacker may attempt to associate their MAC address with another host's IP address, potentially enabling traffic interception on a local network.

---

## 14. DHCP — Dynamic Host Configuration Protocol

DHCP automatically provides network configuration to clients.

A DHCP server can provide information such as:

- IP address
- Subnet mask
- Default gateway
- DNS server
- Lease duration

A simplified DHCP process is commonly remembered as **DORA**:

```text
Discover
   ↓
Offer
   ↓
Request
   ↓
Acknowledgement
```

### SOC relevance

DHCP logs can help correlate an IP address with a device or client at a particular time. This is extremely useful because IP addresses may be dynamically assigned.

For example, if an investigation identifies `192.168.1.50` as the source of suspicious traffic, DHCP records may help determine which endpoint had that address at the relevant time.

---

## 15. DNS — Domain Name System

DNS translates human-readable domain names into IP addresses and supports other types of name-resolution information.

Example:

```text
www.example.com
       ↓ DNS query
93.184.216.34
```

Without DNS, users and applications would frequently need to work directly with IP addresses.

### Common DNS record types

- **A** — maps a hostname to an IPv4 address.
- **AAAA** — maps a hostname to an IPv6 address.
- **CNAME** — creates an alias for another hostname.
- **MX** — identifies mail servers.
- **NS** — identifies authoritative name servers.
- **TXT** — stores text information, commonly used for verification and email-security mechanisms.

### SOC relevance

DNS is one of the most valuable sources of network-security telemetry. Malware may perform DNS lookups for command-and-control infrastructure, while attackers may use suspicious domains, fast-flux infrastructure, or DNS tunneling techniques.

---

## 16. Ports

A port identifies a logical endpoint for network communication at the transport layer.

Examples include:

| Port | Common service |
|---:|---|
| 22 | SSH |
| 25 | SMTP |
| 53 | DNS |
| 80 | HTTP |
| 443 | HTTPS |
| 445 | SMB |
| 3389 | RDP |

Ports do not automatically prove which application is actually running. A service can be configured to listen on a non-standard port.

### SOC relevance

Unexpected listening ports or unusual outbound connections can be useful indicators during investigations.

---

## 17. TCP

TCP is a connection-oriented transport protocol. It provides mechanisms for reliable, ordered delivery of data.

A simplified TCP connection begins with the three-way handshake:

```text
Client                    Server
  | ---- SYN ------------> |
  | <--- SYN/ACK ----------|
  | ---- ACK ------------> |
  |                        |
  |     Connection         |
```

Important TCP flags include:

- SYN
- ACK
- FIN
- RST
- PSH
- URG

### SOC relevance

TCP behavior is useful for identifying reconnaissance and suspicious activity. For example, large numbers of SYN packets to many ports can be associated with port-scanning behavior, although context and additional evidence are required before determining that activity is malicious.

---

## 18. UDP

UDP is connectionless and does not provide TCP-style connection establishment and delivery guarantees.

UDP is commonly used for services such as DNS, DHCP, streaming, and other latency-sensitive or application-specific traffic.

Because UDP does not use a TCP handshake, its traffic patterns differ significantly from TCP.

### SOC relevance

Unexpected UDP traffic, unusual destination ports, high-volume UDP traffic, and abnormal DNS behavior can become useful investigation signals depending on the environment.

---

## 19. ICMP

ICMP is used for network-control and diagnostic functions.

A familiar example is `ping`, which commonly uses ICMP Echo Request and Echo Reply messages.

```text
Host A ---- Echo Request ----> Host B
Host A <---- Echo Reply ------ Host B
```

### SOC relevance

ICMP can be used legitimately for troubleshooting but can also appear in reconnaissance, tunneling, or other attack activity. Analysts therefore evaluate ICMP traffic in context rather than treating every ICMP packet as malicious.

---

## 20. NAT — Network Address Translation

NAT allows private IP addresses to communicate with external networks by translating addresses, commonly at a router or firewall.

Example:

```text
Internal Host
192.168.1.10:51520
       |
       | NAT
       ↓
Public Address
203.0.113.10:62001
       |
       ↓
Internet Server
```

NAT is one reason a SOC cannot always identify an individual internal endpoint from an external source IP alone.

Firewall/NAT logs can provide the correlation needed to map an external connection back to an internal host and port.

---

## 21. Private IP vs Public IP During Internet Communication

A typical endpoint may have a private IP such as `192.168.1.10`. When it accesses an Internet service, the packet normally crosses a NAT device and appears externally using a public IP associated with the network.

Simplified flow:

```text
Laptop
192.168.1.10
     |
     ↓
Router / NAT
192.168.1.1 → Public IP
     |
     ↓
Internet
     |
     ↓
Web Server
```

The internal private address and external public address therefore have different scopes and purposes.

For SOC investigations, firewall and NAT logs are particularly important for correlating these two views of the same connection.

---

## 22. Common Network Protocols a SOC Analyst Should Recognize

| Protocol | Purpose | Security relevance |
|---|---|---|
| HTTP | Web traffic | Clear-text web traffic, malicious requests |
| HTTPS | Encrypted web traffic | Web communication, encrypted C2 possibilities |
| DNS | Name resolution | Suspicious domains, tunneling, C2 indicators |
| DHCP | Network configuration | IP/device correlation |
| ARP | Local IPv4 address resolution | ARP spoofing investigations |
| SSH | Secure remote administration | Brute-force and unauthorized access monitoring |
| RDP | Windows remote access | Brute-force and suspicious remote sessions |
| SMB | File/printer sharing | Lateral movement and file access investigations |
| SMTP | Email transfer | Phishing and malicious-email investigations |
| ICMP | Network diagnostics/control | Reconnaissance and abnormal traffic analysis |

---

## 23. Network Traffic and Packets

Network communication is divided into units of data that travel through the network. A packet contains addressing and control information required for delivery.

A packet capture can expose information such as:

- Source IP
- Destination IP
- Source port
- Destination port
- Protocol
- TCP flags
- Packet size
- Timing
- DNS queries
- Application-layer information when available

This makes packet analysis highly valuable to a SOC analyst.

---

## 24. Wireshark

Wireshark is a network protocol analyzer used to capture and inspect network traffic.

A SOC analyst can use Wireshark to investigate:

- Suspicious IP communication
- DNS requests
- TCP handshakes
- Port scanning
- HTTP traffic
- Protocol anomalies
- Potential malware communication
- Unusual outbound connections

Useful display-filter examples include:

```text
ip.addr == 192.168.1.10
```

```text
tcp.port == 443
```

```text
dns
```

```text
icmp
```

```text
tcp.flags.syn == 1
```

These filters help reduce a large packet capture to traffic relevant to the investigation.

---

## 25. Nmap and Network Reconnaissance

Nmap is a network discovery and security-auditing tool. It can identify hosts, services, ports, and other information depending on the scan configuration and target environment.

From a SOC perspective, reconnaissance matters because attackers may perform discovery before exploitation or lateral movement.

A simplified example of suspicious behavior could be:

```text
One source IP
      |
      +--> Port 22
      +--> Port 80
      +--> Port 135
      +--> Port 139
      +--> Port 445
      +--> Port 3389
```

A SOC analyst should investigate the source, target, timing, frequency, and environment context rather than assuming that every scan is malicious.

---

## 26. Network Security Devices

A production network may contain multiple security and networking controls.

### Firewall

Controls traffic according to configured security policies. Firewall logs can contain source/destination IPs, ports, protocols, actions, and timestamps.

### IDS

An Intrusion Detection System monitors traffic and generates alerts when activity matches configured detection logic or signatures.

### IPS

An Intrusion Prevention System can actively block or prevent traffic according to security policies and detection logic.

### Proxy

A proxy acts as an intermediary between clients and external services and can provide visibility and policy enforcement for web or other traffic.

### VPN

A VPN creates an encrypted tunnel between endpoints or networks. VPN logs can be important when investigating remote access and account activity.

---

## 27. Network Telemetry in a SOC

A SOC rarely depends on a single source of network evidence. Multiple telemetry sources are correlated to understand an event.

Example:

```text
Endpoint
   ↓
Firewall
   ↓
DNS Logs
   ↓
Proxy
   ↓
IDS/IPS
   ↓
SIEM
   ↓
SOC Analyst
```

A SIEM can correlate events from different sources. For example, an endpoint's DNS query can be correlated with a subsequent connection to the resolved IP and a firewall event showing that the connection was allowed.

This correlation provides more context than looking at a single log entry.

---

## 28. SOC Investigation Example

Imagine an endpoint generates an alert for communication with a suspicious external IP.

A SOC analyst can investigate in stages:

### Step 1 — Identify the source

Determine which internal host initiated the communication.

### Step 2 — Examine destination information

Review the destination IP, port, protocol, and timing.

### Step 3 — Check DNS

Determine whether the endpoint recently resolved a domain associated with the destination.

### Step 4 — Review endpoint telemetry

Check the process that generated the connection and determine whether the behavior is expected.

### Step 5 — Review network telemetry

Check firewall, proxy, IDS/IPS, VPN, and other relevant logs.

### Step 6 — Establish a timeline

Correlate timestamps across sources.

### Step 7 — Determine scope

Search for other systems communicating with the same destination.

This is the foundation of network-based incident investigation.

---

## 29. Network Indicators Useful in SOC Investigations

Common network indicators include:

- Source IP address
- Destination IP address
- Source port
- Destination port
- Domain name
- URL
- DNS query
- Protocol
- TCP flags
- User agent
- JA3/JA4 or other TLS-related fingerprints where available
- Connection frequency
- Data volume
- First-seen and last-seen timestamps

These indicators can be correlated with endpoint and identity telemetry to build a complete incident picture.

---

## 30. Key SOC Concepts Learned

By completing Day 2, the following concepts are now part of the networking foundation required for SOC analysis:

- Network fundamentals
- MAC addressing
- IPv4 and IPv6 concepts
- Private and public IP addressing
- OSI model
- TCP/IP model
- Encapsulation and decapsulation
- Ethernet frames
- IP packets
- TCP segments
- Switching
- Routing
- Default gateway
- ARP
- DHCP
- DNS
- Ports and protocols
- TCP three-way handshake
- TCP flags
- UDP
- ICMP
- NAT
- Firewalls
- IDS/IPS
- VPN
- Network telemetry
- Packet analysis
- Wireshark
- Network reconnaissance
- SOC network investigations

---

## 🧪 Practical Work

### Lab 1 — Inspect Local Network Configuration

Commands practiced/used:

```bash
ip addr
ip route
ip neigh
```

Windows equivalents:

```cmd
ipconfig /all
route print
arp -a
```

These commands provide visibility into local interfaces, IP configuration, routing, and neighbor information.

### Lab 2 — DNS Investigation

```bash
dig example.com
nslookup example.com
```

The objective is to understand how DNS queries are generated and how DNS responses provide address-resolution information.

### Lab 3 — Packet Capture with Wireshark

Capture traffic and identify:

- Source and destination MAC addresses
- Source and destination IP addresses
- Source and destination ports
- TCP flags
- DNS queries
- ICMP packets

### Lab 4 — Network Discovery

Use Nmap only against systems you own or are explicitly authorized to test.

Example lab command:

```bash
nmap -sV <authorized-lab-target>
```

The objective is to understand how reconnaissance appears from a defender's perspective.

---

## 🔍 SOC Analyst Perspective

Networking knowledge is not separate from SOC operations. It is one of the foundations of SOC work.

When an alert says:

```text
192.168.1.25 → 203.0.113.50:443
```

a SOC analyst should be able to interpret the basic meaning immediately:

- `192.168.1.25` is the source IP.
- `203.0.113.50` is the destination IP.
- `443` is the destination port.
- TCP/HTTPS may be involved, depending on the observed protocol and service.
- The analyst still needs additional evidence to determine whether the communication is legitimate or malicious.

The next step is correlation: identify the endpoint, user, process, DNS activity, firewall decision, historical behavior, and other related events.

This is how raw network data becomes security intelligence.

---

## 📌 Day 2 Outcome

Day 2 established the networking foundation required for subsequent SOC topics. The focus was on understanding how hosts communicate, how addresses and protocols work at different layers, how network traffic can be inspected, and how SOC analysts use network telemetry during detection and investigation.

**Status: ✅ Completed**
