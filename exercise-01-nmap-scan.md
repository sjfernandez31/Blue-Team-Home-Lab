# Exercise 01 — Nmap Network Scan

## Objective
Use Nmap from Kali Linux to scan Windows Server 2022 and identify open ports, running services, and potential vulnerabilities — simulating what an attacker would do on day one of a real engagement.

---

## Lab Environment
- **Attacker:** Kali Linux 2026.1 — 192.168.10.10
- **Target:** Windows Server 2022 — 192.168.10.20
- **Network:** LAN Segment (labnetwork) — isolated

---

## What is Nmap?
Nmap (Network Mapper) is a free open-source tool used to discover hosts and services on a network. It is used by attackers to find open doors on a target and by defenders to audit their own network for exposure.

---

## Scans Performed

### Scan 1 — Basic Port Scan

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

**What it means:**
| Port | Service | Risk |
|------|---------|------|
| 135/tcp | Microsoft RPC | Medium — used for Windows service communication |
| 139/tcp | NetBIOS | Medium — old file/printer sharing protocol, can leak info |
| 445/tcp | SMB | High — file sharing protocol, target of WannaCry/EternalBlue |
| 5985/tcp | WinRM | Medium — Windows Remote Management, allows remote command execution |

---

### Scan 2 — Service Version Detection

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

**What it means:**
- Confirmed target is running Windows
- Port 5985 is running WinRM over HTTP — potential remote access vector
- SMB version could not be fully identified

---

### Scan 3 — Vulnerability Scan

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

**What it means:**
| Check | Result | Meaning |
|-------|--------|---------|
| samba-vuln-cve-2012-1182 | ERROR | Could not test — SMB negotiation failed |
| smb-vuln-ms10-054 | FALSE | Not vulnerable to this exploit |
| smb-vuln-ms10-061 | ERROR | Could not test — SMB negotiation failed |

**Summary:** No critical vulnerabilities confirmed. Windows Server 2022 is patched against older SMB exploits. However open ports 445 and 5985 represent attack surface that should be monitored and restricted.

---

## Key Findings as an Attacker
1. Target is a Windows machine
2. SMB is open on port 445 — high value target for lateral movement
3. WinRM is open on port 5985 — if credentials are obtained, remote command execution is possible
4. NetBIOS on 139 could leak machine name and workgroup information

---

## Defender Response
*To be documented in next section — reviewing Windows Event Logs to see what the scan looked like from the defender side.*

---

## Commands Reference

| Command | Purpose |
|---------|---------|
| `nmap 192.168.10.20` | Basic port scan |
| `nmap -sV 192.168.10.20` | Service version detection |
| `nmap --script vuln 192.168.10.20` | Check for known vulnerabilities |

---

## Lessons Learned
- Open ports = attack surface. Every open port is a potential entry point.
- SMB (445) and WinRM (5985) are high value targets on Windows machines.
- A vulnerability scan not finding issues does not mean the system is secure — it means known exploits were not confirmed.
- Always scan your own systems before an attacker does.
