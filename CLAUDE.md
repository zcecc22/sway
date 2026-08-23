# Sway Desktop Config - Debian Stable

## Philosophy
- Sway desktop environment config only — no shell dotfiles, no dev tooling
- One install script: `.bin/desktop-setup` (installs and configures the desktop env)
- Solarized Dark theme applied consistently across all tools
- Network via ifupdown2 + wpa_supplicant (no NetworkManager)
- No display manager or auto-launch: start Sway manually via `~/.bin/start-desktop`

## System Components

| Role | Application |
|---|---|
| Window Manager | Sway (Wayland) + autotiling |
| Power Management | TLP + tlp-pd (power profiles via DBus) |
| X11 Compatibility | XWayland |
| Terminal | Alacritty |
| App Launcher | tofi |
| Status Bar | Waybar |
| Notifications | mako |
| Screen Lock | swaylock + swayidle |
| Browser | Firefox ESR |
| Audio | PipeWire (pipewire-pulse + wireplumber) |
| Clipboard | wl-clipboard |
| Fonts | Inconsolata (text), Font Awesome (waybar icons) |

## Hardware Target
Laptop - includes brightness control (brightnessctl), battery status, lid handling.

## Sway Defaults
- Mod key: Super/Logo
- Wallpaper: solid solarized base03 (#002b36)
- Startup brightness: 40% via brightnessctl
- Tiling: autotiling daemon — split direction chosen automatically by container aspect ratio
- Auto-lock: swayidle — lock after 5 min idle, display off after 10 min
- Power profiles: power-profiles-daemon (provided by tlp-pd) surfaced in Waybar
- DBus/systemd env: `dbus-update-activation-environment` propagates `WAYLAND_DISPLAY`, `DISPLAY`, `XDG_CURRENT_DESKTOP` on startup (required for portals and screen sharing)

## Theme: Solarized Dark Palette
```
base03:  #002b36   (background)
base02:  #073642   (background highlights)
base01:  #586e75   (comments, secondary content)
base00:  #657b83   (body text on light bg)
base0:   #839496   (body text)
base1:   #93a1a1   (optional emphasized content)
base2:   #eee8d5   (background on light)
base3:   #fdf6e3   (background on light)
yellow:  #b58900
orange:  #cb4b16
red:     #dc322f
magenta: #d33682
violet:  #6c71c4
blue:    #268bd2
cyan:    #2aa198
green:   #859900
```

## Repository Structure

### Scripts (.bin/)
- `desktop-setup` - Desktop environment package installation
- `start-desktop` - Wayland env vars, startup brightness, exec sway

### Config Files (.config/)
- `sway/config` - Sway WM configuration
- `alacritty/alacritty.toml` - Terminal configuration
- `waybar/config` - Status bar modules
- `waybar/style.css` - Status bar styling
- `mako/config` - Notification daemon styling
- `swaylock/config` - Lock screen styling
- `tofi/config` - App launcher styling
