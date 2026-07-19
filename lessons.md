# Lessons Learned

<!-- Append new entries below. Format:

### YYYY-MM-DD — Brief title
- **Friction**: What went wrong or was suboptimal
- **Fix**: How it was resolved
- **Rule**: Generalizable takeaway for future invocations
-->

### 2026-04-22 — Xcode 26.4 + Runestone 0.5.1 availability build error
- **Friction**: Reinstall with Blink v18.3.0 failed under Xcode 26.4.1 (Swift 6.3). Runestone 0.5.1 `UITextSearchingHelper.swift:170` placed `@available(iOS 16, *)` on the method instead of the extension — stricter Swift 6.3 protocol conformance check rejects it: `protocol 'UIFindInteractionDelegate' requires 'findInteraction(_:sessionFor:)' to be available in iOS 14.0 and newer`.
- **Fix**: Bumped Runestone `1fad339 (0.5.1)` → `592434a (0.5.2)` in `blink-src/Blink.xcodeproj/project.xcworkspace/xcshareddata/swiftpm/Package.resolved`, deleted `DerivedData/Blink-*/SourcePackages/checkouts/Runestone`, rebuilt. Upstream 0.5.2 moved `@available(iOS 16, *)` to the extension level, which is exactly what Swift 6.3 requires.
- **Rule**: When a Blink build fails with a Runestone (or any SwiftPM dep) availability error under a new Xcode, first check if the dep has a newer patch version and edit `Package.resolved` directly rather than rebasing the entire Blink patch. `Package.resolved` is gitignored upstream, so a fresh `setup.sh` clone will naturally pick up the newest dep — this only manifests when reusing an existing `blink-src/`.

### 2026-04-22 — Xcode update leaves iOS platform uninstalled
- **Friction**: After updating to Xcode 26.4.1, `xcodebuild ... -destination 'generic/platform=iOS'` failed with "iOS 26.4 is not installed. Please download and install the platform from Xcode > Settings > Components." Every paired device was marked Ineligible.
- **Fix**: `xcodebuild -downloadPlatform iOS` (8.5 GB download, ~5 min on 15 MB/s connection) installs the iOS 26.4.1 runtime. Build works afterwards.
- **Rule**: `check.sh` should flag missing iOS platform before attempting a build. Also worth noting: `xcrun devicectl list devices` is the authoritative device-detection command; `check.sh`'s older heuristic falsely reports "No paired iPhone/iPad found" even when devices are paired and available.

### 2026-04-22 — v18.6.0 is a branch, not a release tag
- **Friction**: `check.sh` reported "Latest: v18.6.0 -- update available" and SKILL.md talked about upgrading to new versions. Investigation showed Blink publishes versions as **branches** (`refs/heads/v18.6.0`), not tags, and v18.6.0 at that point had no git tag → it was likely pre-release.
- **Fix**: Avoided the upgrade path; fixed the underlying Runestone issue instead.
- **Rule**: Before committing to a full patch-rebase upgrade workflow (30-60 min of manual work), investigate whether the new "version" is an actual release. Blink's `git ls-remote --tags` may be empty; `--heads` lists all in-flight branches. A branch head is not necessarily stable.
