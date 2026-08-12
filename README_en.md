# win_iso_build

[中文说明](https://github.com/adavak/win_iso_build/blob/main/README_cn.md)

Automated Windows ISO build pipeline via GitHub Actions — downloads the official ISO, integrates latest patches, and publishes the result.

## Three-repo architecture

| Repo | Role |
|------|------|
| [Win_ISO_Patching_Scripts](https://github.com/adavak/Win_ISO_Patching_Scripts) | Patch manifests (meta4) + integration scripts (W10UI) |
| **win_iso_build** ← this repo | Actions pipeline: fetch base ISO → verify → patch → release |
| [win_iso_zip](https://github.com/adavak/win_iso_zip) | Base ISO storage + patched ISO releases |

```
Win_ISO_Patching_Scripts  ──(dispatch)──→  win_iso_build  ──(download)──→  win_iso_zip
     (meta4 manifests)                       (Actions build)                  (base ISO + releases)
```

## Trigger

Manual only: `workflow_dispatch`. Normally triggered via API by `Win_ISO_Patching_Scripts` release events.

## Configuration

The build matrix is defined in `.github/version-config.json`:

- `versions` → product name and ISO label
- `locales` → language list (`zh-CN`, `en-US`)
- `releases` → per locale/version: `release_tag` (tag in win_iso_zip) and `expected_hash` (SHA256 of the base ISO)

## Pipeline

1. `generate-matrix` — cleanup old runs, build matrix from `version-config.json`
2. `build` (matrix parallel):
   - Compare meta4 hash vs previous Release → skip if unchanged
   - Download split ISO from `win_iso_zip` → merge with 7z
   - Verify SHA256 → run `Start.cmd` to integrate patches
   - Rename ISO + split (2000MB) → create Release
