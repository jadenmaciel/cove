# Roadmap: Cove

**Phases:** 4 | **Requirements:** 13 mapped | **Mode:** YOLO

## Phase Overview

| # | Phase | Goal | Requirements | Success Criteria |
|---|-------|------|--------------|-----------------|
| 1 | Permanent Workspaces | Save/restore named workspaces across sessions | WS-01, WS-02, WS-03 | 3 |
| 2 | Tab Management | Harder to close accidentally, better nav | TAB-01, TAB-02, TAB-03 | 3 |
| 3 | Split UX | Easier splits, better shortcuts, persistence | SPLIT-01, SPLIT-02, SPLIT-03 | 3 |
| 4 | Liquid Glass UI | Apple glass morphism, polished feel | UI-01, UI-02, UI-03, UI-04 | 4 |

## Phase Details

### Phase 1: Permanent Workspaces

**Goal:** Save/restore named workspaces across sessions with fast switching.

**Requirements:**
- WS-01: Permanent workspaces — save/restore named project workspaces across sessions
- WS-02: Fast workspace switching — keyboard shortcuts and UI for navigating many workspaces quickly
- WS-03: Workspace naming and labeling — clear names for project workspaces

**Success Criteria:**
1. User can name and save a workspace and it persists after app restart
2. User can switch between workspaces using keyboard (Ctrl+Tab, Cmd+1-9)
3. Sidebar shows workspace names clearly with bold active indicator

**Plans:** 3 plans

Plans:
- [ ] 01-01-PLAN.md — Core save/restore: PermanentWorkspaceRecord creation and layout snapshot capture/restore
- [ ] 01-02-PLAN.md — Fast switching: Ctrl+Tab cycling, Cmd+1-9, sidebar click, bold active indicator
- [ ] 01-03-PLAN.md — Naming and deletion: inline rename, archive+kill delete

---

### Phase 2: Tab Management

**Goal:** Tabs are organized, hard to close accidentally, and keyboard-navigable.

**Requirements:**
- TAB-01: Improved tab management — harder to close accidentally, better close confirmation
- TAB-02: Tab organization — visual grouping, better tab bar UX
- TAB-03: Tab keyboard navigation — Cmd+1-9 for tabs, better Ctrl+Tab behavior

**Success Criteria:**
1. Closing a tab with active process prompts for confirmation
2. User can navigate tabs with Ctrl+Tab and Cmd+1-9
3. Tab bar shows tab titles clearly and stays organized

---

### Phase 3: Split UX

**Goal:** Splits are easy to create, navigate, and persist.

**Requirements:**
- SPLIT-01: Improved split/pane UX — clearer split affordances, easier creation
- SPLIT-02: Better keyboard shortcuts for splits — directional focus, creation
- SPLIT-03: Split persistence — splits saved and restored with workspace

**Success Criteria:**
1. User can create splits with Cmd+D and Cmd+Shift+D (horizontal/vertical)
2. User can focus adjacent panes with Option+Arrow keys
3. Splits are restored when workspace reopens

---

### Phase 4: Liquid Glass UI

**Goal:** Cove looks like a polished Apple-first developer tool with glass morphism.

**Requirements:**
- UI-01: Apple Liquid Glass morphism — glassy surfaces, blur materials, `.regularMaterial` styling
- UI-02: Polished macOS-native feel — proper materials, subtle animations, keyboard-first
- UI-03: Better empty states — clear onboarding when no workspaces exist
- UI-04: Dark mode first — all surfaces look great in dark mode

**Success Criteria:**
1. Sidebar and panels use blur materials and glassy surfaces
2. Animations are smooth and subtle (no janky transitions)
3. Empty state shows helpful onboarding when no workspaces exist
4. Dark mode is the default and looks polished

---

## Execution

Run `/gsd-discuss-phase 1` to start Phase 1.
