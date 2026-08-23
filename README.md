# zcecc22 sway config

Minimalist [Sway](https://swaywm.org/) desktop on Debian Stable. Solarized Dark across every tool. One role per tool, no overlap.

This repo is desktop-environment config only — it does not manage shell, editor, or dev-tool dotfiles.

## Stack

| Role | Tool |
|---|---|
| Window Manager | Sway (Wayland) |
| Terminal | Alacritty |
| Status Bar | Waybar |
| App Launcher | tofi |
| Notifications | mako |
| Screen Lock | swaylock + swayidle |
| Audio | PipeWire |
| Browser | Firefox ESR |
| Fonts | Inconsolata, Font Awesome |

## Design decisions

- **No display manager** — Sway starts directly from a TTY via `~/.bin/start-desktop`.
- **No NetworkManager** — `ifupdown2` and `wpa_supplicant` are sufficient for a single machine and keep the service footprint minimal.
- **Solarized Dark everywhere** — consistent palette across Alacritty, Waybar, mako, swaylock, and tofi.

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
| `Super+d` | App launcher |
| `Super+Shift+q` | Close window |
| `Super+Arrow` | Focus window |
| `Super+Shift+Arrow` | Move window |
| `Super+1–9` | Switch workspace |
| `Super+Shift+1–9` | Move window to workspace |
| `Super+f` | Fullscreen |
| `Super+Space` | Toggle floating |
| `Super+Shift+Space` | Toggle focus tiling / floating |
| `Super+r` | Resize mode (arrows to resize, Esc to exit) |
| `Super+l` | Lock screen |
| `Super+Shift+c` | Reload config |
| `Super+Shift+e` | Exit Sway |

Brightness and volume keys work out of the box via `brightnessctl` and `wpctl`.

### Idle behavior

| Idle time | Action |
|---|---|
| 5 min | Screen locks |
| 10 min | Display powers off |
| On suspend | Locks automatically |

## Network

Managed with `ifupdown2` and `wpa_supplicant`.

- **Wired**: configure `/etc/network/interfaces`
- **WiFi**: add credentials to `/etc/wpa_supplicant/wpa_supplicant.conf`, then bring up the interface with `ifup`

## Structure

```
~
├── .bin/
│   ├── desktop-setup                # Desktop env installer
│   └── start-desktop                # Launch Sway
└── .config/
    ├── sway/config                  # Window manager
    ├── alacritty/alacritty.toml     # Terminal
    ├── waybar/config                # Status bar modules
    ├── waybar/style.css             # Status bar styling
    ├── mako/config                  # Notification daemon
    ├── swaylock/config              # Lock screen
    └── tofi/config                  # App launcher
```

## License

Public domain. Use as you like.
