# Scoop Installers & Extraction: Reference

Condensed from official Scoop documentation and repository practices.

Scoop's philosophy strongly favors portable zip archives. When only installer executables or installers are available, Scoop provides mechanisms to extract or run them safely without corrupting the host OS.

## 1. Strategy Priority

1. **Portable Archive (Zip, 7z, Tar)**: Always prefer direct portable archives.
2. **Extractor Trick (`#/dl.7z`)**: If the download is a single `.exe` or `.msi` containing files (like 7-Zip, InnoSetup, NSIS), append `#/dl.7z` to the `url`. Scoop downloads it as `dl.7z` and 7-Zip extracts it automatically into `$dir`.
3. **MSI Extraction (Default behavior)**: `.msi` files specified without an `msi` property are automatically extracted by 7-Zip/MSI extractors into `$dir` without running MSIExec installation or registry modification.
4. **InnoSetup Installer**: Set `"innosetup": true` if InnoSetup is used and cannot be extracted via 7-Zip.
5. **Installer/Uninstaller Script**: Use `installer` and `uninstaller` directives as a last resort.

---

## 2. URL Extension Trick (`#/dl.7z`)

When an app releases a standalone installer or `.exe` that 7-Zip can unpack:

```json
"url": "https://example.com/download/app-setup.exe#/dl.7z",
"extract_dir": "core"
```

- Scoop saves the file as `dl.7z` locally.
- 7-Zip unpacks it.
- `extract_dir` selects the nested directory containing the binaries.

---

## 3. Installer Types & Behavior

### A. NSIS (Nullsoft Scriptable Install System)
- Often 7-Zip can unpack NSIS installers directly using `#/dl.7z`.
- If extraction isn't possible and `installer.script` or `installer.args` must be used:
  - Typical silent arguments: `[ "/S", "/D=$dir" ]`
  - Note: `/D=$dir` must be the last argument and must NOT contain quotes (e.g., `/D=C:\scoop\apps\app\1.0.0`).

### B. MSI (Microsoft Installer)
- **Modern Scoop Approach**: Treat `.msi` as an archive. Omit the deprecated `msi` block. Scoop unpacks `.msi` files into `$dir`.
- If a true MSI install is required via script:
  - Quiet flags: `/quiet`, `/qn`, `TARGETDIR="$dir"`.
  - Admin elevation: Note if an MSI requires admin, it may fail for non-admin Scoop users.

### C. Inno Setup
- Set `"innosetup": true` in the manifest root when 7-Zip cannot unpack it cleanly.
- Silent install arguments when using custom installer script: `/VERYSILENT`, `/SUPPRESSMSGBOXES`, `/NORESTART`, `/DIR="$dir"`.

### D. Direct Executables (`.exe`)
- Single standalone `.exe` files do not require extraction.
- If the binary name in the release includes version tags (e.g. `sonora-v0.45.0-x86_64-pc-windows-msvc.exe`), rename it in `pre_install`:
  ```json
  "pre_install": "Get-ChildItem \"$dir\" -Filter \"sonora-*.exe\" | Rename-Item -NewName \"sonora.exe\""
  ```

---

## 4. Common Installer Issues & Fixes

1. **Installer leaves shortcuts or registry keys outside `$dir`**:
   - Clean them up in `post_install` or `post_uninstall`.
2. **Elevated Privilege Prompts (UAC)**:
   - Prefer portable/extracted binaries so users don't need Administrator privileges.
3. **Installer keeping file on disk**:
   - Use `"keep": true` inside `installer` only if the uninstaller depends on keeping the installer executable. Otherwise, Scoop deletes the installer after execution.
