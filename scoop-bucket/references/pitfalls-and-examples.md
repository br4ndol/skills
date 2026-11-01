# Common Pitfalls & Real Manifest Examples

Condensed from real manifest engineering experience in the `br4ndol` bucket.

---

## 1. Common Pitfalls & Edge Cases

### A. Persisting Files vs Directories
- **The Issue**: Scoop creates hard links for individual files listed in `persist`. If the file does not exist when Scoop runs `persist`, the hard link creation fails or creates a folder instead.
- **Rule**: If persisting a file, ensure it exists before installation finishes (or pre-create it in `pre_install`). If the file is modified by atomic write (app deletes and recreates file on save), hard links break! In such cases, persist parent directory or copy file back and forth in `pre_install` / `pre_uninstall` (like `prismlauncher.json`).

### B. Profile Duplication in AppData
- **The Issue**: Electron or GUI apps default to `%APPDATA%\<app>` or `%LOCALAPPDATA%\<app>` regardless of install location.
- **Solution**: Set up a Directory Junction in `post_install` / `pre_install` pointing `%APPDATA%\<app>` to `$persist_dir\<folder>`, and remove junction in `uninstaller.script` (see `spotifast.json` or `sonora.json`).

### C. NSIS Silent Installer vs Zip
- **The Issue**: Executable installers run silently with `/S`, but might write uninstaller registry keys, desktop shortcuts, or install into AppData.
- **Solution**: Prefer extracting the installer with `#/dl.7z` if 7-Zip can unpack it.

### D. MSI Needing Administrator Privileges
- **The Issue**: Running `.msi` via `msiexec` often requires Admin rights and fails in standard non-privileged Scoop.
- **Solution**: Scoop unpacks `.msi` archives automatically when `msi` block is omitted and no installer script is defined.

### E. GitHub API Pre-Releases in `checkver`
- **The Issue**: `"checkver": "github"` ignores pre-releases completely.
- **Solution**: Use GitHub API releases endpoint (`https://api.github.com/repos/.../releases`) with `jsonpath: "$[0].tag_name"` if pre-releases are needed.

### F. Asset Format Shift Across Versions
- **The Issue**: An app owner changes release format from `.zip` to `.msi` or changes naming schema (e.g. `app-v1.0.zip` to `App_1.0_x64.zip`).
- **Solution**: Ensure `autoupdate.url` handles naming correctly or update `autoupdate` regex matching.

---

## 2. Real Manifest Examples from `br4ndol` Bucket

### Example 1: Portable App with GitHub Digest (`spotifast.json`)
```json
{
    "version": "0.14.2",
    "description": "Spotify, native and fast",
    "homepage": "https://spotifast.rocks/",
    "license": {
        "identifier": "MIT",
        "url": "https://github.com/crmne/spotifast/blob/main/LICENSE"
    },
    "url": "https://github.com/crmne/spotifast/releases/download/v0.14.2/spotifast-v0.14.2-x86_64-pc-windows-msvc.zip",
    "hash": "6dec8f9daa7eda5a30ffbd38f6cfba6d77bd3efe4a43754bd457437a63be432c",
    "extract_dir": "spotifast-v0.14.2-x86_64-pc-windows-msvc",
    "bin": "spotifast.exe",
    "shortcuts": [
        [
            "spotifast.exe",
            "Spotifast"
        ]
    ],
    "persist": [
        "config",
        "data",
        "cache"
    ],
    "checkver": {
        "github": "https://github.com/crmne/spotifast"
    },
    "autoupdate": {
        "url": "https://github.com/crmne/spotifast/releases/download/v$version/spotifast-v$version-x86_64-pc-windows-msvc.zip",
        "hash": {
            "url": "https://api.github.com/repos/crmne/spotifast/releases/tags/v$version",
            "jsonpath": "$.assets[?(@.name == '$basename')].digest"
        },
        "extract_dir": "spotifast-v$version-x86_64-pc-windows-msvc"
    }
}
```

### Example 2: Installer Executable with Extractor Trick (`firefox.json`)
```json
{
    "version": "159.0",
    "description": "Popular open source web browser.",
    "homepage": "https://www.firefox.com/",
    "license": "MPL-2.0",
    "url": "https://archive.mozilla.org/pub/firefox/releases/159.0/win64/en-US/Firefox%20Setup%20159.0.exe#/dl.7z",
    "hash": "sha512:fabd5c7608aec3d6945c2a2038a32cc67b159bd36997598725c90cb042e86a2570a97cc0b1bb4763643ab42dda71c00e4fa242fd52c63b57c6cec9d70b3c1aa5",
    "extract_dir": "core",
    "bin": [
        [
            "firefox.exe",
            "firefox",
            "-profile \"$persist_dir\\profile\""
        ]
    ],
    "shortcuts": [
        [
            "firefox.exe",
            "Firefox",
            "-profile \"$persist_dir\\profile\""
        ]
    ],
    "persist": [
        "distribution",
        "profile"
    ],
    "checkver": {
        "url": "https://product-details.mozilla.org/1.0/firefox_versions.json",
        "jsonpath": "$.LATEST_FIREFOX_VERSION"
    },
    "autoupdate": {
        "url": "https://archive.mozilla.org/pub/firefox/releases/$version/win64/en-US/Firefox%20Setup%20$version.exe#/dl.7z",
        "hash": {
            "url": "https://archive.mozilla.org/pub/firefox/releases/$version/SHA512SUMS",
            "regex": "$sha512\\s+win64/en-US/$basename"
        }
    }
}
```
