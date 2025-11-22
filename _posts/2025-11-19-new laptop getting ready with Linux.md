---
layout: post
title: "Installing & Setting Up My New Linux Laptop: A Real Journey Through Dual-Boot, Partitions & Persistence"
date: 2025-11-25
description: "A complete walkthrough of dual-booting Ubuntu with Windows on a modern Dell NVMe laptop — from partition problems to corrupted USB drives and manual GParted rescue."
comments: true
giscus_comments: true
featured: true
---

Today’s entry is a special one — not about metagenomics, CAZymes, MAGs, or FRed models —  
but about something equally important for any computational biologist:

👉 **setting up a reliable Linux machine.**

If you’ve ever tried installing Ubuntu alongside Windows on a modern NVMe laptop  
(especially Dell Inspiron or XPS), you know the process can be…

**fascinating + frustrating + educational.**

Here’s how my day went.

---

# 🔧 Step 1 — Preparing Windows for Dual-Boot

Before touching Linux, I fixed Windows first.

### ✔ Disabled Fast Startup

Fast Startup locks the NTFS filesystem and blocks Linux installers from modifying disks.  
(For dual-boot, it _must_ be off.)

In my case, hibernation was already disabled — good news.

### ✔ Created Unallocated Space

Using **Windows Disk Management → Shrink Volume**, I freed:

**~1 TB of unallocated space**

This will become:

- `/` root
- `/home`
- swap

Everything looked good… **until things got interesting.**

---

# 💽 Step 2 — Creating the Bootable USB

(and discovering the importance of a _good_ USB drive)

I downloaded **Ubuntu 24.04.3 LTS** and flashed it.

### On Linux (my preferred method):

```bash
sudo dd if=ubuntu.iso of=/dev/sdb bs=4M status=progress oflag=sync
```

On Windows, I also recommend:

👉 Rufus

Select the ISO

Partition scheme = GPT

Target = UEFI

File system = FAT32

Everything looked smooth — until the installer crashed mid-way with:

❌ curtin command install

❌ rsync error 23

❌ “System error detected. Installation failed.”

Root cause:
👉 My USB stick was corrupted.

A bad pendrive = a lot of wasted time.

I grabbed a new 64 GB USB, reflashed, rebooted…
and finally everything started working again.

🧭 Step 3 — Installer Could Not Recognize My Partitions
(The Tricky Part)

Inside the Ubuntu installer → Manual Installation, I saw:

p1 → EFI (FAT32)
p2 → Microsoft Reserved
p3 → Windows (NTFS)
p5 → WINRETOOLS

Free space → ~953 GB
But the installer did NOT show the “+” button to create new partitions.
Meaning: I could not proceed.

🔍 The Fix
Boot into Try Ubuntu, open GParted, and create the partitions manually.

🧱 Step 4 — Manual Partitioning in GParted
(The Correct Final Layout)

Here is exactly what I created:

1️⃣ Root (/)
105 GB
ext4
Mount point: /

2️⃣ Swap
I have 64 GB RAM, so:
17 GB swap
Type: linux-swap

3️⃣ Home (/home)
~902 GB
ext4
Mount point: /home

4️⃣ EFI (existing Windows EFI)
Never create a new EFI partition!
I reused:
/dev/nvme0n1p1 (612 MB FAT32)
Mount: /boot/efi
Do NOT format

📊 Final Partition Table
-Partition Type Mount Use
-nvme0n1p1 VFAT /boot/efi Windows + Ubuntu bootloader
-nvme0n1p4 ext4 / Ubuntu root
-nvme0n1p6 swap — Swap
-nvme0n1p7 ext4 /home User space
-nvme0n1p3 NTFS — Windows C: (untouched)

🚀 Step 5 — Running the Installer Again

I re-opened the Ubuntu installer.
Now the “Review Your Choices” screen looked exactly right:

✔ Correct EFI selected
✔ Correct root, swap, home
✔ No formatting Windows partitions
✔ Installing bootloader to nvme0n1

I clicked Install and…
🎉 It completed successfully!

🧵 Step 6 — First Boot & Setup

After reboot:
GRUB menu appeared (Ubuntu + Windows Boot Manager)
Ubuntu booted perfectly

/home and / mounted correctly
Swap working
System stable

The first software I installed:

VS Code
RStudio
Conda / Mamba
Git
Python packages (pandas, biopython, jupyter, etc.)
GCC, g++, make
Browsers & utilities
My system is finally ready for the real work.

🎉 Final Thoughts

This installation was a journey:
corrupted USB drives
missing partition options
EFI confusion
manual GParted fixes
several reinstalls

But in the end, I now have:

💻 A stable dual-boot Linux system
📦 Clean partitions optimized for bioinformatics
🧬 A fast environment ready for MAGs, CAZymes, MTX, and more

If you're a researcher considering dual-booting Linux, don’t worry if things break —it’s part of the learning process.

![welcome messege ubuntu 24.04](/assets/img/welcome_image.png)
