# ULINZI-PI

**A portable, Raspberry Pi 5–based penetration testing platform for Kenyan SMEs.**

ULINZI-PI ("ulinzi" — Swahili for *protection/defense*) integrates open-source and free-tier vulnerability scanning, exploit verification, and wireless assessment tools into a single, touchscreen-operated device, purpose-built to make routine network security assessment affordable and accessible for Small and Medium Enterprises.

---

## Why ULINZI-PI

SMEs increasingly rely on digital infrastructure but rarely have the budget, staffing, or technical depth for commercial penetration testing tools — and freely available open-source alternatives are typically fragmented, requiring independent installation and configuration of multiple tools per engagement. ULINZI-PI addresses this by bundling a layered, pre-configured toolchain into one low-cost, portable unit that an SME's general IT staff can operate without specialist security expertise.

This project is the artifact developed for the MSc Cybersecurity and Digital Forensics thesis *"Securing Kenyan SMEs: Development of ULINZI-PI, a Raspberry Pi Penetration Testing Solution"* (The Open University of Kenya), following a Design Science Research methodology.

## Hardware

| Component | Spec |
|---|---|
| Board | Raspberry Pi 5 |
| Processor | Broadcom BCM2712, quad-core Arm Cortex-A76 @ 2.4GHz |
| RAM | 8GB |
| Storage | High-speed MicroSD, 12GB+ |
| Connectivity | Dual-band 802.11ac WiFi, Bluetooth 5.0, Gigabit Ethernet |
| Wireless extension | Supplementary long-range, high-gain USB WiFi adapter (extends evil-twin/wireless engagement range beyond the onboard adapter) |
| Display | 7" touchscreen for on-device, real-time engagement monitoring |
| Power | 5V/5A USB-C |
| OS | Kali Linux ARM64 |

## Toolchain

ULINZI-PI orchestrates four primary engines, backed by the full Kali Linux toolset for manual/supplementary testing:

**Orchestrated / automated**
- **[OpenVAS](https://www.openvas.org/)** — primary vulnerability scanner. No host-count restriction, making it the scanner of record for larger SME networks.
- **[Nessus Essentials](https://www.tenable.com/products/nessus/nessus-essentials)** — secondary, cross-validating scanner. Free tier is capped at 16 hosts per activation; scans against larger sites are automatically batched.
- **[Exploit-DB](https://www.exploit-db.com/)** — cross-references confirmed CVEs from OpenVAS/Nessus against documented, verified exploit code to reduce false positives.
- **[Wifiphisher](https://github.com/wifiphisher/wifiphisher)** — automated evil-twin / rogue access point engagements, paired with the high-gain adapter for extended range.

**Manual / supplementary (via Kali Linux base)**
- **Nmap** + **Netdiscover** — network reconnaissance and host discovery
- **Burp Suite** + **Gobuster/dirb** — web application testing
- **Wifite** — wireless auditing and handshake capture ahead of Wifiphisher engagements

> **Known limitation:** WPA/WPA2 handshake cracking is not supported — the platform has no GPU acceleration, making offline brute-force attacks impractical within reasonable timeframes. ULINZI-PI is designed for routine vulnerability visibility, not exhaustive credential attacks.

## Architecture

```
┌─────────────────────────────────────────────┐
│              Touchscreen UI                  │
│         (engagement control + monitoring)    │
└───────────────────┬───────────────────────────┘
                     │
          ┌──────────▼──────────┐
          │  Python Orchestrator │
          └──────────┬──────────┘
                      │
     ┌────────────────┼────────────────┬───────────────┐
     ▼                ▼                ▼                ▼
 OpenVAS          Nessus            Exploit-DB       Wifiphisher
 (primary scan)   (batched,         (CVE / exploit   (evil-twin,
                   16-host cap)      verification)    high-gain adapter)
     │                │                │                │
     └────────────────┴───────┬────────┴────────────────┘
                               ▼
                    Consolidated PDF/HTML
                    vulnerability report
```

## Getting Started

### Prerequisites
- Raspberry Pi 5 (8GB recommended)
- Kali Linux ARM64 image flashed to MicroSD
- Nessus Essentials activation code (free, from [Tenable](https://www.tenable.com/products/nessus/nessus-essentials))
- A supplementary high-gain USB WiFi adapter (optional, recommended for wireless engagements)

### Setup

```bash
# Clone the repository
git clone https://github.com/mohsecurity254/ulinzi-pi.git
cd ulinzi-pi

# Install dependencies
sudo apt update && sudo apt install -y openvas nessus wifiphisher wifite netdiscover gobuster

# Initial OpenVAS feed synchronization (one-time setup — takes 6-8 hours on ARM storage)
sudo gvm-setup

# Configure Nessus Essentials with your activation code
sudo /opt/nessus/sbin/nessuscli fetch --register <ACTIVATION-CODE>
```

> **Tip:** Run the OpenVAS feed sync once during initial setup, not before every engagement — it's the single longest-running step on this hardware. Schedule incremental updates overnight instead.

This will:
1. Run host discovery (Nmap + Netdiscover)
2. Run OpenVAS scan across all discovered hosts
3. Run Nessus scan (auto-batched if host count exceeds 16)
4. Cross-reference confirmed CVEs against Exploit-DB
5. Compile a consolidated vulnerability report

For wireless engagements, run Wifite for handshake capture and auditing, then launch Wifiphisher for evil-twin/rogue-AP testing via the high-gain adapter interface.

## Performance (validated across 5 Kenyan SME sites, 10–50 nodes each)

| Metric | Result |
|---|---|
| Mean full-network scan duration | 41.6 minutes |
| Total vulnerabilities identified | 74 (7 Critical, 18 High, 35 Medium, 14 Low) |
| Repeat-scan consistency | 93.9% |
| Peak CPU utilization | 88.6% |
| Average RAM utilization | 3.2GB / 8GB |

## Roadmap

- [ ] Fully open-source-only mode (OpenVAS-only) to eliminate Nessus's 16-host ceiling
- [ ] Additional wireless hardware options for further evil-twin range extension
- [ ] Optional GPU-accelerated add-on module for offline credential attacks
- [ ] RFID and IoT-specific testing modules

## Ethical Use

ULINZI-PI is intended strictly for **authorized** penetration testing and vulnerability assessment. All testing must be conducted with explicit, written consent from the network owner and within an agreed scope. This tool is provided for educational, research, and legitimate security assessment purposes only. The authors accept no liability for unauthorized or malicious use.

## Citation

If you use ULINZI-PI in academic work, please cite:

```
Jattan, M. D. (2026). Securing Kenyan SMEs: Development of ULINZI-PI, a Raspberry Pi
Penetration Testing Solution [Master's thesis, The Open University of Kenya].
```

Why: ULINZI-PI's own orchestration code sits on top of tools that are themselves GPL-licensed — Wifiphisher and OpenVAS are both GPLv3, and much of the Kali toolset (Nmap, Aircrack-ng family, Gobuster) is GPL or GPL-compatible. If you distribute ULINZI-PI's code alongside or integrated with these tools, GPL-3.0 keeps you license-compatible and avoids the legal ambiguity of pairing a permissive license (MIT/Apache) with GPL dependencies. It also matches the spirit of the project — a copyleft license ensures that if someone builds on ULINZI-PI to make it more capable, those improvements stay open and available to the same SMEs and researchers the project was built for, rather than being closed off in a commercial fork.

Full text: see `LICENSE`.

## Acknowledgements

Developed as part of an MSc Cybersecurity and Digital Forensics thesis at The Open University of Kenya, under the supervision of Dr. Jeremiah Onunga.
