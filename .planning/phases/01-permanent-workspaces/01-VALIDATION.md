---
name: Phase 1 Permanent Workspaces Validation Strategy
description: Validation approach for permanent workspaces feature
type: validation
phase: 1
phase_slug: permanent-workspaces
date: 2026-05-06
---

# Phase 1: Permanent Workspaces - Validation Strategy

## Validation Architecture

### Dimension 8: Nyquist Compliance

This phase implements macOS-native workspace persistence. Key validation targets:

**Core persistence (WS-01):**
- WorkspaceRegistryStore correctly writes PermanentWorkspaceRecord to JSON
- SessionPersistence captures and restores layout snapshot
- App relaunch restores workspace with tabs/splits intact

**Switching (WS-02):**
- Ctrl+Tab cycles workspaces in order
- Cmd+1-9 switches to workspace N
- Sidebar click selects workspace
- Cmd+Tab panel appears for 10+ workspaces

**Naming (WS-03):**
- Auto-name from directory on creation
- Inline rename via double-click
- Bold text indicates active workspace

## Test Approach

**Unit tests:**
- WorkspaceRegistryStore.load() returns existing workspaces
- WorkspaceRegistryStore.save() writes atomic JSON
- SessionWorkspaceSnapshot captures panel layout correctly
- Workspace.restoreSessionSnapshot() reconstructs layout

**Integration tests:**
- Create workspace -> restart app -> workspace restored
- Create 2 workspaces -> switch between them -> correct restore
- Rename workspace -> name persists after restart
- Delete workspace -> terminals killed, workspace removed from sidebar

**Manual verification:**
- Sidebar shows all workspaces after restart
- Bold indicator on active workspace
- Ctrl+Tab cycles through workspaces

## Success Criteria

1. Workspace saved manually persists after Force Quit + relaunch
2. Layout (tabs/splits) restored correctly
3. All keyboard shortcuts work (Ctrl+Tab, Cmd+1-9)
4. Sidebar shows correct workspace names and active state
5. Delete kills all terminal processes in that workspace
