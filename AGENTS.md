# AGENTS.md - Agent Instructions for Kanata Keymap

This repository contains Kanata configuration files (`.kbd`) for keyboard remapping. Agents operating in this repository must adhere to the following guidelines to ensure configuration validity and maintainability.

## 1. Build, Lint, and Test Commands

There is no traditional build system. Kanata configuration is validated by the Kanata binary itself.

### Configuration Validation (Linting)
To verify if a configuration file is syntactically correct and logical:
```bash
kanata --check <file_path>.kbd
```
**Always run this command before committing any changes to a `.kbd` file.**

### Testing Changes
Testing requires running Kanata on a compatible machine. Since this is an agentic environment, you likely cannot "run" the keymap to test physical feel. Focus on structural correctness:
- Ensure every `deflayer` or `deflayermap` has the correct number of keys matching `defsrc`.
- Ensure all aliases (`@name`) used in layers are defined in `defalias` or through templates.
- Ensure all variables (`$name`) used are defined in `defvar`.

---

## 2. Code Style Guidelines

### S-Expression Formatting
Kanata uses S-expressions (Lisp-like syntax).
- **Indentation:** Use 4 spaces for nested blocks.
- **Key Grouping:** In `defsrc` and `deflayer`, group keys by their physical rows to maintain a visual representation of the keyboard layout.
- **Closing Parentheses:** Closing parentheses can be on the same line or a new line depending on the block size, but keep it consistent within the file.

### Naming Conventions
- **Aliases:** Use `kebab-case` for alias names (e.g., `@home-row-mod`).
- **Variables:** Use `kebab-case` for variables in `defvar` (e.g., `$tap-timeout`).
- **Templates:** Use `kebab-case` for template names (e.g., `charmod`).
- **Layers:** Use descriptive names like `base`, `extend`, `symbols`, `fun`.

### Imports and Modularity
Kanata does not support file imports natively in the `.kbd` files. Each `.kbd` file must be self-contained.
- `kanata.kbd`: Basic configuration.
- `kanata_adv.kbd`: Advanced configuration using templates, chords, and variables.
- `kanata_win.kbd`: Windows-specific configuration.

### Types and Logic
- **Home-row Mods:** Use `tap-hold` or `tap-hold-release-timeout` for home-row modifiers. 
- **Timings:** Prefer using variables (`defvar`) for timings to allow global adjustments. Typical values:
    - Tap timeout: 200ms
    - Hold timeout: 300ms
- **Templates:** Use `deftemplate` to DRY (Don't Repeat Yourself) repetitive logic like home-row mod definitions.

### Error Handling
- Kanata configs are sensitive to the number of arguments in commands.
- If a layer has a mismatching number of keys compared to `defsrc`, Kanata will error.
- Use `_` (underscore) in layers to indicate "transparent" (pass-through to lower layer).
- Use `XX` to indicate a disabled key.

---

## 3. Project Specific Patterns

### Home-Row Modifier Symmetry
Maintain strict symmetry between left and right hands for modifiers:
- Left: `a` (alt), `s` (met), `d` (ctl), `f` (sft)
- Right: `;` (alt), `l` (met), `k` (ctl), `j` (sft)

### Template: `charmod`
The advanced configurations use a `charmod` template for home-row mods that includes "streak" detection (disabling the mod during fast typing).
```lisp
(deftemplate charmod (char mod)
  (switch
    ((key-timing $streak-count less-than $streak-time)) $char break
    () (tap-hold-release-timeout $tap-timeout $hold-timeout $char $mod $char reset-timeout-on-press) break
  )
)
```

### Chords and Virtual Keys
Use `defchords` for simultaneous key presses. This is useful for high-frequency actions like `ESC` or `Backspace`.
- Chords should be defined per hand or per region (e.g., `mtl` for main top left).
- Virtual keys (`defvirtualkeys`) can be used to manage complex state transitions, such as switching layers and releasing keys simultaneously.

### Layer Mapping
Prefer `deflayermap` over `deflayer` for new configurations as it is more robust to layout changes (keys are matched by name rather than position).

---

## 4. Workflows and Implementation Steps

### Adding a New Layer
When adding a new functionality (e.g., a "Navigation" layer):
1.  **Define the layer name** in a descriptive way (e.g., `nav`).
2.  **Determine the trigger**: Will it be a `layer-toggle`, `layer-while-held`, or `layer-switch`?
3.  **Use `deflayermap`**:
    ```lisp
    (deflayermap (nav)
      h left
      j down
      k up
      l rght
    )
    ```
4.  **Add the trigger key** to the `base` (or `main`) layer.
5.  **Validate**: Run `kanata --check <file>.kbd`.

### Adjusting Timings
If home-row mods feel sluggish or trigger accidentally:
- Increase `$streak-time` to make streak detection more aggressive (easier to avoid mods while typing fast).
- Decrease `$tap-timeout` for faster tapping response.
- Adjust `$hold-timeout` if modifiers are triggering too late.

---

## 5. Documentation and Layout Visualization

### File Comments
- Use `;` for single-line comments and `;;` for header or block-level comments.
- Keep comments focused on *why* a specific mapping or timing is used.
- Refer to `WARP.md` for specific workflow guidance related to the environment.

### Physical Representation
In `defsrc`, always maintain the physical rows of the keyboard. This helps agents and humans visualize the mapping.
```lisp
(defsrc
  q w e r t   y u i o p
  a s d f g   h j k l ;
)
```

---

## 6. OS Specifics and Repository Structure

### Repository Files
- `kanata.kbd`: The stable, minimal configuration. Use this for standard testing.
- `kanata_adv.kbd`: The experimental/feature-rich version. Most development happens here.
- `kanata_win.kbd` / `kanata_adv_win.kbd`: Windows variants. 

### OS Nuances
- **macOS:** Requires `macos-dev-names-include`. Be careful with the internal keyboard name.
- **Windows:** Certain keys like `lsgt` (Less-than/Greater-than) may require special handling or might not exist on all physical layouts. Use `XX` if a key is unavailable on a specific OS.

---

## 7. Safety and Verification

### Pre-Commit Checklist
1.  **Syntax Check**: `kanata --check <file>.kbd`
2.  **Symmetry Check**: Ensure home-row mods are balanced.
3.  **Layer Completion**: Ensure every key intended for use is defined or explicitly set to `_`.
4.  **No Dead Ends**: Ensure there is always a way to get back to the `base` layer (e.g., `layer-switch main` or `on-release`).

---
*This file is intended for agentic consumption. Last updated: 2026-01-17*
