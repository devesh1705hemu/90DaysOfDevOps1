# Day 14 – Networking Fundamentals & Hands-on Checks

## Objective

Build a practical understanding of core networking concepts and use essential Linux networking commands for troubleshooting.

Today I focused on:

* Understanding the **OSI and TCP/IP models**
* Identifying where **IP, TCP/UDP, DNS, HTTP/HTTPS** fit
* Checking network connectivity and routes
* Inspecting listening ports and active connections
* Testing DNS and HTTP connectivity
* Performing a basic port probe

---

# 1. Quick Networking Concepts

## OSI Model vs TCP/IP Model

### OSI Model

| Layer | Name         | Example              |
| ----- | ------------ | -------------------- |
| L7    | Application  | HTTP, HTTPS, DNS     |
| L6    | Presentation | Encryption, encoding |
| L5    | Session      | Session management   |
| L4    | Transport    | TCP, UDP             |
| L3    | Network      | IP, ICMP             |
| L2    | Data Link    | Ethernet, MAC        |
| L1    | Physical     | Cables, signals      |

### TCP/IP Model

| Layer       | Name                        | Examples              |
| ----------- | --------------------------- | --------------------- |
| Application | Application protocols       | HTTP, HTTPS, DNS, SSH |
| Transport   | End-to-end communication    | TCP, UDP              |
| Internet    | Addressing and routing      | IP, ICMP              |
| Link        | Local network communication | Ethernet, ARP         |

### Simple Mapping

```text
OSI                    TCP/IP

L7 Application    ─┐
L6 Presentation    │
L5 Session         ├── Application
                  │
L4 Transport      ─── Transport
                  
L3 Network        ─── Internet

L2 Data Link      ─┐
L1 Physical       ├── Link
```

---

# 2. Where Common Protocols Fit

| Protocol / Technology | Layer              | Purpose                              |
| --------------------- | ------------------ | ------------------------------------ |
| IP                    | Network / Internet | Addressing and routing               |
| TCP                   | Transport          | Reliable communication               |
| UDP                   | Transport          | Fast connectionless communication    |
| DNS                   | Application        | Domain name resolution               |
| HTTP                  | Application        | Web communication                    |
| HTTPS                 | Application        | Secure web communication             |
| ICMP                  | Network / Internet | Connectivity and diagnostic messages |
| SSH                   | Application        | Secure remote access                 |

### Real Example

```text
curl https://example.com
        ↓
HTTPS
        ↓
TCP
        ↓
IP
        ↓
Network Interface
```

A simple `curl` request involves multiple networking layers working together.

---

# 3. Hands-on Networking Checks

## 3.1 Check Local IP Address

### Command

```bash
hostname -I
```

Alternative:

```bash
ip addr show
```

### Observation

```text
My local IP address:
____________________________
```

### What I Learned

The IP address identifies my machine on the network and allows other devices to communicate with it.

---

# 3.2 Test Network Reachability

### Command

```bash
ping google.com
```

### Observation

```text
Packets transmitted:
____________________________

Packets received:
____________________________

Packet loss:
____________________________

Average latency:
____________________________
```

### What I Learned

`ping` uses ICMP to check whether a destination is reachable and provides basic latency and packet-loss information.

---

# 3.3 Trace the Network Path

### Command

```bash
traceroute google.com
```

If `traceroute` is unavailable:

```bash
tracepath google.com
```

### Observation

```text
Number of hops:
____________________________

Any timeout:
____________________________

Longest/interesting hop:
____________________________
```

### What I Learned

`traceroute` helps identify the path packets take between my machine and the destination.

A timeout at one hop does not automatically mean the final destination is unreachable because some routers intentionally do not respond to traceroute probes.

---

# 3.4 Check Listening Ports

### Command

```bash
ss -tulpn
```

Alternative:

```bash
netstat -tulpn
```

### Example

```text
tcp   LISTEN   0   128   0.0.0.0:22
```

### Observation

```text
Listening service:
____________________________

Port:
____________________________

Protocol:
____________________________
```

### What I Learned

`ss` helps identify services that are listening for incoming network connections.

---

# 3.5 DNS Name Resolution

### Command

```bash
dig google.com
```

Alternative:

```bash
nslookup google.com
```

### Observation

```text
Domain:
____________________________

Resolved IP:
____________________________
```

### What I Learned

DNS converts human-readable domain names into IP addresses that computers can use for network communication.

---

