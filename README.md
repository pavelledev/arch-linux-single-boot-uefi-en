# Arch Linux Single Boot (UEFI)

## Disclaimers:
This guide is intended for users who are new to Linux. It may not be the most advanced or minimal method, but it will help you install a fully functional Arch Linux system with the essential components most people usually need.

The guide works for both laptops and desktops. Laptop users only have one extra step: connecting to a Wi-Fi network.

**Important:** Linux commands are powerful. A single typo can cause the installation to fail. Please read carefully and type exactly as shown.

All commands are provided in separate code blocks. If a command changes, it will appear in its own new block to avoid confusion.


## Tips

### Using Tab Completion
Most Linux commands support Tab completion:

- Type the beginning of a command or filename.
- Press `TAB` to auto-complete it.
- If multiple matches exist, press `TAB` twice to show all possible completions, or press `TAB` repeatedly to cycle through the suggestions.

This helps prevent mistakes and speeds up the installation process. Note that not all commands support Tab completion.


### Step 1: Check if the system is using UEFI

```bash
ls /sys/firmware/efi
```

If the system returns **No such file or directory**, it means the computer is booted in **Legacy (BIOS)** or may not support **UEFI**.


### Step 2 (Optional): Connect to a Wireless Network (Wi-Fi)

If you are using a cable connection (Desktops/PC), you can skip this step.

#### 2.1 Check if the Wi-Fi device is blocked

```bash
rfkill
```

If the Wi-Fi section (`wlan`) is **hard-blocked**, you need to use the physical switch button that the device provide (i don't know why some device has it) 

If it is **soft-blocked** only, you can unblock it with the following command:

```bash
rfkill unblock wlan
```

#### 2.2 Enter the iwd Wi-Fi tool

```bash
iwctl
```

List available wireless device:

```text
device list
```

Choose the wireless device shown in the list (commonly `wlan0`), from this point onward it, `wlan0` will be used in all commands. If your device name is different, make sure to replace `wlan0` with your actual device.

#### 2.3 Scan and connect to a Wi-Fi network

List available networks:

```text
station wlan0 get-networks
```

Connect to your Wi-Fi network (please replace `YourWifiName` with the name of the Wi-Fi you are trying to connect to):

```text
station wlan0 connect "YourWifiName"
```

Enter the Wi-Fi password when prompt, then press **ENTER**.

Exit iwd (iwctl):

```text
exit
```

#### 2.4 Verify the internet connection

```bash
ping archlinux.org
```

If you receive packets, your internet is working.
Press **CTRL + C** to stop the pinging proccess.


### Step 3: Identify the main storage device
> Also commonly known as SSD, HDD,.... I will also refer to them as disk, drive.

List all storage devices:

```bash
lsblk
```

Your primary drive is usually named something like `nvme0n1`, `sda`, `sdb`,.... If you are using an **NVMe SSD**, it will typically appear as `nvme0n1`. If you are using a SATA SSD or HDD, it will usually appear as `sda`.

### Step 4: Partition the Disk
**(Please read the entire step before attempting!)**

Open the partitioning tool:

```bash
cfdisk /dev/your-main-disk
```

Please replace the `your-main-disk` with the disk you identified in **Step 3** (`nvme0n1` or `sda`,...). If you are prompted, please select **GPT** as the partition table.

#### How to use the cfdisk:

- Use the **ARROW KEYS** to navigate.
- Select **Free Space -> New** to create a new partition.
- Enter the desired size (e.g., `500M`, `30G`).
- Select the new partition -> **Type** to set the partition type.
- After creating all partitions and adjusting the correct type, select **Write**, type `yes` and press **ENTER**.
- Select **Quit** to exit `cfdisk`.

#### Suggested partitions layout

| Partition | Size | Type |
|-----------|------|------|
| nvme0n1p1, sda1,... | 512M | EFI System |
| nvme0n1p2, sda2,... | 4G/8G | Linux Swap |
| nvme0n1p3, sda3,... | Remaining space | Linux Filesystem |

> **Tips:** If you don't plan to use hibernation, you can keep the swap partition at **4-8GB**. If you want to enable hibernation, set the swap size to **at least double your RAM** (e.g., 16GB RAM -> 32GB swap).

