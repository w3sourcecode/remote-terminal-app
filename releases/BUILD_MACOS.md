# Building macOS desktop packages

**For most users:** download the pre-built DMGs from [GitHub Releases](https://github.com/w3sourcecode/remote-terminal-app/releases) (Apple Silicon and Intel). No npm required.

**For maintainers:** macOS installers can be built on a Mac locally, or via GitHub Actions — see [RELEASE_CI.md](RELEASE_CI.md).

## Requirements

- macOS 11 or later
- Node.js 22+
- Xcode Command Line Tools

## Build DMG (recommended)

```bash
cd desktop
npm install
npm run build:mac
```

Output: `desktop/release-builds/<timestamp>/`

| Mac | File |
|-----|------|
| Apple Silicon | `arm64/Remote Terminal-<version>-mac-arm64.dmg` |
| Intel | `x64/Remote Terminal-<version>-mac-x64.dmg` |

## Build ZIP (portable)

```bash
npm run build:mac:zip
```

## First launch (unsigned builds)

macOS may block unsigned apps. Open **System Settings → Privacy & Security** and allow **Remote Terminal**, or right-click the app → **Open**.

## Publishing to this repo

After building, upload the `.zip` or `.dmg` to [GitHub Releases](https://github.com/w3sourcecode/remote-terminal-app/releases) with a name like:

`Remote-Terminal-<version>-macOS-arm64.zip`
