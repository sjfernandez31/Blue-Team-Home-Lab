# Lab Setup Notes

## Final Lab Configuration
- **Attacker VM:** Kali Linux 2026.1 — VMware Workstation Pro 26H1
- **Defender VM:** Windows Server 2022 Standard — VMware Workstation Pro 26H1
- **Network:** LAN Segment (labnetwork) — isolated, no internet access
- **Kali IP:** 192.168.10.10
- **Windows Server IP:** 192.168.10.20


## Hypervisor Setup — What We Went Through

### Step 1 — Originally tried VirtualBox for both VMs
- Installed Oracle VirtualBox 7.2
- Kali Linux worked fine
- Windows Server 2022 would not boot — aborted every time

### Step 2 — Troubleshot VirtualBox issues
- Error: E_FAIL 0x80004005
- Cause: Windows 11 virtualization-based security (VBS) conflicting with VirtualBox
- Commands attempted:

bcdedit /set hypervisorlaunchtype off
bcdedit /set vsmlaunchtype off
Disable-WindowsOptionalFeature -Online -FeatureName VirtualMachinePlatform
Disable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V-All

- Changed Paravirtualization from Hyper-V to KVM in VirtualBox settings
- Nothing worked — Windows 11 memory integrity and VBS blocked VirtualBox

### Step 3 — Switched to VMware Workstation Pro
- Downloaded VMware Workstation Pro 26H1 free from broadcom.com
- Created Broadcom account to access download
- Installed and selected personal use (free license)
- Windows Server 2022 booted successfully on first try

### Step 4 — Moved Kali to VMware too
- Originally downloaded Kali as VirtualBox .7z image
- VirtualBox .vbox file could not be opened in VMware
- Downloaded Kali VMware version from kali.org/get-kali/#kali-virtual-machines
- Extracted using 7-Zip
- Opened .vmx file in VMware successfully


## Issues Encountered & Fixes

### VirtualBox Extension Pack Mismatch
- Error: VERR_VERSION_MISMATCH
- Fix: Uninstalled VirtualBox, downloaded fresh 7.2.10 installer from virtualbox.org
- Ran installer as Administrator

### Windows Server VM Aborted in VirtualBox
- Error: E_FAIL 0x80004005
- Root cause: Windows 11 VBS and memory integrity blocking VirtualBox
- Final fix: Switched entirely to VMware Workstation Pro

### VMware EFI Network Timeout on Windows Server
- VM tried to boot from network instead of ISO
- Fix: Removed Floppy device from VM settings
- Pressed any key immediately on boot to load from CD/DVD

### Windows Server License Error on First Boot
- Error: Windows cannot find Microsoft license terms
- Fix: Removed floppy/autoinst file, manually selected
  Windows Server 2022 Standard Evaluation (Desktop Experience) during setup

### Kali VMware Import Error
- Error: .vbox file is not a valid VMware configuration file
- Fix: Downloaded VMware-specific Kali image (.vmx) from kali.org


## Kali Linux Setup — Step by Step

1. Go to kali.org/get-kali/#kali-virtual-machines
2. Click VMware tab and download the 64-bit .7z file
3. Extract using 7-Zip (right click → open with 7-Zip → Extract)
4. Open VMware → Open a Virtual Machine → navigate to extracted folder
5. Select the .vmx file and click Open
6. Edit settings: RAM 4096 MB, CPUs 2
7. Network Adapter → LAN Segment → labnetwork
8. Power on → login with kali/kali
9. Open terminal and run:

sudo apt update && sudo apt upgrade -y
sudo apt autoremove -y

10. Set static IP:

sudo ifconfig eth0 192.168.10.10 netmask 255.255.255.0 up

11. Verify IP:

ip a


## Windows Server 2022 Setup — Step by Step

1. Go to microsoft.com/en-us/evalcenter/evaluate-windows-server-2022
2. Fill out form and download the ISO (SERVER_EVAL_x64FRE_en-us.iso)
3. Open VMware → Create a New Virtual Machine → Typical
4. Select the ISO file
5. Set name: Windows-Server-2022, save to D: drive
6. RAM: 4096 MB, CPUs: 2, Storage: 80 GB
7. Remove Floppy from hardware settings before booting
8. Power on → press any key to boot from ISO
9. Select: Windows Server 2022 Standard Evaluation (Desktop Experience)
10. Complete installation and set Administrator password
11. Network Adapter → LAN Segment → labnetwork
12. Set static IP:
    - Start → type ncpa.cpl → Enter
    - Right click adapter → Properties
    - IPv4 → Use following IP:
      - IP: 192.168.10.20
      - Subnet: 255.255.255.0
13. Turn off Windows Firewall for lab use:
    - Start → Windows Defender Firewall
    - Turn off for private and public networks

## Network Verification

After both VMs are configured, test connectivity from Kali:

ping 192.168.10.20

Successful replies confirm Kali and Windows Server can communicate.


## Commands Reference

| Command | Purpose |
|---------|---------|
| sudo apt update && sudo apt upgrade -y | Update Kali Linux |
| sudo apt autoremove -y | Clean up after update |
| sudo ifconfig eth0 192.168.10.10 netmask 255.255.255.0 up | Set Kali static IP |
| ip a | Verify IP address |
| ping 192.168.10.20 | Test connectivity to Windows Server |
| bcdedit /set hypervisorlaunchtype off | Disable Hyper-V (attempted fix) |
| bcdedit /set vsmlaunchtype off | Disable VSM (attempted fix) |
