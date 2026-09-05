# Void Tutorial

Personal notes for configuring my Void Linux setup

# Prerequisites

This tutorial...
- assumes `x86_64` hardware and compatibility; with Windows OS.
- is meant to be followed exactly, line-by-line, in order to replicate my Void Linux setup.
- is inherently opinionated and suited to my personal preferences for Void Linux.
- assumes basic terminal and command line competency.
- is meant as a basic starting point for absolute Void Linux beginners (myself).
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
2. Validate authenticity of the `sha256sum.txt` file.

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

# First Boot

1. Enter BIOS/UEFI
2. Change boot order to give priority to the USB.
3. Disable Secure Boot.
4. 
