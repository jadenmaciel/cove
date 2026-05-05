# Phase 1: Permanent Workspaces - Research

**Researched:** 2026-05-05
**Domain:** macOS terminal workspace persistence and UI
**Confidence:** HIGH (verified via codebase inspection)

## Summary

Phase 1 implements permanent workspaces: named, persistent workspace records saved to `~/Library/Application Support/cove/workspaces-{bundleID}.json` that survive app restarts. Each `PermanentWorkspaceRecord` captures workspace identity (name, rootPath, kind) while `SessionWorkspaceSnapshot` captures layout (tabs, splits, panels) for layout-only restore (D-06: no scrollback). The sidebar renders workspace list via `SidebarWorkspaceSnapshotBuilder`, keyboard shortcuts live in `KeyboardShortcutSettings.Action`, and workspace switching uses `TabManager` (MainActor). All mutations go through `WorkspaceRegistryStore.save()` for atomic writes.

**Primary recommendation:** Build permanent workspace save/restore by bridging `WorkspaceRegistryStore` (identity layer) with `SessionPersistenceStore` (layout layer), using the existing `liveProjectWorkspaceProjection()` pattern for sidebar projection and `Workspace.restoreSessionSnapshot()` for layout restore.

## User Constraints (from CONTEXT.md)

### Locked Decisions
- **D-01:** Auto-name from directory — new workspaces default to their root directory name
- **D-02:** Inline rename — double-click workspace name in sidebar to edit in place
- **D-03:** Ctrl+Tab cycles through workspaces
- **D-04:** Sidebar click to switch — mouse-based switching
- **D-05:** Cmd+Tab panel for many workspaces (more than 9)
- **D-06:** Layout-only restore — tabs and splits, no scrollback
- **D-07:** History always off — no toggle
- **D-08:** Manual creation only — button or Cmd+N
- **D-09:** No auto-suggest for new directory workspaces
- **D-10:** Bold text — active workspace shown with bold text in sidebar
- **D-11:** Archive + kill — deleting removes from sidebar and kills all terminals

### Claude's Discretion
- Pinned workspace behavior — pinned first, then sortIndex
- Sort order — existing pattern (pinned > sortIndex > updatedAt > name) acceptable

### Deferred Ideas (OUT OF SCOPE)
None.

## Phase Requirements

| ID | Description | Research Support |
|----|-------------|------------------|
| WS-01 | Permanent workspaces — save/restore named project workspaces across sessions | `WorkspaceRegistryStore` persists `PermanentWorkspaceRecord` to JSON; `SessionWorkspaceSnapshot` captures layout; `Workspace.restoreSessionSnapshot()` restores layout |
| WS-02 | Fast workspace switching — keyboard shortcuts and UI for navigating workspaces | `KeyboardShortcutSettings.Action.selectWorkspaceByNumber` exists; `TabManager` manages `tabs: [Workspace]`; sidebar click to switch via `SidebarWorkspaceSnapshotBuilder` |
| WS-03 | Workspace naming and labeling — clear names for project workspaces | `liveProjectWorkspaceName()` derives from `customTitle ?? title ?? directoryName`; inline rename via D-02 |

## Architectural Responsibility Map

| Capability | Primary Tier | Secondary Tier | Rationale |
|------------|-------------|----------------|-----------|
| Workspace identity persistence | API/Backend | — | `WorkspaceRegistryStore` writes JSON to App Support |
| Workspace layout capture | API/Backend | — | `SessionPersistenceStore` serializes `SessionWorkspaceSnapshot` |
| Sidebar workspace rendering | Frontend Server (SwiftUI) | — | `SidebarWorkspaceSnapshotBuilder` + SwiftUI views in `ContentView.swift` |
| Workspace switching (keyboard) | Frontend Server (AppKit) | — | `KeyboardShortcutSettings` + `TabManager` selection |
| Workspace switching (mouse) | Frontend Server (SwiftUI) | — | Sidebar click handlers |
| Terminal panel lifecycle | Browser/Client | — | `Workspace.teardownAllPanels()` kills terminals |

