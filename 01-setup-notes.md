<div align="center">

# ⚙️ LAB SETUP — ENVIRONMENT CONFIGURATION

![STATUS](https://img.shields.io/badge/STATUS-COMPLETE-brightgreen?style=for-the-badge)
![PLATFORM](https://img.shields.io/badge/PLATFORM-VMware%20Workstation%20Pro-lightgrey?style=for-the-badge)
![ATTACKER](https://img.shields.io/badge/ATTACKER-Kali%20Linux%202026.1-red?style=for-the-badge)
![DEFENDER](https://img.shields.io/badge/DEFENDER-Windows%20Server%202022-blue?style=for-the-badge)
![NETWORK](https://img.shields.io/badge/NETWORK-Isolated%20LAN-orange?style=for-the-badge)

</div>

---

## 🖥️ Final Lab Configuration

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

## 📋 Background

This document covers the full setup process I went through to build my Blue Team Home Lab from scratch. I originally planned to run both virtual machines in VirtualBox but ran into conflicts with Windows 11 security features that prevented Windows Server 2022 from booting. I troubleshot every known fix before switching to VMware Workstation Pro, where everything worked on the first try. This document captures every step, every error, and every fix so I can repeat this process or help others do the same.

---

## 🔧 Hypervisor Setup — What I Went Through

### Step 1 — I Originally Tried VirtualBox

I installed Oracle VirtualBox 7.2 and successfully imported Kali Linux. However, every time I tried to boot Windows Server 2022 the VM aborted immediately with no display.

**Error received:**
```
E_FAIL (0x80004005) — The VM session was aborted
Component: SessionMachine
```

### Step 2 — I Troubleshot the VirtualBox Issues

I identified the cause as Windows 11 virtualization-based security (VBS) conflicting with VirtualBox. I ran the following commands in PowerShell as Administrator to attempt to resolve it.

**Command 1 — Disable Hyper-V launch:**
```bash
bcdedit /set hypervisorlaunchtype off
```

**Command 2 — Disable VSM:**
```bash
bcdedit /set vsmlaunchtype off
```

**Command 3 — Disable Virtual Machine Platform:**
```bash
Disable-WindowsOptionalFeature -Online -FeatureName VirtualMachinePlatform
```

**Command 4 — Disable Hyper-V All:**
```bash
Disable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V-All
```

I also changed the Paravirtualization Interface from Hyper-V to KVM inside VirtualBox settings. None of these fixes worked. Windows 11 memory integrity and VBS continued to block VirtualBox from running Windows Server 2022.

### Step 3 — I Switched to VMware Workstation Pro

I downloaded VMware Workstation Pro 26H1 for free from broadcom.com after creating a Broadcom account. I installed it and selected the personal use free license. Windows Server 2022 booted successfully on my first attempt.

### Step 4 — I Moved Kali Linux to VMware

I originally downloaded Kali as a VirtualBox image (.vbox file) which VMware cannot open. I went back to kali.org and downloaded the VMware-specific version (.vmx file), extracted it with 7-Zip, and opened it in VMware successfully.

---

## 🐛 Issues I Encountered and How I Fixed Them

### Issue 1 — VirtualBox Extension Pack Version Mismatch
| | |
|--|--|
| **Error** | VERR_VERSION_MISMATCH |
| **Cause** | Extension Pack version did not match installed VirtualBox version |
| **Fix** | Uninstalled VirtualBox completely, downloaded fresh 7.2.10 installer from virtualbox.org, ran as Administrator |

### Issue 2 — Windows Server VM Aborted in VirtualBox
| | |
|--|--|
| **Error** | E_FAIL 0x80004005 |
| **Cause** | Windows 11 VBS and memory integrity blocking VirtualBox |
| **Fix** | Switched entirely to VMware Workstation Pro |

### Issue 3 — VMware EFI Network Timeout
| | |
|--|--|
| **Error** | EFI Network timeout on every boot attempt |
| **Cause** | VM tried to boot from network instead of ISO |
| **Fix** | Removed Floppy device from VM settings, pressed any key immediately on boot to force ISO boot |

### Issue 4 — Windows Server License Error on First Boot
| | |
|--|--|
| **Error** | Windows cannot find Microsoft license terms |
| **Cause** | Floppy autoinst file interfering with setup |
| **Fix** | Removed floppy device, manually selected Windows Server 2022 Standard Evaluation (Desktop Experience) during setup |

### Issue 5 — Kali VMware Import Error
| | |
|--|--|
| **Error** | .vbox file is not a valid VMware configuration file |
| **Cause** | Wrong image format downloaded |
| **Fix** | Downloaded VMware-specific Kali image (.vmx) from kali.org |

---

## 🐉 How I Set Up Kali Linux

1. I went to kali.org/get-kali/#kali-virtual-machines and clicked the VMware tab.
2. I downloaded the 64-bit .7z file.
3. I extracted it using 7-Zip by right clicking the file and selecting Extract Here.
4. I opened VMware, clicked Open a Virtual Machine, and navigated to the extracted folder.
5. I selected the .vmx file and clicked Open.
6. I clicked Edit virtual machine settings and set Memory to 4096 MB and Processors to 2.
7. I clicked Network Adapter, selected LAN Segment, chose labnetwork, and clicked OK.
8. I clicked Power on this virtual machine.
9. I logged in with username kali and password kali.
10. I right clicked the desktop and opened Terminal.
11. I updated Kali by running:

**Update all packages:**
```bash
sudo apt update && sudo apt upgrade -y
```

**Remove leftover install files:**
```bash
sudo apt autoremove -y
```

12. I set a static IP address so my lab network stays consistent:

**Set static IP:**
```bash
sudo ifconfig eth0 192.168.10.10 netmask 255.255.255.0 up
```

**Verify the IP was applied:**
```bash
ip a
```

I confirmed 192.168.10.10 appeared next to eth0.

---

## 🪟 How I Set Up Windows Server 2022

1. I went to microsoft.com/en-us/evalcenter/evaluate-windows-server-2022.
2. I filled out the form and downloaded the ISO file named SERVER_EVAL_x64FRE_en-us.iso.
3. I opened VMware and clicked Create a New Virtual Machine, selected Typical, and clicked Next.
4. I selected Installer disc image file, browsed to the ISO, and clicked Next.
5. I left the product key blank, named the VM Windows-Server-2022, saved it to my D: drive, and clicked Next.
6. I set the disk size to 80 GB, selected Store virtual disk as a single file, and clicked Next.
7. I clicked Customize Hardware, set Memory to 4096 MB and Processors to 2, then clicked Finish.
8. Before booting I clicked Edit virtual machine settings, selected Floppy, clicked Remove, and clicked OK.
9. I clicked Power on this virtual machine.
10. I immediately clicked inside the VM window and pressed any key to boot from the ISO.
11. I selected Windows Server 2022 Standard Evaluation (Desktop Experience).
12. I accepted the license terms, selected Custom install, selected the disk, and clicked Next.
13. I waited for installation to complete and set my Administrator password.
14. I logged in with the password I set.
15. I clicked Edit virtual machine settings, clicked Network Adapter, selected LAN Segment, chose labnetwork, and clicked OK.
16. Inside Windows Server I clicked Start, typed ncpa.cpl, and pressed Enter.
17. I right clicked the network adapter and clicked Properties.
18. I double clicked Internet Protocol Version 4 (TCP/IPv4).
19. I selected Use the following IP address and entered:

```
IP address:      192.168.10.20
Subnet mask:     255.255.255.0
Default gateway: (leave blank)
```

20. I clicked OK then OK again.
21. I turned off Windows Firewall for lab use by clicking Start, searching Windows Defender Firewall, clicking Turn Windows Defender Firewall on or off, turning off both Private and Public networks, and clicking OK.

---

## 🌐 Network Verification

After both VMs were configured I tested connectivity from my Kali terminal to confirm they could communicate.

**Ping Windows Server from Kali:**
```bash
ping 192.168.10.20
```

Replies confirmed both VMs were connected and my lab was ready. I pressed Ctrl and C to stop the ping.

---

## 📟 Commands Reference

| Command | What It Does |
|---------|-------------|
| `sudo apt update && sudo apt upgrade -y` | Update all Kali packages |
| `sudo apt autoremove -y` | Remove leftover install files |
| `sudo ifconfig eth0 192.168.10.10 netmask 255.255.255.0 up` | Set my Kali static IP |
| `ip a` | Check current IP addresses |
| `ping 192.168.10.20` | Test connectivity to Windows Server |
| `bcdedit /set hypervisorlaunchtype off` | Disable Hyper-V boot |
| `bcdedit /set vsmlaunchtype off` | Disable VSM |
| `Disable-WindowsOptionalFeature -Online -FeatureName VirtualMachinePlatform` | Disable VM Platform |
| `Disable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V-All` | Disable Hyper-V All |

---

## ✅ Result

My lab environment is fully configured and operational. Kali Linux at 192.168.10.10 and Windows Server 2022 at 192.168.10.20 are isolated on a private LAN segment with no internet access. Both VMs can communicate with each other and are ready for attack and defense exercises.

---

## 📸 Screenshots

| Screenshot | Description |
|------------|-------------|
| ![BT Lab 01-1](screenshots/BT%20Lab%2001-1.png) | VMware showing both VMs powered on |
| ![BT Lab 01-2](screenshots/BT%20Lab%2001-2.png) | Kali terminal showing static IP 192.168.10.10 |
| ![BT Lab 01-3](screenshots/BT%20Lab%2001-3.png) | Windows Server network adapter showing IP 192.168.10.20 |
| ![BT Lab 01-4](screenshots/BT%20Lab%2001-4.png) | Kali terminal showing successful ping to Windows Server |
| ![BT Lab 01-5](screenshots/BT%20Lab%2001-5.png) | Basic Nmap scan results |
| ![BT Lab 01-6](screenshots/BT%20Lab%2001-6.png) | Version scan results |
| ![BT Lab 01-7](screenshots/BT%20Lab%2001-7.png) | Vuln scan results |
| ![BT Lab 01-8](screenshots/BT%20Lab%2001-8.png) | Event Viewer filtered 5156 events list |
| ![BT Lab 01-9](screenshots/BT%20Lab%2001-9.png) | Single 5156 event showing source IP 192.168.10.10 |
