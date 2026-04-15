# Parallel USB Imaging for Large-Scale Deployment

**Subject:** OS Imaging Strategy for Mass USB Flash Drive Deployment  
**Method:** Linux-based Parallel Bit-Stream Cloning (`dcfldd`)

---

## Overview
To meet the requirement of imaging **600+ loaner laptops** (using 24 flash drives) efficiently, I implemented a Linux-based parallel imaging workflow. Standard imaging tools, such as Rufus or BalenaEtcher, typically process only one drive at a time. By utilizing the `dcfldd` utility, I transitioned to a **one-to-many** write model, allowing for up to **32 simultaneous USB writes** per workstation.

---

## 1. Tool Installation
The primary utility used is **`dcfldd`**, an enhanced version of the standard `dd` tool developed by the Department of Defense Computer Forensics Laboratory. It is designed for high-integrity imaging; unlike standard `dd`, it supports multiple simultaneous outputs. 

> **Note on Software Status:** This tool is legacy software (last official update in 2006) and is currently maintained by volunteers. Special thanks to the [Resurrecting Open Source Projects](https://github.com/resurrecting-open-source-projects) foundation on GitHub for keeping up-to-date versions in modern package managers.

### Supported Platforms

#### **Debian / Ubuntu**
```bash
sudo apt update && sudo apt install dcfldd
```
* **Source:** [Debian Package Tracker](https://packages.debian.org/trixie/dcfldd)

#### **Arch Linux**
```bash
yay -S dcfldd
```
* **Source:** [AUR Package](https://aur.archlinux.org/packages/dcfldd)

#### **Fedora**
```bash
sudo dnf install dcfldd
```
* **Source:** [Fedora Packages](https://packages.fedoraproject.org/pkgs/dcfldd/dcfldd/epel-8.html)

---

### ⚠️ Platform Warnings

> **MacOS (NOT RECOMMENDED)** > If necessary, use Homebrew: `brew install dcfldd`.  
> *Warning:* macOS automatically mounts drives upon connection, which can interfere with the raw bit-stream copy.

> **Windows (NOT RECOMMENDED)** > Use binaries from [SourceForge](https://dcfldd.sourceforge.net/) or build from the [GitHub source](https://github.com/resurrecting-open-source-projects/dcfldd).  
> *Warning:* Like macOS, Windows' auto-mounting and background file indexing can corrupt the target drive image if not properly ejected.

---

## 2. General Procedure
The process involves designating one "Master" USB drive as the source and cloning its bit structure directly to all target drives.

### Step A: Hardware Preparation
1.  **Source Identification:** Connect the "Master" drive (containing the prepped Windows image) to a dedicated high-speed port.
2.  **Target Connection:** Connect target drives via USB-C hubs and standard USB-A ports.
3.  **Device Verification:** Run the following command to verify device paths:
    ```bash
    lsblk -dno NAME,SIZE,MODEL
    ```

**Example Output:**
```text
❯ lsblk -dno NAME,SIZE,MODEL
sda      28.6G SanDisk 3.2Gen1  # MASTER DRIVE
sdb      28.7G SanDisk 3.2Gen1  # Target 1
sdc      28.6G SanDisk 3.2Gen1  # Target 2
sdd      28.6G SanDisk 3.2Gen1  # Target 3
sde      28.6G SanDisk 3.2Gen1  # Target 4
zram0    15.5G                  # SWAP MEMORY - DO NOT TOUCH
nvme0n1 953.9G SAMSUNG MZVL21   # BOOT DRIVE - DO NOT TOUCH
```

### Step B: Command Execution
Initiate the parallel write using this structure:
```bash
sudo dcfldd if=/dev/source_drive of=/dev/target1 of=/dev/target2 of=/dev/target3 ...
```
* **`if=` (Input File):** The source master drive path.
* **`of=` (Output File):** The destination drive paths. `dcfldd` forks the data stream to all targets simultaneously; writing to 10 drives takes roughly the same time as writing to one.

### Step C: Finalization (The 'Sync' Phase)
Once the transfer is complete, the Linux kernel may still hold data in the write cache. To ensure integrity, you **must** execute:
```bash
sync
```
Once the terminal prompt returns, the drives are physically flashed and safe for removal. Unlike Windows or macOS, there is no need to manually "unmount" as the drives were never mounted by the filesystem.

---

## 3. Key Efficiency Factors
* **Scalability:** Bypasses the overhead of graphical tools. Throughput scales based on the number of available USB controllers on the motherboard.
* **Integrity:** `dcfldd` provides real-time hashing and block-count reporting. Hardware failures on a specific stick are identified immediately without affecting the rest of the batch.
* **Time Savings:** By imaging in batches of nine (limited by my current hardware/dock configuration), I significantly reduced hands-on time. The theoretical maximum is limited only by the USB-A ports supported by the machine's USB controller.

---
**Verification:** Periodically test one drive from each batch on a target laptop to ensure the "Master" image remains viable and hardware integrity is maintained.
