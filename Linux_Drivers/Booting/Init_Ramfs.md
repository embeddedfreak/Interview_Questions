# What is Initramfs?

**Initramfs (Initial RAM Filesystem)** is a **temporary root filesystem loaded into RAM during the Linux boot process**.

Its main purpose is to help the Linux kernel access and mount the **real root filesystem** located on the storage device.

---

# Why Do We Need Initramfs?

When the kernel starts, it may not have the required drivers to access the storage device containing the root filesystem.

Examples:

* Root filesystem is on an NVMe SSD
* Root filesystem is on LVM
* Root filesystem is encrypted
* Root filesystem is on a RAID array

The kernel needs specific drivers/modules to access these devices.

However, those drivers are usually stored on the disk itself.

This creates a chicken-and-egg problem:

```text id="ywyi76"
Need drivers to access disk
↓
Drivers are stored on disk
↓
Cannot access disk without drivers
```

**Initramfs solves this problem.**

---

# Boot Flow with Initramfs

```text id="tjn9if"
Power ON
    ↓
BIOS/UEFI
    ↓
GRUB
    ↓
Kernel + Initramfs loaded into RAM
    ↓
Kernel starts
    ↓
Initramfs becomes temporary root filesystem
    ↓
Required drivers/modules loaded
    ↓
Real root filesystem mounted
    ↓
systemd starts
```

---

# What's Inside Initramfs?

Initramfs is usually a compressed archive that contains:

* Essential device drivers
* Kernel modules
* Basic utilities
* Boot scripts
* Filesystem tools

Typical contents:

```text id="1h8nvz"
/bin
/sbin
/lib
/init
```

The `/init` script is the first script executed inside initramfs.

---

# What Happens Inside Initramfs?

## Step 1

The kernel unpacks initramfs into RAM.

## Step 2

The kernel executes the `/init` script.

## Step 3

The script performs tasks such as:

* Loading required kernel modules
* Detecting storage devices
* Activating LVM volumes
* Unlocking encrypted partitions
* Assembling RAID arrays

## Step 4

The actual root filesystem is mounted.

Example:

```text id="2nj3yx"
/dev/sda2 → /
```

## Step 5

Control is transferred from the temporary root filesystem to the real root filesystem.

This process is called:

```text id="yvs4kp"
switch_root
```

or

```text id="4h36o7"
pivot_root
```

## Step 6

The kernel starts PID 1 (usually systemd).

---

# Interview Example

Suppose the root filesystem resides on an NVMe SSD:

```text id="2wzcyw"
/
└── NVMe SSD
```

Boot sequence:

1. Kernel starts.
2. Kernel cannot yet communicate with the NVMe controller.
3. Initramfs loads the NVMe driver.
4. Kernel gains access to the SSD.
5. Root filesystem is mounted.
6. Boot process continues.

Without initramfs, the kernel would not be able to locate and mount the root filesystem.

---

# How to View Initramfs on a Linux System

List files in the `/boot` directory:

```bash id="jlwmya"
ls /boot
```

Typical output:

```text id="n24jsk"
vmlinuz-6.x.x
initramfs-6.x.x.img
```

or

```text id="zzrj4x"
initrd.img-6.x.x
```

To view initramfs files:

```bash id="m8tx04"
ls -lh /boot/initramfs*
```

---

# Difference Between Kernel and Initramfs

| Kernel                                   | Initramfs                            |
| ---------------------------------------- | ------------------------------------ |
| Core of the operating system             | Temporary root filesystem            |
| Manages CPU, memory, and devices         | Helps mount the real root filesystem |
| Loaded by GRUB                           | Loaded by GRUB along with the kernel |
| Remains active throughout system runtime | Used only during the boot process    |

---

# Simple Interview Answer

> Initramfs (Initial RAM Filesystem) is a temporary root filesystem loaded into RAM by the bootloader along with the Linux kernel. It contains essential drivers, kernel modules, and scripts needed to access and mount the real root filesystem. Once the real root filesystem is mounted, control is transferred from initramfs to the actual root filesystem, and the boot process continues with systemd as PID 1.

---

# One-Line Memory Trick

> **Initramfs is a temporary helper filesystem in RAM that provides the drivers and tools needed to mount the real root filesystem.**

