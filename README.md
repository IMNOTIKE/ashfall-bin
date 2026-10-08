# Ashfall Releases

This repository hosts the **release artifacts and update metadata** for
[Ashfall](https://github.com/IMNOTIKE/AshFall), a free and open-source virtual
tabletop for running tabletop RPG sessions.

## What lives here

Every published **GitHub Release** in this repo contains:

- **Installers** for the platforms we ship:
  - `Ashfall-Setup-<version>.exe` — Windows NSIS installer
  - `Ashfall-<version>-macos-x64.dmg` / `...-arm64.dmg` — macOS DMGs (manual download)
  - `Ashfall-<version>-macos-x64.zip` / `...-arm64.zip` — macOS updater payloads
  - `Ashfall-<version>-linux-x64.AppImage` / `...deb` — Linux packages
- **Update metadata** consumed by the app's auto-updater:
  - `latest.yml` (Windows)
  - `latest-linux.yml` (Linux)
  - `latest-mac.yml` (macOS)
  - `*.blockmap` files (differential downloads for NSIS)

The repo is intentionally a plain, public file host for the updater — it is
**not** the source code (that lives in `IMNOTIKE/AshFall`).

## How updates work

Ashfall self-updates with [`electron-updater`](https://www.electron.build/auto-update),
using GitHub Releases as the distribution point:

1. At startup the app reads its bundled `app-update.yml` (written by
   electron-builder at build time), which points here — provider `github`,
   repo `IMNOTIKE/ashfall-bin`.
2. The app fetches the platform's `latest*.yml` file for the current release.
3. If the version there is newer than the installed version, the updater
   downloads the matching installer payload.
4. The new installer is applied (on Windows, at app quit; on Linux/macOS via the
   downloaded package).

`electron-updater` needs a **strictly increasing version number** — every new
Ashfall tag bumps the version, so old clients always find a newer release.

### Platform notes

| Platform | Update path | Notes |
| -------- | ----------- | ----- |
| Windows  | NSIS installer + differential blockmaps | Fully automatic via `latest.yml` |
| Linux    | AppImage (via `latest-linux.yml`) | AppImage updates require the app to have been installed from an AppImage |
| macOS    | zip + `latest-mac.yml` (Squirrel.Mac) | Auto-install requires the build to be **code-signed** (Developer ID cert); unsigned builds can download but not install |

## How releases get here

This repository is populated automatically by the CI pipeline in
`IMNOTIKE/AshFall`:

1. A version **tag** (`v1.2.3`) is pushed to the AshFall repository.
2. The `Release` workflow (`.github/workflows/release.yml`) builds installers
   for Windows, macOS and Linux on GitHub-hosted runners.
3. The workflow uploads the installers + update metadata to a new GitHub
   Release in this repository (version-matched to the tag).

No manual uploads are required — but there is nothing stopping maintainers from
attaching files by hand if ever needed.

## Notes for maintainers

- Releases are public and read-only for end users; only CI (or maintainers)
  writes here.
- The publish token is stored as the `ASHFALL_ARTIFACT_RELEASE_TOKEN` secret in
  the AshFall repository and needs `repo`/contents permissions on this repo to
  create releases and upload assets.
- Do not delete old releases: clients on older versions need them until they
  update.
- macOS auto-updates will only install on user machines when the pipeline is
  configured to code-sign (a Developer ID certificate); until then macOS
  users should grab the `.dmg` directly.