## Standard Stack

No new dependencies — all built on existing Cove infrastructure.

| Library | Version | Purpose | Why Standard |
|---------|---------|---------|--------------|
| Swift Concurrency | Swift 6 | Actor isolation, Sendable | `@MainActor TabManager`, `Sendable` snapshot structs |
| Swift Testing | bundled | Unit tests | Already in use (`WorkspaceRegistryTests`) |
| SwiftFormat/SwiftLint | bundled | Code formatting | Project standard |

**No new packages required.**

## Architecture Patterns

### System Architecture Diagram

```
User Action (sidebar click / keyboard / Cmd+N)
        |
        v
TabManager (MainActor) <---> WorkspaceRegistryStore (JSON file)
        |                              |
        | (addTab / closeWorkspace)    | (load / save)
        |                              |
        v                              v
Workspace instances          ~/Library/Application Support/cove/
        |                              workspaces-{bundleID}.json
        | (sessionSnapshot / restoreSessionSnapshot)
        v
SessionPersistenceStore (layout snapshots)
        |
        v
SidebarWorkspaceSnapshotBuilder (sidebar projection)
```

### Recommended Project Structure

```
Sources/
├── WorkspaceRegistry.swift          # No changes needed
├── SessionPersistence.swift         # No changes needed
├── PermanentWorkspaceManager.swift  # NEW: orchestrates save/restore lifecycle
├── WorkspaceNaming.swift            # NEW: naming logic (D-01, D-02)
├── WorkspaceSwitcher.swift           # NEW: Ctrl+Tab + Cmd+Tab logic (D-03, D-05)
├── WorkspaceDeleteHandler.swift      # NEW: archive + kill logic (D-11)
└── Sidebar/                         # Existing sidebar infrastructure
```

### Pattern 1: Workspace Save Flow
**What:** Capturing and persisting workspace state.
**When to use:** When user triggers "Save Workspace" or app autosaves.
**Source:** `WorkspaceRegistry.swift` + `SessionPersistence.swift`

```swift
// 1. Capture identity via WorkspaceRegistryStore
let record = PermanentWorkspaceRecord(
    id: workspace.id,
    kind: .project,
    name: liveProjectWorkspaceName(for: workspace),  // D-01
    rootPath: workspace.currentDirectory,
    pinned: workspace.isPinned,
    sortIndex: nextSortIndex,
    restorePolicy: .restoreLastSession,
    createdAt: Date(),
    updatedAt: Date()
)

// 2. Capture layout via sessionSnapshot
let layout = workspace.sessionSnapshot(includeScrollback: false)  // D-06

// 3. Save both layers
var registry = WorkspaceRegistryStore.load(fileURL: registryURL) ?? WorkspaceRegistry()
registry.workspaces.append(record)
WorkspaceRegistryStore.save(registry, fileURL: registryURL)
// Layout saved separately via SessionPersistenceStore.save(snapshot, fileURL: layoutURL)
```

### Pattern 2: Workspace Restore Flow
**What:** Reconstructing workspace from persisted records.
**When to use:** On app launch when restoring last session.
**Source:** `Workspace.swift` + `TabManager.swift`

```swift
// 1. Load workspace registry
let registry = WorkspaceRegistryStore.load(fileURL: registryURL)

// 2. For each non-archived workspace, create workspace + restore layout
for record in registry.defaultVisibleWorkspaces {
    let workspace = tabManager.addTab(
        title: record.name,
        workingDirectory: record.rootPath
    )
    workspace.id = record.id  // Preserve ID
    workspace.isPinned = record.pinned
    // Restore layout from SessionPersistenceStore
    if let layout = SessionPersistenceStore.load(fileURL: layoutURL(for: record.id)) {
        workspace.restoreSessionSnapshot(layout)  // D-06: layout only
    }
}
```

