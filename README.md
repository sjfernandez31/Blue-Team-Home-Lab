<div align="center">

# 🛡️ BLUE TEAM HOME LAB

![STATUS](https://img.shields.io/badge/STATUS-ACTIVE-brightgreen?style=for-the-badge)
![EXERCISES](https://img.shields.io/badge/EXERCISES-2%20COMPLETE-blue?style=for-the-badge)
![PLATFORM](https://img.shields.io/badge/PLATFORM-VMware%20Workstation%20Pro-lightgrey?style=for-the-badge)
![OS](https://img.shields.io/badge/OS-Kali%20Linux%20%7C%20Windows%20Server%202022-purple?style=for-the-badge)
![NETWORK](https://img.shields.io/badge/NETWORK-Isolated%20LAN-orange?style=for-the-badge)

*A hands-on cybersecurity home lab I built to simulate real-world attack and defense scenarios. I document my practice as a blue team defender; detecting, analyzing, containing, and responding to attacks in a controlled environment.*

</div>

---

## 🖥️ Lab Environment

| Component | Details |
|-----------|---------|
| Host OS | Windows 11 Home |
| Hypervisor | VMware Workstation Pro 26H1 |
| Attacker VM | Kali Linux 2026.1 |
| Defender VM | Windows Server 2022 Standard |
| Network | LAN Segment (labnetwork) — isolated, no internet access |
| Attacker IP | 192.168.10.10 |
| Defender IP | 192.168.10.20 |
| RAM Allocated | 4GB Kali / 4GB Windows Server |

---

## 🎯 Objectives

- Practice real-world threat detection and log analysis
- Simulate attacks from the attacker perspective and defend against them
- Build incident response skills using the NIST framework
- Document every finding in professional incident reports

---

## 🧰 Tools

| Tool | Purpose |
|------|---------|
| Nmap | Network scanning and enumeration |
| Hydra | Brute force attack simulation |
| Metasploit | Exploitation framework |
| Wireshark | Network traffic analysis |
| Windows Event Viewer | Log analysis and threat detection |
| PowerShell | Windows administration and scripting |
| Auditpol | Windows audit policy configuration |

---

## 📁 Exercises

| # | Exercise | Status |
|---|----------|--------|
| 01 | [Lab Setup — Environment Configuration](01-lab-setup.md) | ✅ Complete |
| 02 | [Nmap Network Scan](02-nmap-scan.md) | ✅ Complete |
| 03 | Hydra Brute Force Attack | ⏳ Pending |
| 04 | Wireshark Traffic Analysis | ⏳ Pending |
| 05 | Active Directory Attack and Defend | ⏳ Pending |
| 06 | Home SIEM with Wazuh | ⏳ Pending |

---

## 🧠 Skills I'm Building

| Skill | Tools / Methods |
|-------|----------------|
| Network Scanning | Nmap, port enumeration, service detection |
| Threat Detection | Windows Event Viewer, Event ID analysis |
| Log Analysis | Security logs, auditpol, filtering |
| Incident Response | NIST framework — Detect, Contain, Eradicate, Recover |
| Brute Force Defense | Hydra simulation, failed logon detection |
| Traffic Analysis | Wireshark, PCAP review |
| Documentation | GitHub, professional incident reports |

---

## 📂 Repository Structure

```
Blue-Team-Home-Lab/
├── 01-lab-setup.md
├── 02-nmap-scan.md
└── README.md
```

---

<div align="center">

*Built and maintained by Steven Fernandez — github.com/sjfernandez31*

</div>
