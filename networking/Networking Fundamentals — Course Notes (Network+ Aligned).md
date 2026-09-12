

Built to follow the 5-week course structure. Written from general networking/Network+ knowledge as a starting scaffold — meant to be expanded and corrected against what's actually covered in the uCertify labs and readings as I go through them, not a replacement for doing the labs themselves.

---

# Week 1: Network Concepts — Devices, Topologies, and the OSI Model

## Core building blocks

- **Workstation** — an end-user device (desktop, laptop) that people directly interact with to do their work.
- **Server** — a machine providing a resource or service to other machines on the network (files, authentication, web pages, DNS, etc.). Distinction from a workstation isn't hardware, it's _role_ — many "servers" are just regular computers configured to serve something.
- **Host** — the general term for _any_ addressable device on a network (workstations, servers, printers, phones, IoT devices — all hosts).

## Network media (how data physically/wirelessly travels)

- **Twisted pair copper (Ethernet cabling)** — most common wired LAN medium; categories (Cat5e, Cat6, Cat6a) differ mainly in max speed and distance before signal degrades.
- **Fiber optic** — uses light instead of electrical signal; much higher bandwidth and distance, immune to electromagnetic interference, more expensive and physically less flexible than copper.
- **Wireless (RF)** — no physical medium at all; trades reliability/ security convenience for interference susceptibility and shared-medium contention (everyone on the same channel competes for airtime).

## Common topologies

- **Star** — every device connects to a central point (switch/hub). Most common in modern LANs — failure of one device doesn't take down others, but the central point is a single point of failure.
- **Bus** — all devices share one single cable run. Largely historical now (old coax Ethernet); one break takes down the whole segment.
- **Ring** — each device connects to exactly two neighbors, forming a loop; data passes around the ring. Token Ring is the classic (now mostly obsolete) example.
- **Mesh** — devices interconnect directly with many/all others. Highly redundant, expensive to wire at scale — common conceptually in wireless mesh networks and in describing internet backbone routing.

## Common devices and what actually distinguishes them

- **Hub** — dumb, Layer 1 device; repeats every incoming signal out every port. No traffic intelligence at all — basically obsolete today.
- **Switch** — Layer 2 device; learns which MAC address lives on which port and forwards traffic only where it needs to go, instead of broadcasting everywhere like a hub. This is the actual backbone of modern wired LANs.
- **Router** — Layer 3 device; moves traffic _between_ different networks based on IP addressing, not just within one. This is the device making routing decisions, as opposed to a switch's simpler "which port is this MAC on" job.
- **Access Point (AP)** — bridges wireless clients into a wired network, functioning conceptually like a wireless "port" on a switch.

## The OSI Model — why it exists and how to actually use it

The OSI model is a **7-layer conceptual framework** for describing how data moves from an application on one machine to an application on another. It's not something devices literally implement layer-by-layer in practice (real-world networking is closer to the simpler 4-layer TCP/IP model) — its actual value is as a **shared vocabulary for troubleshooting and describing where in the stack a problem or protocol lives**.

|Layer|Name|What it's actually about|Example|
|---|---|---|---|
|7|Application|The actual software/protocol a user interacts with|HTTP, DNS, SMTP|
|6|Presentation|Data formatting/encoding/encryption|TLS encryption, character encoding|
|5|Session|Establishing/maintaining/tearing down a connection session|Session tokens, API session state|
|4|Transport|End-to-end delivery, reliability|TCP (reliable), UDP (fast, no guarantee)|
|3|Network|Logical addressing and routing between networks|IP addresses, routers|
|2|Data Link|Physical addressing within one local network segment|MAC addresses, switches|
|1|Physical|The actual raw bits/electrical signal/light/radio|Cabling, radio waves|

**The practical habit worth building now:** when something breaks, ask "which layer does this actually live at?" — a cable unplugged is Layer 1; a MAC address conflict is Layer 2; "I can ping the router but not the internet" is often Layer 3 (routing); "the website loads but times out loading images" might be Layer 7 (application-specific). This model is a diagnostic tool more than a literal architecture.

## IoT and miscellaneous connected devices

Printers, cameras, and IoT devices are all just additional _hosts_ on the network — the reason they get separate attention in security contexts is that they frequently ship with weak default credentials, infrequent firmware updates, and limited built-in security compared to a general- purpose computer, making them common weak points for lateral movement or botnet recruitment (e.g., Mirai-style IoT botnets).

---

