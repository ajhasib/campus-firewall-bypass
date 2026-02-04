# 🔓 Campus Firewall Bypass & Secure DNS Sinkhole
**Project Status:** Active | **Type:** Network Security Research | **Stack:** Linux, Tailscale, Pi-hole

> ⚠️ **DISCLAIMER: FOR EDUCATIONAL & RESEARCH PURPOSES ONLY**
> This project is a Proof of Concept (PoC) designed to study network isolation vulnerabilities and privacy risks on public/university networks. It demonstrates how "Overlay Networks" (like Tailscale) can traverse restrictive NAT/Firewall rules. The author advocates for responsible network usage and is not liable for policy violations resulting from the deployment of this architecture.

---

## 🧪 Research Motivation: Why This Exists

### 1. The "Hostile Network" Problem
University and Public WiFi networks are often treated as "trusted" zones, but they are actually hostile environments.
* **The Zero-Day Threat:** On a shared campus network, a device is visible to thousands of other students. Without strict isolation, "Malvertising" (malicious ads) can compromise a device within minutes of connecting.
* **Privacy Leaks:** Standard ISPs and University routers log DNS queries, creating a permanent record of every site a student visits.

### 2. Why Commercial Solutions Failed
I evaluated standard market solutions before engineering this custom infrastructure. They were insufficient for this specific environment:

| Solution | Why it Failed in this Scenario |
| :--- | :--- |
| **AdGuard / Local VPN Apps** | Mobile OSs only allow **one** VPN connection at a time. Using AdGuard prevents me from connecting to my actual University Lab VPN or Work VPN. It also drains significant battery by processing traffic locally. |
| **Browser Extensions** | They only protect the web browser. They leave the OS, background apps, and games completely exposed to tracking and ads. |
| **Physical Pi-hole** | I could not plug a Raspberry Pi into the dorm Ethernet because **Client Isolation** (a security feature on Uni routers) blocks my phone from talking to the Pi, rendering it useless. |

---

## 💡 The Engineering Solution
To solve this, I treated the University Network as "Untrusted Transport" only. I engineered a **Hybrid-Cloud Bypass** using:

1.  **Virtualization:** Hosting the security core (Pi-hole) in a Linux VM inside my daily-driver laptop.
2.  **Tunneling:** Using **Tailscale (WireGuard Protocol)** to punch a hole through the University's NAT and Client Isolation, allowing my devices to communicate as if they were on a private LAN.
3.  **Split-Horizon DNS:** Routing *only* DNS requests through the tunnel to minimize latency, while keeping heavy file downloads on the local network.

---

## 🏗️ Architecture Diagram

```mermaid
graph TD
    %% Nodes
    Internet((Internet / Cloud))
    UniFirewall[University Firewall / NAT]
    
    subgraph "Encrypted Overlay Network (Tailscale)"
        Phone(Mobile Phone)
        Laptop(Windows Laptop)
        VM(Linux VM / Pi-hole)
    end

    %% Connections
    Phone -- "DNS Query (100.x.x.x)" --> VM
    Laptop -- "DNS Query (100.x.x.x)" --> VM
    VM -- "Filter Ads & Trackers" --> UniFirewall
    UniFirewall -- "Safe Traffic" --> Internet
    
    %% Styles
    style VM fill:#bbf,stroke:#333,stroke-width:2px
    style UniFirewall fill:#f9f,stroke:#333,stroke-width:2px,stroke-dasharray: 5 5

  ## ⚙️ Under the Hood: Technical Implementation

![Linux](https://img.shields.io/badge/OS-Linux-black?logo=linux) ![WireGuard](https://img.shields.io/badge/Protocol-WireGuard-red) ![Security](https://img.shields.io/badge/Firewall-UFW-green)

### 1. 🧠 The Core (Linux VM)
* **Hypervisor Environment:**  Kali Linux via VMware.
* **Resource Efficiency:** Tuned for minimal footprint (`1 vCPU`, `512MB RAM`) to execute silently in the background of the host machine.


### 2. 🚇 The Tunnel (Tailscale Mesh)
* **NAT Traversal:** Leveraged **DERP Relay Servers** to establish peer-to-peer connections even behind strict University CGNAT and Double-NAT layers.
* **MagicDNS:** Configured *Global Nameservers* to enforce a "Split-Horizon" DNS strategy:
    > **Note:** All connected clients (Android, Windows) are forced to use the VM's Tailscale IP (`100.x.x.x`) for DNS resolution, overriding local ISP/University defaults.

### 3. 🛑 The Engine (Pi-hole FTL)
* **Blocklists:** Aggregated **100,000+ domains** via StevenBlack & Spam404 lists.
* **Regex Filtering:** Custom filters implemented to target CNAME-cloaked tracking domains.
    * *Target:* Telemetry endpoints, Malvertising hosts, and Cross-site trackers.

---

## 📸 Proof of Concept

| **1. Traffic Analysis** |
| :---: |
| ![Dashboard] Dashboard.png |
| *Real-time blocking metrics* | 

---

## 🚀 Impact & Performance Results

* 🛡️ **Privacy Preserved**
    * **100k+** tracking domains and telemetry endpoints neutralized.
* ⚡ **100% Uptime**
    * Maintained connectivity on "hostile" networks where standard device-to-device communication is blocked via Client Isolation.
* 🔒 **Zero-Trust Security**
    * Successfully eliminated "malvertising" vectors without exposing open ports (`53` or `80`) to the public internet.

---

## 🔮 Roadmap & Future Improvements

- [ ] **Recursive DNS (Unbound):** Eliminate reliance on upstream providers (Google/Cloudflare) for true data sovereignty.
- [ ] **Containerization (Docker):** Migrate the VM infrastructure to a `docker-compose` stack for rapid deployment.
- [ ] **Observability Stack:** Implement **Grafana + Prometheus** for long-term data visualization and hardware monitoring.

---
*Maintained by HASIB*