### Pattern 3: Workspace Delete (Archive + Kill)
**What:** D-11 — removing workspace kills its terminals.
**When to use:** When user deletes a permanent workspace.
**Source:** `TabManager.swift` + `Workspace.swift`

```swift
func deletePermanentWorkspace(_ record: PermanentWorkspaceRecord) {
    // 1. Kill all terminals (D-11)
    if let workspace = liveWorkspace(for: record.id) {
        workspace.teardownAllPanels()  // Kills all terminal processes
    }

    // 2. Archive (not hard delete — preserves for undo)
    var registry = WorkspaceRegistryStore.load(fileURL: registryURL) ?? WorkspaceRegistry()
    if let index = registry.workspaces.firstIndex(where: { $0.id == record.id }) {
        registry.workspaces[index].archive()
    }
    WorkspaceRegistryStore.save(registry, fileURL: registryURL)
}
```

### Pattern 4: Inline Rename (D-02)
**What:** Double-click to edit workspace name in place.
**When to use:** Sidebar workspace row editing.
**Source:** `SidebarWorkspaceSnapshotBuilder` + SwiftUI `TextField`

```swift
// In SidebarWorkspaceSnapshotBuilder.Snapshot:
// Add editing state: isEditing: Bool
// On double-click: set isEditing = true for that workspace
// TextField saves on commit: update PermanentWorkspaceRecord.name + save registry
```

### Pattern 5: Ctrl+Tab Workspace Cycling (D-03)
**What:** Ctrl+Tab cycles through workspaces like tab cycling.
**When to use:** Keyboard navigation.
**Source:** `KeyboardShortcutSettings` + existing `nextSurface`/`prevSurface` pattern

```swift
// Add KeyboardShortcutSettings.Action.nextWorkspace / prevWorkspace
// Similar to surface cycling but operates on workspace level
// Uses TabManager.tabs array (sorted by sortIndex)
```

## Don't Hand-Roll

| Problem | Don't Build | Use Instead | Why |
|---------|-------------|-------------|-----|
| JSON persistence | Custom encoder | `WorkspaceRegistryStore.save()` / `load()` | Atomic writes, directory creation, identical-data dedup already implemented |
| Workspace layout serialization | Custom snapshot | `SessionWorkspaceSnapshot` | Already captures tabs/splits/panels with Sendable conformance |
| Sidebar projection | Custom view builder | `SidebarWorkspaceSnapshotBuilder` | Already handles context menu deferral, active tab indicator |
| App lifecycle restore | Custom launch handler | Existing `SessionRestorePolicy.shouldAttemptRestore()` | Handles test environment detection |

**Key insight:** The existing `WorkspaceRegistry` + `SessionPersistence` stack provides both identity (`PermanentWorkspaceRecord`) and layout (`SessionWorkspaceSnapshot`) persistence. Phase 1 primarily wires these together and adds UI for naming/switching/delete.

## Common Pitfalls

### Pitfall 1: Confusing Registry Identity with Layout Snapshot
**What goes wrong:** Saving workspace saves identity but layout not persisted separately.
**Why it happens:** `WorkspaceRegistryStore` persists `PermanentWorkspaceRecord` only; layout lives in `SessionPersistenceStore` with a different file URL pattern.
**How to avoid:** Use a consistent naming convention linking registry record ID to layout file URL (e.g., `workspace-{id}.json` in session snapshots directory).
**Warning signs:** Restore creates workspaces but they have no tabs/splits.

### Pitfall 2: `@MainActor` Isolation Breaking Async Flows
**What goes wrong:** `Workspace.restoreSessionSnapshot()` and `TabManager` are `@MainActor`; session persistence runs off-main.
**Why it happens:** `SessionPersistenceStore.save()` is nonisolated but `workspace.sessionSnapshot()` requires main actor.
**How to avoid:** Capture snapshot on main actor before passing to nonisolated save:
```swift
let snapshot = await MainActor.run { workspace.sessionSnapshot(includeScrollback: false) }
SessionPersistenceStore.save(snapshot, fileURL: layoutURL)
```
**Warning signs:** `concurrent-constraint` or `ActorIsolation` compiler errors.