# Week 2: TCP/IP Addressing, NAT, and Routing

## IP addressing basics

- An **IPv4 address** is a 32-bit number, written as four decimal octets (e.g., `192.168.1.10`).
- A **subnet mask** determines which portion of the address identifies the network versus the individual host — this is what lets a device determine "is this destination on my local network, or do I need to send this to my router?"
- **Private IP ranges** (`10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`) are reserved for internal networks and aren't routable on the public internet directly — this is exactly why NAT exists.

## NAT (Network Address Translation)

NAT lets many devices on a private internal network share a single public IP address when communicating externally. The router/firewall keeps a translation table mapping internal (private IP + port) combinations to the single public IP + a translated port, so return traffic gets routed back to the correct internal device.

**Why this matters beyond "IPv4 address conservation":** NAT incidentally acts as a basic security boundary — an external party can't directly initiate a connection to an internal, NAT'd device without the internal device (or an explicit port-forwarding rule) initiating that mapping first. This isn't a substitute for a real firewall, but it's a meaningful part of why home networks aren't trivially reachable from the open internet by default.

## IP troubleshooting — the logical order

1. **Physical connectivity** — is the cable plugged in / is Wi-Fi connected at all?
2. **Does the device have a valid IP address?** (`ipconfig` / `ifconfig` / `ip a`) — no address, or an autoconfigured `169.254.x.x` address (APIPA on Windows), means DHCP failed.
3. **Can it reach its default gateway?** (`ping <gateway IP>`) — confirms local network connectivity.
4. **Can it reach an external IP directly?** (`ping 8.8.8.8`) — confirms routing to the internet works, independent of DNS.
5. **Can it resolve a domain name?** (`nslookup` / `ping google.com`) — if step 4 works but this fails, the problem is specifically DNS, not general connectivity.

This top-down order matters because each step isolates a specific layer of possible failure — jumping straight to "DNS isn't working" without confirming basic IP connectivity first wastes time chasing the wrong layer.

## Routing basics and protocols

A router needs to know _which direction_ to send traffic for any given destination network — this is what a **routing table** stores. Routes get into that table either by manual configuration (**static routing**) or automatically via a **routing protocol** that routers use to share reachability information with each other.

- **Static routing** — simple, predictable, but doesn't adapt if a link goes down; fine for small/simple networks.
- **Dynamic routing protocols** — routers automatically discover and adapt routes:
    - **RIP (Routing Information Protocol)** — old, simple, uses hop count; mostly legacy today due to slow convergence and hop-count limitations.
    - **OSPF (Open Shortest Path First)** — modern, widely used interior routing protocol; calculates shortest path using link cost, converges fast, common in enterprise networks.
    - **BGP (Border Gateway Protocol)** — the protocol the actual internet runs on between different networks/organizations (autonomous systems); an entirely different scale and trust model from internal routing protocols.

---

# Week 3: Switching, VLANs, and Network Operations

## Layer 2 switching in more depth

A switch builds and maintains a **MAC address table** — mapping which MAC address has been seen on which physical port — by watching traffic as it passes through. This is what lets it forward frames intelligently instead of broadcasting to every port like a hub would.

## VLANs (Virtual LANs)

A VLAN logically segments a single physical switch (or set of switches) into multiple separate broadcast domains, as if they were physically separate networks — without needing separate physical hardware for each group.

**Why this matters practically:** an organization might put Accounting, Engineering, and Guest Wi-Fi on separate VLANs even though they're all plugged into the same physical switches — this limits broadcast traffic, and critically, **restricts what devices can talk to each other by default**, which is a real security boundary (a compromised guest-network device can't directly reach the Accounting VLAN without traffic explicitly being routed between them, typically through a firewall enforcing that boundary).

## Monitoring, metrics, and uptime/downtime

Organizations track network health through metrics like **uptime percentage**, **latency**, **packet loss**, and **bandwidth utilization** — this data feeds into both day-to-day troubleshooting and larger capacity- planning decisions. Monitoring tools (SNMP-based systems, dedicated network monitoring platforms) continuously poll devices for this data rather than relying on manual checks.

## Redundancy and fault tolerance

The goal is eliminating **single points of failure** — if one link, one switch, or one path fails, the network keeps functioning through an alternate path. Concepts here include:

- **Redundant links** between switches, with **Spanning Tree Protocol (STP)** preventing the loops that redundant Layer 2 links would otherwise create (multiple active paths between two switches causes broadcast storms without something disabling the extra paths until needed).
- **Redundant hardware** (dual power supplies, backup ISPs/links) for physical-layer resilience.

