# Project 3 — NetSpecter

**Repository:** https://github.com/praneeth3696/NetSpecter
**Branches:** `main` (v1, the shipped HTTP credential detector), `cli` (v2, multi-protocol engine with TCP reassembly and a TUI), `frontend-backend` (v2 plus a web dashboard)
**Stack:** Python 3, Scapy (libpcap), Rich (terminal UI), argparse
**Nature:** Defensive security tool — a passive network auditor

This is the project that anchors my cybersecurity domain choice, so expect the interviewer to move from this project straight into security questions. Have both ready.

---

## 1. The one-minute pitch

"NetSpecter is a passive network traffic auditor that detects credentials being transmitted in plaintext. It binds to a network interface, captures TCP traffic, and inspects the payloads for authentication data being sent without encryption — form-encoded logins, JSON bodies, URL query parameters, HTTP Basic Auth headers, bearer tokens, multipart fields, and credentials embedded in URLs. Version 2 extends it to FTP, SMTP, POP3, IMAP, and Redis AUTH, adds cloud API key detection for AWS, GitHub, Slack and Stripe, does TCP stream reassembly so credentials split across packet boundaries are not missed, and can replay offline PCAP files for forensic analysis. It is a defensive tool: the point is to demonstrate, concretely and on your own network, why plaintext protocols are indefensible and why HTTPS is not optional."

## 2. The problem it solves

The theory — "HTTP is insecure, use HTTPS" — is universally taught and weakly believed, because it is abstract. Seeing your own password appear on someone else's terminal thirty seconds after you typed it is not abstract.

Beyond the demonstration value, the real-world problem is genuine: despite near-universal HTTPS on the public web, **plaintext protocols persist inside networks**. Legacy internal services, IoT and embedded appliances with unencrypted admin panels, containerised microservices talking over an unencrypted bridge network, staging environments where somebody disabled TLS to debug something, printers, network cameras, old FTP servers, and Redis instances with `AUTH` sent in the clear. Perimeter security assumes the inside is trustworthy; an attacker who has any foothold on the LAN — or a rogue device, or a compromised switch port — can passively harvest credentials without generating a single suspicious connection.

NetSpecter is the tool a defender points at their own network to find those before an attacker does.

## 3. Ethics and authorisation — say this before they ask

This is a dual-use tool, and volunteering the boundary is far better than being asked about it.

"It is passive — it never injects, spoofs, or modifies traffic, so it cannot be used to attack a system, only to observe one. It requires root, so it cannot be run silently on a machine you do not control. The README carries an explicit authorisation warning. I have only ever run it on my own machines and on traffic I generated myself — the demo is `curl -X POST http://testphp.vulnweb.com/login.php -d "username=admin&password=1234"` against a deliberately vulnerable test site, in another terminal on my own laptop. Running it on a network you do not own or have written permission to test is unlawful under the IT Act in India and equivalent legislation elsewhere. Version 2 also masks credentials by default (`admin: s3c****t`) and requires an explicit `--show-secrets` flag, precisely so a demo or a recording does not itself leak anything."

## 4. How it works — the pipeline

```
Network Interface (promiscuous)  or  Offline PCAP file
              |
              v
      Scapy sniff() with a BPF filter    "tcp port 80 or tcp port 23"
              |
              v
      Layer check: IP + TCP + Raw present?          <-- skip anything with no payload
              |
              v
      TCP Stream Reassembler  (v2)                  <-- keyed by the 5-tuple
              |
              v
      Fast byte-level pre-filter                    <-- b"user", b"pass", b"login", b"auth", b"token"
              |                                          reject in microseconds if absent
              v
      Decode UTF-8 with errors='ignore'
              |
              v
      Detector dispatch  (v2: HTTP / FTP / Mail / Redis / Token)
              |
              v
      DetectionResult {type, username, password, confidence, snippet}
              |
              v
      Rich terminal alert  /  TUI dashboard  /  JSON + HTML report
```

**The performance decision worth explaining**: on a busy interface you see thousands of packets per second, and running a set of regular expressions against every payload would not keep up. So there is a **two-stage filter**. Stage one is a plain byte-substring scan for a handful of keywords on the raw `bytes` object — no decoding, no regex, no allocation. Only payloads that survive it are decoded to a string and handed to the regex-based detectors. Most traffic is rejected in stage one at essentially zero cost. The BPF filter in the kernel is an even earlier stage: it discards non-matching packets before they are ever copied to userspace.

## 5. What it detects

**HTTP (v1 and v2):**
| Format | Example |
|---|---|
| Form-urlencoded POST | `username=admin&password=1234` |
| JSON body | `{"user": "admin", "pass": "1234"}`, including nested keys |
| URL query parameters | `GET /login?user=admin&pwd=1234` |
| HTTP Basic Auth | `Authorization: Basic YWRtaW46MTIzNA==` — base64-decoded to reveal `admin:1234` |
| Bearer / Token headers | `Authorization: Bearer eyJhbGci...` |
| Multipart form data | Field names extracted from `Content-Disposition` |
| URL userinfo | `http://user:pass@host/` |
| Insecure session cookies | `sessionid`, `PHPSESSID`, `JSESSIONID`, `connect.sid`, `auth_token` |

