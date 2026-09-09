# Cybersecurity — Domain Deep Dive

This is the domain I choose when asked. Once I say "cybersecurity", the rest of the interview is likely to come from here. Read this file properly and be able to hold a conversation, not just recite definitions.

---

# PART 0 — How to Answer "Which Domain Do You Want?"

Do not just name it. Give a reason that is anchored in something you have actually built, and show you know what the other options are.

> "Cybersecurity. I considered all three seriously — data science interests me and I have done statistical work in ModelAuth, and software development is where most of my hours have gone with ClassRoom Code. But the projects I kept choosing to build on my own time were security ones, and the reason is a specific instinct: I find myself asking what happens when you cannot trust the channel or the counterparty.
>
> NetSpecter came from wanting to prove, rather than assert, that plaintext protocols leak — so I built a passive auditor that captures live traffic and surfaces credentials being sent in the clear. ModelAuth came from noticing that when you call an LLM API, TLS and API keys authenticate the *host* but nothing authenticates the *weights* — so a provider could silently serve you a cheaper model and you would have no evidence either way. Both are the same question: how do you verify something when the other side has no obligation to be honest with you?
>
> That is also why forensic science interests me outside computing. It is the same discipline — reconstructing what happened from the traces left behind, when the person who did it was not trying to help you. I want to work in security because it is the field where that instinct is the job."

**If they ask why not software development**, given ClassRoom Code is the biggest project: "Because the parts of that project I found most interesting were the security decisions — drawing the trust boundary around the AI importer so it can never invent an expected output, returning 404 instead of 403 so the API does not confirm what exists to outsiders, deciding that a teacher's *role* is not enough and assignment to the course is the real permission unit, and refusing to run untrusted code without a sandbox. I enjoyed building the product, but I was drawn to the threat modelling."

---

# PART 1 — Foundations

## 1.1 The CIA triad
- **Confidentiality** — only authorised parties can read the data. Achieved with encryption, access control, and least privilege. Broken by: eavesdropping, data breaches, weak access control.
- **Integrity** — data has not been altered, and alterations are detectable. Achieved with hashing, digital signatures, MACs, and checksums. Broken by: tampering, man-in-the-middle modification, unauthorised writes.
- **Availability** — the system is there when needed. Achieved with redundancy, backups, failover, rate limiting, and DDoS protection. Broken by: denial of service, ransomware, hardware failure.

**Every security control maps to one or more of these, and they trade off.** Encrypting everything and requiring MFA on every action improves confidentiality and hurts availability and usability. Being able to say that trade-off out loud is what separates a memorised answer from an understood one.

## 1.2 Beyond the triad
- **Authentication** — proving who you are.
- **Authorisation** — what you are allowed to do once authenticated. These two are constantly confused; 401 means "I do not know who you are", 403 means "I know, and no".
- **Non-repudiation** — you cannot later deny having done it. Provided by digital signatures and audit logs.
- **Accountability / auditability** — actions are attributable to an identity and recorded.
- **Authenticity** — the data is genuinely from the claimed source.

