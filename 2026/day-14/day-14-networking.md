# Day 14 - Linux Networking Basics

# Quick Concepts

## OSI Model vs TCP/IP Model
TCP: A transport layer protocol that provides reliable, connection-oriented communication between devices.

OSI Model: A seven-layer networking model that describes how data moves across a network, from physical transmission to application-level communication.

### OSI Model (7 Layers)

| Layer | Name         | Example Protocols            |
| ----- | ------------ | ---------------------------- |
| L7    | Application  | HTTP, HTTPS, DNS, FTP        |
| L6    | Presentation | SSL/TLS, JPEG, PNG           |
| L5    | Session      | NetBIOS, RPC                 |
| L4    | Transport    | TCP, UDP                     |
| L3    | Network      | IP, ICMP                     |
| L2    | Data Link    | Ethernet, Wi-Fi              |
| L1    | Physical     | Cables, Fiber, Radio Signals |

### TCP/IP Model (4 Layers)

| Layer       | Example Protocols     |
| ----------- | --------------------- |
| Application | HTTP, HTTPS, DNS, SSH |
| Transport   | TCP, UDP              |
| Internet    | IP, ICMP              |
| Link        | Ethernet, Wi-Fi       |

---

## Protocol Placement in the Stack

| Protocol     | Layer                    |
| ------------ | ------------------------ |
| HTTP / HTTPS | Application Layer        |
| DNS          | Application Layer        |
| TCP          | Transport Layer          |
| UDP          | Transport Layer          |
| IP           | Internet / Network Layer |
| Ethernet     | Link / Data Link Layer   |

---

## OSI vs TCP/IP Mapping

```text
+-------------------+      +------------------+
| OSI Model         |      | TCP/IP Model     |
+-------------------+      +------------------+
| L7 Application    | ---> |                  |
| L6 Presentation   |      |  Application     |
| L5 Session        |      |                  |
+-------------------+      +------------------+
| L4 Transport      | ---> |  Transport       |
+-------------------+      +------------------+
| L3 Network        | ---> |  Internet        |
+-------------------+      +------------------+
| L2 Data Link      | ---> |                  |
| L1 Physical       |      |  Link            |
+-------------------+      +------------------+
```

---

## Real-World Example

### Command

```bash
curl https://example.com
```

### How Data Travels

```text
Application Layer : HTTP/HTTPS Request
         │
         ▼
Transport Layer   : TCP (Port 443)
         │
         ▼
Internet Layer    : IP Packet
         │
         ▼
Link Layer        : Ethernet/Wi-Fi Frame
         │
         ▼
Physical Medium   : Cable / Fiber / Wireless
```

### Summary

* HTTP/HTTPS works at the Application Layer.
* TCP provides reliable communication at the Transport Layer.
* IP handles addressing and routing at the Internet Layer.
* Ethernet or Wi-Fi delivers frames across the local network.
* Every network request passes through these layers before reaching its destination.


## Objective

Perform basic network troubleshooting and diagnostics using Linux networking tools.

---

## 1. Identity Check

### Command

```bash
hostname -I
```

### Output

```bash
172.31.35.110 172.17.0.1
```

### Observation

The server's primary private IP address is **172.31.35.110**, assigned within the AWS VPC. The **172.17.0.1** address belongs to the Docker bridge network.

---

## 2. Reachability Test

### Command

```bash
ping trainwithshubham.com
```

### Output (Sample)

```bash
64 bytes from 3.33.251.168: icmp_seq=1 ttl=244 time=0.836 ms
64 bytes from 3.33.251.168: icmp_seq=2 ttl=244 time=0.847 ms
```

### Observation

The target host responded successfully to ICMP requests. Average latency remained below **1 ms**, indicating excellent network connectivity with no noticeable packet loss.

---

## 3. Path Discovery

### Command

```bash
tracepath google.com
```

### Output

```bash
1?: [LOCALHOST] pmtu 9001
1: ip-172-31-32-1.eu-west-1.compute.internal
2: 99.82.10.96
3: 99.82.10.97
4: no reply
5: no reply
6: no reply
```

### Observation

Traffic passed through AWS internal routers before entering the public internet. Some intermediate hops did not respond, which is common because many routers block ICMP responses for security reasons.

---

## 4. Open Ports and Services

### Command

```bash
ss -tulpn
```

### Example Output

```bash
tcp LISTEN 0 128 0.0.0.0:22
```

### Observation

SSH is actively listening on **port 22**, allowing secure remote administration of the EC2 instance.

---

## 5. DNS Resolution

### Command

```bash
dig google.com
```

### Output

```bash
74.125.193.138
74.125.193.113
74.125.193.139
74.125.193.100
74.125.193.101
74.125.193.102
```

### Observation

The DNS server successfully resolved **google.com** into multiple IPv4 addresses, demonstrating proper DNS functionality and Google's load-balanced infrastructure.

---

## 6. HTTP Connectivity Check

### Command

```bash
curl -I https://trainwithshubham.playground.utho.com/
```

### Output

```bash
HTTP/2 200
server: nginx/1.25.5
```

### Observation

The web server returned an **HTTP 200 OK** response, confirming that the website is reachable and serving content correctly.

---

## 7. Connections Snapshot

### Command

```bash
netstat -an | head
```

### Observation

The output displays active network connections and listening sockets. ESTABLISHED connections represent active communication sessions, while LISTEN entries indicate services waiting for incoming connections.

---

# Key Commands Learned

| Command       | Purpose                                 |
| ------------- | --------------------------------------- |
| `hostname -I` | Display system IP addresses             |
| `ping`        | Test network connectivity and latency   |
| `tracepath`   | Trace the network path to a destination |
| `ss -tulpn`   | View listening ports and services       |
| `dig`         | Query DNS records                       |
| `nslookup`    | Resolve domain names                    |
| `curl -I`     | Check HTTP response headers             |
| `netstat -an` | View active network connections         |

---

# Key Learnings

* Identified network interfaces and IP addresses.
* Verified connectivity using ICMP requests.
* Traced packet routes across network hops.
* Examined listening ports and active services.
* Performed DNS lookups and name resolution.
* Tested web server accessibility using HTTP headers.
* Analyzed active network connections and socket states.

---------
## Mini Task: Port Probe & Interpret

### Identify a Listening Port

Using the `ss -tulpn` command, I found a service listening on port **36843** on localhost.

### Command

```bash
nc -zv localhost 36843
````

### Output

```bash
Connection to localhost (127.0.0.1) 36843 port [tcp/*] succeeded!
```

### Observation

The service running on **port 36843** is reachable from the local machine. The successful connection confirms that the application is actively listening and accepting TCP connections.

### Troubleshooting Note

If the connection had failed, the next steps would be:

1. Check whether the service is running:

   ```bash
   systemctl status <service-name>
   ```

2. Verify that the port is listening:

   ```bash
   ss -tulpn
   ```

3. Check firewall rules:

   ```bash
   sudo ufw status
   ```

4. Review service logs for errors:

   ```bash
   journalctl -xe
   ```


### One-Line Answer (for submission)

> Port **36843** was reachable via `nc -zv localhost 36843`, confirming that the service is actively listening and accepting connections. If it were unreachable, I would check the service status, listening ports, and firewall configuration.

