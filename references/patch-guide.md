# Patch Update Guide for Blink Shell GPL Builder

Guide for updating the `xcode26-free-team.patch` when Blink Shell releases a new version.

---

## When to Update

- `check.sh` reports a new Blink version AND the current patch doesn't apply cleanly
- User explicitly wants to upgrade to a newer Blink release

---

## Step-by-Step Patch Update Process

### 1. Prepare Clean Workspace

```bash
cd Blink-Shell-GPL-Builder

# Save current patch for reference
cp patches/xcode26-free-team.patch patches/xcode26-free-team.patch.bak

# Clone new version
rm -rf blink-src
git clone --depth 1 --branch raw https://github.com/blinksh/blink.git blink-src
cd blink-src
git checkout vNEW_VERSION
```

### 2. Apply Modifications Manually

#### a. Entitlements (`Blink/Blink.entitlements`)

Remove the same 8 keys. This file rarely changes structure between versions:

- `aps-environment`
- `com.apple.developer.associated-domains`
- `com.apple.developer.icloud-container-identifiers`
- `com.apple.developer.icloud-services`
- `com.apple.developer.ubiquity-container-identifiers`
- `com.apple.developer.ubiquity-kvstore-identifier`
- `com.apple.developer.user-fonts`
- `com.apple.developer.web-browser`

#### b. project.pbxproj (Most Likely to Break)

This file changes the most between versions. Apply these logical modifications:

1. **DEVELOPMENT_TEAM**: Search for `DEVELOPMENT_TEAM` and replace any hardcoded values (e.g., `A2H2CL32AG`) with `$(TEAM_ID)`. Also search for `CODE_SIGN_STYLE = Automatic;` lines that lack a corresponding `DEVELOPMENT_TEAM` setting and add `DEVELOPMENT_TEAM = $(TEAM_ID);` to those build configurations.

2. **Remove StoreKit and CloudKit framework links**: Find `StoreKit.framework` and `CloudKit.framework` in the `PBXBuildFile` section and remove those entries. Then find the corresponding references in the Frameworks build phase section and remove those too.

3. **Disable SystemCapabilities**: Set `enabled = 0` for iCloud and Push:
   ```
   com.apple.Push = { enabled = 0; };
   com.apple.iCloud = { enabled = 0; };
   ```

4. **SWIFT_ACTIVE_COMPILATION_CONDITIONS**: Add `BLINK_PUBLISHING_OPTION_DEVELOPER` to the Blink target's build configurations (both Debug and Release). Look for existing `SWIFT_ACTIVE_COMPILATION_CONDITIONS` lines and append the flag.

#### c. SceneDelegate (`Blink/SceneDelegate.swift`)

Find the `_showPaywallIfNeeded()` method and add an early return at the top:

```swift
private func _showPaywallIfNeeded() {
  if FeatureFlags.noSubscriptionNag {
    return
  }
  // ... existing paywall logic
}
```

#### d. developer_setup.xcconfig

This file is created from a template and is user-specific -- it is NOT included in the patch. The setup script handles its creation.

### 3. Generate New Patch

```bash
cd blink-src
git diff HEAD > ../patches/xcode26-free-team.patch
```

### 4. Test Patch Application

```bash
cd ..

# Verify patch applies cleanly to a fresh checkout
rm -rf blink-src-test
git clone --depth 1 --branch raw https://github.com/blinksh/blink.git blink-src-test
cd blink-src-test && git checkout vNEW_VERSION
git apply --check ../patches/xcode26-free-team.patch
```

If `--check` passes with no output, the patch applies cleanly.

### 5. Build and Verify

```bash
cd ..
./setup.sh --version vNEW_VERSION
```

Then open `blink-src/Blink.xcodeproj` in Xcode, set your team, and build to a device or simulator.

---

## Common pbxproj Changes Between Versions

| Change Type | Impact | How to Handle |
|-------------|--------|---------------|
| New targets added | Need `DEVELOPMENT_TEAM` | Add `DEVELOPMENT_TEAM = $(TEAM_ID)` to new target configs |
| Build settings restructured | Line numbers shift, patch context fails | Re-identify the correct locations manually |
| New frameworks added | May trigger capability requirements | Check if any new frameworks need the same removal treatment as StoreKit |
| Targets renamed or removed | Patch references stale target names | Update patch to match new structure |

**Tip:** To see structural changes between versions:
```bash
diff <(git show vOLD:Blink.xcodeproj/project.pbxproj) <(git show vNEW:Blink.xcodeproj/project.pbxproj) | head -200
```

---

## Automation Potential

The logical changes could theoretically be scripted (e.g., a Python script modifying pbxproj as text):

- Find/replace `DEVELOPMENT_TEAM` values
- Remove specific `PBXBuildFile` entries by framework name
- Toggle `SystemCapabilities` flags
- Inject compilation conditions

However, the pbxproj format is fragile -- it uses a custom Apple plist-like syntax where UUIDs tie entries together across sections. A scripted approach requires careful testing against each new version.

**Current recommendation:** Manual modification is more reliable. Use the old patch as a reference for what needs to change, and apply the same logical modifications to the new version's files.
