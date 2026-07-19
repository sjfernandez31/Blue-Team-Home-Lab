<div align="center">

# 📋 EXERCISE 10 — INCIDENT RESPONSE PLAYBOOK

![STATUS](https://img.shields.io/badge/STATUS-COMPLETE-brightgreen?style=for-the-badge&v=2)
![EXERCISE](https://img.shields.io/badge/EXERCISE-10-blue?style=for-the-badge&v=2)
![FRAMEWORK](https://img.shields.io/badge/FRAMEWORK-NIST_IR-FF6B35?style=for-the-badge&v=2)
![INCIDENT](https://img.shields.io/badge/INCIDENT-INC--2026--001-red?style=for-the-badge&v=2)
![SEVERITY](https://img.shields.io/badge/SEVERITY-HIGH-FF0000?style=for-the-badge&v=2)

</div>

---

[← Back to README](README.md)

---

# INCIDENT RESPONSE REPORT
## INC-2026-001 — RDP Brute Force Attack

---

## Incident Summary

| Field | Details |
|-------|---------|
| Incident ID | INC-2026-001 |
| Title | RDP Brute Force Attack Against Domain Controller |
| Severity | High |
| Status | Resolved |
| Date Detected | July 12, 2026 |
| Date Resolved | July 12, 2026 |
| Affected System | WIN-V1LIVRSI0HI.support.local (192.168.10.20) |
| Attacker IP | 192.168.10.101 |
| Attack Vector | Remote Desktop Protocol (RDP) port 3389 |
| Framework | NIST SP 800-61 Incident Response |
| Analyst | Steven Fernandez |

---

## 📋 Background

This document is the formal incident response report for INC-2026-001, an RDP brute force attack detected against the Windows Server 2022 domain controller in the support.local environment. The attack was carried out using the Hydra password spraying tool from a Kali Linux machine on the same network segment. The attack was detected through Windows Security Event Log monitoring and confirmed through PCAP analysis of network traffic captured during the incident.

This report follows the NIST SP 800-61 Incident Response framework covering all six phases: Preparation, Detection and Analysis, Containment, Eradication, Recovery, and Lessons Learned.

---

## Phase 1 — Preparation

Before this incident occurred, the following controls and monitoring capabilities were in place:

**Detection capabilities:**
- Windows Security Event Log auditing enabled on the domain controller
- Advanced Audit Policy configured via auditpol to capture Event ID 5156 (network connections), Event ID 4625 (failed logons), and Event ID 4740 (account lockouts)
- Wazuh SIEM agent deployed on Windows Server 2022 forwarding events to the Wazuh manager
- Network packet capture capability available via tcpdump on Kali Linux

**Access controls:**
- RDP enabled on the domain controller for administrative access
- Account lockout policy configured: lockout after 10 failed attempts, 15 minute lockout duration
- Administrator account protected with a complex password

**Gaps identified before the incident:**
- RDP was exposed on the internal network segment without IP restriction
- No alerting configured for multiple failed logon attempts before lockout threshold was reached
- No network-level detection for port scanning activity

---

## Phase 2 — Detection and Analysis

### Initial Detection

The incident was first detected through Windows Event Viewer on July 12, 2026 at 5:27 PM. A filter on Event ID 4625 revealed 20 failed logon attempts against the Administrator account all within a 30 second window, all originating from the same source IP.

**Event ID 4625 details:**
```
Account Name:    Administrator
Failure Reason:  Unknown user name or bad password
Source Address:  192.168.10.101
Logged:          7/12/2026 5:27 PM
Count:           20 events in under 30 seconds
```

20 failed logon events from one IP in 30 seconds is not a user mistyping their password. That is a tool.

---

### Network Analysis

A packet capture taken during the attack confirmed the source and nature of the activity. Analysis of `hydra-capture.pcap` revealed the following:

**Traffic summary:**
- 323 total packets captured
- 247 packets (76.5%) were RDP traffic on port 3389
- 14 separate TCP streams — each representing a new connection attempt
- Rapid TCP RST packets between each attempt — the network signature of a brute force tool cycling through passwords

**Protocol hierarchy:**
- TCP dominant throughout
- RDP layer visible with TLSv1.2 negotiation attempts
- Pattern consistent with Hydra or similar automated credential testing tool

**TCP stream analysis:**
- Each stream followed the same pattern: SYN, SYN-ACK, ACK (handshake), RDP negotiation attempt, RST (failed), repeat
- Total attack duration approximately 30 seconds from first packet to last

---

### Tool Identification

Based on the network evidence and log analysis, the attack tool was identified as **Hydra v9.6** — an open source password brute force utility. The specific indicators were:

- Rapid sequential connection attempts with consistent timing
- TCP RST after each failed attempt before opening a new stream
- Multiple parallel connection threads visible in the packet capture
- Attack completed in under 30 seconds for 12 password attempts

---

### Attack Timeline

| Time | Event |
|------|-------|
| 5:27:38 PM | First failed logon attempt detected (Event ID 4625) |
| 5:27:38 PM | Hydra begins cycling through password list |
| 5:27:40 PM | Multiple 4625 events accumulating rapidly |
| 5:27:42 PM | Last failed logon attempt logged |
| 5:27:42 PM | Attack completes — correct password found on attempt 11 |
| 5:51:24 AM (7/17) | Second attack — jsmith account locked out (Event ID 4740) |

---

### Scope Assessment

| Question | Finding |
|----------|---------|
| Was access gained? | Yes — Administrator password cracked on attempt 11 |
| Were other systems affected? | No — isolated lab network, single target |
| Was data exfiltrated? | No — attack was simulated in isolated environment |
| Were other accounts targeted? | Yes — jsmith targeted in subsequent attack (Exercise 05) |
| Was malware deployed? | No |
| How long was attacker active? | Under 30 seconds per attack session |

---

## Phase 3 — Containment

### Immediate Actions Taken

**Short-term containment:**

1. Identified attacker IP (192.168.10.101) from Event ID 4625 Source Address field and PCAP analysis
2. Confirmed the attack was from a single source on the internal network segment
3. Documented all failed logon events and timestamps before any log clearing could occur
4. Preserved the packet capture file as forensic evidence
5. Monitored for any successful logon events following the brute force attempts

**Account lockout as automatic containment:**

The existing account lockout policy acted as an automatic containment mechanism during the second attack on jsmith. After 10 failed attempts, the account was automatically locked, preventing further credential testing. Event ID 4740 confirmed the lockout.

**What would be done in a real environment:**

- Block 192.168.10.101 at the firewall immediately
- Disable RDP access from any IP not on the approved admin access list
- Force password reset on the Administrator account
- Notify security leadership and begin formal documentation

---

## Phase 4 — Eradication

### Root Cause

The root cause of this incident was a combination of two factors:

**Factor 1: RDP exposed without IP restriction**
Remote Desktop Protocol was enabled and accessible from any IP on the network segment. There was no firewall rule or network policy restricting which systems could initiate RDP connections to the domain controller. This allowed the attacker to reach port 3389 freely.

**Factor 2: Weak password policy relative to the threat**
While the Administrator password (Admin123!) met basic complexity requirements, it was short enough to appear in a common password wordlist. A 12-password wordlist was sufficient to crack it, meaning any attacker with a standard wordlist would have succeeded.

### Eradication Steps

1. Changed the Administrator password to a longer, randomly generated credential not present in any wordlist
2. Changed the jsmith account password and verified account was unlocked
3. Removed RDP access for the jsmith account (no business need)
4. Documented all attacker activity for the lessons learned phase

---

## Phase 5 — Recovery

### System Restoration

Since this was a controlled lab exercise with no actual damage, recovery focused on restoring the environment to a known good state:

1. Verified Administrator account was secure with new strong password
2. Confirmed jsmith account was unlocked and accessible with correct credentials
3. Verified Wazuh agent was still connected and forwarding events
4. Confirmed network connectivity between lab VMs was restored to normal
5. Cleared the honeypot log and reset the environment for future exercises

### Verification Steps

```powershell
Get-ADUser -Identity jsmith -Properties LockedOut
```

Result: `LockedOut: False` confirmed account restored.

```powershell
Get-ADUser -Identity Administrator -Properties PasswordLastSet
```

Confirmed password was reset after the incident.

---

## Phase 6 — Lessons Learned

### What Went Well

- The account lockout policy automatically stopped the second attack before credentials were found
- Advanced audit logging captured the attacker IP in Event ID 4625 and 4740
- PCAP analysis provided independent network evidence confirming the attack tool and timeline
- Wazuh SIEM detected the brute force activity in real time and generated alerts
- The Sigma detection rule written in Exercise 07 would have alerted on this attack within the first 30 seconds

### What Could Be Improved

| Gap | Recommended Fix |
|-----|----------------|
| RDP open to all IPs | Restrict RDP to specific admin workstation IPs via firewall rule |
| No real-time alerting before lockout | Configure SIEM alert on 5+ failed logons from same IP within 60 seconds |
| Short password on Administrator | Enforce 20+ character random password for privileged accounts |
| No MFA on RDP | Implement multi-factor authentication for all remote access |
| Attacker could enumerate users | Disable null sessions and restrict SMB enumeration |
| Single factor authentication | Implement Windows Hello for Business or equivalent |

### Recommendations

**Immediate (within 24 hours):**
- Restrict RDP access to approved IPs only
- Enforce minimum 20 character password for all privileged accounts
- Configure SIEM alerting on failed logon thresholds

**Short term (within 30 days):**
- Deploy multi-factor authentication for RDP
- Enable Windows Defender Credential Guard
- Implement privileged access workstations for admin tasks
- Deploy network-level detection for port scanning (Snort or Suricata)

**Long term (within 90 days):**
- Implement a formal privileged access management solution
- Conduct regular penetration testing of remote access infrastructure
- Establish a formal security awareness program covering password hygiene
- Review and harden all externally accessible services quarterly

---

## Evidence Inventory

| Evidence Item | Source | Preserved |
|--------------|--------|-----------|
| hydra-capture.pcap | tcpdump on Kali Linux | ✅ |
| Windows Security Event Log exports | Event Viewer on WIN-V1LIVRSI0HI | ✅ |
| Event ID 4625 screenshots (20 events) | Exercise 03 documentation | ✅ |
| Event ID 4625 source IP screenshot | Exercise 03 documentation | ✅ |
| Event ID 4740 account lockout screenshot | Exercise 05 documentation | ✅ |
| Wireshark PCAP analysis screenshots | Exercise 04, 09 documentation | ✅ |
| Wazuh SIEM alert screenshots | Exercise 06 documentation | ✅ |
| Sigma detection rules | Exercise 07 documentation | ✅ |
| Honeypot connection logs | Exercise 08 documentation | ✅ |

---

## Related Exercises

This incident response report draws on evidence and findings from across the entire Blue Team Home Lab:

| Exercise | Contribution to This Report |
|----------|---------------------------|
| Exercise 02 | Nmap scan detection, Event ID 5156 analysis |
| Exercise 03 | Primary incident, Event ID 4625 evidence |
| Exercise 04 | PCAP capture and initial Wireshark analysis |
| Exercise 05 | Second attack on jsmith, Event ID 4740 evidence |
| Exercise 06 | Wazuh SIEM real-time detection evidence |
| Exercise 07 | Sigma detection rules that would have caught this |
| Exercise 08 | Honeypot evidence of attacker reconnaissance |
| Exercise 09 | Deep forensic PCAP analysis and attack timeline |

---

## 💡 Key Takeaways

- Incident response is a process, not a single action. Following a framework like NIST SP 800-61 ensures nothing is missed and the investigation is reproducible
- Every piece of evidence matters. Logs, PCAPs, SIEM alerts, and honeypot data all told parts of the same story. Combined they gave a complete picture
- Automatic controls like account lockout policies buy time and reduce damage before a human analyst can respond
- The best time to prepare for an incident is before it happens. Logging, SIEM, and detection rules need to be in place before the attacker arrives
- Document everything in real time. Waiting until after the incident to write things down means details get lost
- The attacker was on the network for under 30 seconds per session. Without proper logging that attack would have been invisible

---

## 📄 Clean Report Format

The following is this incident report written in plain professional format — no markdown, no badges, no GitHub styling. This is how the same report would look as a formal document submitted to a manager, CISO, or client.

---

# INCIDENT RESPONSE REPORT
## INC-2026-001 — RDP Brute Force Attack Against Domain Controller

**Incident ID:** INC-2026-001
**Severity:** High
**Status:** Resolved
**Date Detected:** July 12, 2026
**Affected System:** WIN-V1LIVRSI0HI.support.local (192.168.10.20)
**Attacker IP:** 192.168.10.101
**Analyst:** Steven Fernandez

---

**Background**

On July 12, 2026 at 5:27 PM, a brute force attack was detected against the Remote Desktop Protocol service on the domain controller WIN-V1LIVRSI0HI. The attack originated from 192.168.10.101 and used an automated credential testing tool to attempt multiple passwords against the Administrator account in rapid succession. The attack was detected through Windows Security Event Log monitoring and confirmed through forensic analysis of network traffic captured during the incident.

**Detection**

Twenty Event ID 4625 failed logon attempts were logged against the Administrator account within a 30 second window, all from the same source IP. Twenty failed logons from one IP in 30 seconds is not a user mistyping their password. That is a tool. Network packet capture confirmed 323 packets were exchanged during the attack, 76.5% of which were RDP traffic on port 3389. Fourteen separate TCP streams were identified, each representing a new connection attempt, with rapid TCP reset packets between each one, the network signature of an automated brute force utility.

**Root Cause**

Two factors enabled this attack. First, RDP was accessible from any IP on the network segment with no firewall restriction limiting which systems could initiate connections to the domain controller. Second, the Administrator password was short enough to appear in a common password wordlist, meaning a 12-entry wordlist was sufficient to crack it.

**Containment**

The attacker IP was identified from Event ID 4625 log entries and the packet capture. All log entries and forensic evidence were preserved before any clearing could occur. The existing account lockout policy served as an automatic containment mechanism during a subsequent attack, locking the jsmith account after 10 failed attempts and preventing further credential testing.

**Eradication**

The Administrator password was changed to a long randomly generated credential not present in any known wordlist. The jsmith account password was also reset and the account was verified to be unlocked. RDP access was restricted and the attack surface was reduced.

**Recovery**

Both affected accounts were verified as secure and accessible with correct credentials. The Wazuh SIEM agent was confirmed still connected and forwarding events. Network connectivity between systems was verified normal.

**Recommendations**

RDP should be restricted to approved administrator IPs only via firewall rule. A SIEM alert should be configured to trigger on five or more failed logons from the same IP within 60 seconds, catching attacks before the lockout threshold is reached. All privileged accounts should enforce a minimum 20 character randomly generated password. Multi-factor authentication should be implemented for all remote access connections.

**Lessons Learned**

The account lockout policy automatically stopped the second attack before credentials were found, which shows that basic hardening controls work. However the first attack succeeded because the same protections were not applied consistently. The most important takeaway is that logging, SIEM, and detection rules need to be in place and tested before an attacker arrives, not after.

---

## 📁 Evidence References

All evidence collected during this incident is documented and preserved across the Blue Team Home Lab exercises. Each link below takes you directly to the relevant documentation and screenshots.

| Evidence | Source | Link |
|----------|--------|------|
| Event ID 4625 — 20 failed logon attempts | Exercise 03 | [Hydra Brute Force Attack](03-hydra-brute-force.md) |
| Attacker IP captured in Security log | Exercise 03 | [Hydra Brute Force Attack](03-hydra-brute-force.md) |
| PCAP capture — 323 packets, 14 TCP streams | Exercise 04 | [Wireshark Traffic Analysis](04-wireshark-analysis.md) |
| Event ID 4740 — jsmith account lockout | Exercise 05 | [Active Directory Attack and Defend](05-ad-attack-defend.md) |
| enum4linux domain enumeration | Exercise 05 | [Active Directory Attack and Defend](05-ad-attack-defend.md) |
| Wazuh SIEM real-time brute force alerts | Exercise 06 | [Home SIEM with Wazuh](06-siem-wazuh.md) |
| Sigma detection rules for this attack | Exercise 07 | [SigmaHQ Detection Rules](07-sigmahq-rules.md) |
| Honeypot reconnaissance capture | Exercise 08 | [Honeypot Deployment and Analysis](08-honeypot.md) |
| Deep forensic PCAP analysis and timeline | Exercise 09 | [PCAP Analysis](09-pcap-analysis.md) |
