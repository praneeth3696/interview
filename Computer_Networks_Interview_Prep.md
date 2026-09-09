# Computer Networks — Interview Preparation

## 1. What is a Computer Network?
A collection of interconnected devices (computers, servers, routers) that can communicate and share resources (data, files, printers) using communication protocols over wired or wireless media.

## 2. OSI Model (7 Layers)
A conceptual framework standardizing network communication functions into 7 layers:
1. **Physical Layer**: transmission of raw bits over a physical medium (cables, signals).
2. **Data Link Layer**: node-to-node data transfer, framing, error detection/correction, MAC addressing (switches operate here).
3. **Network Layer**: logical addressing (IP) and routing of packets between networks (routers operate here).
4. **Transport Layer**: end-to-end communication, segmentation, flow control, error control (TCP/UDP).
5. **Session Layer**: establishes, manages, and terminates sessions between applications.
6. **Presentation Layer**: data translation, encryption/decryption, compression (ensures data is in a usable format).
7. **Application Layer**: interface for end-user applications (HTTP, FTP, SMTP, DNS).

## 3. TCP/IP Model (4 Layers)
A more practical, widely implemented model: **Network Access (Link)**, **Internet**, **Transport**, **Application** — maps roughly to OSI but combines some layers.

## 4. OSI vs TCP/IP
OSI is a theoretical, 7-layer reference model used for understanding/teaching; TCP/IP is a practical, 4-layer model that's actually implemented in real-world networking (the Internet).

## 5. TCP vs UDP
| TCP | UDP |
|---|---|
| Connection-oriented (3-way handshake) | Connectionless |
| Reliable (acknowledgments, retransmission) | Unreliable (best-effort) |
| Ordered delivery | No guaranteed order |
| Flow control & congestion control | No flow control |
| Slower due to overhead | Faster, lower overhead |
| Used for: web (HTTP), email, file transfer | Used for: video streaming, DNS, VoIP, gaming |

## 6. Three-Way Handshake (TCP Connection Establishment)
1. **SYN**: client sends a synchronize request with an initial sequence number.
2. **SYN-ACK**: server acknowledges and sends its own sequence number.
3. **ACK**: client acknowledges, and the connection is established.
Connection termination typically uses a four-way handshake (FIN, ACK, FIN, ACK).

## 7. IP Addressing
A unique logical address assigned to each device on a network.
- **IPv4**: 32-bit address (e.g., 192.168.1.1), ~4.3 billion addresses, written in dotted decimal.
- **IPv6**: 128-bit address, designed to solve IPv4 exhaustion, written in hexadecimal (e.g., 2001:db8::1).

## 8. Classes of IP Addresses (IPv4)
Class A (0-127, large networks), Class B (128-191, medium networks), Class C (192-223, small networks), Class D (224-239, multicast), Class E (240-255, experimental).

## 9. Subnetting
Dividing a large network into smaller sub-networks (subnets) to improve manageability, security, and efficient address utilization, using a subnet mask to distinguish the network and host portions of an IP address.

## 10. Public vs Private IP
**Public IP**: globally unique, routable on the internet. **Private IP**: used within local networks (e.g., 192.168.x.x, 10.x.x.x), not routable on the internet directly, translated via NAT.

## 11. NAT (Network Address Translation)
A technique that maps private IP addresses to a public IP address (and vice versa) when devices on a local network access the internet, conserving public IP addresses and adding a layer of security.

## 12. DNS (Domain Name System)
A hierarchical, distributed naming system that translates human-readable domain names (e.g., google.com) into IP addresses. Works like a phonebook for the internet. Involves DNS resolvers, root servers, TLD servers, and authoritative servers.

## 13. DHCP (Dynamic Host Configuration Protocol)
A protocol that automatically assigns IP addresses and other network configuration (subnet mask, gateway, DNS) to devices on a network, avoiding manual configuration. Process: DORA (Discover, Offer, Request, Acknowledge).

## 14. HTTP vs HTTPS
HTTP (Hypertext Transfer Protocol) is used for transferring web data but is unencrypted (port 80). HTTPS adds a layer of security using SSL/TLS encryption (port 443), protecting data in transit from eavesdropping/tampering.