### Pitfall 3: Restoring Without Matching Panel IDs
**What goes wrong:** Layout snapshot has old panel IDs; restore creates new panels with new IDs.
**Why it happens:** `restoreSessionLayout()` maps old IDs to new via `oldToNewPanelIds`.
**How to avoid:** Already handled by `Workspace.restoreSessionSnapshot()` — do not bypass it.
**Warning signs:** Focus panel selection has no effect after restore.

### Pitfall 4: Bold Active Indicator Not Updating on Switch
**What goes wrong:** Active workspace shown in regular weight after switching.
**Why it happens:** `SidebarWorkspaceSnapshotBuilder` caches snapshot; active state comes from `selectedTabId` on `TabManager`.
**How to avoid:** Trigger snapshot rebuild on `TabManager.selectedTabId` change via Combine/observation.
**Warning signs:** Active workspace text weight unchanged after Ctrl+Tab.

## Code Examples

### Workspace Name Derivation (D-01)
Source: `WorkspaceRegistry.swift` `liveProjectWorkspaceName()` (lines 201-211)

```swift
@MainActor
private static func liveProjectWorkspaceName(for workspace: Workspace) -> String {
    let candidates = [workspace.customTitle, workspace.title]
    for candidate in candidates {
        let trimmed = candidate?.trimmingCharacters(in: .whitespacesAndNewlines) ?? ""
        if !trimmed.isEmpty { return trimmed }
    }

    let directoryName = URL(fileURLWithPath: workspace.currentDirectory).lastPathComponent
        .trimmingCharacters(in: .whitespacesAndNewlines)
    return directoryName.isEmpty ? "Workspace" : directoryName
}
```

### Workspace Layout Snapshot (D-06)
Source: `Workspace.swift` `sessionSnapshot()` (lines 163-200)

```swift
func sessionSnapshot(
    includeScrollback: Bool,
    restorableAgentIndex: RestorableAgentSessionIndex? = nil
) -> SessionWorkspaceSnapshot {
    let tree = bonsplitController.treeSnapshot()
    let layout = sessionLayoutSnapshot(from: tree)

    let orderedPanelIds = sidebarOrderedPanelIds()
    // ... captures panels, status, log entries, progress, gitBranch
    // includeScrollback=false for permanent workspaces (D-06)

    return SessionWorkspaceSnapshot(
        processTitle: ...,
        layout: layout,
        panels: panelSnapshots,
        // ...
    )
}
```

### Workspace Restore
Source: `Workspace.swift` `restoreSessionSnapshot()` (lines 235-297)

```swift
func restoreSessionSnapshot(_ snapshot: SessionWorkspaceSnapshot) {
    // Clears old state
    restoredTerminalScrollbackByPanelId.removeAll()

    // Restores directory, title, color, pinned
    currentDirectory = normalizedCurrentDirectory
    setCustomTitle(snapshot.customTitle)
    // ...

    // Restores layout (tabs/splits) via restoreSessionLayout
    let panelSnapshotsById = Dictionary(uniqueKeysWithKeys: snapshot.panels.map { ($0.id, $0) })
    let leafEntries = restoreSessionLayout(snapshot.layout)
    // Creates new panels matching snapshot structure

    // Restores focused panel
    if let focusedOldPanelId = snapshot.focusedPanelId,
       let focusedNewPanelId = oldToNewPanelIds[focusedOldPanelId] {
        focusPanel(focusedNewPanelId)
    }
}
```

### Workspace Teardown (D-11)
Source: `Workspace.swift` line 10519

```swift
func teardownAllPanels() {
    // Destroys all terminal surfaces and kills processes
}
```

