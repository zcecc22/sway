# Sway Desktop Config - Debian Stable

## Philosophy
- Sway desktop environment config only — no shell dotfiles, no dev tooling
- One install script: `.bin/desktop-setup` (installs Sway and its companion apps only)
- Solarized Dark theme applied consistently across all tools
- Network and power management are assumed already configured on the base system (no NetworkManager; ifupdown2 + wpa_supplicant, TLP/tlp-pd) — this repo doesn't install or manage them
- No display manager or auto-launch: start Sway manually via `~/.bin/start-desktop`

## System Components

| Role | Application |
|---|---|
| Window Manager | Sway (Wayland) + autotiling |
| X11 Compatibility | XWayland |
| Terminal | Alacritty |
| App Launcher | tofi |
| Status Bar | Waybar |
| Notifications | mako |
| Audio | PipeWire (pipewire-pulse + wireplumber) |
| Clipboard | wl-clipboard |
| Fonts | Inconsolata (text), Font Awesome (waybar icons) |

## Hardware Target
Laptop - includes brightness control (brightnessctl), battery status, lid handling. Debian's `brightnessctl` package ships no udev rule (unlike Arch's), so `desktop-setup` installs `etc/udev/rules.d/90-backlight.rules` and adds the user to the `video` group to allow brightness control without root; requires re-login to take effect.

## Sway Defaults
- Mod key: Super/Logo
- Wallpaper: solid solarized base03 (#002b36)
- Startup brightness: 15% via brightnessctl
- Display scaling: `output <name> scale 1.5` (HiDPI panel)
- Borders: 4px pixel border, no gaps
- Workspaces: 5 (matches dwm's 5 tags)
- Tiling: autotiling daemon — split direction chosen automatically by container aspect ratio; Mod+t = split/tile (`layout toggle split`), Mod+m = fullscreen toggle, Mod+f = floating toggle
- Keybindings: letters remapped to match this repo's companion dwm config — Mod+p launcher, Mod+q kill, Mod+Shift+q exit session; Mod+Shift+c reload is a sway-only addition with no dwm analog
- Focus/move: Mod+Arrow keys move focus between tiled windows, Mod+Shift+Arrow keys move the focused window; focus still follows mouse hover (sway default), tiled window borders resize by mouse drag
- Touchpad: tap-to-click via `input type:touchpad { tap enabled }`
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
`client.focused` / focused-workspace color `#174956` is a custom dark cyan (not a standard Solarized value), carried over from the companion dwm config's selected-window scheme.

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
- `tofi/config` - App launcher styling

### System Files (etc/)
- `udev/rules.d/90-backlight.rules` - Grants the `video` group write access to backlight brightness, installed by `desktop-setup`
