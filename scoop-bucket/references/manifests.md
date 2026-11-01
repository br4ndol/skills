# Scoop App Manifests & Field Substitution: Reference

Condensed from official Scoop documentation and repository practices.

---

## 1. Required Manifest Fields

- `version`: Version string (e.g., `"1.2.3"` or `"0.290"`).
- `homepage`: Canonical web address for the app.
- `license`: SPDX identifier string or object `{ "identifier": "...", "url": "..." }`.

---

## 2. Common Manifest Fields

- `description`: Single line summary in English.
- `url`: Download URL or array of URLs.
- `hash`: SHA256 string (or array matching URLs).
- `extract_dir`: Subfolder to extract from archive.
- `bin`: Binary/script shim configuration.
  - String: `"bin": "app.exe"`
  - Single alias in outer array: `"bin": [ [ "app.exe", "alias", "args" ] ]`
- `shortcuts`: Start menu shortcuts `[ [ "target.exe", "Shortcut Name", "args", "icon.ico" ] ]`.
- `notes`: Single string or array of strings (in English).
- `suggest`: Suggest complementary packages `{ "Feature Name": [ "bucket/app" ] }`.
- `env_add_path`: Directory relative to `$dir` added to user PATH.
- `env_set`: Object with environment variables to set.

---

## 3. Single-Architecture Policy (`x86_64`)

In this bucket (`br4ndol`), 64-bit (`x86_64` / `win64`) is used as the target architecture.
Keep `url`, `hash`, and `extract_dir` at the top level of the manifest rather than wrapping them under `architecture.64bit`. This maintains cleaner and more concise JSON files.

---

## 4. Internal Substitution Variables

### Version Variables (e.g. `$version` = `1.2.3.4`)
- `$version`: `1.2.3.4`
- `$underscoreVersion`: `1_2_3_4`
- `$dashVersion`: `1-2-3-4`
- `$cleanVersion`: `1234`
- `$majorVersion`: `1`
- `$minorVersion`: `2`
- `$patchVersion`: `3`
- `$buildVersion`: `4`

### URL Variables (from `autoupdate.url`)
- `$url`: Full URL without fragments (`#/dl.7z`).
- `$baseurl`: Base directory URL without filename or fragments.
- `$basename`: Download filename (e.g., `app-v1.2.3.zip`).

### Captured Variables (from `checkver.regex` groups)
- Named groups `(?<name>...)`:
  - `checkver.replace`: `${name}`
  - `autoupdate`: `$matchName` (First letter after `match` is capitalized!).
- Unnamed groups `(...)`:
  - `checkver.replace`: `${1}`
  - `autoupdate`: `$match1`