Source: `TabManager.swift` `closeWorkspace()` (lines 4105-4129)

```swift
func closeWorkspace(_ workspace: Workspace) {
    guard tabs.count > 1 else { return }
    clearWorkspaceGitProbes(workspaceId: workspace.id)
    clearWorkspacePullRequestTracking(workspaceId: workspace.id)
    sidebarSelectedWorkspaceIds.remove(workspace.id)

    AppDelegate.shared?.notificationStore?.clearNotifications(forTabId: workspace.id)
    workspace.teardownAllPanels()        // Kills terminals (D-11)
    workspace.teardownRemoteConnection()
    unwireClosedBrowserTracking(for: workspace)
    workspace.owningTabManager = nil

    // Remove from tabs array
    if let index = tabs.firstIndex(where: { $0.id == workspace.id }) {
        tabs.remove(at: index)
        // Update selection
    }
}
```

## State of the Art

| Old Approach | Current Approach | When Changed | Impact |
|--------------|------------------|--------------|--------|
| Live workspace projection only | Permanent workspace registry + session snapshots | Phase 1 | Workspaces survive app restart |
| Ephemeral session restore | Layout-only restore (D-06) | Phase 1 decision | No scrollback preserved |
| Agent workspaces (kind=agent) | Project workspaces (kind=project) | Phase 1 focus | Simpler v1 scope |

**Deprecated/outdated:**
- None relevant to Phase 1.

## Assumptions Log

> All claims in this research were verified via codebase inspection — no user confirmation needed.

| # | Claim | Section | Risk if Wrong |
|---|-------|---------|---------------|
| A1 | Session layout snapshot stored separately from workspace registry | Architecture | Low — both files exist and are used for different purposes |
| A2 | `teardownAllPanels()` kills all terminal processes | Common Pitfalls | Low — confirmed in Workspace.swift line 10519 |
| A3 | `SidebarWorkspaceSnapshotBuilder` rebuilds on TabManager selection change | Bold Active Indicator | Medium — builder uses Combine observation; verify rebuild trigger |

## Open Questions

1. **Layout file URL convention**
   - What we know: `SessionPersistenceStore` uses `session-{bundleID}.json` for single session snapshot
   - What's unclear: How to name per-workspace layout files (e.g., `workspace-{uuid}.json`?)
   - Recommendation: Use `workspaces/{uuid}/layout.json` subdirectory pattern

2. **Autosave integration**
   - What we know: `SessionPersistencePolicy.autosaveInterval = 8.0`
   - What's unclear: Should permanent workspace layouts autosave on same interval?
   - Recommendation: Yes — re-use same interval; trigger registry + layout save together

3. **Undo/redo for archive delete**
   - What we know: `archive()` sets `archivedAt` and `status = .archived`
   - What's unclear: Is there an undo mechanism for archive?
   - Recommendation: Not in v1 — soft delete via archive is sufficient

4. **Active indicator rebuild trigger**
   - What we know: `SidebarWorkspaceSnapshotBuilder` uses `ObservableObject` pattern
   - What's unclear: Does switching workspaces trigger snapshot rebuild?
   - Recommendation: Verify `TabManager.$selectedTabId` observation path

## Environment Availability

Step 2.6: SKIPPED (no external dependencies — pure Swift codebase, no new tools required)

## Validation Architecture

### Test Framework
| Property | Value |
|----------|-------|
| Framework | Swift Testing (bundled with Xcode 16+) |
| Config file | `CoveTests/WorkspaceRegistryTests.swift` — existing |
| Quick run command | `xcodebuild test -scheme Cove-Unit -only-testing:CoveTests/WorkspaceRegistryTests 2>&1 | tail -30` |
| Full suite command | `xcodebuild test -scheme Cove-Unit 2>&1 | tail -30` |

