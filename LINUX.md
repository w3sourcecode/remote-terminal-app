# Remote Terminal on Linux

Run the Remote Terminal agent on a Linux server, VPS or desktop and control it from your phone. On a server there is no window: the agent prints a **QR code and a 9-digit pairing code** in your SSH session, and can run as a **systemd service** so it survives reboots and crashes.

[← Back to the main page](README.md)

## Contents

1. [Requirements](#requirements)
2. [Download and install](#download-and-install)
3. [Run it](#run-it)
4. [Pair your phone](#pair-your-phone)
5. [Keep it running (systemd service)](#keep-it-running-systemd-service)
6. [Command reference](#command-reference)
7. [Settings without a desktop](#settings-without-a-desktop)
8. [Where things are stored](#where-things-are-stored)
9. [Troubleshooting](#troubleshooting)
10. [Uninstall](#uninstall)

## Requirements

- Debian, Ubuntu or a derivative, **x86-64 (amd64)**. Other distributions are not packaged yet.
- `sudo` access to install.
- Outbound internet access (cloud mode), or an open port on your LAN (local network mode).
- The Remote Terminal app on your phone ([Google Play](https://play.google.com/store/apps/details?id=com.billten.remoteterminal) / [App Store](https://apps.apple.com/pk/app/remote-terminal-pro/id6771671841)).

## Download and install

Download the latest package:

**[RemoteTerminal-Linux-x64.deb](https://github.com/w3sourcecode/remote-terminal-app/releases/latest/download/RemoteTerminal-Linux-x64.deb)**

Or fetch it straight onto the server:

```bash
curl -fL -o remote-terminal.deb \
  https://github.com/w3sourcecode/remote-terminal-app/releases/latest/download/RemoteTerminal-Linux-x64.deb
```

Install it with `apt`, **not** `dpkg -i`:

```bash
sudo apt install ./remote-terminal.deb
```

The agent embeds a Chromium/Electron runtime that links against GTK and X11 libraries, even in headless mode. `apt install ./file.deb` downloads those for you; `dpkg -i` does not and stops with *"dependency problems - leaving unconfigured"*. If you already ran `dpkg -i`, recover with:

```bash
sudo apt --fix-broken install
```

This installs the app to `/opt/remote-terminal/` and the `remote-terminal` command to `/usr/bin` and `/usr/local/bin`.

## Run it

The agent picks its mode from the environment.

**On a server, over SSH, or in a container** (no display):

```bash
remote-terminal --headless
```

Leaving off `--headless` does the same when no `$DISPLAY` is set. This runs as a plain Node.js process: no window, no X11, roughly 40 MB of RAM.

**On a Linux desktop** (GNOME, KDE, XFCE, …):

```bash
remote-terminal
```

This opens the dashboard window and tray icon, like the Windows and macOS apps.

## Pair your phone

In headless mode the agent prints a box like this, followed by a QR code:

```
╔════════════════════════════════════════════════════════════════════╗
║                  REMOTE TERMINAL LINUX AGENT                       ║
╠════════════════════════════════════════════════════════════════════╣
║  Session Code  : 825 - 020 - 051                                   ║
║  Status        : Waiting for phone scan                            ║
║  Mode          : Cloud Relay                                       ║
║  Host Machine  : ubuntu-vps (Linux x64)                            ║
║  Password      : Open (No password set)                            ║
╚════════════════════════════════════════════════════════════════════╝
```

1. Open Remote Terminal on your phone and tap **Scan QR**, then point the camera at the QR code in your terminal.
2. Or tap **Session ID**, enter the nine digits, and the join password if you set one.

Set a join password on any machine reachable from the internet:

```bash
remote-terminal --headless --password=MySecretPassword
```

Once paired, the session hub offers the terminal (bash/zsh), Files, Clipboard, Programs, Task Manager and Services.

## Keep it running (systemd service)

A service keeps the phone connection alive after you close SSH, starts the agent at boot, and restarts it if it crashes.

```bash
# System-wide service (/etc/systemd/system/remote-terminal.service)
sudo remote-terminal service install

# Or a per-user service (~/.config/systemd/user/remote-terminal.service)
remote-terminal service install
```

Manage it:

```bash
sudo remote-terminal service status
sudo remote-terminal service stop
sudo remote-terminal service start
sudo remote-terminal service restart
sudo remote-terminal service uninstall
```

Read the logs:

```bash
sudo journalctl -u remote-terminal -f          # system service
journalctl --user -u remote-terminal -f         # user service
```

## Command reference

```bash
remote-terminal help                          # all commands
remote-terminal status                        # settings paths and connection info
remote-terminal --mode=local --port=5085      # local network mode instead of the cloud relay
remote-terminal --password=MySecretPassword   # require a join password
remote-terminal --shell=/bin/zsh              # preferred shell
remote-terminal --verbose                     # debug logging
remote-terminal config                        # list and change settings
remote-terminal reset                         # close a stuck pairing session
```

## Settings without a desktop

A server has no settings screen, so use `remote-terminal config`:

```bash
remote-terminal config                        # every setting and its value
remote-terminal config get usage
remote-terminal config set usage on
remote-terminal config unset shell            # back to the default
```

Most settings take effect within a second on a running agent. `mode`, `port`, `shell`, `autostart` and `password` are read at startup, so restart the agent after changing them.

| Setting | Default | What it does |
| :--- | :--- | :--- |
| `usage` | `off` | Share CPU, RAM, disk and network usage with the phone |
| `files` | `on` | Let the phone browse and transfer files |
| `clipboard` | `off` | Share this machine's clipboard with the phone |
| `programs` | `on` | Let the phone list, start and stop programs |
| `services` | `on` | Let the phone list, start, stop and restart services |
| `tasks` | `on` | Let the phone see running tasks and end them |
| `shell` | auto-detected | Shell for new terminals, for example `/bin/bash` |
| `terminal-path` | home directory | Directory new terminals start in |
| `blocklist` | empty | Comma-separated commands the phone may not run |
| `password` | not set | Join password the phone must enter |
| `quit-password` | not set | Password needed to quit or stop the agent |
| `autostart` | `off` | Start at login on desktops (use `service install` on a server) |
| `mode` | `cloud` | Cloud relay or local network API |
| `port` | `5085` | Port for local network mode |

Booleans accept `on`/`off`, `true`/`false`, `yes`/`no`, `1`/`0`. `config get password` shows only whether a password is set, never the password.

The phone hides the **PC usage** panel until sharing is switched on:

```bash
remote-terminal config set usage on
```

If you run the agent with `sudo` or as a system service, run `config` with `sudo` too. Otherwise the two read different settings folders and your change appears to do nothing. `remote-terminal config` prints the folder it is using.

## Where things are stored

| What | Path |
| :--- | :--- |
| Settings and history | `~/.config/remote-terminal/` |
| Log file | `~/.config/remote-terminal/logs/agent.log` |
| System service unit | `/etc/systemd/system/remote-terminal.service` |
| User service unit | `~/.config/systemd/user/remote-terminal.service` |
| Application files | `/opt/remote-terminal/` |

When running as root, `~` is `/root`.

## Troubleshooting

**`error while loading shared libraries` (for example `libgtk-3.so.0`)**
The package was installed without its dependencies. Run `sudo apt --fix-broken install`, and use `apt install ./file.deb` next time. To list what is missing:

```bash
ldd /opt/remote-terminal/remote-terminal-agent | grep "not found"
```

**`Running as root without --no-sandbox is not supported`**
The `remote-terminal` command adds the flag for root automatically. If you start the raw binary yourself, pass `--no-sandbox`.

**`Missing X server or $DISPLAY`**
Use `remote-terminal --headless`, which never starts the graphical runtime.

**The phone can't connect in local network mode**
Allow the port through the firewall:

```bash
sudo ufw allow 5085/tcp                                        # Ubuntu / Debian
sudo firewall-cmd --add-port=5085/tcp --permanent && sudo firewall-cmd --reload   # Fedora / RHEL
```

**"Session already connected on another device"**
A phone scanned the code and closed the app before finishing, which reserves the slot. Close it and start again:

```bash
remote-terminal reset
remote-terminal --headless
```

The pairing code stays the same.

**Service control says it needs elevation**
Listing system services works as any user, but starting and stopping them needs root. Run the agent as a system service (`sudo remote-terminal service install`). A per-user install can only control your own user units.

## Uninstall

```bash
sudo remote-terminal service uninstall
sudo apt remove remote-terminal
rm -rf ~/.config/remote-terminal
```
