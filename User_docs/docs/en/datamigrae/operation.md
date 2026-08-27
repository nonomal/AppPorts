---
outline: deep
---

# Data Migration Operation Guide

This page covers the practical workflow for data directory migration. For technical implementation details, see [Basic Implementation](/en/datamigrae/baseinfo).

## Finding App-Associated Data Directories

1. Switch to the "Data Directories" tab in the main AppPorts window
2. The left panel shows all installed apps
3. Use the top toolbar to switch between "Tool Directories" and "App Data"; when viewing app data, click an app and the right panel displays its associated directories under `~/Library/`

AppPorts automatically scans the following directories, matching by app Bundle ID or name:

| Scan Path | Matching Method |
|-----------|-----------------|
| `~/Library/Application Support/` | Bundle ID or app name |
| `~/Library/Preferences/` | Bundle ID or app name |
| `~/Library/Containers/` | Bundle ID |
| `~/Library/Group Containers/` | Bundle ID |
| `~/Library/Caches/` | Bundle ID or app name |
| `~/Library/WebKit/` | Bundle ID |
| `~/Library/HTTPStorages/` | Bundle ID |
| `~/Library/Application Scripts/` | Bundle ID |
| `~/Library/Logs/` | App name |
| `~/Library/Saved Application State/` | App name |

## Tool Directories (Dot-Folders)

AppPorts can automatically detect dot-folders created by common development tools in the user's home directory:

1. Switch to the "Tool Directories" sub-tab in the Data Directories tab
2. The page lists all detected tool directories with their sizes
3. Each directory shows a priority badge (recommended/optional) and status

If a local tool directory is missing but the canonical location on the selected external storage still contains an AppPorts-managed directory, the item appears as "Needs Relinking". Switching external storage triggers a new scan and refreshes this state. Regular files are not treated as relinkable directories.

For the full supported list, see [Tool Directory Detection](/en/datamigrae/tools).

## Directory Migration (Custom Folders)

The "Directory Migration" tab migrates arbitrary user folders. It is useful for large projects, model files, asset libraries, or tool caches that you want to move to external storage.

1. Switch to "Directory Migration" in the main window
2. Click the "+" button in the "Local Folders" header
3. Choose the local folder to migrate, then choose the target root directory on external storage
4. AppPorts uses `target root/local folder name` as the external destination, saves the configuration, and starts migration

To avoid recursive copies, system-directory migration, or taking over the wrong path, directory migration applies these checks:

- The local folder must be under the current user's home directory and cannot be the entire home directory
- The local path and its parent path must not be symbolic links
- The local folder must not overlap with an already managed data directory or directory-migration entry
- The external target root must be a folder and must not be inside the current user's home directory
- The final external target must not be inside the local folder, and the local folder must not be inside the final external target

After migration, the local pane shows the original path status and the external pane shows the external copy status. Select items in the external pane to "Relink Folder" or "Restore Folder". Removing a configuration only removes it from the directory-migration list; it does not delete real data automatically.

## Migration Operations

### Single Directory Migration

1. Find the directory to migrate in the data directory list
2. Click the "Migrate" button on the right
3. AppPorts performs the following steps:
   - Copy the directory to external storage
   - Write managed link metadata
   - Delete the original local directory
   - Create a symbolic link

### Auto Re-signing

The Data Directories top toolbar contains the "Re-sign after migration" toggle. When enabled, data directory migration triggers signing for the associated app:

1. **Before migration**: Backs up the original signature of the associated app's **real external path** (not the local shell)
2. **After migration**: Executes Ad-hoc re-signing on the **real external app** (silent mode; failures do not show a dialog)

For linked apps, AppPorts automatically resolves the real app path behind the Stub Portal shell or symlink, ensuring signature changes are applied to the actual application package rather than an invalid local shell.

::: tip Container Data Confirmation
When migrating `Containers` or `Group Containers` app data, AppPorts asks before migration whether to re-sign after completion. Accept to back up and Ad-hoc re-sign the associated app; decline to migrate data only.
:::

### Log Context

Data directory operations (migration, restore, normalization, relinking) automatically include associated app context information in logs:

| Field | Description |
|-------|-------------|
| `app_name` | Associated app name |
| `app_status` | App status (Linked, Local, etc.) |
| `app_is_resigned` | Whether the app has been re-signed |
| `app_bundle_id` | App's Bundle ID (read from real path) |
| `app_real_path` | App's real external path |

These fields help pinpoint issues more precisely when exporting diagnostic packages.

### Batch Migration

1. Check multiple directories in the tool directory list
2. Click the "Batch Migrate" button at the bottom
3. AppPorts executes migration sequentially

::: tip 💡 Priority Recommendations
Data directories are classified into three priority levels:

- **Critical** (`critical`): Must work after migration; affects core application functionality
- **Recommended** (`recommended`): Large space savings; high migration benefit
- **Optional** (`optional`): Small size or rebuildable

It is recommended to prioritize migrating directories marked as "Recommended".
:::

## Restore Operations

1. Find the migrated directory in the data directory list (status: "Linked")
2. Click the "Restore" button on the right
3. AppPorts performs the following steps:
   - Delete the local symbolic link
   - Copy data from external storage back to local
   - Delete the external directory (best effort)

If an external target already exists during restore or recovery, AppPorts checks `.appports-link-metadata.plist`. It only reconnects or recovers automatically when metadata fully matches the current source path, destination path, and data directory type; otherwise it reports a destination conflict and leaves the directory untouched.

## Handling Abnormal States

### Needs Normalization

The directory is managed by AppPorts, but the external path is not at the canonical location. Click "Normalize"; AppPorts will move the external data to the canonical path and rebuild the symbolic link.

### Needs Relinking

The data directory still exists on external storage, but the local symbolic link is lost. Click "Relink"; AppPorts will recreate the symbolic link. Relinking only applies when the external target is still a directory. If a regular file occupies the external target, AppPorts stops and leaves the file untouched.

### Existing Soft Link

A user-created symbolic link not created by AppPorts. You can choose "Take Over"; AppPorts will write managed link metadata and manage it going forward.

## Tree View

For data directories containing subdirectories (e.g., multiple app directories under `Application Support`), AppPorts provides a tree grouping view:

- Parent directory shows expand/collapse arrows on the left
- Subdirectories display hierarchical indentation
- Each node independently shows size and status
- Migration/restore operations can be performed on individual subdirectories