## 1.3 Core security principles
- **Defence in depth** — multiple independent layers, so no single failure is fatal. A firewall *and* input validation *and* least privilege *and* monitoring.
- **Least privilege** — every user, process, and service gets the minimum access needed. My NetSpecter needs root only because raw sockets require it, and the better version would use `CAP_NET_RAW` instead of full root — that is least privilege in practice.
- **Fail securely** — on error, deny rather than allow. My ClassRoom Code server **refuses to boot with an insecure production configuration** rather than starting up with defaults, and returns 503 rather than falling back to an unsandboxed executor in production.
- **Zero trust** — never trust based on network location; verify every request, every time. "Assume breach."
- **Separation of duties** — no single person can complete a sensitive action alone.
- **Complete mediation** — check authorisation on every access, not just the first.
- **Open design (Kerckhoffs's principle)** — security must not depend on the secrecy of the design, only on the secrecy of the key. "No security through obscurity."
- **Psychological acceptability** — controls people find unbearable get bypassed, so a usable control that is followed beats a perfect one that is not.
- **Attack surface reduction** — every open port, every dependency, every feature is something to defend. The most secure code is the code you did not write.

## 1.4 Threat, vulnerability, risk, exploit
- **Asset** — something worth protecting.
- **Threat** — a potential cause of harm (an attacker, a flood, an insider).
- **Vulnerability** — a weakness that a threat could exploit.
- **Exploit** — the actual technique or code that uses the vulnerability.
- **Risk** = likelihood × impact. Security is risk *management*, not risk elimination.
- **Zero-day** — a vulnerability with no patch available, sometimes unknown to the vendor.
- **CVE** — the public identifier for a known vulnerability. **CVSS** scores severity from 0 to 10.

## 1.5 Types of attackers
White hat (authorised, ethical), black hat (criminal), grey hat (unauthorised but not malicious), script kiddie (uses others' tools), hacktivist (ideological), insider threat (malicious or negligent, and the hardest to detect), APT (advanced persistent threat — a well-resourced, patient, usually state-backed actor whose goal is long-term access rather than a quick payoff).

---

# PART 2 — Cryptography

## 2.1 Symmetric encryption
One key encrypts and decrypts. Fast, suitable for bulk data. **AES** (128/192/256-bit) is the standard; ChaCha20 is a fast software alternative. DES and 3DES are obsolete.
**The problem it does not solve**: key distribution. Two parties who have never met cannot agree on a shared key over an insecure channel using symmetric crypto alone.
**Modes**: ECB is broken for anything structured because identical plaintext blocks produce identical ciphertext blocks (the famous "ECB penguin"). CBC needs an unpredictable IV. **GCM** is the modern choice because it is authenticated encryption — it provides confidentiality *and* integrity in one construction.

## 2.2 Asymmetric (public key) encryption
A key pair: the public key encrypts (or verifies), the private key decrypts (or signs). **RSA**, **ECC** (smaller keys for equivalent strength, so preferred on mobile and in TLS), Diffie-Hellman for key exchange.
Slow, so it is used to establish a symmetric key rather than to encrypt bulk data.
**Digital signature** is the reverse direction: you sign with your *private* key and anyone verifies with your *public* key, which gives authenticity, integrity, and non-repudiation.

## 2.3 Hashing
A one-way function producing a fixed-length digest. **Properties**: deterministic, fast, pre-image resistant (cannot invert), second-pre-image resistant, collision resistant, and exhibiting the avalanche effect (one bit changed changes roughly half the output bits).
- **MD5 and SHA-1 are broken** — practical collisions exist. Never use them for security.
- **SHA-256 / SHA-3** for integrity.
- **For passwords, use a slow, salted KDF**: bcrypt, scrypt, Argon2, or PBKDF2. Fast hashes are the wrong tool because speed helps the attacker.

**Encryption vs hashing vs encoding — a guaranteed question:**
| | Reversible? | Purpose | Example |
|---|---|---|---|
| Encoding | Yes, trivially, no key | Data representation and transport | Base64, URL encoding |
| Hashing | No | Integrity, password storage | SHA-256, bcrypt |
| Encryption | Yes, with the key | Confidentiality | AES, RSA |

**Base64 is not encryption.** My NetSpecter project decodes HTTP Basic Auth headers in real time specifically to make that point concrete: `Authorization: Basic YWRtaW46MTIzNA==` looks opaque and is one function call away from `admin:1234`.

## 2.4 Password storage — know this in detail
1. Never store plaintext.
2. Never use a fast hash (MD5, SHA-256 alone) — a GPU computes billions per second, so a leaked database of SHA-256 hashes is cracked quickly.
3. **Salt** — a unique random value per user, stored alongside the hash. It defeats rainbow tables and ensures two users with the same password get different hashes.
4. **Pepper** — an additional secret, stored separately from the database (in an HSM or environment config), so a database dump alone is not enough.
5. **Use a slow KDF with a tunable work factor** — bcrypt, scrypt, or **Argon2id** (the current recommendation, resistant to both GPU and side-channel attacks). Increase the work factor as hardware improves.
6. On login, hash the supplied password with the stored salt and compare in **constant time** to avoid timing attacks.

**Related attacks**: brute force (try everything), dictionary attack (try likely passwords), rainbow tables (precomputed hash-to-password lookup, defeated by salting), credential stuffing (reuse leaked credentials from another breach against your site — which is why password reuse is so damaging), and password spraying (one common password against many accounts, to avoid lockout).

## 2.5 TLS/SSL and PKI
- **What TLS provides**: confidentiality (symmetric encryption), integrity (AEAD/MAC), and server authentication (the certificate).
- **Handshake** (TLS 1.3, one round trip): ClientHello with supported ciphers and the SNI; ServerHello with the chosen cipher, the certificate, and key share; both derive the shared secret via ephemeral ECDHE; everything after is encrypted.
- **Certificate** — binds a public key to a domain name, signed by a Certificate Authority. The browser validates the chain up to a trusted root, checks the domain matches, checks expiry, and checks revocation (OCSP/CRL).
- **Forward secrecy** — ephemeral keys mean recording today's traffic and stealing the server's private key tomorrow does not decrypt it.
- **Chain of trust** — root CA → intermediate CA → leaf certificate. Roots are pre-installed in the OS and browser trust stores.
- **What TLS does NOT protect**: metadata. An observer still sees source and destination IPs, ports, packet sizes and timing, and the **SNI hostname in the ClientHello** unless Encrypted Client Hello is used. This is exactly the boundary NetSpecter runs into — under HTTPS its `Raw` layer contains ciphertext and it sees nothing useful.
- **HSTS** — a response header telling the browser to only ever use HTTPS for this domain, preventing SSL-strip downgrade attacks.
- **Certificate pinning** — an application hard-codes the expected certificate or public key, so a rogue CA cannot impersonate the server.

---

# PART 3 — Network Security

## 3.1 Common attacks
- **Sniffing / eavesdropping** — passive capture of traffic. My NetSpecter project is this technique used defensively.
- **MITM** — the attacker relays and possibly modifies traffic between two parties who believe they are talking directly. Enabled on a LAN by ARP spoofing; defeated by TLS with proper certificate validation.
- **ARP spoofing / poisoning** — ARP has no authentication, so an attacker forges ARP replies mapping the gateway's IP to their own MAC, and the victim sends all traffic through them. Mitigations: dynamic ARP inspection on managed switches, static ARP entries for critical hosts, and network segmentation.
- **DNS spoofing / cache poisoning** — injecting false DNS records so a name resolves to an attacker's IP. Mitigation: DNSSEC, DoH/DoT.
- **DoS and DDoS** — exhausting a resource. A **SYN flood** fills the half-open connection table (mitigated by SYN cookies); an **amplification attack** uses a small spoofed request to a service like DNS or NTP that replies with a much larger response to the victim. Mitigation: rate limiting, upstream scrubbing services, anycast, CDNs.
- **Session hijacking** — stealing a session cookie sent over plaintext HTTP or via XSS. Mitigation: `Secure` and `HttpOnly` cookie flags, HTTPS everywhere, session rotation on privilege change.
- **Port scanning** — reconnaissance to map open services. `nmap -sS` (SYN/half-open scan), `-sV` (service version detection), `-O` (OS fingerprinting).
- **Replay attack** — capturing and re-sending a valid message. Mitigated with nonces, timestamps, and sequence numbers. My ClassRoom Code OAuth flow uses a `nonce` for exactly this.
- **Evil twin** — a rogue Wi-Fi access point with a legitimate-looking SSID.
- **VLAN hopping, DHCP starvation, rogue DHCP server** — LAN-layer attacks worth being able to name.

## 3.2 Defences
- **Firewall** — filters by rule. **Stateless** (per packet, in isolation), **stateful** (tracks connections, so return traffic for an established outbound connection is allowed automatically), **next-generation/application-layer** (inspects payloads and can block by application). A **WAF** specifically protects web applications from SQLi, XSS, and similar.
- **IDS vs IPS** — an Intrusion **Detection** System monitors and alerts (out of band, passive); an Intrusion **Prevention** System sits inline and blocks. Detection methods: **signature-based** (matches known patterns; cannot catch novel attacks) and **anomaly-based** (models normal behaviour and flags deviation; catches novel attacks but generates false positives). NetSpecter is a passive, signature-based detector.
- **Network segmentation and VLANs** — limit lateral movement after a breach. A DMZ isolates internet-facing services from the internal network.
- **VPN** — an encrypted tunnel; IPsec, OpenVPN, WireGuard.
- **NAC** — network access control, admitting devices only if they meet policy.
- **SIEM** — Security Information and Event Management: centralises logs from everything, correlates them, and alerts. Splunk, ELK, Wazuh. **SOAR** adds automated response.
- **Honeypot** — a deliberately exposed decoy system, used to detect and study attackers.

## 3.3 Wireless
WEP is broken (RC4 with a weak IV). WPA2 uses AES-CCMP and is still widely used but vulnerable to KRACK and to offline dictionary attacks on the 4-way handshake if the passphrase is weak. WPA3 adds SAE (Simultaneous Authentication of Equals), which gives forward secrecy and resists offline dictionary attacks. Always disable WPS.

---

# PART 4 — Application Security (OWASP Top 10)

Know these by name and be able to give one example and one fix for each.

**A01 — Broken Access Control.** The most common serious flaw. A user accesses data or functions they should not: changing an id in a URL to read someone else's record (IDOR), forced browsing to an admin page, missing checks on an API endpoint.
*My project example*: in ClassRoom Code, holding the `teacher` role is deliberately **not** enough to touch a course — the teacher must be *assigned* to that course, because courses are co-taught. And a request for a resource you have no relationship with returns **404, not 403**, so the API does not confirm what exists to people outside it.
*Fix*: deny by default, enforce authorisation server-side on every request, never trust a client-supplied identifier without checking ownership, and use unguessable identifiers (I use UUIDs).

**A02 — Cryptographic Failures.** Sensitive data transmitted or stored without adequate protection: plaintext HTTP, weak or obsolete algorithms, hard-coded keys, unsalted password hashes.
*My project example*: NetSpecter's entire purpose is finding this class of failure on a live network.
*Fix*: TLS everywhere with HSTS, AES-GCM at rest, Argon2/bcrypt for passwords, and a secrets manager rather than keys in code.

**A03 — Injection (SQLi, command injection, LDAP, NoSQL).** Untrusted input is interpreted as code.
```sql
-- vulnerable: string concatenation
"SELECT * FROM users WHERE name = '" + input + "'"
-- input:  ' OR '1'='1' --      →  authentication bypass
-- input:  '; DROP TABLE users; --

-- safe: parameterised query
"SELECT * FROM users WHERE name = ?"    with input passed as a bound parameter
```
*Fix, in priority order*: parameterised queries / prepared statements (the input is always data, never code), a well-used ORM, least-privilege database accounts, allow-list input validation, and stored procedures that do not themselves concatenate. **Escaping alone is not sufficient.** For command injection, avoid shelling out at all; if you must, pass an argument array rather than a shell string.
*NoSQL injection* is real too — passing `{"$ne": null}` as a password field in an unvalidated MongoDB query matches everything.

**A04 — Insecure Design.** Flaws in the architecture rather than the implementation — no rate limiting on a password reset, a business logic flaw, a missing trust boundary. Cannot be fixed by better coding; needs threat modelling during design.
*My project example*: the ClassRoom Code importer's rule that a model is never asked what the expected output is — only for a reference solution, which the platform then runs — is a design-level trust boundary, not a coding fix.

**A05 — Security Misconfiguration.** Default credentials, unnecessary features enabled, verbose error messages leaking stack traces, directory listing on, missing security headers, cloud storage buckets left public.
*My project example*: `DEV_LOGIN` and `ALLOW_LOCAL_EXECUTION` are refused when `NODE_ENV=production`, and the server refuses to boot without a real `DATABASE_URL` and `JWT_SECRET` rather than starting with insecure defaults.

**A06 — Vulnerable and Outdated Components.** Using a dependency with a known CVE. Log4Shell is the canonical example.
*Fix*: maintain an inventory (SBOM), automate scanning (`npm audit`, `pip-audit`, Dependabot), patch promptly, and remove unused dependencies.

**A07 — Identification and Authentication Failures.** Weak passwords permitted, no MFA, session ids in URLs, sessions that do not rotate on login, unlimited login attempts.
*Fix*: MFA, strong password policy checked against known-breached lists, account lockout or exponential backoff, secure session management, and session rotation on privilege change.

**A08 — Software and Data Integrity Failures.** Trusting code or data from an untrusted source without verification: unsigned updates, a compromised CI/CD pipeline, insecure deserialisation, a malicious npm package.
*This is the category ModelAuth lives in.* Verifying that the model serving your requests is the model you contracted for is a supply-chain integrity problem — the artefact is a set of weights and there is no hash to check.

**A09 — Security Logging and Monitoring Failures.** Attacks not detected because nothing is logged, or logs are not reviewed. The industry average dwell time before detection is measured in months.
*Fix*: log authentication events, access control failures, and input validation failures; centralise into a SIEM; alert on patterns; and protect the logs themselves from tampering. Never log secrets or full credentials.

**A10 — Server-Side Request Forgery (SSRF).** The server is tricked into making a request to an attacker-chosen URL — often to internal services or a cloud metadata endpoint (`169.254.169.254`) to steal credentials.
*Fix*: allow-list permitted destinations, block private IP ranges and link-local addresses, and do not follow redirects blindly.

## 4.1 XSS and CSRF — always asked, and always confused
**XSS (Cross-Site Scripting)** — an attacker gets *their* JavaScript to execute in *your* user's browser in the context of your site, so it can read the DOM, steal cookies, and act as the user.
- **Stored** — the payload is saved on the server (a comment) and served to every viewer. Worst.
- **Reflected** — the payload is in the request (a query parameter) and echoed back in the response; delivered via a crafted link.
- **DOM-based** — the vulnerability is entirely in client-side JavaScript writing untrusted data into the DOM.
- **Fixes**: context-aware output encoding (HTML, attribute, JavaScript, URL contexts each need different escaping), a **Content Security Policy** restricting which scripts may run, `HttpOnly` on session cookies so JavaScript cannot read them, avoiding `innerHTML` and `eval`, and using a framework that escapes by default — React escapes interpolated values automatically, which is why `dangerouslySetInnerHTML` is named that way.

**CSRF (Cross-Site Request Forgery)** — an attacker's site causes the victim's browser to send an *authenticated* request to your site, exploiting the fact that the browser attaches cookies automatically. The attacker cannot read the response; they just cause the action.
- **Fixes**: `SameSite=Lax` or `Strict` cookies (my ClassRoom Code sessions use `SameSite=Lax`), anti-CSRF synchroniser tokens, checking the Origin/Referer header, and requiring re-authentication for sensitive actions.

**The one-line distinction**: XSS abuses the user's trust in the site; CSRF abuses the site's trust in the user's browser. XSS lets the attacker read the response; CSRF only lets them trigger the action.

## 4.2 Secure development practices
Threat model during design (STRIDE: Spoofing, Tampering, Repudiation, Information disclosure, Denial of service, Elevation of privilege). Validate all input on the **server**, using an allow-list. Encode all output for its context. Use parameterised queries. Store secrets outside the code — my repositories ship a `.env.example` with placeholders and gitignore the real `.env`. Apply least privilege everywhere. Keep dependencies patched. Use security headers: `Content-Security-Policy`, `Strict-Transport-Security`, `X-Content-Type-Options: nosniff`, `X-Frame-Options`, `Referrer-Policy`. Add static analysis (SAST) and dependency scanning to CI. Do code review with security in mind. Log security events.

**If a secret is committed to Git, changing it in a later commit is not enough** — it is still in history and must be treated as compromised. Rotate the credential first, then scrub history if needed.

---

# PART 5 — System and Endpoint Security

- **Malware types**: virus (attaches to a host file, needs execution), worm (self-propagating over a network), trojan (disguised as something legitimate), ransomware (encrypts data and demands payment), spyware, keylogger, rootkit (hides itself at a deep level, often kernel-mode), botnet, logic bomb, fileless malware (lives in memory and in legitimate tools).
- **Privilege escalation**: **vertical** (a normal user becomes root) and **horizontal** (a user accesses another user's data at the same level). Linux vectors: misconfigured **setuid** binaries, writable `PATH` directories, sudo misconfiguration, kernel exploits, cron jobs running as root with writable scripts.
- **Buffer overflow** — writing past the end of a buffer to overwrite adjacent memory, classically the return address on the stack, redirecting execution. Mitigations: stack canaries (a guard value checked before return), **ASLR** (randomising memory layout so addresses cannot be predicted), **DEP/NX** (marking data pages non-executable), and using memory-safe languages. This is why C is both fast and dangerous, and why Rust exists.
- **Hardening**: remove unnecessary services, close unused ports, disable default accounts, enforce strong authentication, apply patches, enable a host firewall, use SELinux or AppArmor for mandatory access control, encrypt disks, and enable audit logging.
- **Sandboxing** — restricting what a process can do. Linux **namespaces** (isolating the process, filesystem, network, and user views) plus **cgroups** (limiting CPU, memory, processes, and I/O) are what containers and Judge0's `isolate` are built on. My ClassRoom Code project uses Judge0 for exactly this reason, and its local fallback runner explicitly documents that it is *not* a sandbox.

---

# PART 6 — Identity and Access Management

- **Authentication factors**: something you know (password), have (token, phone), are (biometric). **MFA** combines two or more different categories — a password plus a security question is *not* MFA.
- **Access control models**: **DAC** (the owner sets permissions — standard Unix rwx), **MAC** (a system-wide policy the owner cannot override — SELinux, military classification), **RBAC** (permissions attach to roles, users get roles), **ABAC** (decisions from attributes of the user, resource, action, and environment — the most flexible).
- **OAuth 2.0 is authorisation, not authentication.** It lets an application obtain limited access to a resource on a user's behalf without the user handing over their password. **OpenID Connect** is the authentication layer built on top of it, adding the `id_token`.
  - The **Authorization Code flow** is the secure one: the client gets a short-lived code via the browser redirect and exchanges it for tokens over a back channel, so tokens never appear in the URL. **PKCE** protects public clients (mobile and single-page apps) that cannot keep a secret.
  - **`state`** protects against CSRF on the redirect; **`nonce`** protects against id-token replay. My ClassRoom Code implementation stores both in a short-lived signed cookie and validates them on the callback.
- **JWT** — header.payload.signature, base64url encoded. Signed, **not encrypted**: anyone holding one can read the payload. Verify the signature *and* the `alg` (reject `none` and reject an unexpected algorithm), check `exp`, `iss`, and `aud`. The fundamental weakness is that a stateless token cannot be revoked before it expires — which is why my project keeps sessions short and **re-reads the role from the database on every request** rather than trusting the claim in the token.
- **SSO** — one authentication grants access to many applications. SAML (XML, enterprise) and OIDC (JSON, modern).
- **Session management**: generate ids with a cryptographically secure random source, rotate on login and on privilege change, set an idle and an absolute timeout, invalidate on logout server-side, and set `Secure`, `HttpOnly`, and `SameSite` on the cookie.

---

# PART 7 — Digital Forensics and Incident Response

This connects directly to my stated hobby interest, so be ready to go deeper here than the average candidate.

## 7.1 The incident response lifecycle (NIST)
1. **Preparation** — tooling, playbooks, logging, training, contact lists. The phase that determines whether the rest goes well.
2. **Detection and Analysis** — identify that something happened, determine scope and severity, and triage.
3. **Containment** — short-term (isolate the host) and long-term (patch, rebuild) measures to stop the spread without destroying evidence.
4. **Eradication** — remove the attacker's access, malware, and persistence mechanisms.
5. **Recovery** — restore systems from known-good backups and monitor closely for reinfection.
6. **Lessons Learned** — a blameless post-incident review producing concrete changes.

## 7.2 Forensic principles
- **Order of volatility** — collect evidence from the most volatile first: CPU registers and cache, RAM, network state and connections, running processes, disk, then remote logs and archives. Pulling the plug destroys memory, which may hold the only copy of an encryption key or fileless malware.
- **Chain of custody** — a documented record of who handled the evidence, when, and why. Without it the evidence is inadmissible.
- **Write blockers and imaging** — never analyse the original. Take a bit-for-bit image, hash both the original and the image (SHA-256), and work only on the copy. If the hashes match, you can prove the copy is faithful and unaltered.
- **Locard's exchange principle** — every contact leaves a trace. This is the classical forensics idea that transfers directly to digital investigation: an intruder leaves artefacts even when trying not to. It is exactly the connection I find interesting between forensic science and security.
- **Anti-forensics** — timestomping, log deletion, encryption, steganography, and living off the land (using legitimate system tools so nothing unusual is installed).

## 7.3 Where the evidence is
Memory dumps (Volatility for analysis), disk images (Autopsy, Sleuth Kit), file system metadata and timestamps (MACB: modified, accessed, changed, birth), Windows Registry and Event Logs, Linux `/var/log`, `auth.log`, and shell history, browser history and cache, network captures (Wireshark, tcpdump — and the offline PCAP replay mode I built into NetSpecter v2 is exactly this use case), and cloud provider audit logs.

## 7.4 Frameworks to name
- **MITRE ATT&CK** — a knowledge base of adversary tactics and techniques, organised by phase (initial access, execution, persistence, privilege escalation, defence evasion, credential access, discovery, lateral movement, collection, exfiltration, command and control, impact). It is the common vocabulary for describing what an attacker did.
- **Cyber Kill Chain** (Lockheed Martin) — reconnaissance, weaponisation, delivery, exploitation, installation, command and control, actions on objectives. Simpler and more linear than ATT&CK.
- **NIST Cybersecurity Framework** — Identify, Protect, Detect, Respond, Recover.
- **CIA triad, STRIDE, DREAD** for threat modelling.
- **ISO 27001** for information security management systems.

---

# PART 8 — Offensive Security (Know the Concepts, Stay Ethical)

## 8.1 Penetration testing phases
1. **Reconnaissance** — passive (OSINT, WHOIS, certificate transparency logs, LinkedIn) and active (ping sweeps, DNS enumeration).
2. **Scanning and enumeration** — nmap for open ports and service versions, directory brute-forcing, banner grabbing.
3. **Vulnerability assessment** — Nessus, OpenVAS, Nikto; correlating findings against CVEs.
4. **Exploitation** — Metasploit, Burp Suite for web applications, custom scripts.
5. **Post-exploitation** — privilege escalation, persistence, lateral movement, data access.
6. **Reporting** — the actual deliverable: findings, evidence, business impact, and prioritised remediation.

**Types of engagement**: black box (no prior knowledge), white box (full source and architecture), grey box (partial). **Red team** simulates a real adversary against a defending **blue team**; **purple team** is the two working together.

**Vulnerability assessment vs penetration test**: an assessment enumerates weaknesses broadly, usually with automated tools; a pen test proves exploitability and demonstrates impact by actually chaining them together.

## 8.2 The ethical and legal line — state it unprompted
Every technique here is only legitimate with **explicit written authorisation** defining scope, timing, and rules of engagement. In India the Information Technology Act 2000, particularly sections 43 and 66, makes unauthorised access an offence regardless of intent or harm. "I was only testing" is not a defence. Legitimate places to practise are your own lab, deliberately vulnerable applications (DVWA, Juice Shop, Metasploitable), CTF competitions, and platforms like HackTheBox and TryHackMe, plus **bug bounty programmes** where the scope is published and permission is explicit. I run NetSpecter on my own machines against traffic I generate myself, and the README carries the authorisation warning.

## 8.3 Tools to be able to name
nmap (network and port scanning), Wireshark and tcpdump (packet analysis), Burp Suite and OWASP ZAP (web proxying and testing), Metasploit (exploitation framework), John the Ripper and Hashcat (password cracking), sqlmap (SQL injection), Nikto (web server scanning), Gobuster/ffuf (content discovery), Aircrack-ng (wireless), Autopsy and Volatility (forensics), Kali Linux (the distribution that packages most of them). And Scapy, which I used directly to build NetSpecter rather than using an existing tool.

---

# PART 9 — Questions They Will Ask, With Answers

**"Why cybersecurity?"** — See Part 0. Anchor it in NetSpecter and ModelAuth, and mention forensics.

**"What is the CIA triad?"** — Part 1.1, and add the trade-off point.

**"How would you secure a web application?"** — Answer in layers, and structure it as such. *Transport*: HTTPS everywhere with HSTS, and secure cookie flags. *Authentication*: MFA, Argon2 or bcrypt for passwords, rate limiting and lockout, secure session management with rotation. *Authorisation*: deny by default, enforce server-side on every request, resource-level checks not just role checks. *Input*: validate on the server with an allow-list, parameterised queries for every database access, context-aware output encoding, and a Content Security Policy. *Configuration*: no defaults, no verbose errors, security headers, dependencies patched. *Operations*: log security events centrally, monitor and alert, back up and test restores. *Process*: threat model during design, review code, run SAST and dependency scanning in CI.

**"You find a critical vulnerability in production on a Friday evening. What do you do?"** — Assess exploitability and blast radius first; a theoretical flaw and one being actively exploited are different situations. Escalate immediately to whoever owns the decision rather than acting alone. If it is being exploited, contain — disable the feature, add a WAF rule, or take the endpoint offline — while preserving evidence. Then fix, test, and deploy. Then check the logs for whether it was already exploited, and if user data was affected there are notification obligations. Finally, a blameless post-incident review asking how it got in and how the same class of bug gets caught earlier. The key points to convey: prioritise by real risk, communicate early, contain before eradicating, and preserve evidence.

**"Difference between symmetric and asymmetric encryption, and why does TLS use both?"** — Parts 2.1 and 2.2. Asymmetric solves key distribution but is slow; symmetric is fast but needs a shared key. TLS uses asymmetric crypto to agree on a symmetric key, then symmetric crypto for the actual data. Best of both.

**"How does HTTPS work?"** — Part 2.5, and mention what it does *not* protect.

**"What is SQL injection and how do you prevent it?"** — Part 4, A03. Lead with parameterised queries.

**"XSS vs CSRF?"** — Part 4.1. Give the one-line distinction.

**"How do you store passwords?"** — Part 2.4. Salt plus a slow KDF, and explain why fast hashes are wrong.

**"What is a zero-day?"** — A vulnerability with no available patch, sometimes unknown to the vendor. Dangerous because signature-based defences have nothing to match, so defence in depth and anomaly detection matter more.

**"How would you detect an intrusion?"** — Signature-based detection catches known patterns cheaply but misses novel attacks; anomaly-based detection models normal behaviour and flags deviation, catching novel attacks at the cost of false positives. In practice both, feeding into a SIEM that correlates across sources, with alerting tuned to precision rather than recall because analyst time is the scarce resource. My NetSpecter is a signature-based passive detector, and my ModelAuth is essentially an anomaly detector — so I have built one of each.

**"Explain a security decision you made in your own code."** — Best answers, pick one:
1. *Trust boundary around an LLM*: the ClassRoom Code importer never asks the model what a question's expected output is, because a wrong expected output marks correct students wrong and nobody notices. The model proposes a reference solution; the platform executes it against the real engine and takes the actual result. A question whose solution does not run gets no test cases rather than an invented one.
2. *Information disclosure*: returning 404 rather than 403 for resources you have no relationship with, so the API does not confirm what exists to outsiders.
3. *Refusing to fail open*: `ALLOW_LOCAL_EXECUTION` is rejected in production, and if Judge0 is unreachable production returns 503 rather than falling back to an unsandboxed runner. Failing loudly beats grading wrongly.
4. *Domain restriction anchoring*: matching the OAuth email domain on the full domain, so `college.edu.attacker.com` is rejected — a naive suffix check is a real vulnerability.

**"What security news have you followed?"** — Have one or two genuine, recent examples ready and be honest if you do not. Evergreen examples you can discuss with substance: **Log4Shell** (an unauthenticated remote code execution in a logging library, which showed how deep the software supply chain runs and how few organisations knew what they had deployed), **SolarWinds** (a compromised build pipeline shipping a signed backdoor to thousands of customers — the supply-chain attack that changed how people think about trusting vendors), and **Heartbleed** (a missing bounds check in OpenSSL leaking server memory including private keys, showing that critical infrastructure runs on under-resourced open source).

**"What do you not know yet?"** — Answer this honestly; it lands better than pretending. "I have not done cloud security seriously — IAM policies, cloud misconfiguration, and container security at scale. I have not worked with a SIEM in a real environment. And I have not done malware reverse engineering, which is the area I most want to learn next because it is closest to the forensics side that interests me."

**"How do you keep learning?"** — Building things is my main method — every one of my projects started from a question I could not answer by reading. Beyond that: CTFs and TryHackMe/HackTheBox for hands-on practice, the OWASP documentation, and following disclosed vulnerabilities to understand real root causes rather than categories.

---

# PART 10 — Rapid-Fire Definitions

- **Vulnerability / Exploit / Payload** — the weakness / the technique that uses it / what it delivers.
- **Attack vector** — the path in. **Attack surface** — the sum of all such paths.
- **Phishing** — deceptive message to steal credentials. **Spear phishing** — targeted at a specific person. **Whaling** — targeting an executive. **Vishing** (voice) and **smishing** (SMS).
- **Social engineering** — manipulating people rather than systems. The most effective attack vector, because people are not patchable.
- **Salting** — a unique random value per password to defeat rainbow tables.
- **Nonce** — a number used once, to prevent replay.
- **HSM** — a hardware security module storing keys so they never exist in general-purpose memory.
- **Air gap** — physically disconnected from any network.
- **Data at rest / in transit / in use** — stored, moving, and being processed. Each needs different protection; the third is the hardest and drives confidential computing.
- **RTO / RPO** — recovery time objective (how fast you must be back) and recovery point objective (how much data you can afford to lose).
- **3-2-1 backup rule** — three copies, on two media types, one off site.
- **Patch management** — the unglamorous control that prevents most real breaches.
- **Shadow IT** — unsanctioned tools employees use, which the security team cannot protect.
- **Principle of least astonishment** — a security control that behaves surprisingly will be worked around.
- **Security through obscurity** — hiding the design instead of securing it. Not a control on its own, though it can add a thin layer on top of real ones.
