# Ubuntu + Windows Dual Boot Best Practice

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![GitHub stars](https://img.shields.io/github/stars/charZZH/ubuntu-windows-dual-boot-guide.svg)](https://github.com/charZZH/ubuntu-windows-dual-boot-guide/stargazers)

[← Back to Home](README.md) | [中文版本](README-zh.md)

For **Ubuntu 26.04 LTS + Windows 11 Pro 25H2**, using separate EFI partitions to completely avoid Windows Feature Updates overwriting GRUB bootloader.

## 🎯 Core Principle

Windows Feature Updates always overwrite the shared EFI partition and delete GRUB. **Ubuntu and Windows must each have their own EFI partition** for complete isolation.

## ✅ Step 1: Prepare Space in Windows (Lossless)

1. Right-click "This PC" → Manage → Disk Management
2. Locate your Windows system drive (largest NTFS partition)
3. Right-click → "Shrink Volume", wait for system to calculate available space
4. Enter space to allocate for Ubuntu (recommended 100GB+), click "Shrink"

## ✅ Step 2: Create Ubuntu Installation USB

1. Download latest Ubuntu Desktop ISO
2. Use Rufus to burn to USB (partition type: GPT)
3. Restart computer, boot from USB in BIOS

## ✅ Step 3: Critical! Force Ubuntu to Create Separate EFI

⚠️ Skip this and Ubuntu will use Windows' EFI partition, which will be overwritten later!

1. After entering Ubuntu installer, **DO NOT click install yet**. Press `Ctrl+Alt+T` to open Terminal
2. Find your target disk:
   sudo fdisk -l
Example target disk: /dev/nvme0n1
3. Use parted to temporarily turn off boot flag on Windows EFI:
sudo parted /dev/nvme0n1
  • Type p to view partitions, find Windows EFI partition with boot flag (usually partition 1)
  • Type set 1 boot off to disable boot flag
  • Type p again to confirm boot flag is gone
  • Type q to exit
4. Close Terminal and start normal Ubuntu installation

## ✅ Step 4: Manual Partition Installation

1. At "Installation type" step, MUST select "Manual partitioning"
2. Find the "Free space" you created earlier, click "+"
3. Mount point: /, File system: ext4, click OK
4. ✅ You'll see Ubuntu automatically creates its own EFI partition. Continue installation.

## ✅ Step 5: Restore Boot After Installation (DO NOT reboot!)

1. When installation finishes and asks to reboot, DO NOT reboot yet. Open Terminal again.
2. Restore Windows EFI boot flag:
   sudo parted /dev/nvme0n1
  • Type p to view partitions
  • Type set 1 boot on to restore boot flag
  • Type p to confirm, type q to exit
3. Verify Ubuntu boot priority:
   efibootmgr -v
Make sure BootOrder has Ubuntu before Windows. Adjust with efibootmgr -o if needed.
4. Now you can reboot.

## ✅ Step 6: Make GRUB Detect Windows

Ubuntu disables os-prober by default. Enable it manually:

1. Install GRUB to Ubuntu's own EFI partition
   sudo grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=ubuntu --recheck
2. Edit GRUB config
   sudo nano /etc/default/grub
Find GRUB_DISABLE_OS_PROBER=false, remove the # in front. Press Ctrl+X → Y → Enter to save.
Then scan and update GRUB:
   sudo os-prober
   sudo grub-mkconfig -o /boot/grub/grub.cfg

## ✅ Step 7: Verify Dual Boot Works

After reboot, you'll see GRUB menu with "Ubuntu" and "Windows Boot Manager" options.

1. Boot into Ubuntu first
   • Open file manager, you should see Windows partition
   • Mount it, try reading/writing files to confirm permissions
2. Then boot into Windows
   • Windows boots normally
   • Confirm all files are present
   • Check Windows updates don't affect Ubuntu boot

## 🎯 Final Results

✅ **Boot completely isolated** - Windows updates never touch Ubuntu's EFI partition  
✅ **Systems independent** - Update either OS without breaking the other  
✅ **Data interoperable** - Read/write Windows partitions directly from Ubuntu  

## 🚨 Pitfalls to Avoid

1. **NEVER use shared EFI partition**: Windows 25H2 update 100% overwrites GRUB
2. Installation order: Windows first, then Ubuntu. Never the reverse.
3. Disable "Secure Boot" in BIOS - causes issues with many drivers (including ROCm)

---

**This guide is based on hands-on verification for Windows 11 25H2 + Ubuntu 26.04 (2026).**

If this helped you, please give a Star ⭐️!
