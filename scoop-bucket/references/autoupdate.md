# Scoop checkver & autoupdate: Reference

Condensed from official Scoop documentation and repository practices.

---

## 1. Running Maintenance Scripts

Scripts in `./bin/` wrap Scoop's internal maintainer utilities:

```powershell
.\bin\checkver.ps1 <app>         # Check single app
.\bin\checkver.ps1 *             # Check all apps in bucket
.\bin\checkver.ps1 <app> -u      # Update manifest with autoupdate
.\bin\checkver.ps1 <app> -f -u   # Force update even if version hasn't changed
.\bin\checkhashes.ps1            # Verify hashes of manifests
.\bin\formatjson.ps1             # Format JSON manifests to Scoop standard
```

---

## 2. Configuring `checkver`

### GitHub Releases Standard
```json
"checkver": {
    "github": "https://github.com/owner/repo"
}
```
*Note*: `github` matches release tags. It **ignores pre-releases**.

### GitHub API (When pre-releases or custom tags exist)
```json
"checkver": {
    "url": "https://api.github.com/repos/owner/repo/releases",
    "jsonpath": "$[0].tag_name",
    "regex": "v?([\\d.]+)"
}
```

### Regex on Homepage / HTML
```json
"checkver": {
    "url": "https://example.com/downloads",
    "regex": "App-v(?<version>[\\d.]+)-win64\\.zip"
}
```

---

## 3. Configuring `autoupdate`

```json
"autoupdate": {
    "url": "https://github.com/owner/repo/releases/download/v$version/app-v$version.zip",
    "hash": {
        "url": "$url.sha256"
    },
    "extract_dir": "app-v$version"
}
```

### Hash Extraction Modes (`autoupdate.hash.mode`)
- Default (`extract`): Regex search on `hash.url`.
- `json`: JSONPath query on `hash.url`.
- `download`: Downloads full binary and computes SHA256 locally. Used as default fallback if `hash` block is omitted or fails.

### GitHub API Asset Digest Mode
```json
"autoupdate": {
    "url": "https://github.com/owner/repo/releases/download/v$version/app-$version.zip",
    "hash": {
        "url": "https://api.github.com/repos/owner/repo/releases/tags/v$version",
        "jsonpath": "$.assets[?(@.name == '$basename')].digest"
    }
}
```
