# IP Addressing and Subnetting

## 1. Addressing Strategy

The company network uses the private IPv4 address range `192.168.10.0/24`.

To improve organization and security, the network is divided into two logical networks using VLANs and separate IP subnets:

- VLAN 10: Wired network.
- VLAN 20: Wireless network.

The router supports inter-VLAN routing and acts as the default gateway for both subnets.

## 2. Subnet Calculation

The original network is `192.168.10.0/24`.

It is divided into two `/27` subnets.

A `/27` subnet provides:

- 32 total IP addresses.
- 30 usable host addresses.
- Subnet mask: `255.255.255.224`.

The block size is calculated as:

`256 - 224 = 32`

### Subnet 1 — Wired Network

- Network address: `192.168.10.0`
- Subnet mask: `255.255.255.224`
- First usable address: `192.168.10.1`
- Last usable address: `192.168.10.30`
- Broadcast address: `192.168.10.31`
- VLAN ID: 10

### Subnet 2 — Wireless Network

- Network address: `192.168.10.32`
- Subnet mask: `255.255.255.224`
- First usable address: `192.168.10.33`
- Last usable address: `192.168.10.62`
- Broadcast address: `192.168.10.63`
- VLAN ID: 20

## 3. IP Address Allocation

### VLAN 10 — Wired Network

| Device                  | IP address    | Assignment       |
| ----------------------- | ------------- | ---------------- |
| Router gateway          | 192.168.10.1  | Static           |
| Switch 1                | 192.168.10.2  | Static           |
| Switch 2                | 192.168.10.3  | Static           |
| Access point management | 192.168.10.4  | Static           |
| Printer                 | 192.168.10.5  | Static           |
| PC 1                    | 192.168.10.10 | DHCP reservation |
| PC 2                    | 192.168.10.11 | DHCP reservation |
| PC 3                    | 192.168.10.12 | DHCP reservation |
| PC 4                    | 192.168.10.13 | DHCP reservation |

### VLAN 20 — Wireless Network

| Device            | IP address    | Assignment |
| ----------------- | ------------- | ---------- |
| Router gateway    | 192.168.10.33 | Static     |
| Wireless device 1 | 192.168.10.40 | DHCP       |
| Wireless device 2 | 192.168.10.41 | DHCP       |

## 4. DHCP Configuration

Dynamic Host Configuration Protocol (DHCP) automatically assigns IP configuration to network clients.

The router provides separate DHCP scopes for VLAN 10 and VLAN 20.

DHCP follows four steps:

1. Discover: The client searches for a DHCP server.
2. Offer: The server offers an available IP address.
3. Request: The client requests the offered address.
4. Acknowledgment: The server confirms the assignment.

The DHCP configuration provides the IP address, subnet mask, default gateway, DNS server and lease duration.

## 5. Default Gateway

The default gateway allows devices to communicate with destinations outside their local subnet.

- VLAN 10 uses `192.168.10.1`.
- VLAN 20 uses `192.168.10.33`.

Devices communicating within the same subnet can exchange traffic directly through the switches. Communication between different subnets requires routing through the router.

## 6. Design Benefits

Separating wired and wireless devices provides:

- Better network organization.
- Improved security through access-control rules.
- Easier management of IP address allocation.
- More flexibility for future network expansion.
- Separate DHCP scopes for each network.
