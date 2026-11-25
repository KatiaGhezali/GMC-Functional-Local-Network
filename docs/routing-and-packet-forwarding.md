# Routing and Packet Forwarding

## 1. Overview

This section explains how devices in the company's local area network (LAN) communicate with external networks, including the internet.

The example follows Wireless Device 1 as it accesses an external website.

The network uses two VLANs:

- VLAN 10: Wired devices and network infrastructure.
- VLAN 20: Wireless clients.

The company router provides inter-VLAN routing, internet connectivity, and Network Address Translation (NAT).

## 2. Example Communication Scenario

Wireless Device 1 wants to access:

`https://www.example.com`

Its network configuration is:

| Parameter       | Value             |
| --------------- | ----------------- |
| Device          | Wireless Device 1 |
| IP address      | `192.168.10.40`   |
| Subnet mask     | `255.255.255.224` |
| Default gateway | `192.168.10.33`   |
| DNS server      | `1.1.1.1`         |
| VLAN            | 20                |

For this example, the external server is assigned the illustrative IPv4 address `203.0.113.80`.

This address belongs to a documentation-only address range and does not represent a real internet destination.

## 3. DNS Resolution

The Domain Name System (DNS) translates domain names into IP addresses.

When the user enters `www.example.com`, the laptop first checks its DNS cache.

If the address is not cached, it sends a DNS query to its configured resolver, `1.1.1.1`.

Because the DNS server is outside the laptop's local subnet, the laptop sends the DNS packet to its default gateway.

The router forwards the query toward the DNS server. The DNS resolver responds with the appropriate DNS record.

For this scenario, we assume the laptop obtains the illustrative destination address `203.0.113.80`.

DNS resolution allows the laptop to identify the server's IP address before establishing communication with it.

## 4. Default Gateway and Packet Preparation

The laptop compares the destination address with its own subnet.

Its local network is:

`192.168.10.32/27`

The external destination `203.0.113.80` is outside this subnet.

Therefore, the laptop sends the packet to its default gateway:

`192.168.10.33`

Before sending the Ethernet frame, the laptop must know the gateway's MAC address.

It uses the Address Resolution Protocol (ARP) to discover that address if it is not already in its ARP cache.

In our example, the router's LAN MAC address is:

`02:AA:BB:10:00:01`

The resulting frame and packet contain:

| Field           | Value               |
| --------------- | ------------------- |
| Source MAC      | `02:AA:BB:10:00:40` |
| Destination MAC | `02:AA:BB:10:00:01` |
| Source IP       | `192.168.10.40`     |
| Destination IP  | `203.0.113.80`      |

The destination MAC belongs to the router, not the external web server, because the router is the next hop on the local network.

## 5. Packet Transmission Through the LAN

The wireless device sends its Wi-Fi frame to the access point.

The access point bridges the wireless traffic to its Ethernet connection while preserving the client's VLAN 20 membership.

The frame travels through Switch 1 and reaches the router over the VLAN 20 trunk.

The access point and switches forward traffic within the LAN. They do not independently route the packet to the internet.

## 6. IPv4 Header and Routing

An IPv4 packet contains a header and a payload.

The IPv4 header includes several fields used for identification, delivery, and packet processing.

| Field           | Purpose                                                |
| --------------- | ------------------------------------------------------ |
| Version         | Identifies the IP version                              |
| IHL             | Specifies the IPv4 header length                       |
| Total Length    | Specifies the packet's total size                      |
| Identification  | Helps identify fragments                               |
| Flags           | Controls fragmentation behavior                        |
| Fragment Offset | Identifies a fragment's position                       |
| TTL             | Limits the number of router hops                       |
| Protocol        | Identifies the next-layer protocol, such as TCP or UDP |
| Header Checksum | Detects errors in the IPv4 header                      |
| Source IP       | Identifies the sender                                  |
| Destination IP  | Identifies the intended destination                    |

For this example, the incoming packet contains:

| Field            | Value               |
| ---------------- | ------------------- |
| Source IP        | `192.168.10.40`     |
| Destination IP   | `203.0.113.80`      |
| TTL              | `64` (illustrative) |
| Protocol         | TCP (`6`)           |
| Destination port | `443`               |

HTTPS commonly uses TCP port 443. HTTP/3 is an alternative that uses QUIC over UDP port 443; this example assumes TCP-based HTTPS.

### 6.1 Router processing

When the packet reaches the company router:

1. The router receives the frame on its VLAN 20 interface.
2. It removes the incoming Layer 2 encapsulation.
3. It examines the IPv4 header.
4. It reads the destination IP address.
5. It checks its routing table.
6. It selects the appropriate outgoing interface and next hop.
7. It decrements the IPv4 TTL.
8. It updates the IPv4 header checksum.
9. It creates a new Layer 2 frame for the outgoing link.
10. It forwards the packet.

## 7. Routing Table and Next-Hop Selection

A routing table contains information that helps a router determine where to forward packets.

A simplified routing table for the company router is:

| Destination network | Interface or next hop | Purpose                       |
| ------------------- | --------------------- | ----------------------------- |
| `192.168.10.0/27`   | VLAN 10               | Wired network                 |
| `192.168.10.32/27`  | VLAN 20               | Wireless network              |
| `0.0.0.0/0`         | ISP router            | Default route to the internet |

The first two networks are directly connected to the company router.

The third entry is the default route. It matches destinations for which no more specific route is available.

### 7.1 Longest Prefix Match

Routers generally use Longest Prefix Match to select the most specific matching route.

For example:

