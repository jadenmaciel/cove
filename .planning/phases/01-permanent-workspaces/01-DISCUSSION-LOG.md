# Phase 1: Permanent Workspaces - Discussion Log

> **Audit trail only.** Do not use as input to planning, research, or execution agents.
> Decisions are captured in CONTEXT.md — this log preserves the alternatives considered.

**Date:** 2026-05-06
**Phase:** 1-Permanent Workspaces
**Areas discussed:** Naming, Switching, Scope, Creation, Active Indicator, Deletion

---

## Naming

| Option | Description | Selected |
|--------|-------------|----------|
| Inline edit in sidebar | Double-click workspace name to rename inline | |
| Dialog on first save | Modal dialog on first save asking for name | |
| Auto-name from directory | Default to directory name, rename later | ✓ |

**User's choice:** Auto-name from directory
**Notes:** Minimizes friction — workspace auto-named on creation, user can rename via double-click

---

## Rename Flow (follow-up to Naming)

| Option | Description | Selected |
|--------|-------------|----------|
| Inline (double-click) | Double-click name in sidebar to edit in place | ✓ |
| Via command palette | Cmd+Shift+P then 'Rename Workspace' | |
| Right-click context menu | Right-click workspace for Rename option | |

**User's choice:** Inline (double-click)
**Notes:** Same pattern as tab renaming — consistent with existing UX

---

## Switching

| Option | Description | Selected |
|--------|-------------|----------|
| Cmd+1-9 (keyboard) | Cmd+1-9 switches to workspace 1-9 | |
| Sidebar click | Click workspace in sidebar | ✓ |
| Ctrl+Tab like tabs | Ctrl+Tab cycles through workspaces | ✓ |

**User's choice:** Ctrl+Tab and sidebar click
**Notes:** Both — Ctrl+Tab for keyboard nav, sidebar for mouse users

---

## Many Workspaces (follow-up to Switching)

| Option | Description | Selected |
|--------|-------------|----------|
| Cmd+Tab like app switcher | Holding Cmd+Tab shows panel with all workspaces | ✓ |
| Command palette | Cmd+Shift+P, type workspace name | |
| Just sidebar (no shortcuts) | Sidebar + Ctrl+Tab cycling only | |

**User's choice:** Cmd+Tab like app switcher
**Notes:** macOS app switcher pattern — natural for users

---

## Scope

| Option | Description | Selected |
|--------|-------------|----------|
| Layout only (tabs + splits) | Restore tabs/splits, no scrollback/history | ✓ |
| Full state (layout + history) | Restore everything including scrollback | |
| Ask each time | Dialog on restore: layout only or full history | |

**User's choice:** Layout only (tabs + splits)
**Notes:** Faster, less storage — history not needed for workspace switching

---

## History Setting

| Option | Description | Selected |
|--------|-------------|----------|
| Always off (layout only) | Never save scrollback | ✓ |
| Per-workspace toggle | Right-click > 'Save History' | |
| Settings global toggle | One switch for all workspaces | |

**User's choice:** Always off (layout only)
**Notes:** Simplicity — layout-only is the consistent default

---

## Creation

| Option | Description | Selected |
|--------|-------------|----------|
| Manual only (button/shortcut) | Explicit Cmd+N or + button to create | ✓ |
| Auto on new directory | Auto-create when opening terminal in new directory | |
| Manual + auto-suggest | Suggest creating workspace but let user confirm | |

**User's choice:** Manual only (button/shortcut)
**Notes:** Keeps sidebar clean — user controls when workspaces appear

---

## Active Indicator

| Option | Description | Selected |
|--------|-------------|----------|
| Bold/highlighted text | Active workspace name is bold in sidebar | ✓ |
| Dot/indicator | Small dot next to active workspace | |
| Fills sidebar panel | Active workspace fills main content area | |

**User's choice:** Bold/highlighted text
**Notes:** Standard macOS sidebar pattern

---

## Deletion

| Option | Description | Selected |
|--------|-------------|----------|
| Archive only (hide) | Delete removes from sidebar, terminals stay alive | |
| Archive + kill terminals | Delete removes workspace AND kills all terminals | ✓ |
| Cannot delete | Workspaces cannot be deleted, only archived | |

**User's choice:** Archive + kill terminals
**Notes:** Clean slate on delete — no orphan terminals

---

## Claude's Discretion

- Pinned workspace sort behavior — existing `sortWorkspaces` pattern (pinned > sortIndex > updatedAt > name) acceptable
- No discussion of workspace pinning UX beyond sort order
- No discussion of workspace color/label customization

## Deferred Ideas

None — all discussion stayed within Phase 1 scope.
