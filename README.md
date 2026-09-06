# Void Tutorial

Personal notes for configuring my Void Linux setup

# Prerequisites

This tutorial...
- assumes `x86_64` hardware and compatibility (AMD CPU, Nvidia GPU); with Windows OS. (can be altered)
- assumes your location is North America, United States. (can be altered)
- assumes basic terminal and command line competency.
- is meant to be followed exactly, line-by-line, in order to replicate my Void Linux setup.
- is inherently opinionated and suited to my personal preferences for Void Linux.
- is meant as a basic starting point for absolute Void Linux beginners. (myself)
- may be out of date.

# Download Bootable-USB-Creator

Any will do, this is what I recommend:
- Rufus: https://rufus.ie/en/

# Download ISO & Checksum

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

1. Enter BIOS/UEFI
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

# Install Void Linux

This section...
- is heavily dependent on hardware.
- may need extra configuration or different settings. (refer to [Void Linux Installation Guide](https://docs.voidlinux.org/installation/live-images/guide.html))
- assumes you will install Void Linux to a `64GB` USB Drive.
- assumes you will connect to WiFi.

Create a Bootable USB (Void Linux):

1. Login: `root`
2. Password: `voidlinux`
3. Identify device to install: `fdisk -l` > `/dev/sd?` (refer to [Void Linux Installation Media Preparation](https://docs.voidlinux.org/installation/live-images/prep.html))
4. `void-installer`
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
      - `/dev/sd?` : Select USB/Disk to install Void Linux
   - Partition: (refer to [Partition Notes](https://docs.voidlinux.org/installation/live-images/partitions.html))
      - `/dev/sd?` : Select USB/Disk to install Void Linux
      - `cfdisk`:
         1. Delete all existing partitions.
         2. Create EFI Partition:
            - `512M` (typically between `200MB`-`1GB`)
            - Partition Type: `EFI System`
         3. Create Root Partition:
            - `59.5GB` (typically `30GB` or higher)
            - Partition Type: `Linux filesystem`
         4. Create Swap Partition:
            - `4GB` (typically `4GB` or higher)
            - Partition Type: `Linux swap`
         5. Write changes.
         6. Quit `cfdisk`.
   - Filesystems:
      1. EFI Partition: `/dev/sda1`
         - Filesystem: `vfat`
         - Mount Point: `/boot/efi`
      2. Root Partition: `/dev/sda2`
         - Filesystem: `btrfs`
         - Mount Point: `/`
      3. Swap Partition: `/dev/sda3`
         - Filesystem: `swap`
      4. Done.
   - Install.
5. Enable Services:
   - `acpid`
   - `dhpcd`
   - `dhpcd-eth0`
   - `sshd`
   - `wpa_supplicant`
6. Reboot The System.

# Configure Void Linux

At this point, you have a working base Void Linux System.

This section...
- is highly opinionated.
- is meant to be followed in exact order to ensure smoothest configuration.

## Configure WiFi Internet

1. Test WiFi: `ip link show`
2. Add WiFi with `wpa_supplicant`:
   - Private/Home WiFi (has password):
      1. `su`
      2. `wpa_passphrase WIFINAME WIFIPASSWORD >> /etc/wpa_supplicant/wpa_supplicant.conf`
      3. `exit`
   - Public WiFi (no password):
      1. `su`
      2. `printf 'network={\n\tssid="PUBLICWIFINAME"\n\tkey_mgmt=NONE\n}\n' >> /etc/wpa_supplicant/wpa_supplicant.conf`
      3. `exit`
4. Test Internet Connection: `ping -c 5 voidlinux.org`

## Update System

