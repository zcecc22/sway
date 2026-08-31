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
| Window Manager | Sway (Wayland) + sway-masterstack (master-stack tiling daemon) |
| X11 Compatibility | XWayland |
| Terminal | Alacritty |
| App Launcher | tofi |
| Status Bar | Waybar |
| Audio | PipeWire (pipewire-pulse + wireplumber) |
| Clipboard | wl-clipboard |
| Fonts | Inconsolata (text), Font Awesome (waybar icons) |

## Hardware Target
Laptop - includes brightness control (brightnessctl), battery status, lid handling. Debian's `brightnessctl` package ships no udev rule (unlike Arch's), so `desktop-setup` installs `etc/udev/rules.d/90-backlight.rules` and adds the user to the `video` group to allow brightness control without root; requires re-login to take effect.

## Sway Defaults
- Mod key: Super/Logo
- Wallpaper: solid solarized base03 (#002b36)
- Startup brightness: 15% via brightnessctl
- Display scaling: `output <name> scale 1.25` (HiDPI panel)
- Borders: 4px pixel border on tiled windows (no gaps); floating windows use a `normal` border instead, restoring the titlebar as a drag handle
- Workspaces: 5 (matches dwm's 5 tags)
- Tiling: `sway-masterstack` daemon (i3ipc-based) — dwm-style master/stack layout, `nmaster=1`, `mfact=0.5` (even split, deliberately diverging from the companion dwm config's `mfact=0.6`); Mod+Tab cycles the bottom-of-stack window into master (dwm `cyclemaster` parity), Mod+m toggles monocle mode (scratchpad-based, since Sway has no native equivalent — sway's native fullscreen toggle is no longer bound to anything), Mod+t returns to master-stack mode, Mod+f = floating toggle. A `sway-masterstack status` subcommand prints `[M]`/`[]=` for the focused workspace, driving a `custom/layout` waybar module — dwm-bar parity for the current mode.
- Floating windows: `floating_modifier $mod normal` — Mod+left-drag moves, Mod+right-drag resizes, in addition to the titlebar
- Keybindings: letters remapped to match this repo's companion dwm config — Mod+p launcher, Mod+q kill, Mod+Shift+q exit session; Mod+Shift+c reload is a sway-only addition with no dwm analog
- Focus: `Mod+Tab` is the only keyboard focus-cycling binding (no directional arrow-key focus/move, matching dwm which has none either); focus still follows mouse hover (sway default), tiled window borders resize by mouse drag
- Touchpad: tap-to-click via `input type:touchpad { tap enabled }`
- Idle: `swayidle` powers the display off after 300s (5 min) inactivity, matching dwm's `xset dpms 300 600 600` standby timeout, and powers it back on on resume or before-sleep; no lock daemon, matching dwm (which has none either)
- Notifications: no notification daemon — mako was removed; dwm has none either
- DBus/systemd env: `dbus-update-activation-environment` propagates `WAYLAND_DISPLAY`, `DISPLAY`, `XDG_CURRENT_DESKTOP` on startup, required by dbus-activated systemd user units (PipeWire, WirePlumber) — no portal package (`xdg-desktop-portal-wlr`) is installed, so this is not currently doing anything for screen sharing/file pickers
- Fonts: Alacritty (terminal) is 12pt Inconsolata; Sway's own UI font (titlebars) and Waybar are both 20pt — an intentional split, not an inconsistency

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
- `sway-masterstack` - dwm-style master/stack tiling daemon (i3ipc); `cycle`/`monocle`/`tile` subcommands drive Mod+Tab/Mod+m/Mod+t

### Config Files (.config/)
- `sway/config` - Sway WM configuration
- `alacritty/alacritty.toml` - Terminal configuration
- `waybar/config` - Status bar modules
- `waybar/style.css` - Status bar styling
- `tofi/config` - App launcher styling

### System Files (etc/)
- `udev/rules.d/90-backlight.rules` - Grants the `video` group write access to backlight brightness, installed by `desktop-setup`
