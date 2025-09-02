# DNS Lab in Cisco Packet Tracer

## Objective
Build and configure a simple DNS lab in Cisco Packet Tracer with:
- Two LANs separated by a router
- A DNS server in LAN20 that resolves names in `lab.local`
- A Web server in LAN10 that clients can access using DNS
- PCs in LAN10 receiving IPs via DHCP from the router

---

## Topology & Devices
- **Router**: 2901 (or similar)
- **Switches**: 2× 2960
- **Servers**: 1× DNS, 1× Web
- **PCs**: at least 1 client

### Cabling
- Router G0/0 → Switch0 (LAN10)
- Router G0/1 → Switch1 (LAN20)
- Web Server → Switch0 (LAN10)
- PC(s) → Switch0 (LAN10)
- DNS Server → Switch1 (LAN20)

---

## IP Addressing

| Device / Interface | IP | Subnet Mask | Gateway | Notes |
|--------------------|------------|-------------|------------|----------------|
| R1 G0/0            | 192.168.10.1 | 255.255.255.0 | — | LAN10 GW |
| R1 G0/1            | 192.168.20.1 | 255.255.255.0 | — | LAN20 GW |
| DNS Server         | 192.168.20.2 | 255.255.255.0 | 192.168.20.1 | Runs DNS |
| Web Server         | 192.168.10.100 | 255.255.255.0 | 192.168.10.1 | Runs HTTP |
| PCs (LAN10)        | DHCP | — | — | DNS = 192.168.20.2 |

---

## Router Configuration

```
enable
configure terminal
hostname R1
no ip domain-lookup

interface g0/0
 ip address 192.168.10.1 255.255.255.0
 no shutdown

interface g0/1
 ip address 192.168.20.1 255.255.255.0
 no shutdown

ip dhcp excluded-address 192.168.10.1 192.168.10.20
ip dhcp pool LAN10
 network 192.168.10.0 255.255.255.0
 default-router 192.168.10.1
 dns-server 192.168.20.2
 domain-name lab.local

end
write memory
```

---

## DNS Server Configuration (192.168.20.2)
- **IP Settings**:
  - IP: 192.168.20.2
  - Subnet: 255.255.255.0
  - Gateway: 192.168.20.1

- **Services → DNS**: Turn ON and add records:
  - `www.lab.local` → 192.168.10.100
  - `files.lab.local` → 192.168.20.2
  - (Optional) `app.lab.local` → CNAME → `www.lab.local`

---

## Web Server Configuration (192.168.10.100)
- **IP Settings**:
  - IP: 192.168.10.100
  - Subnet: 255.255.255.0
  - Gateway: 192.168.10.1
  - DNS: 192.168.20.2

- **Services → HTTP**: Turn ON

---

## PC Client Configuration (LAN10)
- Set to **DHCP**
- Should receive:
  - IP: from 192.168.10.21–254
  - Gateway: 192.168.10.1
  - DNS: 192.168.20.2

---

## Testing

1. From PC → Command Prompt:
   ```
   ping 192.168.10.1
   ping 192.168.20.2
   ping 192.168.10.100
   nslookup www.lab.local
   nslookup files.lab.local
   ```

2. Open PC Web Browser:
   - Visit `http://www.lab.local`

---

## Notes
- Make sure router interfaces are `no shutdown`
- Verify DHCP pool excludes router & static IPs
- DNS service must be **ON**
- Default gateways set correctly on all devices
