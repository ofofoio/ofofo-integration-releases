# Ofofo Integration Agent — public release metadata

**Current installers (GitHub Release):** [v1.0.4](https://github.com/ofofoio/ofofo-integration-releases/releases/tag/v1.0.4) — see `VERSION.txt` for the version last exported from the app repo.

This repository holds **`latest-*.yml`** (and other small files) for auto-updates. It stays **separate** from the main app source so you can keep code private while exposing only update metadata.

### Auto-update (important)

The packaged app uses **electron-updater’s generic provider** against **raw.githubusercontent.com** (`mac/latest-mac.yml`, `windows/latest.yml`). Those YAML files list installers with **full `https://github.com/.../releases/download/...` URLs**, because large binaries are **not** stored in Git.

Every **GitHub Release** must still attach the real `.dmg` / `.exe` / `.zip` assets. Older releases that omitted `latest-mac.yml` as a release asset caused **404** errors for the default GitHub provider; the generic + raw-git path avoids depending on that file existing as an asset.

## Why installers are not in Git

GitHub **rejects any file over 100 MB** in standard Git. Full **`.dmg` / `.zip` / `.exe`** builds are usually larger than that, so they **must not be committed**. Use **[GitHub Releases](https://docs.github.com/en/repositories/releasing-projects-on-github/managing-releases-in-a-repository)** to upload installers as release assets.

After you create a release and upload assets, edit the exported `latest-*.yml` so **`path` / `url` point to the asset download URLs** (or use `electron-builder` publish config that emits those URLs). Then commit **only** the YAML files here.

## Layout

| Path | Contents (what to commit) |
|------|-------------------------|
| `mac/` | `latest-mac.yml` (and optional small checksum files) |
| `windows/` | `latest.yml` or platform-specific YAML |
| `linux/` | `latest-linux.yml`, etc. |
| `other/` | Anything the export script could not classify |

`electron-updater` resolves installer URLs **relative to the YAML file** when you use relative `path` entries; if binaries live on GitHub Releases, use **full HTTPS URLs** in the YAML instead.

`VERSION.txt` is written on each export from the integration app’s `package.json` version (for humans / release notes).

## Populate from the integration app (local)

In **`ofofo_AI_Integration`**, after a successful package:

```bash
npm run package:mac      # or package:win / package:linux / package:all
npm run release-export
```

By default this copies from `ofofo_AI_Integration/release/` into this repo (sibling folder). Override the destination:

```bash
OFOFO_RELEASE_REPO=/path/to/this/repo npm run release-export
```

Large artifacts remain on disk under `mac/` / `windows/` / `linux/` but are **ignored by Git** (see `.gitignore`). Upload those files to a **GitHub Release**, then commit the YAML:

```bash
cd /path/to/ofofo-integration-releases
git add mac windows linux VERSION.txt
git status   # confirm only .yml / small files are staged
git commit -m "Release v1.0.1 — update metadata"
git push
```

**Do not press Ctrl+Z during `git push`** — that suspends the process and leaves the remote unchanged. Let the command finish, or use Ctrl+C to cancel cleanly.

## Point the app at this repo

Use the **generic** update provider with a **public HTTPS base URL** per platform, for example:

- macOS: `https://raw.githubusercontent.com/<org>/<repo>/main/mac`  
  (so `latest-mac.yml` is at `.../mac/latest-mac.yml`)

Configure `autoUpdater.setFeedURL` in the main process to that base (or adjust `electron-builder` / app config accordingly).

## Version

Bump **`package.json`** `version` in the integration app before packaging so `latest-*.yml` matches the release you export.
