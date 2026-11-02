# Prerequisites

This tutorial...
- assumes a Windows OS starting point. (not strictly required)
- assumes `x86_64` hardware and compatibility. (AMD CPU, Nvidia GPU, etc.) 
- assumes your location is North America, United States. (can be altered)
- is meant to be followed exactly, line-by-line.
- is my personal guide for installing Void Linux onto a USB.
- requires two storage drives.
   - One for the ISO and another to install the OS.
   - Examples: 1 USB + 1 USB, 1 USB + 1 HDD, 1 USB + 1 SSD, etc.
- may be out of date.

# Download Bootable-USB-Creator

Any will do, this is what I recommend:
- Rufus: https://rufus.ie/en/

# Download ISO, Checksum, & Signature

Void Linux ISO: [void-live-x86_64-XXXXXXXX-base.iso](https://repo-default.voidlinux.org/live/current)

ISO Checksum: [sha256sum.txt](https://repo-default.voidlinux.org/live/current/sha256sum.txt)

Checksum Signature: [sha256sum.sig](https://repo-default.voidlinux.org/live/current/sha256sum.sig)

# Verify Image

Generate a `SHA-256` hash in PowerShell for the downloaded ISO:

`Get-FileHash .\void-live-x86_64-XXXXXXXX-base.iso -Algorithm SHA256`

1. Verify that the generated hash matches exactly the `SHA-256` hash published in the `sha256sum.txt` file.
2. Validate authenticity of the `sha256sum.txt` file. (refer to [Void Linux Verifying Digital Signature](https://docs.voidlinux.org/installation/index.html))

## Example:
> 1. ISO: `void-live-x86_64-20250202-base.iso`
> 2. PowerShell: `Get-FileHash ~\Downloads\void-live-x86_64-20250202-base.iso -Algorithm SHA256`
> 3. Generated Hash: `0F7439F500740F62DD18972CAE448CEC7D8A85032C7EB8F1BF946100D9A92161`
> 4. In the `sha256sum.txt`, find the line corresponding to `void-live-x86_64-20250202-base.iso`:
> - `SHA256 (void-live-x86_64-20250202-base.iso) = 0f7439f500740f62dd18972cae448cec7d8a85032c7eb8f1bf946100d9a92161`
> 5. Both hashes match exactly (case-insensitive), therefore ISO integrity has been validated.

# Create a Bootable USB (ISO)

1. Plug in a USB Drive.
2. Open [Rufus](https://rufus.ie/en/).
3. Select the USB Drive under `Device`.
4. `SELECT` the `void-live-x86_64-XXXXXXXX-base.iso` file.
5. Choose a Volume Label.
6. `START`.

# Boot into ISO USB

1. Change BIOS/UEFI Settings:
   - Change boot order to give priority to the USB.
   - Disable Secure Boot.
2. Boot into ISO USB
   - Open Windows Settings
   - `Windows Update`
   - `Advanced Options`
   - `Recovery`
   - `Advanced Startup`
   - `Use a Device`
   - `UEFI: Removable Device`

# Install Base Void Linux

This section...
- is heavily dependent on hardware.
- may need extra configuration or different settings. (refer to [Void Linux Installation Guide](https://docs.voidlinux.org/installation/live-images/guide.html))
- assumes you will install Void Linux to a `64GB` USB Drive.
- assumes you have access to a WiFi connection.

Create a Bootable USB (Void Linux):

1. Login: `root`
2. Password: `voidlinux`
3. Identify device to install: (refer to [Void Linux Installation Media Preparation](https://docs.voidlinux.org/installation/live-images/prep.html))
   - `lsblk -o NAME,SIZE,TYPE,FSTYPE,LABEL,MOUNTPOINTS`
   - USB Drive name should be should be under `NAME`: `/dev/sd#` (ex. `/dev/sdb`)
   - USB Drive file type should be under `FSTYPE`: (ex. `vfat`)
   - Do NOT choose `VOID_LIVE`.
5. Format storage device & verify filesystem:
   - `Do NOT choose `VOID_LIVE`.
   - `umount /dev/sd#`
   - `mkfs.fat -F32 /dev/sd#` or `mkfs.vfat /dev/sd#` or `mkfs.exfat /dev/sd#` or `mkfs.ext4 /dev/sd#` or etc...
   - `lsblk -f`
7. `void-installer`
   - Keyboard: `us`
   - Network:
      - `WIFINAME`
      - `wpa`
      - `WIFIPASSWORD`
   - Source: `Network`
   - Mirror: `NA` (or closest to your location)
   - Hostname: `void-linux`
   - Locale: `English (United States of America)` (or closest to your location)
   - Timezone: `America\Chicago` (or closest to your location)
   - RootPassword: `***SECUREPASSWORD***`
   - UserAccount:
      - Name: `***name***`
      - Display Name: `***Full Name***`
      - Group Membership:
         - `wheel`    : Elevated privileges for specific system administration tasks.
         - `lp`       : Access to printers.
         - `dialout`  : Access to serial ports.
         - `audio`    : Access to audio devices.
         - `video`    : Access to video devices.
         - `cdrom`    : Access to CD devices.
         - `optical`  : Access to DVD/CD-RW devices.
         - `storage`  : Access to removable storage devices.
         - `scanner`  : Ability to access scanners.
         - `network`  : Used by some networking-related packages.
         - `kvm`      : Ability to use KVM for virtual machines.
         - `input`    : Access to input devices.
         - `plugdev`  : Access to pluggable devices.
         - `xbuilder` : To use `xbps-uchroot(1)` with `xbps-src`.
   - BootLoader:
      - `/dev/sd#` : Select USB/Disk to install Void Linux
   - Partition: (refer to [Partition Notes](https://docs.voidlinux.org/installation/live-images/partitions.html))
      - `/dev/sd#` : Select USB/Disk to install Void Linux
      - `cfdisk`:
         1. Delete all existing partitions.
         2. Create EFI Partition:
            - `1G` (typically between `200MB`-`1GB`)
            - Partition Type: `EFI System`
         3. Create Root Partition:
            - `50G` (typically `30GB` or higher)
            - Partition Type: `Linux filesystem`
         4. Create Swap Partition:
            - `6.3GB` (typically `4GB` or higher)
            - Partition Type: `Linux swap`
         5. Write changes.
         6. Quit `cfdisk`.
   - Filesystems:
      1. EFI Partition: `/dev/sd#1`
         - Filesystem: `vfat`
         - Mount Point: `/boot/efi`
      2. Root Partition: `/dev/sd#2`
         - Filesystem: `btrfs`
         - Mount Point: `/`
      3. Swap Partition: `/dev/sd#3`
         - Filesystem: `swap`
      4. Done.
   - Install.
8. Enable Services:
   - `acpid`
   - `dhcpcd`
   - `dhcpcd-eth0`
   - `sshd`
   - `wpa_supplicant`
9. Reboot The System.

# Additional Notes

At this point, you have a working base Void Linux System.

You are free to customize at will.
