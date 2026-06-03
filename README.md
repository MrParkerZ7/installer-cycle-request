# installer-cycle-request

**Release distribution for [cycle-request](https://github.com/MrParkerZ7/project-cycle-request).**

This repository holds no source code — it is the public home for the packaged
**installers and portable `.zip` builds** of the cycle-request desktop app. Every
release here is **built and published automatically** from the source repo; nothing
is committed by hand.

## Where releases come from

The source lives in **`project-cycle-request`**. Pushing a `vX.Y.Z` tag there runs
its `Release` GitHub Actions workflow, which builds the Electron app on
win / macOS / Linux runners and publishes a GitHub Release **here** with:

| Platform | Installer | Portable |
|----------|-----------|----------|
| Windows  | `*.exe` (NSIS) | `*-win.zip` |
| macOS (arm64) | `*.dmg` | `*-mac.zip` |
| Linux    | `*.AppImage` | `*-linux.zip` |

Plus `latest*.yml` auto-update metadata.

## Getting a build

Grab the asset for your platform from the [latest release](../../releases/latest):

- **Windows** — run the `.exe` installer (choose your install dir), or unzip the
  portable `.zip` and run `cycle-request.exe`.
- **macOS** — open the `.dmg` and drag to Applications, or unzip the `.zip`.
- **Linux** — `chmod +x` the `.AppImage` and run it, or unzip the `.zip`.

## How a release is cut (maintainers)

From the source repo, via the prompt-master `release` command family — bump the
version, tag `vX.Y.Z`, push; the workflow does the rest. See
`project-cycle-request`'s release workflow + the `release-*` commands.

**One-time setup:** the source repo needs a `RELEASE_PAT` secret — a token with
**Contents: read & write** on this repo — so its workflow can publish here
(`GITHUB_TOKEN` is scoped to its own repo only).
