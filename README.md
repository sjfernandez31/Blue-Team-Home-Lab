<div align="center">

# 🛡️ BLUE TEAM HOME LAB

![STATUS](https://img.shields.io/badge/STATUS-ACTIVE-brightgreen?style=for-the-badge)
![EXERCISES](https://img.shields.io/badge/EXERCISES-10%20COMPLETE-informational?style=for-the-badge)
![PLATFORM](https://img.shields.io/badge/PLATFORM-VMware%20Workstation%20Pro-lightgrey?style=for-the-badge)
![OS](https://img.shields.io/badge/OS-Kali%20Linux%20%7C%20Windows%20Server%202022-purple?style=for-the-badge)
![NETWORK](https://img.shields.io/badge/NETWORK-Isolated%20LAN-orange?style=for-the-badge)

*A hands-on cybersecurity home lab I built to simulate real-world attack and defense scenarios. I document my practice as a blue team defender — detecting, analyzing, containing, and responding to attacks in a controlled environment.*

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
| Attacker IP | 192.168.10.101 |
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
| Wireshark | Network traffic analysis and PCAP review |
| netexec | SMB credential testing and AD enumeration |
| enum4linux | Active Directory domain enumeration |
| Windows Event Viewer | Log analysis and threat detection |
| PowerShell | Windows administration and scripting |
| Auditpol | Windows audit policy configuration |
| Wazuh | Open source SIEM and XDR |
| SigmaHQ | Detection rule writing and threat hunting |
| Honeypot | Deception-based threat detection |

---

## 📁 Exercises

| # | Exercise | Status |
|---|----------|--------|
| 01 | [Lab Setup — Environment Configuration](01-lab-setup.md) | ✅ Complete |
| 02 | [Nmap Network Scan](02-nmap-scan.md) | ✅ Complete |
| 03 | [Hydra Brute Force Attack](03-hydra-brute-force.md) | ✅ Complete |
| 04 | [Wireshark Traffic Analysis](04-wireshark-analysis.md) | ✅ Complete |
| 05 | [Active Directory Attack and Defend](05-ad-attack-defend.md) | ✅ Complete |
| 06 | [Home SIEM with Wazuh](06-siem-wazuh.md) | ✅ Complete |
| 07 | [SigmaHQ Detection Rules](07-sigmahq-rules.md) | ✅ Complete |
| 08 | [Honeypot Deployment and Analysis](08-honeypot.md) | ✅ Complete |
| 09 | PCAP Analysis | ✅ Complete |
| 10 | Incident Response Playbook | ✅ Complete |

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
| SIEM Operations | Wazuh log ingestion, alerting, dashboards |
| Detection Engineering | SigmaHQ rule writing and tuning |
| Deception Technology | Honeypot deployment and attacker tracking |
| AD Attack and Defense | enum4linux, netexec, account lockout analysis |
| Documentation | GitHub, professional incident reports |

---

## 📂 Repository Structure

```
Blue-Team-Home-Lab/
├── 01-lab-setup.md               ✅ Complete
├── 02-nmap-scan.md               ✅ Complete
├── 03-hydra-brute-force.md       ✅ Complete
├── 04-wireshark-analysis.md      ✅ Complete
├── 05-ad-attack-defend.md        ✅ Complete
├── 06-siem-wazuh.md              ✅ Complete
├── 07-sigmahq-rules.md           ✅ Complete
├── 08-honeypot.md                ✅ Complete
├── 09-pcap-analysis.md           ✅ Complete
├── 10-ir-playbook.md             ✅ Complete
└── README.md
```

---

## 🤝 Connect

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Steven%20Fernandez-0A66C2?style=for-the-badge&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/stevenjfernandez31)
[![GitHub](https://img.shields.io/badge/GitHub-sjfernandez31-181717?style=for-the-badge&logo=github&logoColor=white)](https://github.com/sjfernandez31)

---

<div align="center">

*Built and maintained by Steven Fernandez — github.com/sjfernandez31*

</div>
