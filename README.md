# signal-updater

A lightweight macOS shell script that checks Signal Desktop's official update server, downloads the latest universal DMG, and installs it — with a native dialog prompt if Signal is currently running.

## Features

- Compares installed vs. latest version using Signal's own update manifest
- Downloads only when a newer version is available (skips if already downloaded)
- Prompts the user via macOS dialog if Signal is open, then quits and relaunches it automatically after updating
- Creates a backup of the current installation and restores it on failure
- Cleans up old Signal DMGs from `~/Downloads`
- Designed to run unattended via a LaunchAgent

## Requirements

- macOS
- Signal Desktop installed under `~/Applications/Signal.app`
- `curl`, `hdiutil`, `osascript`, `PlistBuddy` (all standard on macOS)

## Installation

Install with a single command — no need to clone the repo:

```bash
curl -fsSL https://raw.githubusercontent.com/CandymanRabbit/signal-updater/main/install | bash
```

To install **with scheduling**, pass flags after `bash -s --`:

```bash
# Every day at 09:00 (default)
curl -fsSL https://raw.githubusercontent.com/CandymanRabbit/signal-updater/main/install | bash -s -- -s

# Mon–Fri at 08:30
curl -fsSL https://raw.githubusercontent.com/CandymanRabbit/signal-updater/main/install | bash -s -- -s -d Mon-Fri -t 8:30am
```

<details>
<summary>Clone and install locally instead</summary>

```bash
git clone https://github.com/CandymanRabbit/signal-updater.git
cd signal-updater
chmod +x install signal-update
./install
```
</details>

### Schedule automatic updates

| Flag | Values | Description |
|------|--------|-------------|
| `-s` | — | Enable scheduling (default: every day at 09:00) |
| `-t` | `HH:MM` or `H:MMam/pm` | Override the time of day |
| `-d` | Day names, numbers, ranges, or comma lists | Restrict to specific days of the week |

**Day formats for `-d`:**  
- Names (case-insensitive): `Mon`, `Tue`, `Wed`, `Thu`, `Fri`, `Sat`, `Sun`  
- Numbers: `0`=Sun, `1`=Mon … `6`=Sat  
- Range: `Mon-Fri` or `1-5`  
- Comma list: `Mon,Wed,Fri`  
- Mixed: `Mon-Wed,Fri,6`  

```bash
./install -s                         # every day at 09:00
./install -s -t 14:30                # every day at 14:30
./install -s -t 8:30am               # every day at 08:30
./install -s -d Mon-Fri              # Mon–Fri at 09:00
./install -s -d Mon,Wed,Fri -t 7am   # Mon/Wed/Fri at 07:00
./install -s -d 1-5 -t 22:00         # Mon–Fri at 22:00
```

Scheduling uses a macOS **LaunchAgent** (`~/Library/LaunchAgents/com.user.signal-updater.plist`).  
Logs are written to `~/Library/Logs/signal-updater.log`.

### Uninstall

```bash
curl -fsSL https://raw.githubusercontent.com/CandymanRabbit/signal-updater/main/install | bash -s -- -u
```

Removes the installed script and unloads the LaunchAgent.

## Usage

Run manually at any time:
```bash
signal-update
```

## How it works

1. Reads the installed version from `Signal.app/Contents/Info.plist`
2. Fetches `https://updates.signal.org/desktop/latest-mac.yml`
3. If a newer version exists, downloads the universal DMG to `~/Downloads`
4. Mounts the DMG, copies `Signal.app` to `~/Applications`, then unmounts
5. If Signal was running: prompts the user → quits → installs → relaunches

## License

This project is licensed under the [Creative Commons Attribution-NonCommercial 4.0 International License](LICENSE).  
You may use, share, and adapt this work for **non-commercial purposes only**, with attribution.
