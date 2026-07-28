# Parallel USB Imaging for Large-Scale Deployment

**Subject:** OS Imaging Strategy for Mass USB Flash Drive Deployment
**Method:** Linux-based Parallel Bit-Stream Cloning (`dcfldd`)

---

## Overview

This procedure gives instructions to make OS images for more than 600 laptops. You will use 24 flash drives and a Linux-based parallel imaging procedure. Standard tools (for example: Rufus or BalenaEtcher) write data to one drive at a time. The `dcfldd` utility writes data to a maximum of 32 USB drives at the same time on one workstation.

---

## 1. Tool Installation

You must use the `dcfldd` utility. The Department of Defense Computer Forensics Laboratory made this utility. It is a modified version of the standard `dd` tool. It supports multiple outputs at the same time.

> **NOTE:** The `dcfldd` utility is legacy software (last official update in 2006). Volunteers maintain the utility. You can get the utility from the Resurrecting Open Source Projects foundation on GitHub.
> 
> 

### Supported Platforms

**Debian or Ubuntu**
To install the utility on Debian or Ubuntu, run this command:

```bash
sudo apt update && sudo apt install dcfldd

```

* **Source:** Debian Package Tracker.



**Arch Linux**
To install the utility on Arch Linux, run this command:

```bash
yay -S dcfldd

```

* **Source:** AUR Package.



**Fedora**
To install the utility on Fedora, run this command:

```bash
sudo dnf install dcfldd

```

* **Source:** Fedora Packages.



---

### Platform Warnings

> **CAUTION:** DO NOT USE MACOS. If you must use macOS, use Homebrew to install the utility: `brew install dcfldd`. macOS mounts drives automatically when you connect them. This can cause damage to the raw bit-stream copy.
> 
> 

> **CAUTION:** DO NOT USE WINDOWS. If you must use Windows, use binaries from SourceForge or compile from the GitHub source. Windows mounts drives automatically and makes file indexes in the background. This can cause damage to the target drive image if you do not eject the drive correctly.
> 
> 

---

## 2. General Procedure

In this procedure, you make one USB drive the master drive. You copy the data from the master drive directly to all target drives.

### Step A: Hardware Preparation

1. Connect the master drive to a high-speed USB port.


2. Connect the target drives to USB-C hubs and standard USB-A ports.


3. Run this command to make sure that the device paths are correct:


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

Start the parallel write procedure with this command structure:

```bash
sudo dcfldd if=/dev/source_drive of=/dev/target1 of=/dev/target2 of=/dev/target3 ...

```

* **`if=` (Input File):** This is the path for the master drive.


* **`of=` (Output File):** This is the path for the target drive. The `dcfldd` utility sends the data to all target drives at the same time. To write data to 10 drives requires the same time as to write data to one drive.



### Step C: Finalization (The Sync Phase)

When the transfer is complete, the Linux kernel can keep data in the write cache. To make sure that data has integrity, do these steps:

1. Run this command:


```bash
sync

```


2. Wait for the terminal prompt to show again.


3. Remove the drives.



> **NOTE:** It is not necessary to manually unmount the drives. The filesystem did not mount the drives.
> 
> 

---

## 3. Key Efficiency Factors

* **Scalability:** This procedure does not use graphical tools. The data transfer speed changes in relation to the number of available USB controllers on the motherboard.


* **Integrity:** The `dcfldd` utility gives real-time hashing and block-count reports. You will see hardware failures on a specific drive immediately. A failure on one drive does not have an effect on the other drives.


* **Time Savings:** Image drives in batches of nine to decrease work time. The maximum number of drives is limited by the USB-A ports that the USB controller supports.



---

**Verification:** Test one drive from each batch on a target laptop. Do this periodically to make sure that the master image is good and the hardware operates correctly.
