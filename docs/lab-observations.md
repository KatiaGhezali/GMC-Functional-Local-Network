# Practical Network Investigation

## 1. Objective

The purpose of this practical investigation was to observe real network communication from an Ubuntu Linux laptop. The commands demonstrate IP configuration, routing, connectivity, DNS resolution, HTTPS communication, route discovery, and neighbor resolution.

The observations complement the proposed company LAN design. They were collected on the laptop's existing Wi-Fi network, not on the hypothetical VLANs designed in Parts 1–3.

## 2. Network Interface Configuration

Command:

```bash
ip -br address
```

Observations:

- Active interface: `wlo1`
- Connection type: Wi-Fi
- IPv4 address: `192.168.100.7/24`
- Subnet mask: `255.255.255.0`
- Network address: `192.168.100.0/24`
- Wired interface `enp0s25` was inactive.

## 3. Routing Table

Command:

```bash
ip route
```

Observations:

- Default gateway: `192.168.100.1`
- Active interface: `wlo1`
- Directly connected network: `192.168.100.0/24`

Traffic destined for networks outside the local subnet is forwarded to the default gateway.

## 4. Internet Connectivity

Command:

```bash
ping -c 4 1.1.1.1
```

Results:

- Packets transmitted: 4
- Packets received: 4
- Packet loss: 0%
- Average round-trip time: 7.126 ms

This confirms successful ICMP communication with the external destination.

## 5. DNS Resolution

Command:

```bash
getent ahostsv4 example.com
```

IPv4 addresses returned:

- `172.66.147.243`
- `104.20.23.154`

This demonstrates successful hostname resolution.

## 6. HTTPS Communication

Command:

```bash
curl -I --max-time 10 https://example.com
```

Result:

```text
HTTP/2 200
```

The server successfully processed the HTTP HEAD request. This demonstrates successful application-layer communication over HTTPS.

## 7. Route Discovery

Command:

```bash
tracepath -m 12 1.1.1.1
```

Observed intermediate routers included:

- `192.168.100.1` (local gateway)
- `154.254.160.1`
- `10.104.9.1`
- `10.16.16.16`
- `172.28.16.2`
- `172.17.116.2`
- `41.110.36.3`

Some intermediate routers did not respond to probes. This can happen when routers filter or do not prioritize diagnostic messages.

The discovered path MTU was 1492 bytes.

## 8. Neighbor and MAC Address Resolution

Command:

```bash
ip neigh
```

Important observation:

- Gateway IPv4 address: `192.168.100.1`
- Gateway MAC address: `f0:25:8e:b8:29:86`
- Neighbor state: `REACHABLE`

This demonstrates the relationship between a local IP address and its corresponding link-layer MAC address.

## 9. Conclusion

The practical investigation demonstrated several fundamental networking concepts:

- IP addressing and subnet configuration
- Default gateway selection and routing
- ICMP-based connectivity testing
- DNS hostname resolution
- HTTPS communication
- Router hop discovery
- Neighbor discovery and MAC address resolution

These observations provide practical evidence of how a Linux device communicates across a local network and reaches external services.
