# Part 1 — MAC Addressing and Frame Handling

## 1. Introduction

The Data Link layer (Layer 2) of the OSI model is responsible for transferring frames between devices on the same local network.

It uses MAC addresses to identify network interfaces and Ethernet or Wi-Fi framing to transmit data.

## 2. MAC Addresses

A Media Access Control (MAC) address is a 48-bit identifier associated with a network interface.

It is normally represented using six hexadecimal octets, for example:

`02:AA:BB:CC:DD:01`

For this network design, locally administered unicast MAC addresses are used as illustrative assignments. They do not represent factory-assigned hardware addresses.

### MAC Address Allocation

| Device            | Interface         | MAC address       |
| ----------------- | ----------------- | ----------------- |
| Router            | LAN               | 02:AA:BB:10:00:01 |
| Router            | WAN               | 02:AA:BB:10:00:02 |
| Switch 1          | Management/base   | 02:AA:BB:10:00:03 |
| Switch 2          | Management/base   | 02:AA:BB:10:00:04 |
| PC 1              | Ethernet          | 02:AA:BB:10:00:10 |
| PC 2              | Ethernet          | 02:AA:BB:10:00:11 |
| PC 3              | Ethernet          | 02:AA:BB:10:00:12 |
| PC 4              | Ethernet          | 02:AA:BB:10:00:13 |
| Printer           | Ethernet          | 02:AA:BB:10:00:20 |
| Access point      | Ethernet          | 02:AA:BB:10:00:30 |
| Access point      | Wi-Fi radio/BSSID | 02:AA:BB:10:00:31 |
| Wireless device 1 | Wi-Fi             | 02:AA:BB:10:00:40 |
| Wireless device 2 | Wi-Fi             | 02:AA:BB:10:00:41 |

The router's VLAN 10 and VLAN 20 logical interfaces share the underlying LAN interface MAC address in this router-on-a-stick design. Switch management/base addresses identify the switches; individual switching ports do not require separate MAC addresses for ordinary frame forwarding.

## 3. MAC Address vs IP Address

MAC addresses operate at Layer 2, while IP addresses operate at Layer 3.

A MAC address identifies a network interface for local frame delivery. An IP address provides logical addressing that allows packets to travel between networks.

In the example:

- PC 1 MAC: `02:AA:BB:10:00:10`
- PC 1 IP: `192.168.10.10`
- Printer MAC: `02:AA:BB:10:00:20`
- Printer IP: `192.168.10.5`

Both addressing systems are used when PC 1 communicates with the printer.

## 4. Address Resolution Protocol (ARP)

ARP resolves an IPv4 address into a MAC address on a local network.

When PC 1 wants to communicate with the printer, it first checks its ARP cache.

If no valid mapping exists, PC 1 broadcasts an ARP request asking which device owns `192.168.10.5`.

The printer responds with its MAC address. PC 1 stores the mapping in its ARP cache and can then construct an Ethernet frame addressed to the printer.

An ARP request is broadcast, while the reply is normally unicast.

If the destination is outside the local subnet, the sender resolves the MAC address of its default gateway instead.

## 5. Ethernet II Frame Structure

An Ethernet II frame contains the following fields:

| Field                 |          Size | Purpose                              |
| --------------------- | ------------: | ------------------------------------ |
| Preamble              |       7 bytes | Synchronizes the receiver            |
| Start Frame Delimiter |        1 byte | Indicates the beginning of the frame |
| Destination MAC       |       6 bytes | Identifies the receiving interface   |
| Source MAC            |       6 bytes | Identifies the sending interface     |
| EtherType             |       2 bytes | Identifies the encapsulated protocol |
| Payload               | 46–1500 bytes | Contains the encapsulated data       |
| Frame Check Sequence  |       4 bytes | Detects transmission errors          |

The minimum standard Ethernet frame size is 64 bytes, and the maximum untagged Ethernet frame size is 1518 bytes, excluding the preamble and Start Frame Delimiter.

An IEEE 802.1Q VLAN tag adds four bytes.

Common EtherType values include:

- `0x0800`: IPv4
- `0x0806`: ARP
- `0x86DD`: IPv6

The FCS uses a cyclic redundancy check (CRC) to detect corruption. It does not correct errors.

## 6. PC-to-Printer Communication

PC 1 sends a print job to the printer.

The devices have the following configuration:

- PC 1 IP: `192.168.10.10`
- PC 1 MAC: `02:AA:BB:10:00:10`
- Printer IP: `192.168.10.5`
- Printer MAC: `02:AA:BB:10:00:20`

Both devices belong to VLAN 10 and subnet `192.168.10.0/27`.

The communication follows these steps:

1. PC 1 checks whether the printer is in its local subnet.
2. PC 1 resolves the printer's MAC address using its ARP cache or an ARP request.
3. PC 1 encapsulates the print-job data in an IP packet.
4. PC 1 places the IP packet inside an Ethernet frame.
5. The frame contains PC 1's source MAC and the printer's destination MAC.
6. Switches forward the frame according to their MAC address tables.
7. The printer receives the frame, checks its destination MAC and FCS, and processes the encapsulated data.

The switches forward the frame without changing its source and destination MAC addresses along this same-VLAN path.

## 7. Collision Handling in Ethernet

Traditional shared half-duplex Ethernet uses CSMA/CD (Carrier Sense Multiple Access with Collision Detection).

The process is:

1. A device listens to the medium.
2. If the medium is idle, it begins transmitting.
3. If two devices transmit simultaneously, a collision occurs.
4. The devices detect the collision and stop transmitting.
5. They wait for random backoff intervals.
6. They attempt transmission again.

Modern switched Ethernet normally operates in full-duplex mode. Each device has a dedicated switch link and can transmit and receive simultaneously. CSMA/CD is therefore not used during normal full-duplex operation.

## 8. Collision Handling in Wi-Fi

Wi-Fi uses CSMA/CA (Carrier Sense Multiple Access with Collision Avoidance).

Wireless devices share a radio channel, so they attempt to avoid simultaneous transmissions.

The process is:

1. The device listens to the wireless channel.
2. If the channel is busy, it waits.
3. If idle, it observes the required interframe interval.
4. It selects a random backoff period.
5. It transmits when its backoff counter reaches zero.
6. The receiver sends an acknowledgment when the frame is received successfully.

If an expected acknowledgment is not received, the sender may retransmit.

Optional RTS (Request to Send) and CTS (Clear to Send) control frames can reduce collisions in situations such as the hidden-node problem.

## 9. Ethernet and Wi-Fi Comparison

| Feature            | Traditional Ethernet  | Modern switched Ethernet                  | Wi-Fi                                     |
| ------------------ | --------------------- | ----------------------------------------- | ----------------------------------------- |
| Access mechanism   | CSMA/CD               | Dedicated full-duplex links               | CSMA/CA                                   |
| Collision handling | Detection and backoff | No normal collisions on full-duplex links | Avoidance and retransmission              |
| Medium             | Shared wired medium   | Dedicated switch links                    | Shared radio channel                      |
| MAC acknowledgment | Not normally used     | Not normally used                         | Used for applicable unicast transmissions |

## 10. Conclusion

MAC addressing allows devices to identify local network interfaces, while ARP resolves IPv4 addresses into MAC addresses.

Ethernet frames carry source and destination MAC addresses and encapsulated network-layer packets.

Switches use MAC address learning to forward frames efficiently.

Traditional shared Ethernet uses CSMA/CD, whereas modern switched full-duplex Ethernet avoids the shared-medium collision problem. Wi-Fi uses CSMA/CA to reduce the likelihood of wireless collisions.
