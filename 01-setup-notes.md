# Lab Setup Notes

## Final Lab Configuration
- **Attacker VM:** Kali Linux 2026.1 — VMware Workstation Pro 26H1
- **Defender VM:** Windows Server 2022 Standard — VMware Workstation Pro 26H1
- **Network:** LAN Segment (labnetwork) — isolated, no internet access
- **Kali IP:** 192.168.10.10
- **Windows Server IP:** 192.168.10.20

---

## Hypervisor Setup — What I Went Through

### Step 1 — I originally tried VirtualBox for both VMs
- Installed Oracle VirtualBox 7.2
- Kali Linux worked fine
- Windows Server 2022 would not boot — aborted every time

### Step 2 — I troubleshot VirtualBox issues
- Error: E_FAIL 0x80004005
- Cause: Windows 11 virtualization-based security (VBS) conflicting with VirtualBox

**Command 1 — Disable Hyper-V:**
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

- I changed Paravirtualization from Hyper-V to KVM in VirtualBox settings
- Nothing worked — Windows 11 memory integrity and VBS blocked VirtualBox

### Step 3 — I switched to VMware Workstation Pro
- Downloaded VMware Workstation Pro 26H1 free from broadcom.com
- Created a Broadcom account to access the download
- Installed and selected personal use (free license)
- Windows Server 2022 booted successfully on my first try

### Step 4 — I moved Kali to VMware
- Originally downloaded Kali as a VirtualBox .7z image
- The VirtualBox .vbox file could not be opened in VMware
- Downloaded the Kali VMware version from kali.org/get-kali/#kali-virtual-machines
- Extracted using 7-Zip
- Opened the .vmx file in VMware successfully

---

## Issues I Encountered & How I Fixed Them

### VirtualBox Extension Pack Mismatch
- Error: VERR_VERSION_MISMATCH
- Fix: Uninstalled VirtualBox, downloaded fresh 7.2.10 installer from virtualbox.org
- Ran installer as Administrator

### Windows Server VM Aborted in VirtualBox
- Error: E_FAIL 0x80004005
- Root cause: Windows 11 VBS and memory integrity blocking VirtualBox
- My fix: Switched entirely to VMware Workstation Pro

### VMware EFI Network Timeout on Windows Server
- VM tried to boot from network instead of ISO
- My fix: Removed Floppy device from VM settings
- Pressed any key immediately on boot to load from CD/DVD

### Windows Server License Error on First Boot
- Error: Windows cannot find Microsoft license terms
- My fix: Removed floppy/autoinst file, manually selected Windows Server 2022 Standard Evaluation (Desktop Experience) during setup

### Kali VMware Import Error
- Error: .vbox file is not a valid VMware configuration file
- My fix: Downloaded VMware-specific Kali image (.vmx) from kali.org

---

## How I Set Up Kali Linux — Step by Step

1. Go to kali.org/get-kali/#kali-virtual-machines
2. Click VMware tab and download the 64-bit .7z file
3. Extract using 7-Zip (right click the file → open with 7-Zip → Extract Here)
4. Open VMware → Open a Virtual Machine → navigate to extracted folder
5. Select the .vmx file and click Open
6. Click Edit virtual machine settings
7. Set Memory to 4096 MB
8. Set Processors to 2
9. Click Network Adapter → LAN Segment → select labnetwork → OK
10. Click Power on this virtual machine
11. Login with username: kali and password: kali
12. Right click desktop → Open Terminal

**Step 1 — Update Kali:**
```bash
sudo apt update && sudo apt upgrade -y
```

**Step 2 — Clean up leftover files:**
```bash
sudo apt autoremove -y
```

**Step 3 — Set static IP address:**
```bash
sudo ifconfig eth0 192.168.10.10 netmask 255.255.255.0 up
```

**Step 4 — Verify IP was applied:**
```bash
ip a
```

Look for 192.168.10.10 next to eth0 to confirm it worked.

---

## How I Set Up Windows Server 2022 — Step by Step

1. Go to microsoft.com/en-us/evalcenter/evaluate-windows-server-2022
2. Fill out the form and download the ISO file (SERVER_EVAL_x64FRE_en-us.iso)
3. Open VMware → Create a New Virtual Machine → select Typical → Next
4. Select Installer disc image file and browse to the ISO → Next
5. Leave product key blank, set name to Windows-Server-2022, save to D: drive → Next
6. Set disk size to 80 GB → Store virtual disk as a single file → Next
7. Click Customize Hardware
8. Set Memory to 4096 MB
9. Set Processors to 2
10. Click Finish
11. Before booting: click Edit virtual machine settings → select Floppy → click Remove → OK
12. Click Power on this virtual machine
13. Immediately click inside the VM window and press any key to boot from ISO
14. Select: Windows Server 2022 Standard Evaluation (Desktop Experience)
15. Accept license terms → Custom install → select the disk → Next
16. Wait for installation to complete and set my Administrator password
17. Login with the password I set
18. Click Edit virtual machine settings → Network Adapter → LAN Segment → select labnetwork → OK
19. Inside Windows Server: click Start → type ncpa.cpl → press Enter
20. Right click the network adapter → Properties
21. Double click Internet Protocol Version 4 (TCP/IPv4)
22. Select Use the following IP address and enter:

```
IP address:      192.168.10.20
Subnet mask:     255.255.255.0
Default gateway: (leave blank)
```

23. Click OK → OK
24. Turn off Windows Firewall for lab use:
    - Click Start → search Windows Defender Firewall → open it
    - Click Turn Windows Defender Firewall on or off
    - Turn off for both Private and Public networks
    - Click OK

---

## Network Verification

After both VMs were configured I tested connectivity from my Kali terminal:

**Test connectivity to Windows Server:**
```bash
ping 192.168.10.20
```

Replies coming back confirmed both VMs are connected and my lab is ready.

Press Ctrl + C to stop the ping.

---

## Commands Reference

| Command | What It Does |
|---------|-------------|
| `sudo apt update && sudo apt upgrade -y` | Update all Kali packages |
| `sudo apt autoremove -y` | Remove leftover install files |
| `sudo ifconfig eth0 192.168.10.10 netmask 255.255.255.0 up` | Set my Kali static IP |
| `ip a` | Check current IP addresses |
| `ping 192.168.10.20` | Test my connection to Windows Server |
| `bcdedit /set hypervisorlaunchtype off` | Disable Hyper-V boot (Windows PowerShell) |
| `bcdedit /set vsmlaunchtype off` | Disable VSM (Windows PowerShell) |
