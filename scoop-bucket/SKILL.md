---
name: scoop-bucket
description: Create, update, audit, and test Scoop app manifests (.json) and README entries in the br4ndol bucket. Use whenever creating a new Scoop manifest, updating an app, configuring checkver/autoupdate/persist, testing install/uninstall locally, or maintaining the br4ndol Scoop bucket repository.
---

# Scoop Bucket Manifest Maintenance (`br4ndol` bucket)

This skill provides step-by-step workflows, rules, and guidelines for creating, updating, auditing, testing, and maintaining Scoop app manifests (`.json`) and repository documentation in the `br4ndol` Scoop bucket.

---

## Repository Structure & File Permissions

```
scoop-bucket/
├── SKILL.md                          # Main skill instructions (workflow + core rules)
├── assets/
│   └── json-template.json            # Base JSON template for new manifests
├── references/
│   ├── schema.json                   # Official Scoop manifest JSON Schema (query via grep)
│   ├── manifests.md                  # Manifest fields, required properties, variables
│   ├── autoupdate.md                 # checkver and autoupdate configuration & maintenance
│   ├── installers.md                 # Archives, installers (NSIS, MSI, Inno), #/dl.7z trick
│   ├── persist-and-scripts.md        # Persistent data, junctions, hard links, script hooks
│   └── pitfalls-and-examples.md      # Edge cases, common pitfalls & real manifest examples
├── bucket/                           # READ/WRITE: Location of all Scoop app manifests (.json)
├── bin/                              # Maintenance scripts (checkver.ps1, checkhashes.ps1, formatjson.ps1)
├── scripts/                          # Helper scripts, registry entries (.reg) for apps
├── README.md                         # READ/WRITE: Bucket documentation and app listings
├── LICENSE                           # Repository license (DO NOT EDIT)
└── skills-lock.json                  # DO NOT EDIT MANUALLY
```

### Modification Rules:
- **Allowed to edit**: `bucket/*.json`, `README.md`, `scripts/<app>/*`, and files inside `.agents\skills\scoop-bucket\`.
- **STRICTLY PROHIBITED**: Do NOT edit `skills-lock.json` or `.github\workflows\excavator.yml` by hand.

---

## Reference Reading Guide

Read targeted reference files when performing specific tasks. Paths are relative to the skill base directory (`.agents\skills\scoop-bucket`):
- **Writing/editing manifest structure or fields**: Read `references/manifests.md`.
- **Configuring checkver or autoupdate**: Read `references/autoupdate.md`.
- **Handling installers (NSIS, MSI, InnoSetup) or extracted archives**: Read `references/installers.md`.
- **Configuring persist, directory junctions, hard links, or install scripts**: Read `references/persist-and-scripts.md`.
- **Troubleshooting bugs, edge cases, or reviewing full working examples**: Read `references/pitfalls-and-examples.md`.
- **Validating schema details**: Search `references/schema.json` using `grep` (do not read the whole file).

---

## Step-by-Step Manifest Workflow

### Step 1: Research Release & Collect Metadata
Gather required details from official sources:
- Latest version tag and download URLs.
- Available assets (Zip portable vs EXE/MSI installer).
- Target architecture (`x86_64` / `win64` preferred for this bucket).
- License (SPDX identifier or type: `MIT`, `GPL-3.0-or-later`, `Freeware`, `Proprietary`).
- Homepage URL.

### Step 2: Choose Strategy
1. **Portable Zip/7z/Tar**: Top priority.
2. **Extractor Trick (`#/dl.7z`)**: If only EXE installer is available, append `#/dl.7z` to download URL so 7-Zip unpacks it.
3. **MSI Extraction**: Omit `msi` block so Scoop extracts binaries without installation.
4. **Installer script**: Silent args (`/S`, `/VERYSILENT`) as last resort.

### Step 3: Draft Manifest from Template
- Copy `assets/json-template.json` to `bucket/<app-name>.json`.
- **Name convention**: Lowercase with hyphens (e.g. `spotifast.json`, `g-helper.json`).
- Remove all empty fields and template comments (`"##"`).

### Step 4: Configure `checkver` & `autoupdate`
- Use GitHub releases (`"checkver": { "github": "..." }`) when stable releases are present.
- Use GitHub API endpoint (`https://api.github.com/repos/.../releases`) with `jsonpath` if pre-releases or custom tags are used.
- Include `$version`, `$basename`, or captured groups in `autoupdate.url` and `autoupdate.hash`.

### Step 5: Obtain Real SHA256 Hash
- Download asset or compute hash using real SHA256.
- **NEVER** invent hashes or leave hash fields empty.

### Step 6: Validate & Local Testing
1. **Format JSON**: Run `.\bin\formatjson.ps1 <app>`.
2. **Check Hash & Ver**: Run `.\bin\checkver.ps1 <app>` and `.\bin\checkhashes.ps1 <app>`.
3. **Local Install**: `scoop install .\bucket\<app-name>.json`.
4. **Verify Persist & Links**:
   - Check junction: `Get-Item "<link>" | Select-Object LinkType, Target`.
   - Test uninstall: `scoop uninstall <app-name>` (data in `persist` must remain).
   - Test purge: `scoop uninstall -p <app-name>` (data in `persist` removed).

### Step 7: Update README.md
- Add app entry under `## 📦 Available applications` in **alphabetical order** by visible app name.
- Increment app badge counter: `![Apps](https://img.shields.io/badge/apps-N+1-success?style=for-the-badge)`.
- README entry format:
```html
<details>
<summary><b>App Display Name</b></summary>

<br>

```
scoop install br4ndol/app-name
```

🔗 [Website](https://homepage.url/)

</details>
```

### Step 8: Summary & Transparency
Report to the user:
- What was created or updated.
- What was verified (schema, checkver, local install/uninstall, persist links).
- What could not be verified (if any).

---

## Strict Rules & Conventions

1. **No Invented Hashes or URLs**: All URLs must exist and hashes must match the real file.
2. **Clean JSON**: No unused fields, no `"##"` comments, no empty strings.
3. **Commit Messages**: Use format `app: action` (e.g., `spotifast: update to 0.14.2`, `helium: add manifest`).
4. **Language**: `description` and `notes` fields in manifests must always be in English.
5. **Architectural Style**: Top-level single-arch (`x86_64`) placement for `url`, `hash`, `extract_dir`.

---

## Windows Link Operations (Junctions & Hard Links)

Scoop uses Directory Junctions for folders and Hard Links for files.

### PowerShell & CMD Link Commands
- **Create Junction (Folder)**:
  `New-Item -ItemType Junction -Path "C:\path\link" -Target "C:\path\target"`
- **Remove Junction (Folder)**:
  `cmd /c rmdir "C:\path\link"` OR `(Get-Item "C:\path\link").Delete()`
- **DANGER**: **NEVER** use `Remove-Item -Recurse` on a Directory Junction in PowerShell 5.1! It will delete the original target files!
- **Create Hard Link (File)**:
  `New-Item -ItemType HardLink -Path "C:\path\file.ext" -Target "C:\path\target.ext"`
- **Remove Hard Link (File)**:
  `Remove-Item "C:\path\file.ext"`
- **Check Link Type**:
  `Get-Item "C:\path\link" | Select-Object LinkType, Target`