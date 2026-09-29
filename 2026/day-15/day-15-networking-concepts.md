# Day 15 – Networking Concepts: DNS, IP, Subnets & Ports

## Objective

Build a strong foundation in the networking concepts used daily by DevOps and Cloud engineers.

Today I focused on:

* DNS and name resolution
* IPv4 addressing
* Public vs private IP addresses
* CIDR notation and subnetting
* Common network ports
* Connecting DNS, IPs, subnets, and ports during troubleshooting

---

# 1. DNS – How Names Become IPs

## What is DNS?

DNS (Domain Name System) translates human-readable domain names into IP addresses.

When I enter `google.com` in a browser:

```text
Browser
   ↓
DNS Resolver
   ↓
DNS Server
   ↓
IP Address
   ↓
Google Server
   ↓
HTTP/HTTPS Response
```

### What happens?

1. The browser checks its local DNS cache.
2. If the IP is not cached, a DNS resolver looks up `google.com`.
3. The resolver obtains the IP address from the DNS hierarchy.
4. The browser uses that IP address to connect to the server.

---

## Common DNS Record Types

| Record | Purpose                                           |
| ------ | ------------------------------------------------- |
| A      | Maps a domain name to an IPv4 address             |
| AAAA   | Maps a domain name to an IPv6 address             |
| CNAME  | Creates an alias pointing to another domain name  |
| MX     | Specifies mail servers for a domain               |
| NS     | Specifies authoritative name servers for a domain |

---

## DNS Hands-on

### Command

```bash
dig google.com
```


### Example

```text
google.com.    300    IN    A    142.250.x.x
```

### What I Learned

The **A record** provides the IPv4 address of a domain, while the **TTL** tells DNS resolvers how long the record can be cached.

---

# 2. IP Addressing

## What is an IPv4 Address?

An IPv4 address is a 32-bit address used to identify a device or network interface.

It is written as four decimal octets:

```text
192.168.1.10
```

Each octet ranges from:

```text
0 - 255
```

Example:

```text
192 . 168 . 1 . 10
 └──── Network ────┘
             └ Host ┘
```

The exact network and host portions depend on the subnet mask or CIDR prefix.

---

## Public vs Private IP

### Private IP

Used inside private networks such as home networks, corporate networks, and AWS VPCs.

Example:

```text
192.168.1.10
```

### Public IP

An IP address that can be publicly routable on the Internet.

Example:

```text
8.8.8.8
```

---

## Private IPv4 Ranges

| Range                           | CIDR             |
| ------------------------------- | ---------------- |
| `10.0.0.0 - 10.255.255.255`     | `10.0.0.0/8`     |
| `172.16.0.0 - 172.31.255.255`   | `172.16.0.0/12`  |
| `192.168.0.0 - 192.168.255.255` | `192.168.0.0/16` |

These addresses are not directly routable on the public Internet.

---

## IP Hands-on

### Command

```bash
ip addr show
```

### Identify Your IP

```text
Private IP:
____________________________

Interface:
____________________________
```

### What I Learned

Private IP addresses are commonly used for internal communication, while public IP addresses provide Internet-reachable addressing when routing and security rules allow it.

---

# 3. CIDR & Subnetting

## What Does `/24` Mean?

Consider:

```text
192.168.1.0/24
```

The `/24` means that the first **24 bits** represent the network portion.

The remaining:

```text
32 - 24 = 8 bits
```

are available for host addressing.

The corresponding subnet mask is:

```text
255.255.255.0
```

---

## CIDR Examples

| CIDR  | Subnet Mask       | Total IPs | Usable Hosts |
| ----- | ----------------- | --------: | -----------: |
| `/24` | `255.255.255.0`   |       256 |          254 |
| `/16` | `255.255.0.0`     |    65,536 |       65,534 |
| `/28` | `255.255.255.240` |        16 |           14 |

### Formula

For a traditional IPv4 subnet:

```text
Total IPs = 2^(32 - prefix)
```

For typical subnets:

```text
Usable Hosts = Total IPs - 2
```

The two reserved addresses are normally the network address and broadcast address.

---

## Why Do We Subnet?

Subnetting divides a large network into smaller logical networks.

Benefits include:

* Better IP address management
* Network segmentation
* Smaller broadcast domains
* Improved security boundaries
* Easier routing and infrastructure organization

### Example

