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
| Status Bar | swaybar (native Sway bar) + sway-status |
| Audio | PipeWire (pipewire-pulse + wireplumber) |
| Clipboard | wl-clipboard |
| Fonts | Inconsolata |

## Hardware Target
Laptop - includes brightness control (brightnessctl) and battery status. Debian's `brightnessctl` package ships no udev rule (unlike Arch's), so `desktop-setup` installs `etc/udev/rules.d/90-backlight.rules` and adds the user to the `video` group to allow brightness control without root; requires re-login to take effect. Lid-switch behavior is left to systemd-logind's own default policy (`HandleLidSwitch=suspend`) rather than configured here — out of scope for this repo the same way network and power management are.

## Sway Defaults
- Mod key: Super/Logo
- Wallpaper: solid solarized base03 (#002b36)
- Startup brightness: 50% via brightnessctl
- Display scaling: `output <name> scale 1.25` (HiDPI panel)
- Borders: 4px pixel border on tiled windows (no gaps); floating windows use a `normal` border instead, restoring the titlebar as a drag handle
- Workspaces: 9
- Tiling: `sway-masterstack` daemon (i3ipc-based) — dwm-style master/stack layout, `nmaster=1`, `mfact=0.55` (master gets the wider share, sized so both master and stack columns clear an 80-col terminal at the panel's full width); Mod+Tab cycles the bottom-of-stack window into master (dwm `cyclemaster` parity), Mod+m toggles monocle mode (scratchpad-based, since Sway has no native equivalent — sway's native fullscreen toggle is no longer bound to anything), Mod+t also returns to master-stack mode (an explicit exit, alongside Mod+m's toggle), Mod+f = floating toggle. A `sway-masterstack status` subcommand prints `[M]`/`[]=` for the focused workspace, and a `sway-masterstack title` subcommand prints the focused window's title; `sway-status` shells out to both as the first two segments of the swaybar status line — dwm-bar parity for the current mode and window title, now that native swaybar has no custom-module or window-title slot of its own. The daemon wakes `sway-status` immediately on every mode change and workspace switch by writing to a FIFO (`$XDG_RUNTIME_DIR/sway-status.fifo`) that `sway-status`'s main loop blocks on, rather than it polling `sway-masterstack status`/`title` on a timer — deliberately not an OS signal, since a signal delivered into the running bash script can interrupt bash's own read of its script source mid-reparse and kill the process (confirmed live, reproducible from a quick burst of workspace switches).
- Floating windows: `floating_modifier $mod normal` — Mod+left-drag moves, Mod+right-drag resizes, in addition to the titlebar
- Keybindings: letters remapped to match this repo's companion dwm config — Mod+p launcher, Mod+q kill, Mod+Shift+q exit session; Mod+Shift+c reload is a sway-only addition with no dwm analog
- Focus: `Mod+Tab` is the only keyboard focus-cycling binding (no directional arrow-key focus/move, matching dwm which has none either); focus still follows mouse hover (sway default), tiled window borders resize by mouse drag
- Touchpad: tap-to-click and middle-click emulation via `input type:touchpad { tap enabled; middle_emulation enabled }`
- Idle: `swayidle` powers the display off after 300s (5 min) inactivity, matching dwm's `xset dpms 300 600 600` standby timeout, and powers it back on on resume or before-sleep; no lock daemon, matching dwm (which has none either)
- Notifications: no notification daemon — mako was removed; dwm has none either
- DBus/systemd env: `dbus-update-activation-environment` propagates `WAYLAND_DISPLAY`, `DISPLAY`, `XDG_CURRENT_DESKTOP` on startup, required by dbus-activated systemd user units (PipeWire, WirePlumber) and by the `xdg-desktop-portal`/`xdg-desktop-portal-wlr`/`xdg-desktop-portal-gtk` packages `desktop-setup` installs, which use it to pick up the Wayland session for screen sharing/file pickers
- Fonts: Alacritty (terminal) is 12pt Inconsolata — an intentional split from the rest of the UI, not an inconsistency. Sway's own UI font (titlebars), tofi, and the swaybar status line are all point-sized at 20 (Pango points, `font pango:Inconsolata 20`) — the same literal font directive in every case now that swaybar is native Sway UI rather than a separate GTK app with its own CSS unit system

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

Alacritty's bright ANSI colors deliberately diverge from a literal Solarized mapping: `bright.green/yellow/blue/cyan` repeat their normal-intensity values (the canonical mapping would use base1/base01/base0/base00, which reads as washed-out grey in a terminal), and `bright.black` is base01 rather than base03 (so it stays visible). `bright.red`/`bright.magenta` do follow the canonical mapping (orange, violet). This is the well-known variant most Solarized terminal themes ship instead of the literal 16-color mapping.

## Repository Structure

### Scripts (.bin/)
- `desktop-setup` - Desktop environment package installation
- `start-desktop` - Wayland env vars, startup brightness, exec sway
- `sway-masterstack` - dwm-style master/stack tiling daemon (i3ipc); `cycle`/`monocle`/`tile` subcommands drive Mod+Tab/Mod+m/Mod+t; `status`/`title` subcommands feed `sway-status`
- `sway-status` - swaybar status line script (BAT/BRI/VOL/NET/clock, plus sway-masterstack's layout indicator and focused window title); polls every 1s, SIGUSR1-driven immediate refresh from sway-masterstack

### Config Files (.config/)
- `sway/config` - Sway WM configuration
- `alacritty/alacritty.toml` - Terminal configuration
- `tofi/config` - App launcher styling

### System Files (etc/)
- `udev/rules.d/90-backlight.rules` - Grants the `video` group write access to backlight brightness, installed by `desktop-setup`
