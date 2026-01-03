# Control by CMD - Windows Command Line Guide

This guide covers essential Windows Command Prompt (CMD) commands for BitLocker analysis, disk management, and system preparation.

---

## Table of Contents

- [Prerequisites](#prerequisites)
- [Running CMD as Administrator](#running-cmd-as-administrator)
- [Disk Management Commands](#disk-management-commands)
- [BitLocker Commands](#bitlocker-commands)
- [TPM Commands](#tpm-commands)
- [USB Drive Management](#usb-drive-management)
- [Network Commands](#network-commands)
- [System Information](#system-information)
- [Troubleshooting](#troubleshooting)

---

## Prerequisites

### Check Your Windows Version
```cmd
winver
```

### Check System Information
```cmd
systeminfo
```

### Check if Running as Administrator
```cmd
whoami /priv
```

If you see `SeDebugPrivilege` and other privileges, you're running as admin.

---

## Running CMD as Administrator

### Method 1: Search Menu
1. Press `Win` key
2. Type `cmd`
3. Right-click **Command Prompt**
4. Select **"Run as administrator"**
5. Click **"Yes"** on UAC prompt

### Method 2: Run Dialog
1. Press `Win + R`
2. Type `cmd`
3. Press `Ctrl + Shift + Enter` (automatically runs as admin)

### Method 3: File Explorer
1. Navigate to `C:\Windows\System32`
2. Find `cmd.exe`
3. Right-click → **"Run as administrator"**

---

## Disk Management Commands

### List All Disks
```cmd
diskpart
list disk
```

**Example Output:**
```
  Disk ###  Status         Size     Free     Dyn  Gpt
  --------  -------------  -------  -------  ---  ---
  Disk 0    Online          238 GB      0 B        *
  Disk 1    Online           14 GB      0 B
  Disk 2    Online          931 GB      0 B        *
```

### Select a Disk
```cmd
select disk 1
```

*Replace `1` with your disk number*

### View Disk Details
```cmd
detail disk
```

### List Volumes
```cmd
list volume
```

### List Partitions
```cmd
list partition
```

### Clean a Disk (⚠️ ERASES ALL DATA)
```cmd
select disk 1
clean
```

### Remove Read-Only Attribute
```cmd
select disk 1
attributes disk clear readonly
```

### Exit Diskpart
```cmd
exit
```

---

## BitLocker Commands

### Check BitLocker Status
```cmd
manage-bde -status
```

**Example Output:**
```
BitLocker Drive Encryption: Configuration Tool version 10.0.19041
Copyright (C) 2013 Microsoft Corporation. All rights reserved.

Disk volumes that can be protected with BitLocker Drive Encryption:
Volume C: []
[OS Volume]

    Size:                 237.85 GB
    BitLocker Version:    2.0
    Conversion Status:    Fully Encrypted
    Percentage Encrypted: 100.0%
    Encryption Method:    XTS-AES 128
    Protection Status:    Protection On
    Lock Status:          Unlocked
    Identification Field: None
    Key Protectors:
        TPM
        Numerical Password
```

### Check BitLocker Status for Specific Drive
```cmd
manage-bde -status C:
```

### List Key Protectors
```cmd
manage-bde -protectors -get C:
```

### Enable BitLocker
```cmd
manage-bde -on C: -RecoveryPassword
```

### Disable BitLocker
```cmd
manage-bde -off C:
```

### Unlock Drive with Recovery Key
```cmd
manage-bde -unlock C: -RecoveryPassword [48-DIGIT-KEY]
```

**Example:**
```cmd
manage-bde -unlock C: -RecoveryPassword 123456-789012-345678-901234-567890-123456-789012-345678
```

### Backup Recovery Key to File
```cmd
manage-bde -protectors -get C: > C:\recovery-key.txt
```

### Add Recovery Password Protector
```cmd
manage-bde -protectors -add C: -RecoveryPassword
```

---

## TPM Commands

### Check TPM Status
```cmd
tpm.msc
```

*This opens the TPM Management console*

### Get TPM Information (PowerShell)
```cmd
powershell Get-Tpm
```

**Example Output:**
```
TpmPresent                : True
TpmReady                  : True
TpmEnabled                : True
TpmActivated              : True
TpmOwned                  : True
ManufacturerId            : 1229870147
ManufacturerVersion       : 7.62
ManagedAuthLevel          : Full
OwnerClearDisabled        : True
AutoProvisioning          : Enabled
```

### Clear TPM (⚠️ DANGEROUS - Requires Physical Presence)
```cmd
powershell Clear-Tpm
```

---

## USB Drive Management

### List All Drives
```cmd
wmic logicaldisk get name,description,volumename,size
```

### Identify USB Drives
```cmd
wmic diskdrive get model,interfacetype,index,size
```

### Format USB Drive (⚠️ ERASES ALL DATA)
```cmd
diskpart
list disk
select disk 1
clean
create partition primary
format fs=ntfs quick label="MyUSB"
assign letter=E
exit
```

### Remove USB Drive Letter (Unmount)
```cmd
diskpart
list volume
select volume 3
remove letter=E
exit
```

*Replace `3` with your volume number and `E` with your drive letter*

---

## Network Commands

### Check Network Configuration
```cmd
ipconfig /all
```

### Test Internet Connection
```cmd
ping google.com
```

### Download File (PowerShell)
```cmd
powershell Invoke-WebRequest -Uri "https://example.com/file.iso" -OutFile "C:\Downloads\file.iso"
```

---

## System Information

### Get Computer Name
```cmd
hostname
```

### Get Current User
```cmd
whoami
```

### Get System Manufacturer and Model
```cmd
wmic computersystem get manufacturer,model
```

### Get BIOS Information
```cmd
wmic bios get manufacturer,name,version,releasedate
```

### Check Windows Version Details
```cmd
ver
```

### Get Detailed System Info
```cmd
systeminfo | findstr /B /C:"OS Name" /C:"OS Version" /C:"System Manufacturer" /C:"System Model" /C:"BIOS Version"
```

### Check Disk Space
```cmd
wmic logicaldisk get name,freespace,size
```

---

## Troubleshooting

### "Access Denied" Error

**Solution:** Run CMD as Administrator
```cmd
# Right-click CMD → "Run as administrator"
```

### "The request is not supported" (BitLocker)

**Cause:** Drive not encrypted or BitLocker not available

**Solution:** Check Windows edition
```cmd
wmic os get caption
```

BitLocker requires Windows Pro, Enterprise, or Education.

### Can't Access Physical Drive

**Solution:** Disable drive automount
```cmd
diskpart
automount disable
exit
```

### USB Not Recognized

**Solution:** Refresh disk list
```cmd
diskpart
rescan
list disk
exit
```

### TPM Not Found

**Check if TPM is enabled in BIOS:**
1. Restart computer
2. Enter BIOS (F2, F10, Del, or Esc during boot)
3. Look for "Security" or "TPM" settings
4. Enable TPM
5. Save and exit

---

## Useful One-Liners

### Quick Disk Identification
```cmd
wmic diskdrive list brief
```

### Quick BitLocker Status Check
```cmd
manage-bde -status | findstr "Protection Status"
```

### Export All System Info to File
```cmd
systeminfo > C:\system-info.txt
```

### List Only Removable Drives
```cmd
wmic logicaldisk where "DriveType=2" get deviceid,volumename,size
```

### Check Free Space on C: Drive
```cmd
wmic logicaldisk where "DeviceID='C:'" get FreeSpace,Size
```

---

## Safety Warnings

### ⚠️ Commands That Can Cause Data Loss

These commands are **DANGEROUS** and will erase data:
```cmd
# DO NOT run these unless you're absolutely sure!
diskpart clean              # Erases entire disk
format                      # Erases partition
manage-bde -off            # Decrypts drive (vulnerable during process)
```

### ✅ Safe Commands (Read-Only)

These commands only **read** information and won't modify anything:
```cmd
diskpart list disk         # Just lists disks
manage-bde -status        # Just checks status
systeminfo                # Just displays info
wmic diskdrive list       # Just lists drives
```

---

## Tips & Best Practices

### 1. Always Verify Disk Numbers

Before running commands on disks, **triple-check** you have the right disk number:
```cmd
diskpart
list disk
detail disk
# Verify size and name match what you expect!
```

### 2. Save Recovery Keys

Before enabling BitLocker or making changes:
```cmd
manage-bde -protectors -get C: > C:\Users\%USERNAME%\Desktop\recovery-key.txt
```

### 3. Use Tab Completion

Press `Tab` to auto-complete file paths and commands in CMD.

### 4. View Command History

Press `↑` (up arrow) to cycle through previous commands.

### 5. Copy Output to Clipboard
```cmd
systeminfo | clip
```

Now paste (`Ctrl+V`) the output anywhere.

### 6. Pause Output for Long Results
```cmd
systeminfo | more
```

Press `Space` to scroll through results page by page.

---

## Quick Reference Card

| Task | Command |
|------|---------|
| List disks | `diskpart` → `list disk` |
| BitLocker status | `manage-bde -status` |
| TPM status | `powershell Get-Tpm` |
| System info | `systeminfo` |
| Check admin rights | `whoami /priv` |
| List USB drives | `wmic logicaldisk where "DriveType=2"` |
| Network config | `ipconfig /all` |
| BIOS info | `wmic bios get name,version` |

---

## Additional Resources

- [Microsoft BitLocker Documentation](https://docs.microsoft.com/en-us/windows/security/information-protection/bitlocker/)
- [Diskpart Command Reference](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/diskpart)
- [Windows Command-Line Reference](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/windows-commands)

---

## Need Help?

If you encounter issues:

1. Verify you're running CMD as Administrator
2. Check Windows version compatibility
3. Review error messages carefully
4. Search Microsoft documentation
5. Open an issue in this repository

---

**⚠️ Reminder:** Always have proper authorization before running these commands on any system. Unauthorized access is illegal.