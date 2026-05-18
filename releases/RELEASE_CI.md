# Automated macOS release builds

macOS installers are built on **GitHub Actions** (`macos-latest`) so users do not need npm or a local Mac to download them.

## One-time setup

In [remote-terminal-app](https://github.com/w3sourcecode/remote-terminal-app) → **Settings** → **Secrets and variables** → **Actions**, add:

| Secret | Purpose |
|--------|---------|
| `REMOTE_TERMINAL_SOURCE_PAT` | Classic PAT with `repo` scope for private repo `w3sourcecode/remote-terminal` |

Optional (source repo `remote-terminal` only): `RELEASE_PUBLISH_TOKEN` — PAT with `repo` on `remote-terminal-app` if you trigger builds from the source repo workflow instead.

## Attach Mac builds to an existing release

1. Ensure `desktop/package.json` version on `remote-terminal` **main** matches the release (e.g. `0.1.9` for tag `v0.1.9`).
2. Open **Actions** → **Attach macOS builds to release** → **Run workflow**.
3. Enter tag `v0.1.9` (or publish a new release to run automatically).

Artifacts uploaded:

- `Remote-Terminal-<version>-macOS-arm64.dmg`
- `Remote-Terminal-<version>-macOS-x64.dmg`
- `README-macOS.txt`

## First launch on Mac

Unsigned builds: right-click **Remote Terminal** → **Open**, or allow in **System Settings → Privacy & Security**.
