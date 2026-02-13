---
name: blink-builder
description: >-
  This skill should be used when the user asks to "build Blink Shell", "install Blink on iPhone",
  "sideload Blink", "Blink Shell GPL build", "build iOS terminal app", "建置 Blink",
  "安裝 Blink 到 iPhone", "Blink 編譯", "free sideload iOS app", mentions Blink Shell,
  or discusses building iOS apps from GPL source with free Apple developer signing.
version: 0.1.0
---

# Blink Shell GPL Builder

Build and sideload Blink Shell (iOS terminal with Mosh/SSH) using free Apple Personal Team signing.

- Fork repo: https://github.com/JonesHong/Blink-Shell-GPL-Builder
- Upstream: https://github.com/blinksh/blink (GPL v3)
- Tested version: v18.3.0

## Pre-Flight: Version & Health Check

**CRITICAL: Run these checks EVERY TIME this skill is invoked, BEFORE doing anything else.**

1. Check latest available Blink version:
   ```bash
   git ls-remote --heads https://github.com/blinksh/blink.git | grep -E 'v[0-9]+\.' | sed 's/.*refs\/heads\///' | sort -t. -k1,1n -k2,2n -k3,3n | tail -1
   ```

2. If `~/Blink-Shell-GPL-Builder/` exists, run the health check:
   ```bash
   bash ~/Blink-Shell-GPL-Builder/check.sh
   ```

3. Report findings to user before proceeding:
   - Current tested version vs latest available
   - Patch compatibility (flag if version mismatch detected)
   - Signing certificate expiry status
   - Device reachability

## Quick Path (Fork Already Cloned)

When `~/Blink-Shell-GPL-Builder/` exists:

| Task | Command |
|------|---------|
| Health check | `./check.sh` |
| Reinstall (same version) | `./reinstall-blink.sh` |
| Full rebuild | `./setup.sh` |
| Silent rebuild (launchd) | `./reinstall-blink-auto.sh` |
| Update to new version | Follow Version Upgrade Workflow below |

## Full Setup (From Scratch)

### Prerequisites

- macOS with Xcode installed (Xcode > Settings > Platforms > install iOS)
- Apple ID signed into Xcode (Xcode > Settings > Accounts)
- iPhone paired via USB once (Wi-Fi deployment works afterward)

### One-Click Install

```bash
git clone https://github.com/JonesHong/Blink-Shell-GPL-Builder.git
cd Blink-Shell-GPL-Builder
./setup.sh
```

`setup.sh` handles: clone source, apply patch, collect Team ID / Bundle ID / Device UDID, build, install.

### Modes

| Flag | Behavior |
|------|----------|
| `./setup.sh --auto` | Fully automated (default) |
| `./setup.sh --manual` | Step-by-step with prompts |
| `./setup.sh --dry-run` | Preview only, no changes |

## Version Upgrade Workflow

When a new Blink version is available and the current patch fails to apply:

1. Clone the new version to a temp directory.
2. Compare changed files against the patched version (especially `project.pbxproj`).
3. Apply the same logical modifications:
   - Remove unsupported entitlements from `Blink.entitlements` (iCloud, IAP, Push).
   - Remove `StoreKit.framework` and `CloudKit.framework` from the Frameworks build phase.
   - Replace all `DEVELOPMENT_TEAM` values with `$(TEAM_ID)`.
   - Add `DEVELOPMENT_TEAM` to any build configs missing it.
   - Disable iCloud and Push under `SystemCapabilities`.
   - Add `BLINK_PUBLISHING_OPTION_DEVELOPER` to `SWIFT_ACTIVE_COMPILATION_CONDITIONS`.
   - Add `noSubscriptionNag` check in `SceneDelegate._showPaywallIfNeeded()`.
4. Generate new patch:
   ```bash
   git diff HEAD > patches/xcode26-free-team.patch
   ```
5. Test build and install on device.
6. Commit updated patch to the fork.

For step-by-step details: read `references/patch-guide.md`

## Signing Lifecycle

| Day | Event |
|-----|-------|
| 0 | Build and install via `setup.sh` or `reinstall-blink.sh` |
| 5 | launchd auto-resign triggers (if configured) |
| 7 | Certificate expires; app stops launching |

Auto-resign setup: `setup.sh` Step 7 configures this, or set up manually via `docs/AUTO-REINSTALL.md` in the fork repo.

## Troubleshooting Quick Reference

| Error | Fix |
|-------|-----|
| "Personal Team doesn't support iCloud/IAP/Push..." | Patch removes these. Re-apply patch or verify entitlements. |
| "Signing requires a development team" | `DEVELOPMENT_TEAM` missing from build config. Check xcconfig. |
| StoreKit IAP capability required | Remove `StoreKit.framework` from Frameworks build phase (not just weak link). |
| Paywall "no valid product selected" | Requires `BLINK_PUBLISHING_OPTION_DEVELOPER` flag + SceneDelegate check. |
| Patch doesn't apply cleanly | Version mismatch. Follow Version Upgrade Workflow. |
| iPhone not found | Mac and iPhone must be on same Wi-Fi. Verify with `xcrun devicectl list devices`. |
| App crashes on launch after 7 days | Certificate expired. Run `./reinstall-blink.sh` to re-sign. |

For detailed troubleshooting with root cause analysis: read `references/troubleshooting.md`

## Key Files in Fork

| File | Purpose |
|------|---------|
| `setup.sh` | One-click build and install (auto/manual/dry-run) |
| `check.sh` | Health check (versions, signing, device, environment) |
| `reinstall-blink.sh` | Quick rebuild and reinstall |
| `reinstall-blink-auto.sh` | Silent rebuild for launchd auto-resign |
| `patches/xcode26-free-team.patch` | All source modifications for free signing |
| `templates/developer_setup.xcconfig.template` | Xcode build config template |
| `templates/com.blink-reinstall.plist.template` | launchd plist for auto-resign |

## Free Sideloading Constraints (2026)

- **7-day certificate validity** -- no workaround without a paid account
- **Maximum 3 sideloaded apps** per free Apple ID
- **Bonjour/mDNS required** -- Mac and iPhone must be on the same local network
- **Trust developer on device** -- Settings > General > VPN & Device Management

## Continuous Improvement

This skill evolves with each use. After every invocation:

1. **Reflect** — Identify what worked, what caused friction, and any unexpected issues
2. **Record** — Append a concise lesson to `lessons.md` in this skill's directory
3. **Refine** — When a pattern recurs (2+ times), update SKILL.md directly

### lessons.md Entry Format

```
### YYYY-MM-DD — Brief title
- **Friction**: What went wrong or was suboptimal
- **Fix**: How it was resolved
- **Rule**: Generalizable takeaway for future invocations
```

Accumulated lessons signal when to run `/skill-optimizer` for a deeper structural review.

## Additional Resources

### Reference Files
- **`references/troubleshooting.md`** -- Detailed build error solutions with root cause analysis
- **`references/patch-guide.md`** -- How to update patches for new Blink versions
