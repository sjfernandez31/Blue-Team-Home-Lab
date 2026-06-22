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
## Initial Defender Investigation — What We Tried First

### What We Expected
After running the Nmap scan we went to Windows Event Viewer expecting to see the attacker's IP in the Security logs immediately.

### What We Actually Found
- Event IDs 4624 (Successful Logon) and 4672 (Special Privileges) were present
- No source IP address was visible in the 4624 events
- No Event ID 5156 or 5157 (network connection events) were showing up at all

### Why It Happened
Windows Server 2022 does **not** enable full network connection logging by default. Basic Nmap port scans do not fully authenticate so they don't always trigger clean logon events. The default audit policy is not sufficient to catch port scan activity.

### The Lesson
**Default Windows logging will miss a port scan entirely.** A SOC analyst relying only on default settings would have no idea the scan happened. This is why hardening audit policies is a critical part of server configuration.

### Fix Applied
Enabled advanced audit logging using auditpol — documented in the Defender Response section below.

## Defender Response — Windows Event Log Analysis

### Step 1 — Enable Connection Logging on Windows Server

Default Windows Server logging does not capture port scan activity. Run these commands in Command Prompt as Administrator:

**Enable Filtering Platform Connection logging:**
```cmd
auditpol /set /subcategory:"Filtering Platform Connection" /success:enable /failure:enable
```

**Verify it is enabled:**
```cmd
auditpol /get /subcategory:"Filtering Platform Connection"
```

Result should show: Success and Failure

### Step 2 — Re-run the Nmap Scan from Kali

```bash
nmap 192.168.10.20
```

### Step 3 — Check Windows Event Viewer

1. Open Event Viewer on Windows Server
2. Expand Windows Logs → Security
3. Click Filter Current Log
4. Filter for Event ID: 5156
5. Look for entries showing Source Address 192.168.10.10

### What We Found

Event ID 5156 entries confirmed the following:

| Field | Value |
|-------|-------|
| Event ID | 5156 |
| Source Address | 192.168.10.10 (Kali Linux) |
| Destination | 192.168.10.20 (Windows Server) |
| Application | System |
| Ports detected | 135, 138, 139, 445, 5985 |

### What This Means
- Every port Nmap probed was logged with the attacker's IP
- A SOC analyst monitoring these logs would immediately see the scan
- The source IP 192.168.10.10 would be flagged for investigation
- This is how real intrusion detection works at the network level

### Lessons Learned
- Default Windows logging is NOT enough — advanced audit policies must be enabled
- Port scans leave traces in 5156 events when proper logging is configured
- As a defender always check the source IP on 5156 events — multiple ports from one IP in a short time = port scan
- Enable Filtering Platform Connection logging on every Windows Server you manage

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
