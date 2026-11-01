# Persistent Data & PowerShell Scripts: Reference

Condensed from the Scoop wiki ("Persistent data", "Pre/Post (un)install scripts") and repository practices.

---

## 1. Persistent Data (`persist`)

Data that must survive updates lives in `~\scoop\persist\<app>\` (or `C:\ProgramData\scoop\persist\<app>` for global installs). Inside scripts, this path is available as `$persist_dir`.

### Declaration Syntax

```json
"persist": [
    "settings.json",
    "data_folder",
    [ "config/app.cfg", "app.cfg" ]
]
```
- A single string persists a file or directory relative to `$dir`.
- A 2-element array `[ "source", "target" ]` renames the item inside `$persist_dir`.

### Link Types
- **Directories**: Scoop creates Directory Junctions (`New-Item -ItemType Junction`).
- **Files**: Scoop creates Hard Links (`New-Item -ItemType HardLink`).

---

## 2. Common Persist Traps & Fixes

### Trap A: File Persist when File Does Not Exist Yet
Scoop requires hard link target files to exist on disk. If an app creates its config file only on first launch:
- **Solution**: Pre-create the file or directory in `pre_install` before Scoop attempts to persist it, or manage the junction manually in `pre_install` / `post_install`.

### Trap B: App Writes Profiles to `%APPDATA%` or `%LOCALAPPDATA%`
If an app ignores its install folder and writes to `%APPDATA%\<AppName>` or `%LOCALAPPDATA%\<AppName>`:
- **Option 1**: Check if a CLI flag exists (e.g. `firefox -profile "$persist_dir\profile"`).
- **Option 2**: Pass arguments via `bin` shim:
  ```json
  "bin": [
      [ "app.exe", "app", "--config \"$persist_dir\\config.json\"" ]
  ]
  ```
- **Option 3 (Junction Hack)**: Manually link `%APPDATA%\<AppName>` to `$persist_dir` in `post_install` / `pre_install` and clean up link in `uninstaller.script`.
  *Real Example from `spotifast.json`:*
  ```powershell
  $link = "$env:APPDATA\paolino\spotifast\config"
  $target = "$persist_dir\config"
  if (!(Test-Path "$target")) { New-Item "$target" -ItemType Directory -Force | Out-Null }
  New-Item -Path $link -ItemType Junction -Value $target | Out-Null
  ```

---

## 3. Script Hooks & Variables

Available hooks: `pre_install`, `installer.script`, `post_install`, `pre_uninstall`, `uninstaller.script`, `post_uninstall`.

### Available Script Variables

| Variable | Description |
|---|---|
| `$app` | Application manifest name (e.g. `spotifast`) |
| `$architecture` | `64bit` or `32bit` |
| `$cmd` | Current subcommand: `install`, `update`, or `uninstall` |
| `$version` | Version string being installed |
| `$global` | `$true` if installed with `--global` |
| `$dir` | Application installation directory (see quirk below) |
| `$persist_dir` | Application persist directory (`~\scoop\persist\<app>`) |
| `$bucketsdir` | Scoop buckets path (`~\scoop\buckets`) |
| `$manifest` | Deserialized manifest PowerShell object |

### The `$dir` Variable Quirk
- In `pre_install`, `pre_uninstall`, `post_uninstall`, `installer.script`, `uninstaller.script`: `$dir` expands to `~\scoop\apps\<app>\<version>`.
- In `post_install`: `$dir` expands to `~\scoop\apps\<app>\current`.

---

## 4. Helper Functions inside Scripts

- `appdir <app>`: Returns the base install directory of another installed Scoop app.
  ```powershell
  if (Test-Path "$(appdir vscodium)\current\bin\codium.cmd") { ... }
  ```

---

## 5. Directory Junctions vs Hard Links Management

### Junctions (Directories)
- **Create**: `New-Item -ItemType Junction -Path "<link>" -Target "<target>"`
- **Remove**: `cmd /c rmdir "<link>"` OR `(Get-Item "<link>").Delete()`
- **NEVER**: Do NOT run `Remove-Item -Recurse "<link>"` on a junction in Windows PowerShell 5.1! It will delete the contents of the target folder!

### Hard Links (Files)
- **Create**: `New-Item -ItemType HardLink -Path "<link>" -Target "<target>"`
- **Remove**: `Remove-Item "<link>"` (Safe for hard linked files).
