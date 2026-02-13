# Blink Shell GPL Build Troubleshooting

Comprehensive troubleshooting guide for building Blink Shell from GPL source.
Based on actual experience building Blink v18.3.0 on Xcode 26.2 with a free (Personal) Apple developer team.

---

## Issue 1: Personal Team Capability Restrictions

**Error:**
```
Cannot create a iOS App Development provisioning profile.
Personal development teams do not support the iCloud, In-App Purchase, Fonts,
Associated Domains, Default Web Browser, Font Enumeration, and Push Notifications capabilities.
```

**Root cause:** `Blink/Blink.entitlements` declares capabilities that require a paid Apple Developer account.

**Fix:** Remove these 8 entitlement keys from `Blink/Blink.entitlements`:

1. `aps-environment`
2. `com.apple.developer.associated-domains`
3. `com.apple.developer.icloud-container-identifiers`
4. `com.apple.developer.icloud-services`
5. `com.apple.developer.ubiquity-container-identifiers`
6. `com.apple.developer.ubiquity-kvstore-identifier`
7. `com.apple.developer.user-fonts`
8. `com.apple.developer.web-browser`

Also disable iCloud and Push in `SystemCapabilities` inside `Blink.xcodeproj/project.pbxproj`:
```
com.apple.Push = { enabled = 0; };
com.apple.iCloud = { enabled = 0; };
```

---

## Issue 2: StoreKit Framework Triggering IAP Capability Requirement

**Error:** Even after removing entitlements, Xcode still complains about the In-App Purchase capability.

**Root cause:** `StoreKit.framework` is explicitly linked in the Frameworks build phase. Xcode auto-detects this and requires the IAP capability, which Personal teams cannot provision.

**Key insight:** Weak linking (`Optional` status) does NOT fix this. The framework's presence in the build phase is what triggers the check.

**Fix:** Remove both `StoreKit.framework` AND `CloudKit.framework` from:
- The `PBXBuildFile` section (the build file entries)
- The `Frameworks` build phase (the references in the framework linking phase)

**Why it still works:** The Swift compiler auto-links frameworks when it encounters `import StoreKit` in source code. Explicit linking in the build phase is redundant for Swift sources.

---

## Issue 3: Subscription Paywall on First Launch

**Error:** App launches but shows a "Buy" dialog that spins forever, then displays "no valid product selected."

**Root cause (two parts):**

1. `PublishingOptions.current` defaults to `.appStore` unless the `BLINK_PUBLISHING_OPTION_DEVELOPER` compilation flag is set. The `.appStore` path tries to validate StoreKit subscriptions.
2. `FeatureFlags.noSubscriptionNag` was DEFINED in the codebase but NEVER REFERENCED in the actual paywall display code.

**Fix (both required):**

1. Add `BLINK_PUBLISHING_OPTION_DEVELOPER` to `SWIFT_ACTIVE_COMPILATION_CONDITIONS` in the Blink target's build settings (both Debug and Release configurations).

2. Add an early return in `SceneDelegate._showPaywallIfNeeded()`:
   ```swift
   private func _showPaywallIfNeeded() {
     if FeatureFlags.noSubscriptionNag {
       return
     }
     // ... existing paywall logic
   }
   ```

---

## Issue 4: DEVELOPMENT_TEAM Not Set

**Error:** "Signing requires a development team" for one or more targets/configurations.

**Root cause:** In `project.pbxproj`:
- 6 build configurations had a hardcoded team ID (`A2H2CL32AG` -- the Blink developers' team)
- 27 build configurations had no `DEVELOPMENT_TEAM` setting at all

**Fix:**
- Replace all hardcoded `DEVELOPMENT_TEAM = A2H2CL32AG` with `DEVELOPMENT_TEAM = $(TEAM_ID)`
- Add `DEVELOPMENT_TEAM = $(TEAM_ID)` to all 27 missing configurations
- The `$(TEAM_ID)` variable is resolved from `developer_setup.xcconfig`

---

## Issue 5: Bundle ID Mismatch

**Error:** Embedded binary bundle ID mismatch -- extension targets have bundle IDs that don't match the main app's bundle ID prefix.

**Fix:** Set a consistent `BUNDLE_ID` in `developer_setup.xcconfig`. The xcconfig value propagates to all targets via `$(BUNDLE_ID)` references in the build settings.

Example `developer_setup.xcconfig`:
```
TEAM_ID = YOUR_TEAM_ID
BUNDLE_ID = com.yourname.blink
```

---

## Issue 6: Patch Fails to Apply on Newer Blink Version

**Error:**
```
error: patch does not apply
```

**Root cause:** `project.pbxproj` structure changes between Blink versions -- new targets, reordered sections, changed line numbers. The patch context lines no longer match.

**Diagnosis:**
```bash
git apply --check patches/xcode26-free-team.patch
```
This will show which hunks fail without modifying files.

**Fix:** Follow the version upgrade workflow in `references/patch-guide.md` -- manually re-apply the logical changes to the new version and generate a fresh patch.

---

## Issue 7: GPL Builder sed Corrupting project.pbxproj

**Symptom:** The upstream GPL builder's `sed` replacement commands can break the pbxproj XML/plist structure. Xcode fails to open the project or shows unexpected errors.

**Root cause:** `sed` replacements on pbxproj are brittle -- they can match unintended lines or produce malformed output.

**Fix options:**
- Use `./setup.sh --overwrite` to re-clone from scratch and re-apply the patch
- Manually inspect and fix corrupted lines in `project.pbxproj`
- Use `git diff` to see what `sed` changed and revert specific damage

---

## Issue 8: Swift Package Resolution Failures

**Symptoms:**
- `swiftui-cached-async-image` fails to resolve because its `Package.swift` has issues with Swift 6
- `SwiftCBOR` tracks the `master` branch, causing unstable resolution

**Fix:**
```bash
# Clean SPM cache
rm -rf ~/Library/Caches/org.swift.swiftpm

# Then re-resolve
cd blink-src
xcodebuild -resolvePackageDependencies -project Blink.xcodeproj -scheme Blink
```

If a specific package is fundamentally broken with the current Swift version, you may need to fork the dependency or pin to a compatible commit.
