# Blink Shell GPL Builder

Build and sideload Blink Shell (iOS terminal with Mosh/SSH) using free Apple Personal Team signing.

## Overview

This skill automates the process of building and installing **Blink Shell** (a GPL-licensed iOS terminal app) on your iPhone using free Apple developer signing. Perfect for developers who need SSH/Mosh access on iOS without paying for an Apple Developer Program membership.

- **Fork repo**: [github.com/JonesHong/Blink-Shell-GPL-Builder](https://github.com/JonesHong/Blink-Shell-GPL-Builder)
- **Upstream**: [github.com/blinksh/blink](https://github.com/blinksh/blink) (GPL v3)
- **Tested version**: v18.3.0

## Quick Start

### Prerequisites
- macOS with Xcode installed
- Apple ID signed into Xcode
- iPhone paired via USB or same Wi-Fi network

### One-Click Install
```bash
git clone https://github.com/JonesHong/Blink-Shell-GPL-Builder.git
cd Blink-Shell-GPL-Builder
./setup.sh
```

## Key Commands

| Task | Command |
|------|---------|
| Health check | `./check.sh` |
| Reinstall (same version) | `./reinstall-blink.sh` |
| Full rebuild | `./setup.sh` |
| Silent rebuild (auto-resign) | `./reinstall-blink-auto.sh` |

## Features

- **7-day certificate**: Free Apple signing works for 7 days before needing re-signing
- **Auto-resign**: Optional launchd setup for automatic re-signing every 5 days
- **No costs**: Uses free Apple Personal Team (no Developer Program fee)
- **Patch management**: Handles version upgrades with automatic patch detection

## Constraints & Limitations

- **7-day certificate validity** — no workaround without a paid account
- **Maximum 3 sideloaded apps** per free Apple ID
- **Local network required** — Mac and iPhone must be on same Wi-Fi
- **Device trust needed** — Must approve developer on device after install

## Resource Links

For detailed documentation on:
- Version upgrades and patch generation
- Troubleshooting and common errors
- Auto-resign setup and launchd configuration

See the [fork repository](https://github.com/JonesHong/Blink-Shell-GPL-Builder) for full guides.

## License

This skill is provided as-is. Blink Shell itself is GPL v3. Fork is maintained for free sideloading workflows.
