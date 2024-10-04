### Step-By-Step Installation

1. **Connect to Wi-Fi**
   - Use `iwctl` to connect:
     ```bash
     iwctl
     # Inside iwctl
     device list
     station <device> scan
     station <device> get-networks
     station <device> connect <SSID>
     exit
     ```

2. **Set Time and Date**
   ```bash
   timedatectl set-ntp true
   ```
   - Ensures your system clock is accurate.

3. **List Block Devices**
   ```bash
   lsblk
   ```
   - Displays all attached storage devices.

4. **Partition the Disk**
   ```bash
   cfdisk /dev/<drive_name>
   ```
   - [How to use cfdisk](https://www.geeksforgeeks.org/cfdisk-command-in-linux-with-examples/)
   - **Select Label Type:**
     - Choose `gpt` for UEFI systems.
   - **Create Partitions:**
     - **EFI System Partition (ESP):**
       - Size: `1G` or `512M`
       - Type: `EFI System`
     - **Swap Partition (Optional):**
       - Size: `8G` (match your RAM size if desired)
       - Type: `Linux swap`
     - **Root Partition:**
       - Size: Remaining space
       - Type: `Linux filesystem`
   - **Write Changes** and **Exit** `cfdisk`.

5. **Format the Partitions**
   - **Format EFI System Partition as FAT32:**
     ```bash
     mkfs.fat -F32 /dev/<esp_partition>
     ```
   - **Format Root Partition as ext4:**
     ```bash
     mkfs.ext4 /dev/<root_partition>
     ```
   - **Initialize Swap Partition:**
     ```bash
     mkswap /dev/<swap_partition>
     swapon /dev/<swap_partition>
     ```
   - **Note:** Swap is optional if you have sufficient RAM (16GB or more).

6. **Mount the Partitions**
   - **Mount Root Partition:**
     ```bash
     mount /dev/<root_partition> /mnt
     ```
   - **Create and Mount EFI Directory:**
     ```bash
     mkdir -p /mnt/boot
     mount /dev/<esp_partition> /mnt/boot
     ```
     - In Arch Linux, it's standard to mount the ESP at `/boot`.

7. **Install Essential Packages**
   ```bash
   pacstrap /mnt base linux linux-firmware vim networkmanager grub efibootmgr intel-ucode man-db man-pages texinfo
   ```
   - **Explanation:**
     - `base`: Minimal package set.
     - `linux`: Latest Linux kernel.
     - `linux-firmware`: Necessary firmware.
     - `vim` (or `nano`): Text editor.
     - `networkmanager`: Manages network connections.
     - `grub`: Bootloader.
     - `efibootmgr`: Required for UEFI boot management.
     - `man-db`: Man page database utilities
     - `man-pages`: Collection of man pages for Linux
     - `texinfo`: Tools for reading and creating GNU info pages
     - `intel-ucode`: Verify first with `;scpu | grep 'Vendor ID` first before installing. It must display 'GenuineIntel' or 'AuthenticAMD'

8. **Generate fstab File**
   ```bash
   genfstab -U /mnt >> /mnt/etc/fstab
   ```
   - Generates `/etc/fstab` with UUIDs for reliable device identification.

9. **Change Root into the New System**
   ```bash
   arch-chroot /mnt
   ```
   - Switches from the live USB environment to your installed system.

10. **Enable NetworkManager**
    ```bash
    systemctl enable NetworkManager
    ```
    - Ensures networking is active on boot.

11. **Set Timezone**
    ```bash
    ln -sf /usr/share/zoneinfo/Asia/Manila /etc/localtime
    hwclock --systohc
    ```
    - Replace `Region/City` with your locale (e.g., `America/New_York`).
    - Synchronizes hardware clock.

12. **Localization**
    - **Edit Locale Generation File:**
      ```bash
      vim /etc/locale.gen
      ```
      - Uncomment `en_US.UTF-8 UTF-8` (remove the `#` at the beginning).
      - Uncomment en_US.UTF-8 UTF-8, en_PH.UTF-8 UTF-8

    - **Generate Locales:**
      ```bash
      locale-gen
      ```
    - **Set System Language:**
      ```bash
      echo "LANG=en_US.UTF-8" > /etc/locale.conf
      ```

13. **Set Hostname**
    ```bash
    echo "your_hostname" > /etc/hostname
    ```
    - Replace `your_hostname` with your desired system name.

14. **Set Root Password**
    ```bash
    passwd
    ```
    - Enter and confirm a strong password for the root user.

15. **Install and Configure GRUB Bootloader**
    - **Install GRUB for UEFI Systems:**
      ```bash
      grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB
      ```
    - **Generate GRUB Configuration:**
      ```bash
      grub-mkconfig -o /boot/grub/grub.cfg
      ```
    - **Note:** If you see messages about missing Linux images, ensure the `linux` package was installed during `pacstrap`.

16. **Exit chroot Environment**
    ```bash
    exit
    ```

17. **Unmount Partitions and Reboot**
    ```bash
    umount -R /mnt
    reboot
    ```
    - Remove the installation media after the system restarts.