| Route             | Prefix |
| ----------------- | -----: |
| `0.0.0.0/0`       |      0 |
| `203.0.0.0/16`    |     16 |
| `203.0.113.0/24`  |     24 |
| `203.0.113.80/32` |     32 |

If several routes match the destination, the route with the longest matching prefix is selected.

In our scenario, no more specific route exists for the illustrative external destination. The company router therefore uses its default route and forwards the packet toward the ISP router.

The next hop is the next router that receives the packet. It is not necessarily the final destination.

## 8. Layer 2 Encapsulation and TTL

When a router forwards a packet, it removes the incoming Layer 2 frame and creates a new Layer 2 frame for the outgoing network.

For the incoming frame:

- Source MAC: the wireless device's MAC address.
- Destination MAC: the company router's LAN MAC address.

For the outgoing frame on an Ethernet WAN link:

- Source MAC: the company router's WAN MAC address.
- Destination MAC: the ISP next-hop MAC address.

The IP packet is carried inside the new frame.

Before NAT, the source and destination IP addresses normally remain unchanged during ordinary routing. However, the Layer 2 MAC addresses are replaced for the new link.

### 8.1 Time To Live

Time To Live (TTL) is an IPv4 header field that prevents packets from circulating indefinitely.

Each router that forwards the packet decreases its TTL by one.

Example:

| Stage                      | TTL |
| -------------------------- | --: |
| Laptop sends packet        |  64 |
| Company router forwards it |  63 |
| ISP router forwards it     |  62 |
| Next router forwards it    |  61 |

If TTL reaches zero, the router discards the packet and normally sends an ICMP Time Exceeded message to the sender.

TTL is effectively used as a hop limit, rather than a measurement of elapsed time.

## 9. Network Address Translation (NAT)

The wireless device uses the private IP address `192.168.10.40`.

Private IPv4 addresses are intended for internal networks and are not directly routed across the public internet.

The company router therefore uses Network Address Translation (NAT) to translate the private source address into an address usable on its public-facing connection.

For illustration, assume the company's router has the documentation-only WAN address:

`192.0.2.25`

Before source NAT:

| Field          | Value           |
| -------------- | --------------- |
| Source IP      | `192.168.10.40` |
| Destination IP | `203.0.113.80`  |

After source NAT:

| Field          | Value          |
| -------------- | -------------- |
| Source IP      | `192.0.2.25`   |
| Destination IP | `203.0.113.80` |

The external server sees the translated source address.

The address `192.0.2.25` is reserved for documentation and is not a real public internet address.

## 10. Port Address Translation (PAT)

Port Address Translation (PAT) allows multiple internal devices to share one public IPv4 address.

The router translates transport-layer source ports as well as source IP addresses.

For example, two devices may establish separate connections:

| Device            | Private address and source port |
| ----------------- | ------------------------------- |
| Wireless Device 1 | `192.168.10.40:51514`           |
| PC 1              | `192.168.10.10:51515`           |

The router can assign different translated ports:

| Device            | Translated address and port |
| ----------------- | --------------------------- |
| Wireless Device 1 | `192.0.2.25:62001`          |
| PC 1              | `192.0.2.25:62002`          |

These port assignments are illustrative.

The router maintains a NAT/PAT translation table to associate each translated connection with its original internal address and port.

This allows multiple devices to access the internet through one public IPv4 address.

NAT translates addresses, PAT also translates ports, and routing determines the forwarding path. These functions work together but are not interchangeable.

## 11. Return Traffic

When the external server sends a response, it addresses the response to the company's translated public IP address and port.

For Wireless Device 1, the illustrative response initially contains:

| Field            | Value          |
| ---------------- | -------------- |
| Source IP        | `203.0.113.80` |
| Destination IP   | `192.0.2.25`   |
| Destination port | `62001`        |

The response reaches the company router through the ISP.

The router checks its NAT/PAT translation table and finds the matching connection:

`192.0.2.25:62001` maps to `192.168.10.40:51514`.

The router translates the destination back to the laptop's private address and port.

The resulting packet contains:

| Field            | Value           |
| ---------------- | --------------- |
| Source IP        | `203.0.113.80`  |
| Destination IP   | `192.168.10.40` |
| Destination port | `51514`         |

The router identifies that `192.168.10.40` belongs to its directly connected VLAN 20 network.

It forwards the packet through VLAN 20. The access point then delivers the response to Wireless Device 1 over Wi-Fi.

The laptop receives the response, allowing the browser to display the website.

## 12. Complete Communication Summary

The outgoing communication follows this path:

1. Wireless Device 1 creates a packet for the external destination.
2. The access point bridges the wireless traffic onto the wired LAN.
3. Switch 1 forwards the traffic toward the company router.
4. The router checks its routing table and selects the ISP as the next hop.
5. The router applies NAT/PAT as configured.
6. The packet travels through the ISP and internet routers.
7. The external web server receives the request.

The response follows the reverse logical path:

1. The web server sends its response to the company's public-facing address.
2. The ISP forwards it to the company router.
3. The router checks its NAT/PAT translation table.
4. The router restores the destination to the laptop's private address.
5. The router forwards the packet through VLAN 20.
6. Switch 1 and the access point deliver it to Wireless Device 1.

## 13. Conclusion

Routing enables communication between different IP networks by using routing tables and next-hop selection.

Layer 2 frames are replaced as packets cross routed links, while IP addressing generally identifies the original source and final destination throughout the journey, subject to address translation.

TTL prevents packets from circulating indefinitely, and NAT/PAT allows devices using private IPv4 addresses to communicate with external networks through a shared public-facing address.

Together, routing, switching, DNS, and NAT/PAT provide the connectivity required for employees to access external services from the company's LAN.
