# Phase 1: Permanent Workspaces - Context

**Gathered:** 2026-05-06
**Status:** Ready for planning

## Phase Boundary

Save/restore named workspaces across sessions. Users can name, switch between, and manage multiple terminal workspaces. Workspaces persist in the sidebar and restore their tab/split layout on app relaunch.

## Implementation Decisions

### Naming
- **D-01:** Auto-name from directory — new workspaces default to their root directory name (e.g. `~/Projects/cove`)
- **D-02:** Inline rename — double-click workspace name in sidebar to edit in place

### Switching
- **D-03:** Ctrl+Tab cycles through workspaces (same as tab cycling behavior)
- **D-04:** Sidebar click to switch — mouse-based switching
- **D-05:** Cmd+Tab panel for many workspaces — holding Cmd+Tab shows workspace switcher (macOS app switcher style) when more than 9 workspaces exist

### Scope
- **D-06:** Layout-only restore — workspaces restore tabs and split layout, no terminal scrollback/history
- **D-07:** History always off — no per-workspace or global toggle for history saving

### Creation
- **D-08:** Manual creation only — workspaces created explicitly via button or Cmd+N shortcut
- **D-09:** No auto-suggest for new directory workspaces

### Active Indicator
- **D-10:** Bold text — active workspace shown with bold text in sidebar

### Deletion
- **D-11:** Archive + kill — deleting a workspace removes it from sidebar and kills all its running terminals

### Claude's Discretion
- Pinned workspace behavior — not discussed; open to standard approach (pinned first, then sortIndex)
- Sort order — not discussed; existing sortWorkspaces pattern (pinned > sortIndex > updatedAt > name) acceptable

## Canonical References

No external specs — requirements fully captured in decisions above.

## Existing Code Insights

### Reusable Assets
- `WorkspaceRegistry.swift` — `PermanentWorkspaceRecord` model and `WorkspaceRegistryStore` already persist workspaces to JSON at `~/Library/Application Support/cove/workspaces-{bundleID}.json`
- `SessionPersistence.swift` — `SessionSnapshotSnapshot` model captures full layout state (tabs, splits, panels) for autosave/restore cycle
- `Workspace+EqualizeSplitsSupport.swift` — existing workspace model supports splits via `SplitOrientation`
- `SidebarBonsplitTabWorkspaceDropOverlay.swift` — existing sidebar workspace projection already renders workspace list

### Established Patterns
- `WorkspaceRegistryStore.save()` uses atomic write with directory creation — follow same pattern for any new persistence
- `liveProjectWorkspaceProjection()` derives workspace name from `workspace.customTitle ?? workspace.title ?? directoryName` — naming strategy aligns with D-01
- `SessionPersistencePolicy.autosaveInterval = 8.0` — autosave cadence established; workspace persistence can use same interval

### Integration Points
- New "Save Workspace" action must update `WorkspaceRegistryStore` AND create/update `PermanentWorkspaceRecord`
- Restore flow: load `WorkspaceRegistry`, then reconstruct layout from `SessionWorkspaceLayoutSnapshot`
- Workspace deletion: remove from `WorkspaceRegistry` + kill all terminal processes in that workspace

## Specific Ideas

No specific references from discussion — decisions cover all major behaviors.

## Deferred Ideas

None — discussion stayed within phase scope.

---

*Phase: 1-Permanent Workspaces*
*Context gathered: 2026-05-06*
