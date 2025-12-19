# WARP.md

This file provides guidance to WARP (warp.dev) when working with code in this repository.

## Repository overview
This repo is a single-purpose Kanata configuration:
- `kanata.kbd`: the entire keymap definition (device selection + physical key matrix + aliases + the `base` layer).

There are no build/lint/test scripts or project tooling in this repo.

## Common commands / workflows
No commands are defined in-repo. Usage depends on how Kanata is installed/launched on the machine (CLI invocation vs. service/daemon).

Typical workflow in this repo:
1. Edit `kanata.kbd`.
2. Restart/reload Kanata so it picks up the updated configuration.

If you need an exact launch/reload command, look for:
- a system service definition (e.g. launchd/systemd/Windows task), or
- a shell profile / dotfiles entry that starts Kanata.

## `kanata.kbd` architecture (big picture)
Kanata configs are declarative and usually follow this structure (this repo matches it):

### 1) `(defcfg ...)`: global behavior + device selection
- `process-unmapped-keys yes`: unmapped keys are still passed through.
- `macos-dev-names-include (...)`: a device allow-list for macOS. In this repo it includes:
  - `Apple Internal Keyboard / Trackpad`
  - `Kinesis Adv360`

If you add/remove keyboards you want this config to apply to on macOS, update this list.

### 2) `(defsrc ...)`: the physical key matrix
`defsrc` enumerates the physical keys this config expects (rows roughly correspond to a standard ANSI layout). Layer definitions must match this shape.

### 3) `(defalias ...)`: reusable behaviors (home-row mods)
This repo defines “home-row modifier” aliases using `tap-hold`:
- Left hand:
  - `a` → tap `a`, hold `lalt`
  - `s` → tap `s`, hold `lmet`
  - `d` → tap `d`, hold `lctl`
  - `f` → tap `f`, hold `lsft`
- Right hand (mirrored):
  - `;` → tap `;`, hold `lalt`
  - `l` → tap `l`, hold `lmet`
  - `k` → tap `k`, hold `lctl`
  - `j` → tap `j`, hold `lsft`

All aliases currently use the same timing: `tap-hold 200 200 ...`.

If you adjust home-row feel/latency, do it here so all layers inherit the change.

### 4) `(deflayer base ...)`: the active mapping
The `base` layer is mostly passthrough except that the home-row letters are replaced by the aliases (e.g. `@a`, `@s`, etc.).

When adding new layers, keep these conventions:
- Preserve the `defsrc` layout shape exactly.
- Prefer aliases (in `defalias`) for reusable behaviors instead of duplicating `tap-hold` logic across layers.
- Maintain left/right symmetry for home-row mods unless there’s a specific reason to diverge.
