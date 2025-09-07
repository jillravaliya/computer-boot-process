# 🖥️ Linux Boot Process (From Power On → Login Screen)

This document explains the **entire Linux boot process** step by step. Every stage is described with what it does, **where it lives** (ROM, RAM, SSD, etc.), and how it hands over control to the next stage.

---

## 1. Power On → CPU Looks for Firmware

* When you press the power button:

  * CPU is **blank** (RAM empty, no OS yet).
  * CPU fetches its very first instructions from **firmware stored in ROM/Flash chip** on the motherboard.
  * This firmware is either **BIOS (old)** or **UEFI (modern)**.

📍 **Stored in:** ROM/Flash chip (on motherboard).

---

## 2. Firmware Stage (BIOS/UEFI)

### Role:

* Perform **POST (Power-On Self-Test):** check CPU, RAM, keyboard, GPU.
* Initialize basic hardware.
* Locate a boot device (SSD, NVMe, USB, etc.) using boot order.
* Hand over control to the bootloader.

### Details:

* **BIOS/MBR method:**

  * Looks at first sector of disk → **MBR (512 bytes)**.
  * Finds stage 1 bootloader there.

* **UEFI method (modern):**

  * Uses a small area of non-volatile RAM (**NVRAM**) to store boot entries.
  * Reads the **EFI System Partition (ESP)** from SSD/NVMe.
  * Loads a UEFI application (usually **GRUB bootloader**) into RAM.

📍 **Stored in:**

* Firmware in ROM/Flash chip.
* Boot entries in **NVRAM**.
* Bootloader files in **EFI partition (on SSD/NVMe)**.

---

## 3. Bootloader (GRUB or similar)

### Role:

* Bootloader is the **middleman** between firmware and kernel.
* Resides in the **/boot** directory (on Linux root partition or EFI partition).
* Tasks:

  * Display splash/menu.
  * Let you choose kernel/OS.
  * Load **kernel** and **initramfs** into RAM.

📍 **Stored in:** EFI partition (UEFI) or /boot (on SSD/NVMe).  
📤 **Copied to:** RAM before execution.

👉 Think of the bootloader as a **bus driver 🚍** that picks up the kernel + initramfs from SSD and drops them into RAM.

---

## 4. Kernel Loads (Main Boss 👑)

### Role:

* Kernel = Core of OS.
* Bootloader loads the **compressed kernel image** from SSD into RAM.
* Kernel uncompresses itself inside RAM.
* Initializes memory management, CPU scheduling, and low-level drivers.

📍 **Stored in:** SSD (/boot/vmlinuz-…)
📤 **Copied to:** RAM, then uncompressed.

---

## 5. Initramfs (Temporary Root FS)

### Role:

* Along with kernel, bootloader also loads **initramfs** into RAM.
* Initramfs = **small temporary filesystem** used before the real root FS.
* Contains:

  * Drivers for storage controllers.
  * Tools to find and mount the actual root filesystem.
  * The `udev` system to detect hardware.

📍 **Stored in:** SSD (/boot/initramfs-…)  
📤 **Copied to:** RAM, mounted temporarily.

👉 Think of initramfs as a **toolbox 🧰** that helps the kernel stand on its own.

---

## 6. Mount Real Root Filesystem

### Role:

* Kernel uses initramfs to:

  * Detect SSD/NVMe.
  * Mount the real root filesystem (`/`).
* Once successful:

  * Initramfs is discarded from RAM (to free memory).
  * Control is handed over to `init` (first process).

📍 **Root FS stored in:** SSD/HDD/NVMe (/, /home, /usr, etc.).

---

## 7. Init/Systemd Stage

### Role:

* `init` (or modern `systemd`) is the **first userspace process**.
* Responsibilities:

  * Start all background services (daemons).
  * Mount additional filesystems (/home, /var, etc.).
  * Set up networking, logging, user sessions.
  * Launch the login screen or desktop.

📍 **Stored in:** SSD (/sbin/init or /lib/systemd/).  
📤 **Runs in:** RAM.

---

## 8. User Space (You Log In 🎉)

* Finally, system reaches a point where you:

  * Get a text login (tty) OR graphical login (GDM, SDDM, LightDM).
  * Enter username/password.
  * Desktop environment or shell starts.

📍 **All apps stored in:** SSD/HDD/NVMe (/usr/bin, /usr/lib).  
📤 **Executed from:** RAM.

---

# 🧠 Summary Flow (Memory + Storage)

1. **Firmware (ROM → CPU)** → wakes system.
2. **NVRAM** → remembers boot order.
3. **Bootloader (SSD → RAM)** → loads kernel + initramfs.
4. **Kernel (SSD → RAM)** → unpacks, initializes.
5. **Initramfs (SSD → RAM)** → temporary helpers.
6. **Root FS (SSD)** → mounted by kernel.
7. **Init/Systemd (SSD → RAM)** → starts services.
8. **User Space** → you log in.

---

# ⚡ Why SSD/NVMe = Faster Boot

* HDD (spinning disk) = slow seeks, ~100 MB/s.
* SSD (SATA) = no moving parts, ~500 MB/s.
* NVMe SSD = direct PCIe lanes, multi-GB/s bandwidth.
* Firmware (UEFI) can **directly talk to NVMe** → kernel/initramfs copied into RAM much faster.

---

# 🔑 Key Terms

* **ROM/Flash:** Permanent chip storing firmware (UEFI/BIOS).
* **NVRAM:** Small memory in firmware to store boot configs.
* **RAM:** Temporary workspace; everything must be loaded here to run.
* **SSD/NVMe:** Storage where kernel, initramfs, and root FS live.
* **Bootloader:** Middleman that transfers kernel/initramfs into RAM.
* **Kernel:** Core of Linux OS.
* **Initramfs:** Temporary root filesystem.
* **Systemd:** First process managing all services.

---

# 🔄 Complete Flow Recap

1. Power on → CPU runs firmware from **ROM (UEFI/BIOS)**.  
2. Firmware (BIOS/UEFI) runs POST, checks hardware, reads **NVRAM settings**.  
3. Firmware loads **bootloader** from SSD (MBR or EFI partition) → into RAM.  
4. Bootloader loads **kernel (compressed)** + **initramfs** → into RAM.  
5. Kernel uncompresses, initializes drivers, mounts temporary **initramfs**.  
6. Initramfs mounts the **real root filesystem** from SSD.  
7. Kernel executes **init/systemd (PID 1)**.  
8. Systemd launches services → login screen/desktop.  

✅ At this point → Linux is fully booted.
```
