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
- **Terminal font is smaller than the rest of the UI** — Alacritty runs at 12pt Inconsolata; Sway's titlebars and tofi are 20pt (Pango points). Waybar is sized in CSS pixels instead (`font-size: 20px`, since GTK CSS doesn't share Pango's point-size property) — chosen to land close to the same visual size at this display's scale, not a literal unit match.
- **Alacritty's bright ANSI colors deliberately diverge from a literal Solarized mapping** — `bright.green/yellow/blue/cyan` repeat their normal-intensity values instead of using base1/base01/base0/base00 (which the canonical mapping would produce, but reads as washed-out grey), and `bright.black` is base01 rather than base03 (so it stays visible instead of blending into the background). `bright.red`/`bright.magenta` do follow the canonical mapping (orange, violet). This is the well-known variant most Solarized terminal themes ship instead of the literal 16-color mapping.

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

## How the master-stack tiling works

Sway has no dwm-style master/stack layout built in, so `sway-masterstack` is a small daemon (~700 lines of Python, using [i3ipc](https://github.com/altdesktop/i3ipc-python)) that layers one on top. It runs continuously, subscribed to Sway's IPC event stream, and re-tiles on every window open, close, float, and move — none of the tiling behavior described here is native Sway, it's all enforced by this daemon watching and reacting.

The model it enforces, independently per workspace:

- **`nmaster=1`, one master and one stack** — the most recently created or promoted window is always master; whatever was master gets demoted to the top of the stack.
- **`mfact=0.55`** — master gets the wider share, sized so both master and stack columns clear an 80-col terminal at the panel's full width.
- **`Super+Tab` promotes, rather than just cycling focus** — it pulls the *bottom* of the stack into master, mirroring dwm's `cyclemaster()`, not the more common "step focus to the next window" binding.
- **Monocle is built on the scratchpad, since Sway has no equivalent** — Sway's tabbed/stacking layouts always draw a tab strip, and dwm's monocle draws none. `Super+m` toggles monocle on the focused workspace: entering moves every window but the focused one into the scratchpad; exiting (`Super+m` again, or `Super+t`) brings them back and rebuilds the master/stack invariant from marks each window kept while hidden.
- **The current mode is exposed for the status bar** — `sway-masterstack status` prints `[]=` or `[M]` for the focused workspace, feeding waybar's `custom/layout` module. The daemon signals waybar (`pkill -RTMIN+8 waybar`) on every mode change and workspace switch instead of waybar polling on a timer — the same feedback dwm's own bar gives for free.

Sway's tree has no first-class notion of "the master" or "the stack," so the daemon tracks both with workspace-namespaced marks (`_ms_master_<workspace>`, `_ms_stack_<workspace>`) and re-derives layout from those marks after every event, rather than keeping a separate in-memory model that could drift from a tree it doesn't fully control. If you're adapting `.bin/sway-masterstack` for your own setup, the source has inline notes on a few of the sharper edges this surfaced along the way — sway marks being unique tree-wide (not per-workspace) by default, and a daemon/CLI race specifically on monocle exit — worth reading before changing the mark-handling code.

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

[Unlicense](LICENSE) — public domain. Use as you like.