## 15. Common Application Layer Protocols
- **HTTP/HTTPS**: web browsing.
- **FTP**: file transfer.
- **SMTP**: sending email.
- **POP3/IMAP**: receiving/retrieving email.
- **DNS**: domain name resolution.
- **DHCP**: dynamic IP assignment.
- **Telnet/SSH**: remote login (SSH is encrypted, Telnet is not).

## 16. Switch vs Router vs Hub
- **Hub**: a basic Layer 1 device that broadcasts incoming data to all ports; no intelligence, causes collisions.
- **Switch**: a Layer 2 device that forwards data only to the intended device using MAC addresses, reducing unnecessary traffic.
- **Router**: a Layer 3 device that connects different networks and routes packets between them based on IP addresses.

## 17. MAC Address vs IP Address
**MAC Address**: a physical, hardware-based address (48-bit) burned into the network interface card, used at the Data Link layer for local delivery. **IP Address**: a logical address used at the Network layer for routing across networks, can change.

## 18. ARP (Address Resolution Protocol)
A protocol used to map a known IP address to its corresponding MAC address on a local network, enabling communication at the Data Link layer.

## 19. Error Detection Techniques
- **Parity Check**: adds a parity bit to detect single-bit errors.
- **Checksum**: sums up data segments to detect errors during transmission.
- **CRC (Cyclic Redundancy Check)**: a more robust polynomial-division-based technique to detect errors in data blocks.

## 20. Flow Control
Mechanisms to ensure a fast sender doesn't overwhelm a slow receiver:
- **Stop-and-Wait**: sender sends one frame, waits for acknowledgment before sending the next.
- **Sliding Window**: sender can send multiple frames before needing an acknowledgment, improving efficiency.

## 21. Congestion Control
Techniques to prevent network overload when too much data is sent, causing packet loss/delay. TCP uses algorithms like **Slow Start**, **Congestion Avoidance**, **Fast Retransmit**, and **Fast Recovery**.

## 22. Routing Algorithms
- **Distance Vector Routing** (e.g., RIP): routers share their routing tables with neighbors; uses Bellman-Ford algorithm.
- **Link State Routing** (e.g., OSPF): each router builds a complete map of the network topology using Dijkstra's algorithm.

## 23. Circuit Switching vs Packet Switching
**Circuit Switching**: a dedicated communication path is established for the entire duration of a call (e.g., traditional telephone networks) — reliable but resource-inefficient. **Packet Switching**: data is broken into packets, sent independently, and reassembled at the destination (e.g., the Internet) — efficient and resilient, used by TCP/IP.

## 24. Sockets and Ports
A **socket** is an endpoint for sending/receiving data across a network, identified by an IP address + port number. **Ports** identify specific processes/services on a device (e.g., port 80 for HTTP, port 443 for HTTPS, port 21 for FTP).

## 25. Firewall
A network security device/software that monitors and filters incoming/outgoing traffic based on predefined security rules, acting as a barrier between trusted and untrusted networks.

## 26. VPN (Virtual Private Network)
Creates a secure, encrypted tunnel over a public network (like the internet), allowing users to send/receive data as if their devices were directly connected to a private network.

## 27. Proxy Server
An intermediary server that sits between client and destination server, forwarding requests on behalf of the client — used for caching, anonymity, filtering, and load balancing.

## 28. Bandwidth vs Latency vs Throughput
- **Bandwidth**: maximum data transfer capacity of a link (bits/sec).
- **Latency**: time delay for data to travel from source to destination.
- **Throughput**: actual rate of successful data transfer, often less than bandwidth due to overhead/congestion.

## 29. Collision Domain vs Broadcast Domain
**Collision Domain**: a network segment where data packets can collide when sent simultaneously (reduced by switches). **Broadcast Domain**: a network segment where a broadcast message reaches all devices (reduced by routers).

## 30. Wireless Networking Basics
- **Wi-Fi**: wireless LAN technology (IEEE 802.11 standards).
- **SSID**: the name broadcast by a wireless network.
- **WPA2/WPA3**: security protocols for encrypting Wi-Fi traffic.

## 31. Load Balancing
Distributing incoming network traffic across multiple servers to ensure no single server is overwhelmed, improving availability and reliability.