This connects directly to **disaster recovery** planning — redundancy is the technical mechanism, disaster recovery is the broader organizational plan for what happens (backups, failover procedures, recovery time objectives) when something fails despite redundancy.

---

# Week 4: Network Security Fundamentals

## Core security concepts

- **Threat** — anything that could cause harm (an attacker, a piece of malware, a natural disaster).
- **Vulnerability** — a weakness that could be exploited (unpatched software, weak password policy, exposed port).
- **Mitigation** — the control put in place to reduce risk from a vulnerability (patching, firewall rules, access controls).

## Wireless security

- **WPA3** (and before it, WPA2) — the current standard encryption/ authentication protocols for Wi-Fi; WEP is long obsolete and trivially broken.
- **Enterprise vs. Personal (PSK) mode** — Personal mode uses a single shared passphrase for everyone; Enterprise mode ties authentication to individual user credentials (via RADIUS/802.1X), which matters a lot for larger organizations wanting per-user accountability and easy individual revocation.
- **SSID broadcasting and hiding** — hiding an SSID is weak security through obscurity at best; it doesn't meaningfully stop a determined attacker (SSIDs are visible in the traffic itself to anyone actively monitoring), and mainly just reduces casual discovery.

## Securing IoT devices

Same theme from Week 1 — IoT devices frequently ship with weak/default credentials and infrequent updates. Practical mitigations: change default credentials immediately, isolate IoT devices on their own VLAN separate from sensitive systems, and disable unnecessary services/ports on the device itself where configurable.

## VPNs

A VPN creates an encrypted tunnel between two points over an otherwise untrusted network (like the public internet), so traffic between them is protected from anyone observing the network in between. Common use cases: remote employees securely reaching internal company resources, or site- to-site VPNs connecting two office locations' networks together securely over the internet instead of needing a dedicated private line.

## Physical security

Easy to overlook in a purely network-focused course, but genuinely foundational: if someone can physically access a switch, server, or network closet, most logical security controls become far easier to bypass (plugging in a rogue device, resetting hardware, physically stealing a drive). Locked server rooms, cable management preventing unauthorized physical taps, and controlled physical access are all part of a complete security posture, not just software/configuration controls.

---

# Week 5: Network Troubleshooting

## A general troubleshooting methodology (not just tool-specific steps)

1. **Identify the problem** — gather information, don't assume; ask what changed recently.
2. **Establish a theory of probable cause** — form a hypothesis based on symptoms.
3. **Test the theory** — confirm or rule it out before acting further.
4. **Establish a plan of action** — once the cause is confirmed, plan the fix, including any needed downtime/communication.
5. **Implement the fix**, then **verify full system functionality** — don't just confirm the original symptom is gone, check nothing else broke as a side effect.
6. **Document findings, actions, and outcomes** — this closes the loop and builds an institutional knowledge base for next time.

This structured approach (a general IT troubleshooting model, not networking-specific) matters because random guess-and-check wastes far more time than a disciplined process, especially under pressure during an actual outage.

## Common tools

- **`ping`** — basic reachability test (ICMP echo).
- **`traceroute` / `tracert`** — shows the path (hop by hop) traffic takes to a destination, useful for identifying _where_ along a route a problem is occurring.
- **`ipconfig` / `ifconfig` / `ip a`** — view local interface configuration (IP, subnet, gateway).
- **Packet capture tools (Wireshark, tcpdump)** — inspect actual traffic at the packet level when higher-level symptoms don't reveal the cause.
- **Network monitoring/SNMP tools** — ongoing visibility rather than point-in-time checks, catching degradation trends before they become full outages.

## Optimization considerations

Beyond fixing outright failures, network optimization involves managing **bandwidth allocation** (e.g., QoS — Quality of Service — prioritizing latency-sensitive traffic like voice/video over bulk downloads), monitoring for capacity trends before they become bottlenecks, and periodically reviewing whether the network's actual topology/hardware still matches current organizational needs as usage grows.

---

## Ongoing to-do as I go through the actual labs

- [ ]  Verify/expand OSI layer examples against what uCertify specifically covers
- [ ]  Add specific subnetting practice once that's covered in labs (CIDR notation, calculating usable host ranges)
- [ ]  Expand routing protocol section with actual configuration syntax once covered
- [ ]  Add specific wireless security lab findings (WPA3 config specifics, if covered hands-on)