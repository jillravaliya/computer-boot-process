# 🖥️ Linux Boot Process 
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

## 6. Init (systemd takes over)

- After the kernel finishes loading (step 5), it looks for the very first **userspace process** to run.  
- Traditionally this was `init`, but now on almost all modern Linux distros it’s **systemd**.  
- This process is always assigned **PID = 1**, because it’s the first process.  

### Responsibilities of systemd:
- Initialize the environment (set up what user-space needs).  
- Mount additional filesystems (like `/proc`, `/sys`, swap, network filesystems).  
- Start background services (**daemons**):  
  - Networking  
  - Printing  
  - Logging  
  - SSH server, etc.  
- Manage **targets (runlevels)** → earlier `init` used runlevels (0–6), now `systemd` uses targets:  
  - `multi-user.target` → like runlevel 3 (text-based login).  
  - `graphical.target` → like runlevel 5 (GUI login).  
  - `rescue.target` → single-user emergency mode.  
- Stays alive in the background → **all other processes are its children**.  

👉 **In short:** Kernel hands over to `systemd`, and `systemd` becomes the “master organizer” of user-space.  

---

## 7. Login Process

Now the system is ready to accept a user.

### Two types of login:

**1. Text Login (TTY)**  
- Provided by the `getty` program.  
- If you press `Ctrl + Alt + F1…F6`, you get **virtual terminals**.  
- You log in by typing your **username** and **password**.  
- After authentication (`/etc/passwd` and `/etc/shadow` are checked), it starts your **default shell** (usually Bash).  
- Useful for servers or troubleshooting (because you don’t need graphics).  

**2. GUI Login (Graphical Login Manager)**  
- Provided by a **Display Manager** (e.g., GDM, LightDM, SDDM).  
- Shows a graphical login screen.  
- Once authenticated, it launches your **desktop environment** (GNOME, KDE, XFCE, etc.).  
- GUI login still runs on top of text login (just automated and hidden).  

👉 **Key point:** GUI is optional, TTY is fundamental. Even if GUI fails, you can always log in via TTY.  

---

## 8. Shell

After login, you don’t directly interact with the kernel. You interact through the **shell**.  

- **Shell = command-line interpreter**.  
- Examples: **Bash** (Bourne Again Shell), **Zsh**, **Fish**.  

### Role of the shell:
1. Takes your command (e.g., `ls -l`).  
2. Parses it (splits into program + arguments).  
3. Asks the kernel (via system calls) to execute.  
4. Displays the result back to you.  

- The shell is **not the kernel**, but a **middleman** between user and kernel.  
- Analogy:  
  - Kernel = *engine of the car*.  
  - Shell = *steering wheel to control the engine*.  

### Why shell matters?
- Without a shell, you’d have to write machine code directly to talk to the kernel → impossible for normal users.  
- Shell provides:  
  - Commands  
  - Scripting power   

👉 This makes the system **usable and powerful** for both admins and developers.  

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


---

# 🖥️ From Power Button → Linux Desktop

---

## 🔹 Step 1. Firmware / VBIOS Init (0.0 – 0.5 sec)

**Who’s in charge?**  
- CPU microcode  
- Motherboard UEFI firmware  
- GPU’s VBIOS firmware  

**What happens?**  
- CPU wakes → executes UEFI firmware from flash ROM.  
- UEFI runs **POST** (Power-On Self-Test): checks memory, CPU, keyboard, GPU.  
- GPU wakes → its own **VBIOS firmware** runs.  
- VBIOS sets up a **framebuffer** = tiny VRAM block + dumb scanout to monitor.  
  - No 3D, no compositing.  
  - Just: “draw pixels at this resolution.”  
- Monitor EDID (spec sheet) read → firmware picks safe resolution (often 1024×768 or native).  

**What you see:** motherboard/UEFI logo or vendor splash.  

---

## 🔹 Step 2. Bootloader (GRUB/systemd-boot) (0.5 – 1.5 sec)

**Who’s in charge?**  
- UEFI firmware jumps into bootloader from EFI partition.  

**What happens?**  
- Bootloader (GRUB/systemd-boot) runs in VBIOS framebuffer.  
- Shows **simple text menu** or **bitmap logo** (still firmware graphics).  
- Lets user select kernel/OS.  

**What you see:** black screen with GRUB menu, or vendor logo + “press key to enter menu.”  

---

## 🔹 Step 3. Kernel Loads + KMS Takeover (1.5 – 2.5 sec)

**Who’s in charge?**  
- Linux kernel (just loaded into RAM).  

**What happens?**  
- Kernel probes PCIe bus → finds GPU.  
- Loads **DRM driver** (`amdgpu`, `nouveau`, `i915`, etc.).  
- **KMS (Kernel Mode Setting):**
  - Reads monitor EDID.  
  - Reprograms GPU pipeline: resolution, refresh, multiple monitors.  
  - Allocates VRAM buffers.  
- Firmware framebuffer → replaced by DRM framebuffer.  

**What you see:**  
- Brief flicker/blank → control switches firmware → kernel.  
- Higher-res Ubuntu splash screen (purple background, logo).  

---

## 🔹 Step 4. Early Userland (Plymouth Splash) (2.5 – 4.0 sec)

**Who’s in charge?**  
- **initramfs** + early `systemd` services.  

**What happens?**  
- initramfs mounts real root filesystem.  
- `plymouth` daemon runs → draws animated splash.  
- Still DRM framebuffer → only **2D drawing**.  

**What you see:** Ubuntu logo glowing / dots animation.  

---

## 🔹 Step 5. Display Manager (GDM, SDDM, LightDM) (4.0 – 6.0 sec)

**Who’s in charge?**  
- `systemd` → starts display manager service.  

**What happens?**  
- GDM starts **Wayland (or Xorg fallback)** server.  
- GPU acceleration enabled (DRM + Mesa/OpenGL/Vulkan).  
- **Compositor (Mutter)** begins:
  - Windows, cursor, vsync, rendering pipeline.  
- GDM shows **login greeter window** (mini GNOME session).  

**What you see:** graphical login screen (username, password).  

---

## 🔹 Step 6. Desktop Session (6.0 – 9.0 sec)

**Who’s in charge?**  
- `gnome-session` + user’s `systemd --user` + **Mutter** compositor.  

**What happens?**  
- After login: `gnome-session` starts user services.  
- Creates **cgroups** (CPU/mem sandbox).  
- Mutter + Wayland now handle all drawing:
  - Apps render into GPU buffers.  
  - Compositor assembles them.  
  - KMS scans out → monitor.  
- GNOME Shell overlays UI (top bar, dock, animations).  

**What you see:** full GNOME desktop, smooth animations, ready.  

---

## 🕒 Timing (SSD + Modern CPU/GPU, Ubuntu Case)

- **0.0–0.5s** → Firmware init (UEFI + VBIOS splash)  
- **0.5–1.5s** → GRUB menu  
- **1.5–2.5s** → Kernel loads GPU driver, KMS takeover (screen flicker)  
- **2.5–4.0s** → Plymouth splash animation  
- **4.0–6.0s** → GDM login screen  
- **6.0–9.0s** → Desktop session fully loaded  

---

## 💡 Big Picture

Ubuntu logo → login page feels “instant,”  
but really it’s a **relay race**:  

**Firmware → Bootloader → Kernel+KMS → Plymouth → Display Manager → GNOME Session.**