## 32. Ping and Traceroute
- **Ping**: uses ICMP to test reachability between two devices and measure round-trip time.
- **Traceroute**: shows the path (hops/routers) packets take to reach a destination, useful for diagnosing network issues.

---

## 33. What Happens When You Type a URL and Press Enter (full walkthrough)
This is the single most asked networking question. Answer it as a chain, layer by layer.

1. **URL parsing** — the browser splits the URL into scheme (`https`), host (`www.google.com`), port (implicit 443), path, and query string.
2. **HSTS check** — if the domain is on the browser's HSTS preload list, `http://` is upgraded to `https://` before any request leaves.
3. **DNS resolution** — browser cache → OS cache → hosts file → configured resolver (usually the ISP or 8.8.8.8). If the resolver has no cached answer it walks the hierarchy: root server → TLD server (`.com`) → authoritative name server for `google.com` → A/AAAA record with the IP.
4. **ARP** — to send the packet, the machine needs the MAC address of the next hop (the default gateway). ARP broadcasts "who has 192.168.1.1?" and caches the reply.
5. **TCP three-way handshake** — SYN, SYN-ACK, ACK with the server IP on port 443.
6. **TLS handshake** — ClientHello (supported ciphers, SNI), ServerHello + certificate, certificate chain validation against trusted CAs, key exchange (ECDHE), then both sides derive a symmetric session key. Everything after this is encrypted.
7. **HTTP request** — `GET / HTTP/1.1` with `Host`, `User-Agent`, `Cookie`, `Accept-Encoding` headers.
8. **Server processing** — reverse proxy / load balancer picks a backend, the application builds a response, possibly hitting a database and a cache.
9. **HTTP response** — status line (`200 OK`), headers, body. Browser parses HTML, builds the DOM, and issues parallel requests for CSS, JS, images.
10. **Render** — DOM + CSSOM → render tree → layout → paint. JavaScript executes and may issue further XHR/fetch calls.
11. **Connection teardown** — either kept alive (HTTP keep-alive / HTTP2 multiplexing) or closed with a four-way FIN handshake.

Say the words "cache at every step" — DNS cache, ARP cache, browser cache, CDN cache. Interviewers like that.