Instead of putting everything into:

```text
10.0.0.0/16
```

we can create separate subnets:

```text
10.0.1.0/24    → Application
10.0.2.0/24    → Database
10.0.3.0/24    → Monitoring
```

---

# 4. Ports – The Doors to Services

## What is a Port?

A port is a logical endpoint used to identify a specific service or application on a host.

An IP address identifies the **host**.

A port identifies the **service** on that host.

```text
IP Address + Port
       ↓
10.0.1.50:3306
       ↓
Database Service
```

---

## Common Ports

|  Port | Service | Protocol / Usage       |
| ----: | ------- | ---------------------- |
|    22 | SSH     | Remote server access   |
|    80 | HTTP    | Web traffic            |
|   443 | HTTPS   | Secure web traffic     |
|    53 | DNS     | Domain name resolution |
|  3306 | MySQL   | Database               |
|  6379 | Redis   | In-memory data store   |
| 27017 | MongoDB | Database               |

---

## Check Listening Ports

### Command

```bash
ss -tulpn
```

Alternative:

```bash
netstat -tulpn
```

### What I Learned

A service must be listening on the expected port before clients can connect to it.

---

# 5. Putting It Together

## Scenario 1

### Command

```bash
curl http://myapp.com:8080
```

### What networking concepts are involved?

The browser or `curl` first needs DNS to resolve `myapp.com` to an IP address. It then connects to that IP using TCP on port `8080`, and sends an HTTP request to the application.

```text
myapp.com
    ↓
DNS
    ↓
IP Address
    ↓
TCP :8080
    ↓
HTTP
    ↓
Application
```

---

# 6. Database Connectivity Troubleshooting

## Scenario

```text
Application
     ↓
10.0.1.50:3306
     ↓
MySQL Database
```

The application cannot connect to the database.

### What should I check first?

### Step 1: Test Network Reachability

```bash
ping 10.0.1.50
```

### Step 2: Test Port Connectivity

```bash
nc -zv 10.0.1.50 3306
```

### Step 3: Check Database Service

On the database server:

```bash
systemctl status mysql
```

### Step 4: Check Listening Port

```bash
ss -tulpn | grep 3306
```

### Step 5: Check Firewall / Cloud Rules

For AWS, check:

* Security Group
* Network ACL
* Route Table
* VPC configuration
* Database binding address

### Troubleshooting Flow

```text
Can I reach the server?
        ↓
Can I reach port 3306?
        ↓
Is MySQL running?
        ↓
Is MySQL listening on 3306?
        ↓
Is the firewall allowing traffic?
        ↓
Are AWS Security Groups/NACLs allowing traffic?
        ↓
Is the application using the correct credentials/configuration?
```

---

# 7. Quick Revision

## DNS

```text
Domain Name → DNS → IP Address
```

## IP

```text
IP Address → Identifies a network interface/host
```

## CIDR

```text
192.168.1.0/24
        ↓
24 Network Bits
8 Host Bits
```

## Port

```text
IP + Port → Specific Service
```

Example:

```text
10.0.1.50:3306
       ↓
    MySQL
```

---

# 8. Useful Commands

```bash
# DNS
dig google.com

# Network interfaces
ip addr show

# Check connectivity
ping google.com

# Check listening ports
ss -tulpn

# Test a specific port
nc -zv 10.0.1.50 3306

# HTTP request
curl -I https://google.com

# Check service
systemctl status mysql
```

---

# 9. What I Learned

### 1. DNS

DNS translates human-readable domain names into IP addresses, allowing applications to locate services without requiring users to remember IP addresses.

### 2. IP & Subnetting

IP addresses identify network interfaces, while CIDR and subnetting divide networks into manageable and isolated sections.

### 3. Ports

Ports identify specific services running on a host. Understanding IP + port combinations is essential when troubleshooting application connectivity.

---

# 10. Key Takeaways

* DNS resolves domain names to IP addresses.
* IPv4 addresses contain 32 bits.
* Private IPs are used inside internal networks.
* CIDR defines the network prefix and available host space.
* Subnetting helps with organization, routing, and network segmentation.
* Ports identify services running on a host.
* `3306` is commonly used by MySQL.
* `6379` is commonly used by Redis.
* `27017` is commonly used by MongoDB.
* Connectivity troubleshooting should move from network reachability to port availability, service status, and security controls.

---

