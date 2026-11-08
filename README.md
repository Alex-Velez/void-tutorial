# Void Tutorial

Personal notes for configuring my Void Linux setup

# Prerequisites

This section...
- is my personal configuration.
- is meant to be followed in exact order to ensure smoothest configuration.
- assumes you have already booted and logged into a fresh install of base Void Linux.
- will most likely require elevated privileges for most commands.

## Configure WiFi Internet

1. Enable `wpa_supplicant` service:
   - `sv status wpa_supplicant`
   - `ln -s /etc/sv/wpa_supplicant /etc/runit/runsvdir/default`
   - `ip link show`
   - `ip link set wlp3s0 up`
3. Test WiFi: `wpa_cli status`
4. Add WiFi:
   - `wpa_cli scan`
   - `wpa_cli scan_results`
   - `wpa_cli add_network`
   - `wpa_cli set_network 0 ssid '"WIFINAME"'`
      - Private WiFi: `wpa_cli set_network 0 psk "PASSWORD"`
      - Public WiFi: `wpa_cli set_network 0 key_mgmt NONE`
   - `wpa_cli enable_network 0`
   - `wpa_cli select_network 0`
   - `wpa_cli status`
5. Test Internet Connection: `ping -c 5 voidlinux.org`

## Firmware

```
xbps-install -S void-repo-nonfree
xbps-install -S void-repo-multilib
xbps-install -S void-repo-multilib-nonfree
xbps-install -Suy
xbps-install -S linux-firmware
xbps-install -S linux-firmware-amd
```

## Tools

```
xbps-install -S helix
xbps-install -S nushell
chsh -s /usr/bin/nu
```

## Desktop

### Global Environment Variables

`hx /etc/environment`:
```
QT_QPA_PLATFORM=wayland
ELM_DISPLAY=wl
SDL_VIDEODRIVER=wayland,x11
XDG_SESSION_TYPE=wayland
```

### Graphical Packages

```
xbps-install -S fastfetch
xbps-install -S dbus 
ln -s /etc/sv/dbus /etc/runit/runsvdir/default
xbps-install -S nvidia
xbps-install -S nerd-fonts
xbps-install -S elogind
xbps-install -S tlp 
ln -s /etc/sv/tlp /etc/runit/runsvdir/default
xbps-install -S qt5-wayland qt6-wayland
xbps-install -S kwayland
xbps-install -S xorg-server-xwayland
xbps-install -S xdg-desktop-portal xdg-desktop-portal-gtk
xbps-install -S niri
```

### Desktop Launcher

Setup local binary folder with correct permissions:
```
mkdir -p ~/.local/bin
chown -R "$USER":"$(id -gn)" ~/.local
chmod -R u+rwX ~/.local
```

`hx ~/.config/nushell/env.nu`:
```nu
$env.PATH = ($env.PATH | prepend ($env.HOME | path join ".local" "bin"))
$env.EDITOR = 'hx'
```

`hx ~/.config/nushell/config.nu`:
```nu
$env.config.show_banner = "short"

# local bin files
source ~/.local/bin/desktop.nu
source ~/.local/bin/custom_banner.nu

^fastfetch --disable-linewrap
```

Create desktop launcher scripts:
```
touch ~/.local/bin/custom_banner.nu
chmod +x ~/.local/bin/custom_banner.nu 
touch ~/.local/bin/desktop.nu
chmod +x ~/.local/bin/desktop.nu 
```

`hx ~/.local/bin/custom_banner.nu`:
```nu
#!/usr/bin/env nu

use std/dt [datetime-diff pretty-print-duration]

export def custom_greeting [] {

let dt = (datetime-diff (date now) 2019-05-10T09:59:12-07:00)

$"(ansi green)     __  ,(ansi reset)
(ansi green) .--\(\)°'.' (ansi reset)Welcome to (ansi green)Nushell(ansi reset),
(ansi green)'|, . ,'   (ansi reset)based on the (ansi green)nu(ansi reset) language,
(ansi green) !_-\(_\\    (ansi reset)where all data is structured!

It's been this long since (ansi green)Nushell(ansi reset)'s first commit:
(pretty-print-duration $dt)
"
}
```

`hx ~/.local/bin/desktop.nu`:
```nu
#!/usr/bin/env nu

use custom_banner.nu custom_greeting

def desktop [] {
    # save TTY dimensions
    let old_size = (^stty -F /dev/tty size | str trim | split row " ")

    # launch niri
    try {
        dbus-run-session -- niri --session
    } catch {
        print "Niri exited with an error."
    } finally {
        # restore TTY dimensions
        ^stty -F /dev/tty  rows ($old_size | get 0) cols ($old_size | get 1)

        # restore TTY state
        ^stty -F /dev/tty sane
        ^reset

        # display banner
        print (custom_greeting)
        print $"(ansi green_bold)Startup Time:(ansi reset) ($nu.startup-time)"
    }
}
```


## Login Screen

```
xbps-install -S greetd tuigreet
```

`hx /etc/greetd/config.toml`:
```toml
[terminal]
vt = 1

[default_session]
command = "tuigreet --time --remember --remember-session --cmd nu"
user = "greeter"
```

```
useradd -r -s /sbin/nologin -M -d /var/empty greeter
ln -s /etc/sv/greetd /etc/runit/runsvdir/default
sv down acpid
rm -f /var/service/acpid
reboot
```

## Essential Software

```
xbps-install -Suy alacritty broot fuzzel mako Waybar swaybg wl-clipboard grim slurp swaylock curl xz
curl -fsSL https://github.com/zen-browser/updates-server/raw/refs/heads/main/install.sh | $SHELL
```

## Audio

```
xbps-install -S pipewire
mkdir -p /etc/pipewire/pipewire.conf.d
ln -s /usr/share/examples/wireplumber/10-wireplumber.conf /etc/pipewire/pipewire.conf.d/
ln -s /usr/share/examples/pipewire/20-pipewire-pulse.conf /etc/pipewire/pipewire.conf.d/
```

## Bluetooth

```
xbps-install -S bluez libspa-bluetooth
ln -s /etc/sv/bluetoothd /var/service/
```

## Gaming

```
xbps-install -S steam xwayland-satellite libgcc-32bit libstdc++-32bit libdrm-32bit libglvnd-32bit libva-32bit nvidia-libs-32bit
xbps-install -Suy
```

# Notes

To be continued, with further instructions and more detail as I grow my personal daily-driver Void Linux system.

I just wanted a way to replicate my setup, since I often forget commands, applications, and configs; in case I need to reinstall Void Linux onto another device.

# Final System
| | |
| - | - |
| architecture | x86_64 |
| cpu | AMD Ryzen 9 8945HS |
| gpu | NVIDIA GeForce RTX 4070 |
| filesystem | btrfs |
| init | runit |
| display_server | Wayland |
| compositor | niri |
| login_manager | greetd |
| audio | pipewire |
| shell | nushell |
| terminal | alacritty |
| editor | helix |
