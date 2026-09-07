# Void Tutorial

Personal notes for configuring my Void Linux setup

# Prerequisites

This section...
- is highly opinionated.
- is meant to be followed in exact order to ensure smoothest configuration.
- assumes you have already booted and logged into a fresh install of base Void Linux.
- will most likely require elevated privileges for most commands.

## Configure WiFi Internet

1. Test WiFi: `ip link show`
2. Add WiFi with `wpa_supplicant`:
   - Private/Home WiFi (has password): `wpa_passphrase WIFINAME WIFIPASSWORD >> /etc/wpa_supplicant/wpa_supplicant.conf`
   - Public WiFi (no password): `printf 'network={\n\tssid="PUBLICWIFINAME"\n\tkey_mgmt=NONE\n}\n' >> /etc/wpa_supplicant/wpa_supplicant.conf`
4. Enable `wpa_applicant` Service: `ln -s /etc/sv/wpa_supplicant /etc/runit/runsvdir/default`
5. Test Internet Connection: `ping -c 5 voidlinux.org`

## Firmware

```
xbps-install -Su
xbps-install -S void-repo-nonfree
xbps-install -S void-repo-multilib
xbps-install -S void-repo-multilib-nonfree
xbps-install -S linux-firmware-amd
xbps-install -S linux-firmware
```

## Desktop

```
xbps-install -S helix
xbps-install -S dbus 
ln -s /etc/sv/dbus /etc/runit/runsvdir/default
xbps-install -S nvidia
xbps-install -S xorg
xbps-install -S nerd-fonts
xbps-install -S elogind
xbps-install -S tlp 
ln -s /etc/sv/tlp /etc/runit/runsvdir/default
xbps-install -S qt5-wayland qt6-wayland
echo "QT_QPA_PLATFORM=wayland" >> /etc/environment
xbps-install -S kwayland
echo “ELM_DISPLAY=wl” >> /etc/environment
echo "SDL_VIDEODRIVER=wayland" >> /etc/environment
xbps-install -S xorg-server-xwayland
echo "XDG_SESSION_TYPE=wayland" >> /etc/environment
xbps-install -S xdg-desktop-portal xdg-desktop-portal-gtk
xbps-install -S niri
```

## Login Screen

```
xbps-install -S greetd tuigreet
hx /etc/greetd/config.toml
```

`/etc/greetd/config.toml`:
```toml
[terminal]
vt = 1

[default_session]
command = "tuigreet --time --remember --remember-session --cmd 'dbus-run-session niri --session'"
user = "greeter"
```

```
useradd -r -s /sbin/nologin -d /var/empty greeter
rm /var/service/agetty-tty1
ln -s /etc/sv/greetd /etc/runit/runsvdir/default
sv down acpid
rm -f /var/service/acpid
reboot
```

## Essential Software

```
xbps-install -Su alacritty nushell broot fuzzel mako Waybar swaybg wl-clipboard grim slurp swaylock curl xz
curl -fsSL https://github.com/zen-browser/updates-server/raw/refs/heads/main/install.sh | $SHELL
```

## Audio

```
xbps-install -S pipewire
mkdir -p /etc/pipewire/pipewire.conf.d
ln -s /usr/share/examples/wireplumber/10-wireplumber.conf /etc/pipewire/pipewire.conf.d/
mkdir -p /etc/pipewire/pipewire.conf.d
ln -s /usr/share/examples/pipewire/20-pipewire-pulse.conf /etc/pipewire/pipewire.conf.d/
```

## Bluetooth

```
xbps-install -S libspa-bluetooth
```

# Notes

To be continued, with further instructions and more detail as I grow my personal daily-driver Void Linux system.

I just wanted a way to replicate my setup, since I often forget commands, applications, and configs; in case I need to reinstall Void Linux onto another device.
