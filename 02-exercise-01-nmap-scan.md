<div align="center">

# 🔍 EXERCISE 01 — NMAP NETWORK SCAN

![STATUS](https://img.shields.io/badge/STATUS-COMPLETE-brightgreen?style=for-the-badge)
![EXERCISE](https://img.shields.io/badge/EXERCISE-01-blue?style=for-the-badge)
![TOOL](https://img.shields.io/badge/TOOL-Nmap-red?style=for-the-badge)
![ATTACKER](https://img.shields.io/badge/ATTACKER-Kali%20Linux-purple?style=for-the-badge)
![DEFENDER](https://img.shields.io/badge/DEFENDER-Windows%20Server%202022-blue?style=for-the-badge)

</div>

---

[← Back to README](README.md)

---

## 🖥️ Lab Environment

| Component | Details |
|-----------|---------|
| Attacker VM | Kali Linux 2026.1 |
| Defender VM | Windows Server 2022 Standard |
| Attacker IP | 192.168.10.10 |
| Defender IP | 192.168.10.20 |
| Network | LAN Segment (labnetwork) — isolated |
| Tool Used | Nmap 7.98 |

---

## 📋 Background

Nmap (Network Mapper) is a free open-source tool used to discover hosts and services on a network. It is one of the first tools an attacker or penetration tester uses when targeting a system. By scanning for open ports and running services, an attacker can map out exactly what is exposed and what might be exploitable. As a blue team defender, I need to understand what Nmap reveals so I can detect it, respond to it, and lock down what should not be visible.

In this exercise I acted as the attacker first, running three Nmap scans against my Windows Server 2022 VM. I then switched to the defender role, investigated the Windows Security logs, and documented everything I found.

---

## ⚔️ Attacker Phase — Scans Performed

### Scan 1 — Basic Port Scan

I ran a basic Nmap scan to discover what ports were open on the target.

**Command:**
```bash
nmap 192.168.10.20
```

**Results:**
```
PORT      STATE  SERVICE
135/tcp   open   msrpc
139/tcp   open   netbios-ssn
445/tcp   open   microsoft-ds
5985/tcp  open   wsman
```

**What I found:**

| Port | Service | Risk Level | Notes |
|------|---------|------------|-------|
| 135/tcp | Microsoft RPC | Medium | Used for Windows service communication |
| 139/tcp | NetBIOS | Medium | Old file and printer sharing protocol, can leak system info |
| 445/tcp | SMB | High | File sharing protocol — target of WannaCry and EternalBlue exploits |
| 5985/tcp | WinRM | Medium | Windows Remote Management — allows remote command execution if credentials obtained |

---

### Scan 2 — Service Version Detection

I ran a version detection scan to identify exactly what software was running on each open port.

**Command:**
```bash
nmap -sV 192.168.10.20
```

**Results:**
```
PORT      STATE  SERVICE       VERSION
135/tcp   open   msrpc         Microsoft Windows RPC
139/tcp   open   netbios-ssn   Microsoft Windows netbios-ssn
445/tcp   open   microsoft-ds  ?
5985/tcp  open   http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)

Service Info: OS: Windows
```

**What I found:**
- I confirmed the target is running Windows
- Port 5985 is running WinRM over HTTP — a potential remote access vector if I had valid credentials
- SMB version could not be fully identified but the port is open and exposed

---

### Scan 3 — Vulnerability Scan

I ran a vulnerability scan using Nmap scripts to check for known exploits on the open ports.

**Command:**
```bash
nmap --script vuln 192.168.10.20
```

**Results:**
```
Host script results:
samba-vuln-cve-2012-1182: Could not negotiate a connection — ERROR
smb-vuln-ms10-054: false
smb-vuln-ms10-061: Could not negotiate a connection — ERROR
```

**What I found:**

| Check | Result | Meaning |
|-------|--------|---------|
| samba-vuln-cve-2012-1182 | ERROR | Could not test — SMB negotiation failed |
| smb-vuln-ms10-054 | FALSE | Not vulnerable to this exploit |
| smb-vuln-ms10-061 | ERROR | Could not test — SMB negotiation failed |

No critical vulnerabilities were confirmed. Windows Server 2022 is patched against older SMB exploits. However ports 445 and 5985 remain open and represent real attack surface that should be monitored and restricted.

---

## 🔑 Key Findings as an Attacker

1. The target is confirmed as a Windows machine
2. SMB is open on port 445 — a high value target for lateral movement and ransomware delivery
3. WinRM is open on port 5985 — if I obtained valid credentials I could execute commands remotely
4. NetBIOS on port 139 could leak machine name and workgroup information

---

## 🛡️ Defender Phase — Event Log Analysis

### What I Expected

After running the Nmap scan I went to Windows Event Viewer expecting to see the attacker IP address in the Security logs immediately.

### What I Actually Found

When I opened Event Viewer and checked the Security log I found the following:

- Event IDs 4624 (Successful Logon) and 4672 (Special Privileges) were present
- No source IP address was visible in the 4624 events
- No Event ID 5156 or 5157 network connection events were showing up at all

### Why It Happened

Windows Server 2022 does not enable full network connection logging by default. Basic Nmap port scans do not fully authenticate so they do not always trigger clean logon events. The default audit policy is not sufficient to catch port scan activity.

**The lesson:** Default Windows logging will miss a port scan entirely. A SOC analyst relying only on default settings would have no idea the scan happened. This is why hardening audit policies is a critical part of server configuration.

---

### Step 1 — I Enabled Advanced Audit Logging

I opened Command Prompt as Administrator on Windows Server and ran the following commands.

**Enable Filtering Platform Connection logging:**
```cmd
auditpol /set /subcategory:"Filtering Platform Connection" /success:enable /failure:enable
```

**Verify logging is enabled:**
```cmd
auditpol /get /subcategory:"Filtering Platform Connection"
```

The result confirmed Success and Failure logging was now active.

### Step 2 — I Re-ran the Nmap Scan from Kali

```bash
nmap 192.168.10.20
```

### Step 3 — I Checked Windows Event Viewer

1. I opened Event Viewer on Windows Server
2. I expanded Windows Logs and clicked Security
3. I clicked Filter Current Log on the right side
4. I filtered for Event ID 5156
5. I looked for entries showing Source Address 192.168.10.10

### What I Found

Event ID 5156 entries confirmed the following:

| Field | Value |
|-------|-------|
| Event ID | 5156 |
| Source Address | 192.168.10.10 (Kali Linux) |
| Destination | 192.168.10.20 (Windows Server) |
| Application | System |
| Ports Detected | 135, 138, 139, 445, 5985 |

Every port Nmap probed was logged with the attacker IP address. A SOC analyst monitoring these logs would immediately see the scan and flag 192.168.10.10 for investigation.

---

## ✅ Result

I successfully completed a full attack and defense cycle for network scanning. As the attacker I identified 4 open ports and confirmed the target OS. As the defender I discovered that default Windows logging is insufficient to detect port scans, enabled advanced audit logging, re-ran the scan, and captured the attacker IP in Event ID 5156 logs. All findings are documented with screenshots below.

---

## 📟 Commands Reference

| Command | Purpose |
|---------|---------|
| `nmap 192.168.10.20` | Basic port scan |
| `nmap -sV 192.168.10.20` | Service version detection |
| `nmap --script vuln 192.168.10.20` | Vulnerability scan |
| `auditpol /set /subcategory:"Filtering Platform Connection" /success:enable /failure:enable` | Enable connection logging |
| `auditpol /get /subcategory:"Filtering Platform Connection"` | Verify logging is enabled |

---

## 💡 Lessons Learned

- Open ports are attack surface — every open port is a potential entry point
- SMB on port 445 and WinRM on port 5985 are high value targets on Windows machines
- A vulnerability scan not finding issues does not mean the system is secure — it means known exploits were not confirmed
- Default Windows Server logging is insufficient — advanced audit policies must always be enabled
- Port scans leave traces in Event ID 5156 when proper logging is configured
- Multiple 5156 events from one source IP in a short time window is a clear sign of a port scan
- Always scan your own systems before an attacker does

---

## 📸 Screenshots

| Screenshot | Description |
|------------|-------------|
| ![BT Lab 02-1](screenshots/BT%20Lab%2002-1.png) | Basic Nmap scan results showing open ports |
| ![BT Lab 02-2](screenshots/BT%20Lab%2002-2.png) | Version detection scan results |
| ![BT Lab 02-3](screenshots/BT%20Lab%2002-3.png) | Vulnerability scan results |
| ![BT Lab 02-4](screenshots/BT%20Lab%2002-4.png) | Event Viewer filtered list of 5156 events |
| ![BT Lab 02-5](screenshots/BT%20Lab%2002-5.png) | Single 5156 event showing source IP 192.168.10.10 |
