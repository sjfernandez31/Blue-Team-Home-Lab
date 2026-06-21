# Lab Setup Notes

## Hypervisor Setup
- Originally installed VirtualBox 7.2 for both VMs
- Windows Server 2022 would not boot in VirtualBox due to Windows 11 
  virtualization-based security conflicts
- Switched to VMware Workstation Pro 26H1 for Windows Server 2022
- Kali Linux remains in VirtualBox

## Issues Encountered & Fixes

### VirtualBox Extension Pack Mismatch
- Error: VERR_VERSION_MISMATCH
- Fix: Uninstalled old VirtualBox, downloaded fresh installer from virtualbox.org, 
  ran as Administrator

### Windows Server VM Aborted in VirtualBox
- Error: E_FAIL 0x80004005
- Cause: Windows 11 virtualization-based security (VBS) conflicting with VirtualBox
- Commands run:
  bcdedit /set hypervisorlaunchtype off
  bcdedit /set vsmlaunchtype off
  Disable-WindowsOptionalFeature -Online -FeatureName VirtualMachinePlatform
- Final fix: Switched to VMware Workstation Pro

### VMware EFI Network Timeout
- Fix: Removed Floppy device from VM settings, booted from ISO manually

## Kali Linux Setup
- Downloaded from kali.org as .7z VirtualBox image
- Extracted using 7-Zip
- Imported .vbox file into VirtualBox
- RAM: 4096 MB, CPUs: 2
- Network: Internal Network (labnetwork)
- Updated with: sudo apt update && sudo apt upgrade -y
- Default credentials: kali/kali

## Windows Server 2022 Setup
- Downloaded eval ISO from Microsoft Evaluation Center
- Installed in VMware Workstation Pro
- Selected: Windows Server 2022 Standard Evaluation (Desktop Experience)
- RAM: 4096 MB, CPUs: 2, Storage: 80 GB
- Network: NAT (to be changed to Host-only for lab isolation)

## Commands Reference
| Command | Purpose |
|---------|---------|
| sudo apt update && sudo apt upgrade -y | Update Kali Linux |
| sudo apt autoremove -y | Clean up after update |
| bcdedit /set hypervisorlaunchtype off | Disable Hyper-V |
| bcdedit /set vsmlaunchtype off | Disable VSM |