**Base64 is not encryption** — this is the single best teaching point in the tool. HTTP Basic Auth base64-encodes `username:password`, which looks opaque and is trivially reversible with one function call. NetSpecter decodes it in real time to make that concrete.

**Other protocols (v2):** FTP `USER`/`PASS` on port 21; SMTP `AUTH PLAIN` and `AUTH LOGIN`; POP3 `USER`/`PASS`; IMAP `LOGIN`; Redis `AUTH` in both inline and RESP forms.

**Cloud and API secrets (v2):** AWS access keys (`AKIA[0-9A-Z]{16}`), GitHub tokens (`ghp_`, `github_pat_`), Slack tokens (`xoxb-`, `xoxp-`, `xoxa-`), Stripe live secret keys (`sk_live_`), and JWTs — where it decodes the base64 header and claims in real time to extract subject, role, and email, which demonstrates that **a JWT is signed, not encrypted**: anyone who intercepts one can read its entire payload.

## 6. TCP stream reassembly — the hardest part of v2

**The problem v1 had**, and it is stated honestly in the v1 README's limitations section: it inspects each packet's payload independently. But **TCP is a byte stream, not a message protocol**. A login POST whose body straddles an MTU boundary arrives as two segments, and `password=` might end one packet while the value begins the next. Neither packet matches on its own, so the credential is missed.

**The solution**: track flows keyed by the **5-tuple** (source IP, source port, destination IP, destination port, protocol), buffer each unidirectional flow's payload in a `StreamBuffer`, and run detection against the accumulated buffer rather than the isolated segment.

**The engineering constraints that come with it** — this is what makes it a good interview answer, because a naive implementation is a denial-of-service on yourself:
- **Bounded per-flow buffers** — capped at 64 KB, and when the cap is hit the buffer keeps the most recent half rather than growing without limit. An attacker (or just a large upload) must not be able to exhaust memory.
- **A bounded flow table** — capped at 1000 concurrent flows.
- **Flow timeout and cleanup** — flows idle for more than 45 seconds are evicted, because TCP connections are not always closed cleanly and a leaked flow table entry is a slow memory leak.
- The reassembler returns both the raw segment and the accumulated stream, so a detector can choose which it wants.

This is a genuine, self-imposed resource-management problem, and it maps directly onto operating-systems concepts: bounded buffers, eviction policy, and timeout-based garbage collection.

## 7. The code, module by module

**v1 (`main` branch):**
- `main.py` — argparse CLI with a `scan` subcommand and an optional `--iface`. Enforces two preconditions before doing anything: `os.name == "posix"` and `os.geteuid() == 0`, each with a clear error message rather than an obscure permission failure.
- `sniffer.py` — wraps Scapy's `sniff()` with the BPF filter `tcp port 80 or tcp port 23`, `store=False` so packets are not accumulated in memory, and a `prn` callback per packet. The handler checks `pkt.haslayer(IP) and pkt.haslayer(TCP) and pkt.haslayer(Raw)` before extracting the payload, and wraps everything in a bare `except` — **deliberately**, so one malformed packet can never kill the capture loop. That is a real design decision: a monitoring tool that dies on the first weird packet is worse than useless.
- `detector_wrapper.py` — the fast byte pre-filter and the bytes-to-string bridge.
- `detectors/http_credential_detector.py` — ~613 lines, the core. Pre-compiled regexes (compiled once at import, not per packet), separate sets of known username and password field names, handlers for each format, URL-decoding, base64 decoding for Basic Auth, and a return contract of `{type, username, password, confidence, raw_snippet}` with a ≤200-character context window.
- `formatter.py` — Rich-based alert panels.
- `tests/` — a unit test suite for the detection engine, which can be run without any network access because it feeds payload strings directly.

**v2 (`cli` and `frontend-backend` branches):**
- `core/models.py` — `FlowKey` and `DetectionResult` dataclasses, giving every detector a common output type.
- `core/stream_reassembler.py` — `StreamBuffer` and `TCPStreamReassembler`.
- `core/detector_engine.py` — the dispatcher. It holds one instance of each protocol detector, applies the expanded fast-keyword pre-filter, and additionally lets traffic through on well-known ports (21, 23, 25, 110, 143, 6379) even when no keyword matched, because a protocol command like `USER bob` contains none of the generic keywords.
- `detectors/` — `ftp_detector.py`, `mail_detector.py`, `redis_detector.py`, `token_detector.py`, alongside the HTTP one.
- `reporting/reporter.py` — structured JSON/JSONL export and a self-contained executive HTML audit report with remediation advice.
- `ui/dashboard.py` — a Rich `Live` full-screen TUI with packet counters, throughput, an active-flow monitor, and a live incident feed.
- Offline mode: `netspecter pcap <file>` replays a capture for post-incident forensics, which needs no root and no live interface.

## 8. Networking concepts this project demonstrates

This is the bridge into the Computer Networks questions, and being able to walk it is worth a lot.

