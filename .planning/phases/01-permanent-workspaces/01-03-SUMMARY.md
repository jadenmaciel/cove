# 01-03 SUMMARY: Naming and Deletion for Permanent Workspaces

## Actions Taken

- Added inline workspace rename: double-click on workspace name in sidebar shows TextField with auto-focus; Enter submits; Escape reverts
- Added `deleteWorkspace(_:)` method to TabManager: shows confirmation dialog, calls `teardownAllPanels()` (D-11), archives record via `archive()`, saves registry, closes workspace tab
- Added `renamePermanentWorkspace(tabId:name:)` to TabManager: updates `PermanentWorkspaceRecord.name` in registry and saves
- Added context menu "Delete Workspace…" menu item in `workspaceContextMenu`
- Added localization strings for `contextMenu.deleteWorkspace`, `dialog.deleteWorkspace.title`, `dialog.deleteWorkspace.message`

## Files Modified

- `Sources/ContentView.swift`
  - Added `@State private var editingWorkspaceId: UUID?`, `@State private var renameText: String`, `@State private var renameOriginalName: String` to `TabItemView`
  - Replaced `Text(workspaceSnapshot.title)` with conditional: when `editingWorkspaceId == tab.id`, shows `TextField` with `$renameText` binding; else shows `Text` with `onTapGesture(count: 2)` to enter edit mode
  - Added `commitRename()` method: validates (not empty, max 255 chars, no `/` or `\0`), calls `tabManager.renamePermanentWorkspace()`, exits edit mode
  - Added "Delete Workspace…" `Button` in `workspaceContextMenu` that calls `tabManager.deleteWorkspace(tab)`
- `Sources/TabManager.swift`
  - Added `deleteWorkspace(_: Workspace)`: confirmation dialog, `teardownAllPanels()`, archive record, `WorkspaceRegistryStore.save()`, `closeWorkspace()`
  - Added `renamePermanentWorkspace(tabId: UUID, name: String)`: loads registry, updates record name, saves
- `Resources/Localizable.xcstrings`
  - Added `contextMenu.deleteWorkspace`, `dialog.deleteWorkspace.title`, `dialog.deleteWorkspace.message` for en, ja, zh-Hans, zh-Hant, ko, de

## Key Findings

- `closeWorkspace()` already calls `workspace.teardownAllPanels()` internally, so `deleteWorkspace()` avoids double-killing by calling `teardownAllPanels()` first before `closeWorkspace()`
- `PermanentWorkspaceRecord.archive()` is a mutating func that sets `archivedAt = Date()` and `status = .archived`
- Registry save URL is obtained via `WorkspaceRegistryStore.defaultRegistryFileURL()` on macOS
- The `commitRename()` validates: empty = cancel, >255 chars or contains `/` or `\0` = revert to original

## Verification

```
grep -n "editingWorkspaceId\|renameText\|commitRename" Sources/ContentView.swift | head -15
grep -n "deleteWorkspace\|renamePermanentWorkspace\|teardownAllPanels\|archive()" Sources/TabManager.swift | head -15
```

Both return expected matches. No Swift LSP errors in modified files. Zig build error is pre-existing and unrelated.