# 3.6 HTTP/HTTPS Check

### Command

```bash
curl -I https://example.com
```

### Observation

```text
HTTP Status Code:
____________________________

Server:
____________________________

Response:
____________________________
```

Example:

```text
HTTP/2 200
```

### What I Learned

`curl -I` retrieves HTTP response headers without downloading the complete webpage.

A `200` status generally indicates a successful request.

---

# 3.7 Active Connections Snapshot

### Command

```bash
netstat -an | head
```

Alternative:

```bash
ss -ant
```

### Observation

```text
ESTABLISHED connections:
____________________________

LISTEN connections:
____________________________
```

### What I Learned

Network connection states help troubleshoot whether applications are actively communicating or waiting for connections.

---

# 4. Mini Network Check

## Target

For today's checks, I used:

```text
Target: google.com
```

### Commands

```bash
ping google.com
```

```bash
traceroute google.com
```

```bash
dig google.com
```

```bash
curl -I https://google.com
```

### Summary

| Check      | Result | Observation                   |
| ---------- | ------ | ----------------------------- |
| IP Address | ⬜      | Local IP identified           |
| Ping       | ⬜      | Latency / packet loss checked |
| Traceroute | ⬜      | Network path checked          |
| DNS        | ⬜      | Domain resolved               |
| HTTP       | ⬜      | HTTP response checked         |

---

# 5. Mini Task: Port Probe & Interpret

## Step 1: Identify a Listening Port

Run:

```bash
ss -tulpn
```

Example:

```text
tcp LISTEN 0 128 0.0.0.0:22
```

Here, port `22` is commonly used by SSH.

Record your port:

```text
Service:
____________________________

Port:
____________________________
```

---

## Step 2: Test the Port

Use:

```bash
nc -zv localhost <port>
```

Example:

```bash
nc -zv localhost 22
```

For a web service:

```bash
curl -I http://localhost:<port>
```

### Result

```text
Port reachable: YES / NO

Output:
____________________________
```

---

## Step 3: Interpret the Result

### If the port is reachable

```text
The port is open and the service is accepting connections.
```

### If the port is not reachable

Check:

```bash
systemctl status <service>
```

Then check:

```bash
ss -tulpn
```

And investigate firewall rules:

```bash
sudo ufw status
```

For AWS environments, also verify:

* Security Group rules
* Network ACLs
* Instance firewall
* Service binding address
* Application status

---

# 6. Troubleshooting Flow

When a service is not reachable, follow this order:

```text
Application Running?
        ↓
Service Listening?
        ↓
Correct Port?
        ↓
Local Connectivity?
        ↓
Firewall Rules?
        ↓
Security Group / NACL?
        ↓
Network Route?
        ↓
DNS Resolution?
        ↓
Remote Connectivity?
```

### Useful Commands

```bash
systemctl status <service>
```

```bash
ss -tulpn
```

```bash
ping <target>
```

```bash
traceroute <target>
```

```bash
dig <domain>
```

```bash
curl -I <url>
```

```bash
nc -zv <host> <port>
```

---

# 7. Key Takeaways

* IP handles addressing and routing.
* TCP provides reliable, connection-oriented communication.
* UDP is connectionless and generally has lower protocol overhead.
* DNS translates domain names into IP addresses.
* HTTP/HTTPS is used for web communication.
* `ping` checks basic reachability and latency.
* `traceroute` helps identify the network path.
* `ss` helps identify listening ports and active connections.
* `dig` helps troubleshoot DNS.
* `curl` is useful for testing HTTP/HTTPS services.
* Port connectivity does not always mean the application itself is healthy.
* In AWS, networking problems can involve Security Groups, NACLs, routes, DNS, and VPC configuration.

---

# 8. Troubleshooting Cheat Sheet

| Problem              | First Command                |
| -------------------- | ---------------------------- |
| Find local IP        | `hostname -I`                |
| Check connectivity   | `ping <host>`                |
| Check network path   | `traceroute <host>`          |
| Find listening ports | `ss -tulpn`                  |
| Check DNS            | `dig <domain>`               |
| Check HTTP           | `curl -I <url>`              |
| Test a port          | `nc -zv <host> <port>`       |
| Check service        | `systemctl status <service>` |
| Check firewall       | `sudo ufw status`            |

---



### Commands I should remember

```bash
hostname -I
ping
traceroute
ss -tulpn
dig
curl -I
nc -zv
systemctl status
```

---