## 34. TLS/SSL Handshake in Slightly More Detail
- **Purpose**: confidentiality (encryption), integrity (MAC/AEAD), authentication (the server's certificate).
- **Asymmetric crypto is used only to agree on a key**; the actual data is encrypted with fast symmetric crypto (AES-GCM, ChaCha20).
- **Certificate**: binds a public key to a domain name, signed by a Certificate Authority. The browser validates the chain up to a root it already trusts, checks the domain matches, checks expiry, and checks revocation (OCSP/CRL).
- **TLS 1.3** cut the handshake to one round trip and removed old ciphers (RC4, CBC modes, static RSA key exchange).
- **Forward secrecy**: ephemeral Diffie-Hellman (ECDHE) means recording today's traffic and stealing the server key tomorrow still does not decrypt it.

## 35. HTTP Versions
| Version | Key idea | Problem it solved |
|---|---|---|
| HTTP/1.0 | One request per TCP connection | — |
| HTTP/1.1 | Keep-alive, pipelining, `Host` header, chunked transfer | Connection setup cost |
| HTTP/2 | Binary framing, multiplexed streams on one connection, header compression (HPACK), server push | Head-of-line blocking at HTTP level |
| HTTP/3 | Runs over QUIC (UDP), built-in TLS 1.3, connection migration | Head-of-line blocking at TCP level, faster handshakes |

## 36. Important HTTP Concepts
- **Methods**: GET (read, safe, idempotent), POST (create, not idempotent), PUT (replace, idempotent), PATCH (partial update), DELETE (idempotent).
- **Status codes**: 1xx informational, 2xx success (200 OK, 201 Created, 204 No Content), 3xx redirection (301 permanent, 302 temporary, 304 Not Modified), 4xx client error (400 Bad Request, 401 Unauthorized, 403 Forbidden, 404 Not Found, 409 Conflict, 429 Too Many Requests), 5xx server error (500, 502 Bad Gateway, 503 Service Unavailable).
- **Stateless**: HTTP itself remembers nothing between requests. State is added on top with cookies, sessions, or tokens.
- **Idempotent** means sending the same request twice has the same effect as sending it once.

## 37. Cookies, Sessions, and Tokens
- **Cookie**: a small key-value pair the server sets with `Set-Cookie` and the browser returns on every subsequent request to that domain.
- **Important cookie flags**: `HttpOnly` (JavaScript cannot read it, blocks XSS theft), `Secure` (only sent over HTTPS), `SameSite=Lax/Strict` (blocks the cookie on cross-site requests, mitigates CSRF).
- **Session**: server stores state and gives the client an opaque session id in a cookie.
- **Token (JWT)**: a signed, self-contained blob — header.payload.signature, base64url encoded. The server verifies the signature instead of looking up a session store, so it scales horizontally, but it cannot be revoked easily before expiry.

## 38. CORS (Cross-Origin Resource Sharing)
Browsers enforce the **same-origin policy**: a page on origin A cannot read responses from origin B by default. Origin = scheme + host + port. CORS is the server saying "it is fine" via `Access-Control-Allow-Origin`. For non-simple requests the browser first sends an **OPTIONS preflight** asking whether the method and headers are allowed. `Access-Control-Allow-Credentials: true` is required to send cookies cross-origin, and then the origin cannot be `*`.

## 39. CIDR and Subnetting Worked Example
CIDR notation `192.168.1.0/24` means the first 24 bits are the network portion, leaving 8 host bits.
- Total addresses = 2^(32 - prefix). Usable hosts = that minus 2 (network address and broadcast address).
- `/24` → 256 addresses, 254 usable, mask 255.255.255.0.
- `/26` → 64 addresses, 62 usable, mask 255.255.255.192.
- `/30` → 4 addresses, 2 usable — used for point-to-point router links.

**Worked example**: split `192.168.10.0/24` for departments needing 100, 50, 20, and 10 hosts. Use **VLSM**, allocating largest first:
- 100 hosts → needs 128 addresses → `/25` → 192.168.10.0/25 (.0 – .127)
- 50 hosts → needs 64 → `/26` → 192.168.10.128/26 (.128 – .191)
- 20 hosts → needs 32 → `/27` → 192.168.10.192/27 (.192 – .223)
- 10 hosts → needs 16 → `/28` → 192.168.10.224/28 (.224 – .239)

## 40. Well-Known Port Numbers
| Port | Service | Port | Service |
|---|---|---|---|
| 20/21 | FTP data / control | 143 | IMAP |
| 22 | SSH / SFTP | 443 | HTTPS |
| 23 | Telnet (plaintext) | 445 | SMB |
| 25 | SMTP | 993 | IMAPS |
| 53 | DNS | 995 | POP3S |
| 67/68 | DHCP server / client | 3306 | MySQL |
| 80 | HTTP | 5432 | PostgreSQL |
| 110 | POP3 | 6379 | Redis |
| 123 | NTP | 27017 | MongoDB |
| 161 | SNMP | 1521 | Oracle |

Ranges: 0–1023 well-known, 1024–49151 registered, 49152–65535 ephemeral/dynamic.

## 41. Common Network Attacks (relevant to my NetSpecter project)
- **Packet sniffing / eavesdropping** — passively reading traffic on a shared medium or a mirrored port. This is exactly what NetSpecter does defensively.
- **MITM (man in the middle)** — attacker sits between two parties, often via ARP spoofing on a LAN.
- **ARP spoofing / poisoning** — forging ARP replies so the victim's ARP cache maps the gateway IP to the attacker's MAC.
- **DNS spoofing / cache poisoning** — injecting a false DNS record so a name resolves to an attacker's IP.
- **DDoS** — overwhelming a service from many sources. SYN flood exhausts the half-open connection table.
- **Session hijacking** — stealing a session cookie sent over plaintext HTTP.
- **Port scanning** — probing which ports are open (nmap) as reconnaissance.

## 42. Network Troubleshooting Commands
| Command | What it tells you |
|---|---|
| `ping <host>` | Reachability and round-trip time (ICMP echo) |
| `traceroute` / `tracert` | Every router hop on the path and where latency appears |
| `nslookup` / `dig` | What DNS returns for a name, and from which server |
| `ipconfig` / `ifconfig` / `ip addr` | Local interfaces, IPs, and masks |
| `netstat -tulpn` / `ss -tulpn` | Listening ports and which process owns them |
| `arp -a` | The local ARP cache (IP to MAC mappings) |
| `curl -v <url>` | Full request/response including headers and TLS details |
| `tcpdump` / Wireshark | Raw packet capture and protocol decoding |
| `nmap` | Open ports and services on a target |

---

# Practice Questions with Answers (Computer Networks)

### Conceptual / Definition-based

**1. Explain the OSI model layers with real-world examples for each.**
Physical — the Ethernet cable or Wi-Fi radio carrying bits. Data Link — a switch forwarding a frame using MAC addresses, plus CRC error detection. Network — a router choosing a path using IP addresses. Transport — TCP guaranteeing your file download arrives complete and in order. Session — establishing and maintaining a connection between two applications, for example an RPC session. Presentation — TLS encryption, JPEG encoding, character-set conversion. Application — your browser speaking HTTP or a mail client speaking SMTP. Mnemonic top-down: "All People Seem To Need Data Processing".

**2. What is the TCP/IP model, and how does it differ from OSI?**
TCP/IP has four layers: Network Access, Internet, Transport, Application. It collapses OSI's Physical + Data Link into Network Access, and OSI's Session + Presentation + Application into Application. OSI was designed first as a reference model and is used for teaching; TCP/IP was built from working protocols and is what the internet actually runs. OSI strictly separates service, interface, and protocol; TCP/IP does not.

**3. What happens when you type a URL in a browser and press Enter?**
See section 33 above — go through it as a chain: URL parse, HSTS, DNS, ARP, TCP handshake, TLS handshake, HTTP request, server processing, response, rendering, teardown.

**4. What is the three-way handshake in TCP?**
Client sends SYN with its initial sequence number x. Server replies SYN-ACK with its own sequence number y and acknowledgement x+1. Client replies ACK with y+1. Both sides now know the other is alive and have agreed on starting sequence numbers, so ordering and retransmission can work.

**5. What is DNS, and how does domain name resolution work step by step?**
DNS maps human-readable names to IP addresses. Resolution order: browser cache, OS cache, hosts file, then the recursive resolver. If the resolver has nothing cached it queries a root server (which returns the `.com` TLD servers), then the TLD server (which returns the authoritative name servers for the domain), then the authoritative server (which returns the A record). The answer is cached at each level for the TTL. Record types worth naming: A (IPv4), AAAA (IPv6), CNAME (alias), MX (mail), NS (name server), TXT (arbitrary text, used for SPF/DKIM/verification).

**6. What is DHCP, and explain the DORA process.**
DHCP automatically assigns IP configuration so nobody has to set it by hand. **D**iscover — client broadcasts looking for a DHCP server. **O**ffer — server offers an available IP with lease terms. **R**equest — client broadcasts that it accepts that specific offer (broadcast so other servers know to withdraw theirs). **A**cknowledge — server confirms and commits the lease. The client also receives the subnet mask, default gateway, and DNS servers. Leases expire and are renewed at 50% of the lease time.

### Comparison-based

**7. Difference between TCP and UDP, with use cases.**
TCP is connection-oriented, reliable, ordered, flow- and congestion-controlled, with a 20-byte header — used for HTTP, email, SSH, file transfer, database connections. UDP is connectionless, unreliable, unordered, with an 8-byte header and no congestion control — used for DNS queries, DHCP, video and voice streaming, online gaming, and QUIC. The trade-off is reliability versus latency: TCP retransmits a lost packet, which stalls everything behind it; a video call would rather drop a frame than pause.

**8. Difference between a hub, switch, and router.**
Hub: Layer 1, repeats every incoming signal to all ports, one collision domain, obsolete. Switch: Layer 2, learns MAC addresses into a CAM table and forwards frames only to the correct port, each port is its own collision domain, but all ports share one broadcast domain. Router: Layer 3, connects different networks, forwards based on IP and a routing table, and separates broadcast domains.

**9. Difference between MAC address and IP address.**
MAC is 48-bit, physical, assigned by the manufacturer, flat (no hierarchy), used for delivery within one local network segment, and does not change as the packet travels. IP is 32- or 128-bit, logical, assigned by the network, hierarchical (so routing scales), and used for end-to-end delivery across networks. In one transmission the source and destination IPs stay constant end to end, while the source and destination MACs are rewritten at every hop.

**10. Difference between HTTP and HTTPS.**
HTTP is plaintext on port 80 — anyone on the path can read and modify it. HTTPS is HTTP inside a TLS tunnel on port 443, giving encryption, integrity, and server authentication via certificates. HTTPS costs an extra handshake, but TLS 1.3 and session resumption make that small. My NetSpecter project exists precisely to demonstrate what HTTP leaks.

**11. Difference between circuit switching and packet switching.**
Circuit switching reserves a dedicated path for the whole session — guaranteed bandwidth and constant delay, but the capacity is wasted when idle, and setup takes time (the classic telephone network). Packet switching splits data into independently routed packets that share links statistically — efficient and fault-tolerant, but with variable delay and possible reordering or loss.

**12. Difference between IPv4 and IPv6.**
IPv4 is 32-bit (~4.3 billion addresses), dotted decimal, needs NAT because of exhaustion, has an optional checksum and variable-length header, and supports broadcast. IPv6 is 128-bit, hexadecimal with colons, has a fixed 40-byte header for faster routing, built-in IPsec support, stateless address autoconfiguration (SLAAC), no broadcast (multicast and anycast instead), and no header checksum.

**13. Difference between public and private IP addresses.**
Public IPs are globally unique and routable on the internet, assigned by IANA/ISPs. Private IPs (10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16 as per RFC 1918) are reusable inside any organisation and are never routed on the public internet; a NAT device translates them. Private addressing conserves the IPv4 space and incidentally hides internal hosts.

**14. Difference between forward proxy and reverse proxy.**
A forward proxy sits in front of clients and acts on their behalf — used for content filtering, caching, and hiding client identity. A reverse proxy sits in front of servers and acts on their behalf — used for load balancing, TLS termination, caching, and hiding backend topology. Nginx as a reverse proxy is the common example; a corporate web filter is a forward proxy.

### Scenario/Application-based

**15. A webpage isn't loading — how do you troubleshoot?**
Work up the layers. Is the interface up and does it have an IP (`ip addr`)? Can I reach the gateway (`ping 192.168.1.1`)? Can I reach the internet by IP (`ping 8.8.8.8`) — if yes but names fail, it is DNS. Does the name resolve (`dig example.com`)? Is the path broken somewhere (`traceroute`)? Is the port open (`telnet host 443` or `nc -zv`)? Does the request itself work (`curl -v`)? Finally, is it the browser (cache, extensions, certificate error)? Naming this ordered progression matters more than any single command.

**16. How would you design a subnet for a company with 4 departments needing different numbers of hosts?**
Use VLSM and allocate the largest requirement first so blocks stay aligned. See section 39 for the full worked example. Key points: reserve 2 addresses per subnet for network and broadcast, round up to the next power of two, and keep room for growth.

**17. Explain how NAT lets many devices share one public IP.**
The router keeps a translation table. When an inside host sends a packet, the router rewrites the source IP to its public IP and the source port to a unique free port, recording the mapping (inside IP:port ↔ public IP:port). The reply arrives addressed to that public port, and the router looks up the table and rewrites it back. This is PAT / NAT overload. The side effect is that inbound connections cannot reach an inside host unless a port-forward rule exists.

**18. Why does video streaming use UDP while file downloads use TCP?**
For live video, a packet that arrives late is useless — retransmitting it wastes time and stalls the stream, so it is better to drop it and keep playing. For a file, every byte must be exact, so reliability wins over latency. Note that on-demand streaming like YouTube largely uses TCP/QUIC with buffering; it is real-time calls and gaming that prefer UDP.

**19. How does a firewall decide to block or allow traffic?**
It evaluates each packet or connection against an ordered rule set matching on source/destination IP, port, protocol, and interface, with a default-deny policy at the end. A **stateless** filter judges each packet in isolation. A **stateful** firewall tracks connection state, so return traffic for an established outbound connection is allowed automatically. A **next-generation / application-layer** firewall inspects the payload and can block by application or content.

**20. Explain how HTTPS ensures secure communication.**
See section 34. Three properties: confidentiality from symmetric encryption, integrity from authenticated encryption, and authentication from the CA-signed certificate. Asymmetric crypto is used only to establish the shared key.

### Protocol / Layer-specific

**21. What is ARP and why is it needed alongside IP?**
IP identifies the destination logically, but a frame can only be delivered on a LAN using a MAC address. ARP resolves the next-hop IP to a MAC by broadcasting a request; the owner replies with its MAC, which is cached. Without ARP the network layer would have no way of handing packets to the data link layer. Note the security weakness: ARP has no authentication, which is why ARP spoofing works.

**22. Difference between flow control and congestion control?**
Flow control protects the **receiver** — it stops a fast sender from overrunning a slow receiver's buffer, using the advertised receive window. Congestion control protects the **network** — it stops senders from collectively overloading routers, using the congestion window computed from loss and delay signals. The sender transmits the minimum of the two windows.

**23. Explain sliding window and how it improves on stop-and-wait.**
Stop-and-wait sends one frame and waits a full round trip before sending the next, so link utilisation is roughly (transmission time) / (transmission time + RTT) — terrible on high-latency links. Sliding window lets the sender have up to N unacknowledged frames in flight, keeping the pipe full. Go-Back-N retransmits the whole window on loss; Selective Repeat retransmits only the lost frame but needs a receiver buffer.

**24. What is CRC, and how does it detect errors?**
The sender treats the data as a binary polynomial, divides it by an agreed generator polynomial, and appends the remainder as check bits. The receiver divides the whole received block by the same generator; a zero remainder means no detected error. CRC catches all single-bit errors, all double-bit errors, all odd numbers of bit errors, and any burst error shorter than the CRC length — far stronger than parity or a simple checksum, and cheap in hardware.

**25. Explain distance vector vs link state routing.**
Distance vector (RIP): each router knows only its neighbours, periodically sends its whole routing table to them, and runs Bellman-Ford. Simple, but slow to converge and prone to count-to-infinity, mitigated by split horizon and poison reverse. Link state (OSPF): each router floods link-state advertisements so every router builds an identical map of the topology, then runs Dijkstra locally. Faster convergence and loop-free, but more memory and CPU.

**26. What are sockets, and how do they relate to ports and IPs?**
A socket is the OS endpoint for network communication. A TCP connection is uniquely identified by a **4-tuple**: source IP, source port, destination IP, destination port. A server socket binds to a well-known port and listens; each accepted connection produces a new socket sharing that port but differing in the client side of the tuple. That is why one server on port 443 can serve thousands of clients.

### Tricky / Deep-dive

**27. Why three-way handshake and not two-way?**
Because both directions need their initial sequence numbers acknowledged. A two-way exchange would confirm only the client's sequence number, leaving the server unsure its own was received. It also prevents an old, delayed duplicate SYN from opening a phantom connection — the final ACK proves the client is actually present and responding now.

**28. Collision domain vs broadcast domain?**
A collision domain is the set of devices whose transmissions can collide on a shared medium — each switch port is its own collision domain, so switches shrink them. A broadcast domain is the set of devices that receive a broadcast frame — a switch forwards broadcasts everywhere, so all its ports are one broadcast domain. Routers and VLANs split broadcast domains.

**29. What happens during TCP connection termination?**
Four-way: side A sends FIN, side B ACKs it (A can no longer send but B still can — half-close), then B sends its own FIN, and A ACKs it. A then waits in **TIME_WAIT** for 2×MSL to make sure the final ACK arrived and that stray old segments die before the same 4-tuple is reused.

**30. Explain TCP congestion control phases.**
**Slow start**: cwnd begins at ~1 MSS and doubles every RTT — exponential growth until it hits the slow-start threshold. **Congestion avoidance**: growth becomes linear (+1 MSS per RTT) to probe capacity gently. **Fast retransmit**: three duplicate ACKs imply one segment was lost while later ones arrived, so retransmit immediately instead of waiting for a timeout. **Fast recovery**: halve cwnd and continue in congestion avoidance rather than collapsing to slow start. A full timeout is treated as severe congestion and does reset to slow start. (TCP Reno behaviour; modern stacks often use CUBIC or BBR.)

**31. Why is UDP "unreliable", and when is that acceptable?**
UDP does not acknowledge, retransmit, reorder, or control rate — it just sends a datagram. That is acceptable whenever late data is worthless (live audio/video, gaming), when the application implements its own reliability (QUIC), or when the exchange is a single small request-response where a retry at application level is cheaper than a handshake (DNS).

**32. Difference between latency, bandwidth, and throughput?**
Bandwidth is the theoretical maximum capacity of the link. Throughput is what you actually achieve after overhead, congestion, and loss. Latency is the delay for one bit to travel end to end, and it is made of propagation delay, transmission delay, queuing delay, and processing delay. A satellite link can have huge bandwidth and terrible latency — capacity and delay are independent.

### Miscellaneous

**33. What is a VPN and how does it secure communication?**
A VPN encapsulates and encrypts your traffic inside a tunnel to a VPN server, so intermediate networks see only encrypted packets to the VPN endpoint. It provides confidentiality on untrusted networks, lets remote users appear to be inside a private network, and hides your real IP from the destination. Protocols: IPsec, OpenVPN, WireGuard.

**34. What is load balancing, and what algorithms are used?**
Distributing incoming requests over a pool of backends for availability and scale. Algorithms: round robin, weighted round robin, least connections, least response time, IP hash (for session affinity), and consistent hashing (used in distributed caches so adding a node moves few keys). Layer 4 balancing works on IP/port; Layer 7 balancing can route on URL path or headers. Health checks remove dead backends.

**35. What common port numbers should you know?**
See section 40 — at minimum 20/21 FTP, 22 SSH, 23 Telnet, 25 SMTP, 53 DNS, 80 HTTP, 110 POP3, 143 IMAP, 443 HTTPS, 3306 MySQL, 5432 PostgreSQL, 27017 MongoDB, 1521 Oracle.

**36. What is the role of a proxy server?**
It intermediates requests. Uses: caching to cut bandwidth and latency, access control and content filtering, anonymity, logging and auditing, TLS termination, and load balancing when deployed as a reverse proxy.

---

# Rapid-Fire One-Liners

- **MTU** — largest frame payload a link can carry, typically 1500 bytes on Ethernet. Exceeding it forces fragmentation.
- **TTL** — hop counter in the IP header, decremented by each router; at zero the packet is dropped and ICMP Time Exceeded is sent. This is how traceroute works.
- **ICMP** — control and error messaging protocol used by ping and traceroute; not a data transport.
- **NIC** — network interface card, holds the MAC address.
- **VLAN** — logically separate broadcast domains on the same physical switch.
- **Gateway** — the router a host sends traffic to when the destination is outside its own subnet.
- **Loopback** — 127.0.0.1, traffic that never leaves the machine.
- **Multicast** — one sender to a subscribed group; **broadcast** — one sender to everyone on the segment; **anycast** — one sender to the nearest of several identical destinations (used by DNS root servers and CDNs).
- **CDN** — geographically distributed cache of static content close to users.
- **Half duplex vs full duplex** — one direction at a time versus both simultaneously.
- **Piggybacking** — carrying an acknowledgement inside a data frame going the other way.
- **Nagle's algorithm** — buffers small writes to avoid sending many tiny packets; disabled with TCP_NODELAY for latency-sensitive apps.
- **Socket states** — LISTEN, SYN_SENT, SYN_RECEIVED, ESTABLISHED, FIN_WAIT, TIME_WAIT, CLOSED.

---

# Linking Networks to My Projects

Interviewers love it when theory connects back to what you built. Prepare these:

- **NetSpecter** is a live demonstration of most of this file: it uses raw sockets via Scapy, applies a BPF filter (`tcp port 80 or tcp port 23`), inspects the TCP payload above the IP layer, and reassembles TCP segments because credentials can straddle MTU boundaries. It proves why HTTP is unsafe and HTTPS is not optional. It also touches ports (21 FTP, 25 SMTP, 110 POP3, 143 IMAP, 6379 Redis), Basic Auth, and cookie theft.
- **ClassRoom Code** exercises the application layer: HTTP methods and status codes across a REST API, Google OAuth 2.0 redirects, session cookies with `HttpOnly`, `Secure` and `SameSite=Lax` flags, CORS and a Vite dev proxy so the browser stays same-origin, and a client-server split talking to Judge0 over HTTP.
- **ModelAuth** talks to a local Ollama server over an HTTP REST API on port 11434 using an OpenAI-compatible client, with retries on failure and concurrent requests via a thread pool.
