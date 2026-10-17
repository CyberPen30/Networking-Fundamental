# Advanced Networking Concepts and Analysis

This repository contains a comprehensive collection of practical networking laboratories demonstrating core and advanced Linux networking concepts. The exercises cover TCP/IP packet analysis, network routing, network segmentation, IP addressing, subnetting, and network troubleshooting using Kali Linux and the OWASP Broken Web Applications virtual machine.

The project emphasizes hands-on learning through command-line networking utilities, packet inspection tools, and Linux networking features to understand how devices communicate across networks.

## Table of Contents

- [Environment Prerequisites](#environment-prerequisites)
- [Lab 1: Understanding Network Layers and TCPIP Model](#lab-1-understanding-network-layers-and-tcpip-model)
- [Lab 2: Exploring Network Interfaces and Packet Transmission](#lab-2-exploring-network-interfaces-and-packet-transmission)
- [Lab 3: TCPIP Protocol Stack and Packet Inspection](#lab-3-tcpip-protocol-stack-and-packet-inspection)
- [Lab 4: Simulating Network Routing and VLAN Configuration](#lab-4-simulating-network-routing-and-vlan-configuration)
- [Lab 5: IP Address Analysis and Network Report](#lab-5-ip-address-analysis-and-network-report)
- [Learning Outcomes](#learning-outcomes)
- [Safety and Lab Guidelines](#safety-and-lab-guidelines)

---

## Environment Prerequisites

To successfully execute the laboratories in this repository, the following environment and tools are required.

### Operating Environment

- **Host operating system:** Kali Linux.
- **Target system:** OWASP Broken Web Applications virtual machine, configured and reachable on the lab network.
- **Virtualization platform:** VirtualBox or VMware.
- **Terminal access:** Kali Linux terminal with appropriate administrative privileges for privileged networking operations.

### Networking Tools

| Category | Tools |
|---|---|
| Connectivity testing | `ping`, `traceroute` |
| Interface configuration | `ip`, `ifconfig` |
| Connection monitoring | `netstat`, `ss` |
| Network scanning | `nmap` |
| Address resolution | `arp`, `ip neigh`, `arp-scan` |
| Packet capture | `tcpdump`, Wireshark, `tshark` |
| Routing and segmentation | `ip route`, `ip netns`, `veth` |
| Firewall management | `iptables` |
| Bandwidth monitoring | `iftop`, `nload` |
| DNS and IP analysis | `dig`, `getent`, `traceroute` |

Some utilities may not be installed by default. Install missing packages as needed:

```bash
sudo apt update
sudo apt install -y \
  iproute2 net-tools iputils-ping traceroute \
  nmap tcpdump wireshark tshark arp-scan \
  dnsutils iftop nload
```

**Note:** Package names and availability can vary by Kali release. Administrative privileges may be required for packet capture, interface configuration, routing, and firewall operations.

---

## Lab 1: Understanding Network Layers and TCP/IP Model

### Objective

Explore the OSI and TCP/IP networking models and understand how their layers support communication between Kali Linux and the OWASP Broken Web Applications virtual machine.

### Key Activities

#### 1. Reachability and Path Analysis

Use `ping` to test whether the target responds to ICMP Echo Requests:

```bash
ping -c 4 <OWASP_IP>
```

Use `traceroute` to investigate the path packets take toward the target:

```bash
traceroute <OWASP_IP>
```

These commands help investigate connectivity, response latency, packet loss, and possible routing issues. A failed response does not always indicate that a destination is offline because ICMP traffic may be filtered.

#### 2. Active Network Connections

Inspect network connections and socket states:

```bash
ss -ant
```

Alternatively, use:

```bash
netstat -ant
```

Look for local and remote addresses, port numbers, and TCP states such as `ESTABLISHED`, `LISTEN`, and `TIME_WAIT`.

Filter output for the target IP:

```bash
ss -ant | grep '<OWASP_IP>'
```

The results depend on whether a connection is active or has recently been closed. A short-lived connection may disappear before the command is run.

#### 3. TCP and UDP Protocol Scanning

Perform a TCP connect scan against the authorized lab target:

```bash
nmap -sT <OWASP_IP>
```

Perform a UDP scan:

```bash
sudo nmap -sU <OWASP_IP>
```

TCP and UDP provide different transport-layer services. TCP establishes connections and supports acknowledgements and retransmissions, while UDP sends datagrams without providing built-in delivery guarantees.

Nmap may report ports as `open`, `closed`, `filtered`, or `open|filtered`. These states describe what the scan could determine; they should not be interpreted as proof that every unreported service is absent.

#### 4. Link-Layer Address Resolution

Inspect the local IPv4 neighbor table:

```bash
ip neigh show
```

You can also try:

```bash
arp -a
```

On a directly connected IPv4 network, ARP helps resolve a destination IP address to a MAC address. If the target is on another subnet, Kali normally resolves the MAC address of its next-hop router rather than the remote target's MAC address.

### Expected Learning Outcomes

- Explain the seven OSI layers and four TCP/IP layers.
- Identify the role of ICMP, IP, TCP, UDP, and ARP.
- Use basic network troubleshooting commands.
- Interpret scan results without assuming that every port is accessible.

---

## Lab 2: Exploring Network Interfaces and Packet Transmission

### Objective

Inspect network interfaces, capture packets, monitor network statistics, and understand how traffic is transmitted to and from the OWASP virtual machine.

### Key Activities

#### 1. Inspect Network Interfaces

List available interfaces and their addresses:

```bash
ip -br link
ip -br addr
```

For a detailed interface view:

```bash
ip addr show
```

Legacy alternative:

```bash
ifconfig -a
```

Identify the loopback interface (`lo`) and the interface used to reach the OWASP VM. Interface names may be `eth0`, `ens33`, `enp0s3`, or another name depending on the system.

#### 2. Capture Network Packets

Capture traffic associated with the target:

```bash
sudo tcpdump -i <INTERFACE> host <OWASP_IP>
```

To capture TCP traffic:

```bash
sudo tcpdump -i <INTERFACE> tcp and host <OWASP_IP>
```

To save a packet capture for later analysis:

```bash
sudo tcpdump -i <INTERFACE> host <OWASP_IP> -w lab2-capture.pcap
```

Open the saved capture in Wireshark:

```bash
wireshark lab2-capture.pcap
```

You can also launch Wireshark and select the interface from the available capture interfaces.

Useful Wireshark display filters include:

```text
ip.addr == <OWASP_IP>
tcp.port == 80
arp
icmp
tcp.flags.syn == 1
```

Replace placeholders with actual values before using the commands or filters.

#### 3. Examine Network Statistics

Display interface statistics:

```bash
ip -s link
```

Summarize socket statistics:

```bash
ss -s
```

Legacy interface statistics:

```bash
netstat -i
```

Inspect transmitted and received packets, errors, dropped packets, and interface MTU values. Increasing error or drop counters can indicate a problem, but their meaning should be investigated in context.

#### 4. Monitor Bandwidth

Use `iftop` to view traffic by host:

```bash
sudo iftop -i <INTERFACE>
```

Alternatively, use `nload` to view interface throughput:

```bash
nload <INTERFACE>
```

Generate authorized lab traffic while monitoring the interface. Record observed throughput rather than assuming congestion from a single measurement.

#### 5. Troubleshoot an Interface

**Warning:** Disabling an interface may interrupt your SSH session or network access. Use a disposable lab VM and confirm that you have console access before proceeding.

Temporarily disable the chosen interface:

```bash
sudo ip link set <INTERFACE> down
```

Restore it:

```bash
sudo ip link set <INTERFACE> up
```

Verify the resulting state:

```bash
ip -br link
ip -br addr
ip route
```

An interface being administratively up does not guarantee connectivity. The interface may also require an IP address, a valid route, and a functioning virtual or physical network connection.

### Expected Learning Outcomes

- Identify network interfaces and their IP addresses.
- Capture and inspect packets.
- Read interface statistics and socket summaries.
- Monitor bandwidth and diagnose basic connectivity failures.

---

## Lab 3: TCP/IP Protocol Stack and Packet Inspection

### Objective

Inspect network traffic at the link, network, transport, and application layers. Investigate TCP handshakes, IP headers, ICMP messages, UDP datagrams, and operating-system fingerprinting.

### Key Activities

#### 1. Review the TCP/IP Model

The four-layer TCP/IP model can be mapped approximately to the OSI model:

| TCP/IP Layer | Corresponding OSI Layers | Examples |
|---|---|---|
| Application | Application, Presentation, Session | HTTP, DNS, SSH |
| Transport | Transport | TCP, UDP |
| Internet | Network | IPv4, IPv6, ICMP |
| Link | Data Link, Physical | Ethernet, Wi-Fi, ARP in IPv4 LAN operation |

The models provide different ways of organizing networking functions. Actual implementations may not follow a strict one-protocol-per-layer separation.

#### 2. Analyze the TCP Three-Way Handshake

Capture TCP traffic to the OWASP target:

```bash
sudo tcpdump -i <INTERFACE> -nn 'tcp and host <OWASP_IP>'
```

To save a capture:

```bash
sudo tcpdump -i <INTERFACE> -nn 'tcp and host <OWASP_IP>' \
  -w lab3-tcp-handshake.pcap
```

Generate a connection to a service that is known to be listening and authorized for testing. For example, if HTTP is available:

```bash
curl -v --connect-timeout 5 http://<OWASP_IP>/
```

The standard TCP handshake is:

1. **SYN:** The client requests a TCP connection.
2. **SYN-ACK:** The server acknowledges the request and responds with its own synchronization flag.
3. **ACK:** The client acknowledges the server's response.

Common TCP flags include:

- `SYN`: Initiates connection establishment.
- `ACK`: Acknowledges received sequence-space data or connection setup.
- `FIN`: Signals that a sender has finished sending data.
- `RST`: Resets or rejects a connection.
- `PSH`: Requests prompt delivery of received data to the application.
- `URG`: Indicates that the urgent pointer field is significant.

TCP reliability uses sequence numbers, acknowledgements, retransmissions, checksums, and flow control. Congestion control also regulates the amount of traffic sent into the network.

#### 3. Inspect IP Packet Headers

Capture IP traffic:

```bash
sudo tcpdump -i <INTERFACE> -nn 'ip and host <OWASP_IP>'
```

In Wireshark, expand the **Internet Protocol Version 4** or **Internet Protocol Version 6** section.

Important IPv4 header fields include:

- **Source IP:** Originating IPv4 address.
- **Destination IP:** Intended IPv4 address.
- **TTL (Time to Live):** Decremented by routers to prevent packets circulating indefinitely.
- **Protocol:** Identifies the encapsulated protocol, such as TCP or UDP.
- **Header checksum:** Detects errors in the IPv4 header.
- **Identification, flags, and fragment offset:** Used for IPv4 fragmentation and reassembly.
- **Total length:** Size of the IPv4 packet, including its header.

TCP sequence numbers belong to the TCP header, not the IP header.

#### 4. Analyze HTTP and SSH Traffic

For HTTP traffic on port 80:

```bash
sudo tcpdump -i <INTERFACE> -nn 'tcp port 80 and host <OWASP_IP>'
```

If an HTTP service is available:

```bash
curl -v --connect-timeout 5 http://<OWASP_IP>/
```

Plain HTTP may expose request methods, URLs, headers, and unencrypted content in a packet capture. Do not assume that every application uses port 80 or that a service is available simply because a port is conventional.

For SSH traffic on port 22:

```bash
sudo tcpdump -i <INTERFACE> -nn 'tcp port 22 and host <OWASP_IP>'
```

SSH encrypts application data after the relevant negotiation stages. A packet capture can still reveal metadata such as IP addresses, ports, packet sizes, timing, and connection setup, but encrypted payloads generally cannot be read as ordinary application text.

#### 5. Investigate ICMP and Ping

Capture ICMP packets:

```bash
sudo tcpdump -i <INTERFACE> -nn icmp
```

In Wireshark, use:

```text
icmp
```

For IPv4, ICMP Echo Request messages use Type 8 and Echo Reply messages use Type 0. ICMP also reports certain network errors, including destination-unreachable and time-exceeded conditions.

The TTL in the IPv4 header is different from the ICMP message Type and Code fields. TTL helps limit packet lifetime; a Time Exceeded response may be generated when a packet's TTL reaches zero at a router.

#### 6. Analyze UDP Traffic

Capture UDP traffic:

```bash
sudo tcpdump -i <INTERFACE> -nn 'udp and host <OWASP_IP>'
```

UDP has a compact header containing source port, destination port, length, and checksum fields. Unlike TCP, UDP does not establish a connection or provide built-in acknowledgements and retransmissions.

UDP is useful for applications such as DNS, streaming, voice and video transport, and protocols that implement their own reliability or prioritize low overhead.

No UDP traffic may appear unless an application or service generates it during the capture.

#### 7. Perform OS Detection with Nmap

Only run operating-system detection against the authorized OWASP lab target:

```bash
sudo nmap -O <OWASP_IP>
```

Nmap estimates the operating system by analyzing how the target responds to a series of network probes. Characteristics can include TCP options, initial TTL values, window sizes, IP identification behavior, and responses to different probe types.

OS detection is an estimate, not a guarantee. Firewalls, virtualization, network address translation, and shared network stacks can reduce accuracy.

#### 8. Inspect ARP Traffic

Inspect the neighbor table:

```bash
ip neigh show
```

If `arp-scan` is installed and the target is on the same Layer 2 network:

```bash
sudo arp-scan -I <INTERFACE> <OWASP_IP>
```

ARP requests ask which device owns a particular IPv4 address on a local link. The reply can provide a MAC address. ARP does not normally resolve a remote Internet host's MAC address across routers.

#### 9. Troubleshoot a TCP/IP Problem

Start with read-only diagnostics:

```bash
ip -br addr
ip route
ip neigh show
ping -c 4 <OWASP_IP>
traceroute <OWASP_IP>
ss -s
```

Check the interface state, IP address, subnet prefix, route, neighbor resolution, target service, and firewall rules. If changing configuration, document the original settings and make changes only in a controlled lab. Restore all temporary changes when finished.

### Expected Learning Outcomes

- Recognize TCP handshake and termination behavior.
- Read IP and transport-layer header fields.
- Distinguish TCP, UDP, ICMP, HTTP, SSH, and ARP traffic.
- Explain retransmissions, sequence numbers, TTL, and protocol limitations.
- Interpret Nmap OS detection as an estimate.

---

## Lab 4: Simulating Network Routing and VLAN Configuration

### Objective

Use Linux routing tools, network namespaces, virtual Ethernet pairs, and firewall rules to simulate segmented network environments and study how packets move between networks.

### Important Technical Distinction

**A Linux network namespace is not itself an IEEE 802.1Q VLAN.** A namespace provides an isolated network stack with its own interfaces, routes, and related network resources. It can be used to simulate network segmentation in a lab. Actual VLAN tagging generally uses VLAN interfaces configured with an 802.1Q VLAN ID, often on a parent interface or virtual switch.

### Key Activities

#### 1. Inspect and Add Static Routes

Inspect the routing table:

```bash
ip route show
ip route get <OWASP_IP>
```

A static route can be added using:

```bash
sudo ip route add <DESTINATION_NETWORK>/<PREFIX> \
  via <GATEWAY_IP> dev <INTERFACE>
```

For example, in a lab where `192.168.4.0/24` is reachable through gateway `192.168.1.1` on `eth0`:

```bash
sudo ip route add 192.168.4.0/24 via 192.168.1.1 dev eth0
```

This example works only if the gateway is reachable through the specified interface and the route fits the existing topology. Do not copy it blindly into a different network.

Verify the route:

```bash
ip route show
ip route get <OWASP_IP>
```

Remove a temporary route when finished:

```bash
sudo ip route del <DESTINATION_NETWORK>/<PREFIX> via <GATEWAY_IP>
```

Static routing requires correct destination prefixes, reachable next hops, and return routes. An incorrect route can disrupt connectivity.

#### 2. Create Network Namespaces and Virtual Ethernet Pairs

Create two isolated namespaces:

```bash
sudo ip netns add vlan1
sudo ip netns add vlan2
```

Create a virtual Ethernet pair:

```bash
sudo ip link add veth1 type veth peer name veth2
```

Move one endpoint into each namespace:

```bash
sudo ip link set veth1 netns vlan1
sudo ip link set veth2 netns vlan2
```

Enable loopback and the virtual interfaces:

```bash
sudo ip netns exec vlan1 ip link set lo up
sudo ip netns exec vlan2 ip link set lo up

sudo ip netns exec vlan1 ip link set veth1 up
sudo ip netns exec vlan2 ip link set veth2 up
```

Assign addresses to the two ends of the point-to-point virtual link:

```bash
sudo ip netns exec vlan1 ip addr add 192.168.10.1/24 dev veth1
sudo ip netns exec vlan2 ip addr add 192.168.10.2/24 dev veth2
```

Test connectivity between the namespaces:

```bash
sudo ip netns exec vlan1 ping -c 4 192.168.10.2
```

This demonstrates connectivity between two namespaces over a virtual Ethernet pair. It does **not** automatically connect either namespace to the OWASP VM. Communication with an external target requires a suitable topology, addressing, routes, and potentially forwarding and firewall configuration.

Inspect namespace interfaces and routes:

```bash
sudo ip netns exec vlan1 ip addr
sudo ip netns exec vlan1 ip route

sudo ip netns exec vlan2 ip addr
sudo ip netns exec vlan2 ip route
```

#### 3. Understand Subnetting and Address Assignment

A subnet prefix specifies which bits of an IP address identify the network and which bits identify a host.

For example:

- `192.168.10.1/24` belongs to network `192.168.10.0/24`.
- `192.168.10.2/24` belongs to the same network.
- A `/24` IPv4 subnet contains 256 total addresses, traditionally with 254 usable host addresses for ordinary host assignment.

For a namespace to reach a target on another subnet, it generally needs a route to that subnet or a default route via a reachable router. The router needs forwarding enabled and an appropriate return path.

#### 4. Configure IP Forwarding and Firewall Rules

**Caution:** Firewall and forwarding changes can affect host connectivity. Perform these exercises on a disposable lab VM, record existing settings, and avoid flushing firewall rules on a shared or production system.

Check current IPv4 forwarding:

```bash
sysctl net.ipv4.ip_forward
```

Temporarily enable forwarding:

```bash
sudo sysctl -w net.ipv4.ip_forward=1
```

Inspect the existing filter rules:

```bash
sudo iptables -S
sudo iptables -L -n -v
```

A forwarding rule can permit traffic between specifically configured subnets:

```bash
sudo iptables -A FORWARD \
  -s 192.168.10.0/24 -d 192.168.20.0/24 -j ACCEPT
```

This is only an example. The rule will not create the required routes, interfaces, or return path. Rule order and existing policies also affect behavior.

To block forwarding from one lab subnet to the authorized target:

```bash
sudo iptables -A FORWARD \
  -s 192.168.10.0/24 -d <OWASP_IP>/32 -j DROP
```

Replace `<OWASP_IP>` with the actual IPv4 address. A `FORWARD` rule affects routed traffic passing through the host; it does not normally block traffic originated by the host itself, which traverses the `OUTPUT` chain.

Inspect rule counters after testing:

```bash
sudo iptables -L FORWARD -n -v --line-numbers
```

Remove a specific temporary rule by its exact rule specification or line number. Line numbers may change after a deletion, so inspect the chain again before removing another rule. Restore the original forwarding setting and firewall configuration when the exercise is complete.

#### 5. Monitor Traffic with tcpdump

Capture traffic on a virtual interface:

```bash
sudo tcpdump -i <INTERFACE> -nn
```

Capture traffic involving the target:

```bash
sudo tcpdump -i <INTERFACE> -nn host <OWASP_IP>
```

When using namespaces, run a capture on the relevant virtual interface or inside the namespace. Compare packets at both ends of the virtual Ethernet pair to understand which packets arrive and which are forwarded.

### Expected Learning Outcomes

- Read and configure Linux routing tables.
- Build isolated network stacks using namespaces.
- Connect namespaces using virtual Ethernet pairs.
- Understand why namespaces and 802.1Q VLANs are related but different concepts.
- Explain the roles of IP forwarding, routing, firewall rules, and return routes.

---

## Lab 5: IP Address Analysis and Network Report

### Objective

Investigate IPv4 addressing, DNS resolution, historical address classes, subnet masks, CIDR prefixes, and usable host capacity using selected public websites and practical subnetting calculations.

### Key Activities

#### 1. Select Ten Websites

Choose ten publicly accessible domain names, ideally from different domain categories. Examples include:

- `apple.com`
- `mit.edu`
- `wikipedia.org`
- `iana.org`
- `example.com`
- Other approved domains with `.com`, `.org`, `.edu`, or country-code top-level domains.

A domain suffix does not determine the IP address class, physical location, or hosting provider of a website.

#### 2. Resolve Domain Names to IP Addresses

Use `getent`:

```bash
getent ahosts apple.com
```

Or use DNS tools:

```bash
dig apple.com A
dig apple.com AAAA
```

- An **A record** contains an IPv4 address.
- An **AAAA record** contains an IPv6 address.

You can use `ping` to test reachability and, in many cases, observe the resolved address:

```bash
ping -c 4 apple.com
```

However, `ping` is not a complete DNS inventory tool. A domain may resolve to multiple addresses, return different results over time, or use CDN and load-balancing infrastructure. ICMP may also be blocked.

Record the date and time of testing because DNS answers and routing can change.

#### 3. Record the Results

Use a table like this in your report:

| Website | Resolved IP address | Address family | Resolution command | Notes |
|---|---|---|---|---|
| `apple.com` | Record actual result | IPv4/IPv6 | `dig` or `getent` | Record observed details |
| `mit.edu` | Record actual result | IPv4/IPv6 | `dig` or `getent` | Record observed details |
| `wikipedia.org` | Record actual result | IPv4/IPv6 | `dig` or `getent` | Record observed details |

Continue until all ten domains have been documented. Do not use illustrative values as if they were collected results.

#### 4. Classify IPv4 Addresses

Traditional IPv4 classful addressing uses the first octet:

| Class | First-octet range | Default mask | Historical default prefix |
|---|---:|---|---|
| A | 1–126 for ordinary unicast classful networks | `255.0.0.0` | `/8` |
| B | 128–191 | `255.255.0.0` | `/16` |
| C | 192–223 | `255.255.255.0` | `/24` |
| D | 224–239 | Not a host unicast class | Multicast |
| E | 240–255 | Not ordinary host unicast | Reserved/experimental range |

Important qualifications:

- `127.0.0.0/8` is reserved for loopback.
- Private IPv4 ranges are `10.0.0.0/8`, `172.16.0.0/12`, and `192.168.0.0/16`.
- Special-purpose addresses include link-local `169.254.0.0/16` and the documentation ranges `192.0.2.0/24`, `198.51.100.0/24`, and `203.0.113.0/24`.
- Classful addressing is largely historical. Modern networks use CIDR prefixes such as `/20`, `/24`, and `/27`.

A public website's resolved address should not be assigned a default classful mask as though that were necessarily its actual network prefix. The actual routed prefix may differ and should be investigated separately when relevant.

#### 5. Calculate Host Capacity

For a conventional IPv4 subnet, the common usable-host formula is:

\[
\text{Usable hosts} = 2^{(32-p)} - 2
\]

where \(p\) is the CIDR prefix length. This formula generally applies to ordinary IPv4 subnets from `/0` through `/30`, with special handling required for `/31` point-to-point links and `/32` host routes.

| Prefix | Subnet mask | Total addresses | Traditional usable hosts |
|---|---|---:|---:|
| `/24` | `255.255.255.0` | 256 | 254 |
| `/25` | `255.255.255.128` | 128 | 126 |
| `/26` | `255.255.255.192` | 64 | 62 |
| `/27` | `255.255.255.224` | 32 | 30 |
| `/28` | `255.255.255.240` | 16 | 14 |
| `/29` | `255.255.255.248` | 8 | 6 |
| `/30` | `255.255.255.252` | 4 | 2 |
| `/31` | `255.255.255.254` | 2 | 2 on supported point-to-point links |
| `/32` | `255.255.255.255` | 1 | A single address/host route |

Traditional classful default capacities, when applied to ordinary networks, are:

- Class A `/8`: \(2^{24} - 2 = 16,777,214\) usable host addresses.
- Class B `/16`: \(2^{16} - 2 = 65,534\) usable host addresses.
- Class C `/24`: \(2^8 - 2 = 254\) usable host addresses.

These are historical default network sizes, not the number of devices a public website's particular IP address can support. For modern network planning, use the actual subnet prefix.

#### 6. Calculate Network Details

For a chosen IPv4 network, document:

- Network address
- Subnet mask
- Broadcast address
- First usable address
- Last usable address
- Total addresses
- Usable host capacity

Example: `192.168.10.0/26`

- Network address: `192.168.10.0`
- Subnet mask: `255.255.255.192`
- Broadcast address: `192.168.10.63`
- First usable address: `192.168.10.1`
- Last usable address: `192.168.10.62`
- Total addresses: 64
- Traditional usable hosts: 62

#### 7. Analyze the Network Path

Run traceroute against one selected website:

```bash
traceroute apple.com
```

If available, specify a TCP-based trace when ICMP or UDP probes are filtered:

```bash
sudo traceroute -T -p 443 apple.com
```

Interpret each hop as a responding router or network-layer device along the probe path, where visible. Some hops may not respond because of filtering, rate limiting, or device configuration. Asterisks do not necessarily mean the network path is broken.

#### 8. Investigate GeoIP Information

If permitted by the assignment, use a reputable IP geolocation resource to investigate selected public addresses. Record the provider and lookup date.

GeoIP results are estimates and may identify a data center, ISP registration location, or network headquarters rather than the physical location of a specific server. CDN addresses can represent distributed infrastructure. Do not infer exact server location from an IP address alone.

#### 9. Optional Subnetting and Diagnostic Exercises

For additional practice:

- Divide a sample `/24` network into `/25`, `/26`, `/27`, and `/28` subnets.
- Calculate the network and broadcast addresses for each subnet.
- Explain how to identify and resolve an IP address conflict using interface configuration, neighbor tables, and appropriate logs.
- If studying ICMP rate behavior, prefer a controlled, low-rate test against an isolated lab host. Do not perform a ping flood or denial-of-service test against public websites or systems without explicit authorization.

### Expected Learning Outcomes

- Resolve domain names and distinguish IPv4 from IPv6.
- Explain historical IPv4 address classes and modern CIDR notation.
- Calculate subnet masks, network addresses, broadcast addresses, and host capacity.
- Interpret traceroute and GeoIP results with appropriate limitations.
- Present findings in a professional network analysis report.

---

## Learning Outcomes

After completing the five laboratories, students should be able to:

1. Explain the OSI and TCP/IP models and map their functions.
2. Use Linux commands to inspect interfaces, routes, neighbors, and active connections.
3. Capture and analyze packets with `tcpdump`, `tshark`, and Wireshark.
4. Identify common TCP flags and explain TCP connection establishment and termination.
5. Compare TCP and UDP behavior and identify ICMP and ARP traffic.
6. Configure static routes and create isolated network namespaces.
7. Explain the difference between network namespaces and IEEE 802.1Q VLANs.
8. Apply controlled firewall rules and inspect their effects.
9. Calculate IPv4 subnet details and host capacities.
10. Document technical results with accurate command output and screenshots.

---

## Safety and Lab Guidelines

- Test only the OWASP VM and other systems you own or are explicitly authorized to assess.
- Keep intentionally vulnerable virtual machines on an isolated lab network whenever possible.
- Use `sudo` only when required and understand the effect of each privileged command.
- Do not scan arbitrary public systems or run denial-of-service exercises without explicit authorization.
- Before changing routes, interfaces, forwarding settings, or firewall rules, record the original configuration.
- Restore temporary settings after completing the lab.
- Redact public IPs, credentials, tokens, and unrelated sensitive information from screenshots before publishing.
- Clearly distinguish observed results from expected behavior and hypothetical examples.

## Conclusion

These laboratories connect networking theory with practical Linux tools. They demonstrate how interfaces, routes, addressing, transport protocols, packet inspection, and firewall policies work together to support communication and troubleshooting.

The final report should include actual command outputs, annotated screenshots, observations, issues encountered, troubleshooting steps, and conclusions for each exercise. Accurate evidence and clear technical explanations are essential parts of the project.
