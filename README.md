# zcecc22 sway config

Minimalist [Sway](https://swaywm.org/) desktop on Debian Stable. Solarized Dark across every tool. One role per tool, no overlap.

This repo is desktop-environment config only — it does not manage shell, editor, or dev-tool dotfiles.

## Stack

| Role | Tool |
|---|---|
| Window Manager | Sway (Wayland) + sway-masterstack |
| Terminal | Alacritty |
| Status Bar | Waybar |
| App Launcher | tofi |
| Audio | PipeWire |
| Fonts | Inconsolata, Font Awesome |

## Design decisions

- **No display manager** — Sway starts directly from a TTY via `~/.bin/start-desktop`.
- **Network and power management are out of scope** — the base system is expected to already have networking (no NetworkManager — `ifupdown2` + `wpa_supplicant`) and TLP/tlp-pd set up; `desktop-setup` only installs Sway and its companion apps.
- **Solarized Dark everywhere** — consistent palette across Alacritty, Waybar, and tofi.
- **Screen blanks but never locks** — `swayidle` powers the display off after 5 minutes idle, mirroring dwm's `xset dpms 300 600 600`; there's no lock daemon, matching dwm (which has none either).
- **No notification daemon** — mako was removed; dwm has none either.
- **Terminal font is smaller than the rest of the UI** — Alacritty runs at 12pt Inconsolata; Sway's titlebars and Waybar are both 20pt. Intentional, not a mismatch.

## Prerequisites

- Debian Stable (trixie)
- TTY login — no display manager needed or installed

## Install

Clone into your home directory:

```bash
git clone git@github.com:zcecc22/sway.git ~/sway
```

Run the setup script:

```bash
~/sway/.bin/desktop-setup
```

Then symlink or copy `.config/*` and `.bin/*` into place (e.g. `~/.config`, `~/.bin`).

## Usage

Start Sway from a TTY:

```bash
~/.bin/start-desktop
```

### Key bindings

`Super` is the mod key.

| Key | Action |
|---|---|
| `Super+Return` | Open terminal |
| `Super+p` | App launcher |
| `Super+q` | Close window |
| `Super+1–5` | Switch workspace |
| `Super+Shift+1–5` | Move window to workspace |
| `Super+t` | Return to master-stack (tile) mode |
| `Super+m` | Toggle monocle mode |
| `Super+f` | Toggle floating |
| `Super+Tab` | Cycle: promote bottom-of-stack window into master (tile mode) / swap visible window (monocle mode) |
| `Super+Shift+c` | Reload config |
| `Super+Shift+q` | Exit Sway |

Focus still follows the mouse, and tiled window borders resize by dragging with the mouse — no dedicated keybinding for resizing. Floating windows have a titlebar to drag, and can also be moved with `Super`+left-drag or resized with `Super`+right-drag from anywhere on the window.

Brightness and volume keys work out of the box via `brightnessctl` and `wpctl`.

## Structure

```
~
├── .bin/
│   ├── desktop-setup                # Desktop env installer
│   ├── start-desktop                # Launch Sway
│   └── sway-masterstack             # dwm-style master/stack tiling daemon
├── .config/
│   ├── sway/config                  # Window manager
│   ├── alacritty/alacritty.toml     # Terminal
│   ├── waybar/config                # Status bar modules
│   ├── waybar/style.css             # Status bar styling
│   └── tofi/config                  # App launcher
└── etc/
    └── udev/rules.d/90-backlight.rules  # Backlight group perms (installed by desktop-setup)
```

## License

Public domain. Use as you like.