- **The layer stack in practice** — Scapy hands you a packet you index by layer: `pkt[IP].src`, `pkt[TCP].dport`, `bytes(pkt[Raw].payload)`. That makes encapsulation concrete: the application payload is inside the TCP segment, inside the IP packet, inside the Ethernet frame.
- **Raw sockets and why they need root** — normal sockets only give you your own process's traffic. Capturing everything on the wire needs `AF_PACKET`/`SOCK_RAW` (or BPF on BSD/macOS), which is privileged because it would otherwise let any user read every other user's network traffic.
- **Promiscuous mode** — the NIC normally discards frames not addressed to its MAC; promiscuous mode accepts everything on the segment. On a modern switched network you only see your own traffic plus broadcasts, so a realistic attack also requires ARP spoofing or a mirrored/SPAN port — which is worth saying, because it shows I understand the difference between a lab demo and a real threat model.
- **BPF (Berkeley Packet Filter)** — the filter expression is compiled and executed **in the kernel**, so non-matching packets are discarded before being copied to userspace. Filtering in Python instead would be orders of magnitude slower.
- **Port-to-protocol mapping** — 21 FTP, 23 Telnet, 25 SMTP, 80 HTTP, 110 POP3, 143 IMAP, 6379 Redis.
- **Why HTTPS defeats it entirely** — TLS encrypts the payload before it reaches TCP, so the `Raw` layer contains ciphertext. All that remains visible is metadata: source and destination IPs, ports, packet sizes and timings, and the **SNI field in the ClientHello**, which reveals the hostname in plaintext (unless Encrypted Client Hello is in use). Being able to say exactly what remains visible under TLS is a strong answer.

## 9. Questions they will ask, with answers

**"How is this different from Wireshark?"**
Wireshark is a general-purpose protocol analyser: it captures and decodes everything and leaves interpretation to you. NetSpecter is purpose-built and opinionated — it answers one question, "is anything leaking credentials on this network right now?", and it answers it in real time with a one-line verdict instead of requiring you to write display filters and read hex. Wireshark is the microscope; this is the smoke alarm. Practically it is also headless, scriptable, and produces a JSON and HTML audit report you can hand to someone.

**"Why does it need root?"**
Opening a raw socket in promiscuous mode is a privileged operation, because without that restriction any unprivileged user could read every other user's network traffic. `main.py` checks `os.geteuid() != 0` and exits with a clear message rather than failing obscurely inside Scapy. On Linux you can alternatively grant `CAP_NET_RAW` to the binary instead of running the whole thing as root, which is the more careful approach and something I would add.

**"What are its limitations?"** (v1's README lists these honestly, which is deliberate.)
- It cannot see inside HTTPS. That is the point of HTTPS.
- v1 does no TCP stream reassembly, so split-packet payloads are missed — which is exactly what v2 fixes.
- Limited handling of compressed bodies (`gzip`, `br`) and HTTP/2 binary framing, since HTTP/2 uses HPACK header compression, so header-based detection would need a HPACK decoder.
- On a switched network it only sees its own segment's traffic without a SPAN port or ARP spoofing.
- Regex-based detection has false positives — a field named `token` might not be a secret — which is why every result carries a confidence level rather than a binary verdict.

**"How would you make it production-grade?"**
Move the hot path off Python — the pre-filter and reassembly in Rust or C, or use `eBPF`/`XDP` to filter in the kernel. Add a proper protocol state machine rather than regex over a byte buffer. Store findings in a time-series database and alert through a SIEM rather than printing to a terminal. Add a rules engine so detections are configuration rather than code. Run it as a systemd service with `CAP_NET_RAW` instead of root. And add sampling and back-pressure so it degrades gracefully under load instead of dropping packets silently.

**"How do you fix what it finds?"**
That is the report's job, and the HTML output includes remediation advice. In order: enforce TLS everywhere including internal services, redirect HTTP to HTTPS and set **HSTS**, mark session cookies `Secure` and `HttpOnly` and `SameSite`, never use HTTP Basic Auth outside TLS, rotate any credential the tool observed because it must be treated as compromised, move secrets out of URLs (they end up in server logs, browser history, and `Referer` headers), use a secrets manager rather than API keys in code or config files, disable plaintext FTP/Telnet in favour of SFTP/SSH, and require `AUTH` over TLS or a network policy for Redis.

**"What did you learn?"**
Two things. First, that the gap between "I know HTTP is insecure" and "I have watched my own password appear on another terminal" is enormous, and the second one changes behaviour. Second, that a monitoring tool's hardest problems are not detection logic but resource management and robustness: bounded buffers, flow eviction, never crashing on malformed input, and being fast enough that you are not the reason packets get dropped.

**"Which branch should I look at?"**
`main` is v1 — the clean, shipped HTTP credential detector, which is the version described on my resume. `cli` is v2 with the multi-protocol engine, TCP reassembly, PCAP replay, the TUI dashboard, and reporting. `frontend-backend` is v2 plus a web dashboard. I keep them separate rather than merging because v1 is a small, readable, working tool and v2 is a substantially larger system that is still being extended.
