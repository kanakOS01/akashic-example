---
title: Gravitype Textual Framework for TUI
type: adr
source_repositories:
  - gravitype
generated_at: 2026-07-14T00:00:00Z
akashic_version: 1.0
---

## Status

**Accepted** — Currently implemented; primary UI framework.

## Context

Gravitype is a terminal-based typing game requiring:
- Rich text rendering (colors, styles, animations)
- Keyboard input handling with low latency
- Terminal-agnostic portability (Unix/Windows/macOS)
- Reactive state management (score, lives, level updates)
- Widget library for reusable components

Initial prototypes considered:
1. Raw terminal I/O (curses/ncurses)
2. Rich Python library (rich + input)
3. **Textual framework** (Textualize)

## Decision

Use **Textual** framework (version >= 8.2.7) for all UI concerns.

### Key Design Choices

- **Reactive State:** Use `reactive` properties for score, level, lives; UI auto-updates
- **Screen Stack:** Use push/pop for menu → game → game_over transitions
- **CSS Styling:** Separate CSS from Python; use theme compilation at startup
- **Widget Composition:** Build UI from smaller widgets (header, board, input, footer)
- **Event Binding:** Use `@on()` decorators and `on_*()` methods for input handling

## Consequences

### Positive

1. **Rich Features:** Colors, styles, animations built-in; no manual escape codes
2. **Responsive:** Event-driven architecture; input processed immediately
3. **Portable:** Works on macOS, Linux, Windows, and over SSH
4. **Developer Friendly:** Python; familiar syntax; good documentation
5. **Hot Reload:** CSS changes apply live during development (`watch_css=True`)
6. **Accessible:** Terminal is inherently more accessible than GUI

### Negative

1. **Terminal Limited:** Cannot do anything GUI can do (no graphics, limited colors)
2. **Learning Curve:** Textual concepts (reactive, widgets, events) take time
3. **Performance:** Terminal I/O slower than native GUI; ~60fps is ceiling
4. **Dependencies:** Adds Textual as core dependency; version lock required
5. **Testing:** Terminal interaction harder to test; screenshot testing needed

## Alternatives Considered

### A. Curses / NCurses
**Pros:** Available everywhere; minimal dependencies; battle-tested
**Cons:** Low-level API; lots of boilerplate; no reactive state; tedious
**Rejected:** Too much manual work; poor developer experience

### B. Rich + Manual Input Loop
**Pros:** Lightweight; familiar Rich library
**Cons:** No event-driven architecture; DIY animation; DIY event handling
**Rejected:** Would reinvent Textual; Textual better suited

### C. Qt / PyQt (GUI)
**Pros:** Feature-rich; professional look; cross-platform
**Cons:** Violates terminal-only goal; heavier dependencies; overkill for typing game
**Rejected:** Not a terminal application; changes project scope

## Implementation Evidence

**File:** `gravitype/tui/app.py`
- `GravitypeApp` extends Textual App
- `reactive` properties for game state
- Screen stack management (main, game, game_over)

**File:** `gravitype/tui/widgets/game_board.py`
- Falling word rendering with CSS positioning
- Collision detection in `check_match()`
- Word animation via y_position updates

**File:** `gravitype/tui/widgets/header.py`
- Score, level, lives display
- Synced with reactive properties

**File:** `gravitype/core/config.py`
- Theme selection loaded from config
- Theme CSS compiled before app startup

## Open Questions / Gaps

1. **Performance:** Can we maintain 60fps with many words? Optimize rendering?
2. **Mobile Terminal:** Does Textual work on mobile SSH clients (e.g., SSH apps)?
3. **Accessibility:** How well do screen readers work with Textual?
4. **Multiplayer:** Could Textual support network play? (Beyond current scope)

## Related ADRs

- `adr/gravitype-word-difficulty-scaling.md` (proposed)
- `adr/gravitype-audio-implementation.md` (proposed for music/SFX)
