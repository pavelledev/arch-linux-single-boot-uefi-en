# Arch Linux Single Boot (UEFI)

## Disclaimers:
- This guide is intended for users who are new to Linux.

- It may not be the most advanced method, but it will help you install a **bare-minimum Arch Linux system**. 

- ⚠️ This installation does **not include** a Desktop Environment, audio, graphics drivers, or other additional software — it will give you a **fully functional Arch Linux system with a black console screen**.

- The guide works for both laptops and desktops. Laptop users only have one extra step: connecting to a Wi-Fi network.

**Important:** Linux commands are powerful. A single typo can cause the installation to fail. Please read carefully and type exactly as shown.
All commands are provided in separate code blocks. If a command changes, it will appear in its own new block to avoid confusion.

## Tips:

### Using Tab Completion
Most Linux commands support Tab completion:

- Type the beginning of a command or filename.
- Press `TAB` to auto-complete it.
- If multiple matches exist, press `TAB` twice to show all possible completions, or press `TAB` repeatedly to cycle through the suggestions.

This helps prevent mistakes and speeds up the installation process. Note that not all commands support Tab completion.

## Step-by-Step Arch Linux UEFI Installation

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


### Step 3: Enable Network time synchronization

#### 3.1 Enable automatic time sync with NTP servers

```bash
timedatectl set-ntp true
```

#### 3.2 Check the status to make sure it's active

```bash
timedatectl status
```

> Check if `synchronized: yes`.


### Step 4: Identify the main storage device
> Also commonly known as SSD, HDD,.... I will also refer to them as disk, drive.

List all storage devices:

```bash
lsblk
```

Your primary drive is usually named something like `nvme0n1`, `sda`, `sdb`,.... If you are using an **NVMe SSD**, it will typically appear as `nvme0n1`. If you are using a SATA SSD or HDD, it will usually appear as `sda`.

### Step 5: Partition the Disk
**(Please read the entire step before attempting!)**

Open the partitioning tool:

```bash
cfdisk /dev/your-main-disk
```

Please replace the `your-main-disk` with the disk you identified in **Step 4** (`nvme0n1` or `sda`,...). If you are prompted, please select **GPT** as the partition table.

#### How to use the cfdisk:

- Use the **ARROW KEYS** to navigate.
- Select **Free Space -> New** to create a new partition.
- Enter the desired size (e.g., `500M`, `30G`).
- Select the new partition -> **Type** to set the partition type.
- After creating all partitions and adjusting the correct type, select **Write**, type `yes` and press **ENTER**.
- Select **Quit** to exit `cfdisk`.

#### Suggested partitions layout

| Name | Partition | Size | Type |
|------|-----------|------|------|
| EFI Partition | nvme0n1p1, sda1,... | 512M | EFI System |
| Swap Partition | nvme0n1p2, sda2,... | 4G/8G | Linux Swap |
| Root Partition | nvme0n1p3, sda3,... | Remaining space | Linux Filesystem |

> Name is used for **Step 6** below!
> **Tips:** If you don't plan to use hibernation, you can keep the swap partition at **4-8GB**. If you want to enable hibernation, set the swap size to **at least double your RAM** (e.g., 16GB RAM -> 32GB swap).


### Step 6: Format the partitions

#### 6.1 Format the EFI partition

```bash
mkfs.fat -F32 /dev/your-efi-partition
```

> Replace `/dev/your-efi-partition` with your EFI partition in **Step 4** (e.g., `/dev/nvme0n1p1`, `/dev/sda1`,...)

#### 6.2 Format the swap partition

```bash
mkswap /dev/your-swap-partition
```
```bash
swapon /dev/your-swap-partition
```

> Replace `/dev/your-swap-partition` with your Swap partition in **Step 4** (e.g., `/dev/nvme0n1p2`, `/dev/sda2`,...)

#### 6.3 Format the root partitions

```bash
mkfs.ext4 /dev/your-root-partition
```

> Replace `/dev/your-root-partition` with your Root partition in **Step 4** (e.g., `/dev/nvme0n1p2`, `/dev/sda2`,...)
> **Tips:** Please double-check that you are formatting the correct partitions to avoid errors and data loss.


### Step 7: Mount the partitions
> From this step onward, you should have a clear understanding of which partition is which.
> For clarity, I won’t repeat reminders about the partition layout in every command.

#### 7.1 Mount the root partition

```bash
mount /dev/your-root-partition /mnt
```
#### 7.2 Create mount points for EFI and a home directory

```bash
mkdir -p /mnt/boot
```
```bash
mkdir -p /mnt/home
```

#### 7.3 Mount the EFI partition

```bash
mount /dev/your-efi-partition /mnt/boot
```


### Step 8: Install the base packages
> Please read this entire step before executing any commands!
Install the essential packages for your Arch Linux system:

```bash
pacstrap /mnt base base-devel linux linux-firmware linux-headers vim
```

- If you are using an **Intel CPU**, include this at the end:
```bash
intel-ucode
```

- If you are using an **AMD CPU**, include this at the end:
```bash
amd-ucode
```


### Step 9: Basic system configurations
**This step setup the essential system configurations for your Arch Linux so please follow closely!**

#### 9.1 Generate Fstab

```bash
genfstab -U /mnt >> /mnt/etc/fstab
```

#### 9.2 Enter the new system

```bash
arch-chroot /mnt
```

#### 9.3 Set the timezone

```bash
ln -sf /usr/share/zoneinfo/Region/City /etc/localtime
```

> Replace `/Region/City` with your local timezone in the [List of tz database time zones](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones).
> For example I live in Vietnam so I personally use `ln -sf /usr/share/zoneinfo/Asia/Ho_Chi_Minh /etc/localtime`.

#### 9.4 Synchronize hardware clock

```bash
hwclock --systohc
```

#### 9.5 Configure locales
> Tips to using vim is at the end of this step (9.5)

1. Open the locale configuration files:
```bash
vim /etc/locale.gen
```

2. Uncomment the line:
> Note that it en_US not es_US, i've had friends who uncomment the wrong line!
> I've explained `uncomment` at the end of this step (9.5)!

```
en_US.UTF-8 UTF-8
```

3. Generate the locale:

```bash
locale-gen
```

4. Set the system default locale:

```bash
echo LANG=en_US.UTF-8 > /etc/locale.conf
```

> **Tip for beginners using vim:**  
> - Press `[I]` (the I (i not L) key in your keyboard) to enter insert mode.
> - Remove the `#` at the beginning of the line to uncomment.  
> - Press `[ESC]`, then type `:wq` and press `ENTER` to save and exit.

#### 9.6 Set hostname

1. Set your device's hostname:

```bash
echo your-hostname > /etc/hostname
```

> Replace `your-hostname` with your desired hostname like `pavelle-pc` for me personally.

2. Edit `/etc/hosts`:

```bash
vim /etc/hosts
```

- Add the following line, remember to replace `your-hostname` with the hostname you've set in **Step 9.5** and use the **TAB** key where `<TAB>` were indicated:

```text
127.0.1.1<TAB>your-hostname.localdomain<TAB>your-hostname
```

#### 9.7 Set root password

```bash
passwd
```

- Enter the root password and repeat it *(it will be invisible while typing)*.


### Step 10: Create an user account

#### 10.1 Add a new user

```bash
useradd -m username
```

> Replace `username` with your disired username.
> From this point onward, replace `username` with the username you've set in this step (10.1).

#### 10.2 Set the user password

```bash
passwd username
```

- Enter the password and repeat it *(it will be invisible while typing like last time)*.

#### 10.3 Add the user to common groups

```bash
usermod -aG wheel,audio,video,optical,storage,power username
```

#### 10.4 Enable sudo privileges
> I've explained how to use vim at the end of **Step 9**.

```bash
EDITOR=vim visudo
```

- Add this line below the line `root ALL=(ALL:ALL) ALL`:

```text
username ALL=(ALL) ALL
```

- Uncomment this line *(should be right below where you are at)*:

```text
%wheel ALL=(ALL:ALL) ALL
```


### Step 11: Install a bootloader (GRUB)

#### 11.1 Install GRUB and networking tools

```bash
pacman -S grub efibootmgr networkmanager network-manager-applet
```

#### 11.2 Enable NetworkManager

```bash
systemctl enable NetworkManager
```

#### 11.3 Install GRUB to the EFI partition
> Take this step carefully because it really easy to make a typo!

```bash
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB
```

#### 11.4 Generate GRUB configuration

```bash
grub-mkconfig -o /boot/grub/grub.cfg
```

### Step 12: Exit, reboot and login to the new system

- Exit the now fully configured system

```bash
exit
```

- Unmount all partitions

```
umount -R /mnt
```

- Reboot the device

```
reboot
```

- After rebooting, you will be prompt to enter your `username` and `password` you've set for the username in **Step 10**.

### Step 13 (Optional): Connecting to Wi-Fi again

- Enter network manager tool:

```bash
sudo nmtui
```

- Enter your password
- From this point onward the tool will be clear and easy enough for you to use, just use the **ARROW KEYS** to navigate like `cfdisk`.

### Step 14: Update the system
> You should do this daily because Arch Linux is a rolling-release distro, a long time of not updating may cause trouble!

```
sudo pacman -Syu
```

### Congratulations!

You have successfully installed a **bare-minimum Arch Linux system**! 🎉  
This is just the base system. I will create more guides on how to build a **complete, fully functional Arch Linux system**, and I will leave links to them below when available.  