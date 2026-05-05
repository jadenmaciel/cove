# Cove — macOS Workspace Cockpit

## What This Is

Native macOS terminal workspace app built on Ghostty/cmux. A workspace cockpit for running terminal sessions, AI coding agents (Claude Code, Codex), and browser panes side-by-side. Feels like a polished Apple-first developer tool — not a web dashboard.

## Core Value

A local-first macOS workspace where multiple terminal workspaces with tabs and splits feel instant, organized, and beautiful — with Apple Liquid Glass aesthetics.

## Requirements

### Validated

(None yet — ship to validate)

### Active

- [ ] **WS-01**: Permanent workspaces — save/restore named project workspaces across sessions
- [ ] **WS-02**: Better workspace switching — fast nav between many workspaces
- [ ] **TAB-01**: Improved tab management — harder to close accidentally, better organization
- [ ] **SPLIT-01**: Improved split/pane UX — better keyboard shortcuts, easier management
- [ ] **UI-01**: Apple Liquid Glass morphism — glassy surfaces, blur materials, Codex-like aesthetic
- [ ] **UI-02**: Polished macOS-native feel — proper materials, animations, keyboard-first workflows

### Out of Scope

- Browser panes (deferred — gap but not current priority)
- SSH workspaces (deferred — gap but not current priority)
- Inspector drawer (deferred — gap but not current priority)

## Context

**Current state (halfway):**
- Notification rings: working
- Sidebar: working
- Basic workspaces/tabs/splits: functional but rough UX
- Liquid Glass / Codex-style aesthetics: not implemented

**Inspiration:**
- Apple Liquid Glass: https://developer.apple.com/documentation/TechnologyOverviews/adopting-liquid-glass
- Codex app aesthetic (Anthropic's own app)
- Conductor / T3 Code for workspace-first layout

**Tech:**
- Swift/AppKit (native macOS)
- Ghostty terminal engine (libghostty via submodule)
- Bonsplit for tab/pane management

## Constraints

- **Tech stack**: Swift/AppKit, native — no Electron/Tauri
- **Aesthetic**: Apple Liquid Glass, dark-mode-first, macOS-native
- **Performance**: Fast startup, low memory, GPU-accelerated terminal

## Key Decisions

| Decision | Rationale | Outcome |
|----------|-----------|---------|
| Liquid Glass aesthetic | Codex sets the bar; Cove should match Apple's latest design language | — Pending |
| Permanent workspaces first | Core workflow blocker — current workspaces don't persist well | — Pending |

---
*Last updated: 2026-05-06 after initialization*
