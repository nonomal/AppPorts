---
outline: deep
---

# Changelog

## v1.8.0

### New Features

- Added custom local scan directories: the "Mac Local Apps" header now has a "+" button to add extra local app scan directories. Useful for tools like JetBrains Toolbox and Steam that install apps outside `/Applications`. Added directories are persisted and automatically monitored for changes (#48).
- Added Stub Portal version sync: when an external app is updated via the App Store, the local Stub Portal's version info is now automatically synced and the macOS Launch Services cache is refreshed. The "Open With" menu no longer shows stale version numbers (#50).
- Added tool directory detection for Gradle (`~/.gradle`), Android development data (`~/.android`), and Flutter/Dart Pub cache (`~/.pub-cache`) (#49).
- Added Directory Migration: add arbitrary user folders in the "Directory Migration" tab, migrate large projects, models, asset libraries, or tool caches to external storage, and relink or restore them later (#54).
- Added protected-app migration warning: before migrating App Store apps or root-owned apps, AppPorts warns that automatic deletion or replacement may fail due to permissions and suggests manually moving the app in Finder before creating a link (#55).

### Improvements

- Faster app scanning: Info.plist reads per app reduced from 7 to 1 (via in-memory cache), significantly improving scan speed.
- Scan timeout protection: the `codesign` subprocess now has a 10-second timeout, preventing large app signature checks from blocking the entire scan indefinitely.
- Directory size calculation safety cap: a 500,000 file count limit has been added to recursive size calculations, preventing runaway enumeration on Electron and other large app bundles.
- Scan trace logging: per-app TRACE logging added to the scan loop, making it easier to identify which app is slow or stuck during scanning.
- More precise data directory matching: fixed bundle ID suffix extraction to filter generic TLD words like `app`, `com`, `org`. Previously, bundle IDs like `cn.trae.app` would trigger scanning of 720+ unrelated system containers.
- More complete tool-directory relink detection: when a local tool directory is missing but a managed directory still exists at the canonical external-storage location, AppPorts shows it as "Needs Relinking"; switching external storage refreshes tool-directory state automatically.
- Improved localization and accessibility: app, data-directory, and custom-directory statuses, sort/filter labels, settings toggles, and status badges now follow the selected language more consistently and expose clearer accessibility labels.
- App sizes now use a session-level cache, reducing cases where sizes return to "Calculating" or disappear after a refresh (#55).
- Safer data-directory rollback: before creating the link, AppPorts renames the local source to a hidden safety backup. If link creation or backup cleanup fails, it keeps the local backup and external copy where possible to avoid losing both sides (#54).

### Fixes

- Fixed Trae and similar apps scanning extremely slowly — the generic suffix `app` from the bundle ID caused `~/Library/Containers/` to scan hundreds of unrelated directories.
- Fixed local Stub Portal version info not updating after external apps are updated via the App Store, causing the "Open With" menu to show stale versions.
- Fixed the refresh button not triggering Stub Portal version sync.
- Fixed data-directory relinking or normalization potentially treating an external regular file as a directory; regular files are now rejected and left untouched.
- Fixed multi-line dialog bodies falling back to Chinese in some languages, completed Russian UI translations, and localized the Stub Portal "external storage not connected" system dialog based on the system language (#55).

## v1.7.0

### New Features

- Added "Pending Move Out" status: when the real local app is newer than the app with the same name on external storage, AppPorts marks it as pending move out, indicating that the local newer version can be safely migrated out to replace the external older copy.
- Added re-sign confirmation for data migration: before migrating data inside an app container, AppPorts can ask whether to automatically apply Ad-hoc re-signing to the related app after migration, reducing the risk of unrecognized data, warnings, or launch failures after container data migration (#44).

### UI Improvements

- Rearranged the top toolbar: app/data-directory tab buttons now use a more compact icon + text style.
- Optimized the data-directory action bar: the Tool Directories / App Data switch, post-migration re-sign toggle, restore original signature button, and refresh button now live in the top toolbar.
- Added a "Pending Move Out" app status badge for apps whose local version is newer than the external old copy.
- Localized the data migration re-sign confirmation dialog, including title, body text, and buttons.

### Improvements

- Strengthened app migration safety: when the external destination already exists, AppPorts only auto-cleans it if it is identified as an AppPorts-managed old portal, a stale migration remnant, or the app is in "Pending Move Out" state.
- Strengthened data-directory recovery checks: automatic recovery no longer relies on similar directory size and now requires full AppPorts metadata matching.
- Made app data scanning more stable: results from older scan tasks no longer overwrite the data-directory list for the currently selected app.
- Improved escaping for admin commands and AppleScript: paths containing quotes, backslashes, spaces, or Chinese characters are handled more safely.
- Improved localization: fixed help content, prompts, and data migration confirmation text that could remain in Chinese or be incomplete after switching languages, and completed translations for all supported languages (#43).

### Fixes

- Fixed data directory migration incorrectly treating a real external directory as a recoverable target.
- Fixed app migration potentially deleting a real external app with the same name by mistake.
- Fixed unstable detection and cleanup of old external AppPorts portals or stale migration remnants.
- Fixed malformed AppleScript or admin commands when paths contain special characters.
- Fixed background migration or post-migration re-signing reading the app after the selected app had changed.
- Fixed the "Pending Move Out" status badge not appearing in the app list.

## v1.6.2

- New: Auto re-sign at login. Automatically re-signs migrated apps with expired signatures each time the user logs in, no manual action needed. Enabled by default, can be turned off in Settings
- Improvement: Stub Portal now uses a native Mach-O binary launcher instead of the legacy bash script, fixing the issue where double-clicking associated documents in Finder could not open the external app (#42)
- Improvement: About page layout optimized with scrollable content area, fixing content being cut off when the window is too small
- Fix: Native Stub Portal being incorrectly identified as a regular local app
- Fix: Unable to properly clean up native Stub Portal when moving apps back to local storage
- Fix: App shell being treated as a complete app during link-back-to-local operations
- Fix: AutoResignInstaller silently succeeding when installation fails

## v1.6.1

- Fixed: Auto-re-signing after data directory migration now correctly signs the real external app instead of the local stub shell
- Fixed: Re-signing and signature restore operations now correctly resolve the real path for linked apps
- Fixed: "Re-signed" status detection for linked apps now correctly identifies the signing status of the real external app
- Improved: Log output includes structured error codes and related path information

## v1.6.0

- Migrated apps no longer show arrow badges
- Auto-updating apps are no longer broken by updates after migration
- Added app signature management feature to fix "Damaged" prompts after migration
- External storage disconnection now shows red "Orphaned Link" warnings
- macOS 15.1+ users can install App Store apps directly to external drives
- Data directory migration is safer: prevents accidental system directory migration, auto-recovers from interruption
- Scanning and size calculation are faster; list no longer jumps
- File copying to external storage is more stable; no more errors on interruption
- App status badges redesigned with richer information and clickable details
- App list no longer loses selection after refresh; data directories support tree view
- UI refinements: search, sort, group cards, icon loading, etc.
- Added Martian language option
- Automated test updates

## v1.5.5

- Added macOS 15.1+ App Store app external installation support
- Added auto re-signing feature (auto-executed after data directory migration)
- Added `LocalizationAuditTests` localization audit tests
- Improved Stub Portal Info.plist generation logic
- Fixed Launchpad icon loss issue for some apps after migration

## v1.4.0

- Added data directory tree view
- Added tool directory detection (30+ development tools)
- Added diagnostic package export feature
- Improved self-update detection (Chrome, Edge, and other custom updaters)
- Fixed auto-recovery mechanism after migration interruption

## v1.3.0

- Added data directory migration feature
- Added code signature management (backup/restore original signatures)
- Added Sparkle and Electron app auto-detection
- Improved locked migration protection (`chflags uchg`)
- Fixed badge display issues in Finder

## v1.2.0

- Added Stub Portal migration strategy (replacing Deep Contents Wrapper)
- Added iOS app migration support (Mac version iOS apps)
- Improved batch migration performance
- Fixed issue where some apps could not launch after restore

## v1.1.0

- Added multi-language support (20+ languages)
- Added app suite directory migration (e.g., Microsoft Office)
- Improved external storage offline detection
- Fixed symbolic link penetration issue with Deep Contents Wrapper strategy

## v1.0.0

- First official release
- Supported app migration to external storage (Deep Contents Wrapper / Whole App Symlink)
- Supported app restore and link management
- Supported FolderMonitor real-time file system monitoring