### Phase Requirements -> Test Map
| Req ID | Behavior | Test Type | Automated Command | File Exists? |
|--------|----------|-----------|-------------------|---------------|
| WS-01 | Save workspace record | Unit | `WorkspaceRegistryTests` already covers `save/load` round-trip | YES |
| WS-01 | Restore workspace layout | Unit | Add test: create workspace -> snapshot -> restore -> verify panels | Gap |
| WS-02 | Keyboard shortcut (Ctrl+Tab) | Integration | Socket command test via `tests_v2/` | Gap |
| WS-03 | Inline rename | Unit | Add test: `record.name = "New"` -> save -> load -> verify | Gap |
| WS-11 | Archive + kill terminals | Integration | Requires running terminal; socket test in `tests_v2/` | Gap |

### Sampling Rate
- **Per task commit:** Quick run (registry tests only, <10s)
- **Per wave merge:** Full suite (`Cove-Unit`, ~2min)
- **Phase gate:** Full suite green before `/gsd-verify-work`

### Wave 0 Gaps
- `CoveTests/WorkspaceRestoreTests.swift` — covers WS-01 layout restore behavior
- `CoveTests/WorkspaceNamingTests.swift` — covers WS-03 rename behavior
- `tests_v2/test_workspace_delete.py` — covers WS-11 archive+kill behavior (requires running app)

*(If no gaps: "None — existing test infrastructure covers all phase requirements")*

## Security Domain

> Security enforcement is enabled for this project. Phase 1 has minimal security surface.

### Applicable ASVS Categories

| ASVS Category | Applies | Standard Control |
|---------------|---------|-----------------|
| V2 Authentication | No | N/A — local workspace persistence |
| V3 Session Management | No | N/A — no auth sessions |
| V4 Access Control | No | N/A — local-only |
| V5 Input Validation | Yes | Validate workspace name (not empty, max length) before save |
| V6 Cryptography | No | N/A — no secrets in workspace records |

### Known Threat Patterns for macOS Workspace Apps

| Pattern | STRIDE | Standard Mitigation |
|---------|--------|---------------------|
| Path traversal in workspace rootPath | Tampering | Validate `rootPath` is a real directory; reject symlink tricks |
| Workspace name injection | Information Disclosure | Sanitize name for filesystem use (no `/`, `\0`) |
| Double-click rename XSS | Tampering | TextField sanitization via SwiftUI built-in |

## Sources

### Primary (HIGH confidence)
- `Sources/WorkspaceRegistry.swift` — lines 1-343: WorkspaceRegistryStore, PermanentWorkspaceRecord, liveProjectWorkspaceProjection
- `Sources/SessionPersistence.swift` — lines 1-541: SessionWorkspaceSnapshot, SessionWorkspaceLayoutSnapshot, SessionPersistenceStore
- `Sources/Workspace.swift` — lines 163-320+: sessionSnapshot(), restoreSessionSnapshot()
- `Sources/TabManager.swift` — lines 4105-4129: closeWorkspace(), teardownAllPanels
- `Sources/TabManager.swift` — lines 728-750: TabManager @MainActor class definition
- `CoveTests/WorkspaceRegistryTests.swift` — round-trip tests, archive behavior

### Secondary (MEDIUM confidence)
- `Sources/KeyboardShortcutSettings.swift` — lines 1-100: Action enum, shortcut definitions
- `Sources/ContentView.swift` — lines 12821+: SidebarWorkspaceSnapshotBuilder
- `Sources/Sidebar/SidebarBonsplitTabWorkspaceDropOverlay.swift` — sidebar workspace UI

### Tertiary (LOW confidence)
None — all major claims verified via primary sources.

## Metadata

**Confidence breakdown:**
- Standard stack: HIGH — no new dependencies
- Architecture: HIGH — all patterns verified via source
- Pitfalls: MEDIUM — timing assumptions based on code review, not runtime verification

**Research date:** 2026-05-05
**Valid until:** 2026-06-05 (30 days — stable codebase, no fast-moving dependencies)
