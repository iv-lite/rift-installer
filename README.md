# rift-installer

A niri-like window management setup for macOS, built on [Rift](https://github.com/acsandmann/rift) (a tiling and scrolling window manager) and [Ghostty](https://ghostty.org/) (terminal). Driven by the same Cmd-layered arrow chords as the previous Paneru config — Cmd+Option for columns/navigation, Cmd+Ctrl for displays, Ctrl+Option for workspaces — that don't fight macOS defaults.

## Requirements

- macOS 14+ (Rift is a universal binary for Apple Silicon and Intel)
- Homebrew (auto-installed if missing)
- "Displays have separate Spaces" enabled (Rift refuses to run without it — the installer sets it)
- No SIP disable required

## Quick start

```sh
./install
```

Re-running `./install` upgrades an existing setup: Homebrew components (Rift, Ghostty) are updated, configs are refreshed from this repo (previous copies kept as `*.bak`), and the service is restarted so the new binary/config apply immediately.

The installer runs these steps from `scripts/`:

| Step                | What it does                                                                  |
| ------------------- | ----------------------------------------------------------------------------- |
| `install-deps`      | Install Homebrew if missing                                                   |
| `configure-system`  | Enable "Displays have separate Spaces"                                        |
| `install-ghostty`   | Install Ghostty + write `~/.config/ghostty/config` (frameless title bar)      |
| `install-antigen`   | Install Antigen + write `~/.config/zsh/antigen.zsh` + wire it into `~/.zshrc` |
| `install-rift`      | Install Rift (`acsandmann/tap/rift`) + write `~/.config/rift/config.toml`     |
| `grant-permissions` | Open the Accessibility pane + print instructions (best-effort)                |
| `enable-services`   | `rift service install` + start (restart on re-run)                            |

## After install

1. Log out and back in (Cmd+Shift+Q) — applies the enabled separate-Spaces setting.
2. Rift tiles in an niri-style scrolling strip; new windows are appended at the end and never resize existing windows.
3. If the Accessibility grant failed, grant it manually: System Settings → Privacy & Security → Accessibility, then run `rift service restart`.
4. Ghostty opens frameless (`macos-window-buttons = hidden` in `~/.config/ghostty/config`).

## Keybindings

| Chord                                  | Action                                                              |
| -------------------------------------- | ------------------------------------------------------------------- |
| Cmd + Option + ←/↓/↑/→                 | Focus the window left/down/up/right                                 |
| Cmd + Option + Shift + ←/↓/↑/→         | Move the focused window (swap)                                      |
| Cmd + Option + W / + Shift + W         | Cycle column width forward / backward (0.3 / 0.5 / 1.0)             |
| Cmd + Option + Space / + Shift + Space | Center the focused column / snap the strip into view                |
| Ctrl + Option + ↑ / ↓                  | Previous / next workspace                                           |
| Ctrl + Option + Shift + ↑ / ↓          | Move the focused window to the previous / next workspace and follow |
| Cmd + Option + Tab                     | Switch to the last workspace                                        |
| Cmd + Ctrl + ← / →                     | Focus the previous / next display                                   |
| Cmd + Ctrl + Shift + ← / →             | Move the focused window to the previous / next display              |
| Cmd + Ctrl + ↑ / ↓                     | Warp the mouse to the display above / below                         |
| Cmd + Option + V                       | Toggle floating / tiled                                             |
| Cmd + Option + O / + Shift + O         | Stack into the neighbour column / pull out of a stack               |
| Cmd + Ctrl + T                         | Open / activate Ghostty                                             |
| Cmd + Option + Shift + R               | Reload the rift config                                              |
| Cmd + Option + Ctrl + Q                | Save and exit rift                                                  |
| 3-finger horizontal swipe              | Page through columns; overscroll past the strip switches workspace  |

## Window borders

Rift has no native window-border setting, so the niri-style colors this config would like to use are recorded as comments at the top of `config/rift/config.toml` and are **not** applied:

- `active-border` `#2b303cd4`
- `inactive-border` `#2b303cd6`

To draw real borders, run [JankyBorders](https://github.com/FelixKratz/JankyBorders) alongside rift:

```sh
brew tap FelixKratz/formulae && brew install borders
# ~/.config/borders/bordersrc:
#   active_color=0xd42b303c
#   inactive_color=0xd62b303c
```

## Troubleshooting

**Windows don't tile/move at all** — run these in order:

1. `rift-cli query displays` — JSON means the daemon is up; "cannot connect" means run `rift service restart`.
2. `rift-cli execute space toggle-activated`, then open two windows — if tiling starts, the current Space was inactive. Press `Alt + Z` to toggle it from the keyboard.
3. `tail -n 100 "/tmp/rift_$USER.err.log"` — check for config, permission, or macOS 27 errors.
4. `defaults read com.apple.spaces spans-displays` — must print `1`; if not, enable "Displays have separate Spaces" and log out/in.

If a Space is stuck inactive after a reinstall, press `Alt + Z` (the bundled activation key is restored in this config). On macOS 27, if automatic activation misbehaves, set `default_disable = true` in `config/rift/config.toml` and activate each Space manually with `Alt + Z`.

## Uninstall

```sh
./uninstall
```

Stops the rift service, moves installed configs to `~/.config/backups/uninstall-<timestamp>/`, removes the antigen block from `~/.zshrc`, and optionally uninstalls the `rift` formula. Ghostty and Antigen are left installed.

## Project layout

```
install                   Main installer (runs scripts/*)
uninstall                 Uninstaller (stop + backup + optional brew removal)
scripts/                  Per-component install/system/accessibility steps
config/rift/              Rift config (niri-like scrolling layout + keybindings)
config/ghostty/           Ghostty config (frameless title bar)
config/zsh/               Antigen bundles (oh-my-zsh + typewritten)
```

Configs are installed to `~/.config/{rift,ghostty,zsh}`; existing files are backed up (`.bak`) before overwriting. Rift hot-reloads `~/.config/rift/config.toml` on save.
