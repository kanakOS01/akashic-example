---
title: Gravitype UI Engine
type: service
source_repositories:
  - gravitype
generated_at: 2026-07-14T00:00:00Z
akashic_version: 1.0
---

## Purpose

The UI Engine provides the Textual-based terminal interface for Gravitype, managing screen lifecycle, widget rendering, input handling, and visual state updates.

## Public API

### Screen Management
- **`GravitypeApp`** — Main application entry point; orchestrates screen transitions
- **Methods:**
  - `show_menu()` — Return to main menu screen
  - `start_new_game()` — Initialize and display game screen with state reset
  - `end_game()` — Transition to game over screen and persist high score
  - `action_open_github()` — Open project GitHub in browser

### Game Board Widget
- **`GameBoard`** — Renders falling words, manages animations
  - `check_match(typed: str) -> int` — Validate user input, return score gained
  - `clear_board()` — Reset word list and state
  - `is_paused` — Boolean state for pause overlay

### Header Widget
- **`HeaderWidget`** — Display score, level, category, lives
- **`MainHeader`** — Navigation header with menu buttons

## Dependencies

- Textual >= 8.2.7 — TUI framework for widgets, screens, and reactive updates
- `gravitype.core.config` — Theme, sound, and game settings
- `gravitype.core.words` — Word data

## Data Ownership

Owns:
- Screen state and navigation stack
- Widget tree and render state
- User input event handling (keyboard, mouse)
- CSS styling (theme compilation at startup)

Shares:
- Game state (score, level, lives, category) with App reactive properties
- Config state with Config service

## Operational Notes

- Theme is compiled to `theme_active.tcss` at app startup from selected theme + base styles
- Input field auto-focuses after each correct match to maintain gameplay UX
- Pause state disables input but keeps board rendering (for visual feedback)
- Flash animations (hit/miss) use `set_timer()` for 0.15s CSS class toggles
- High score persists to `.gravitype_config.json` on game end

## Known Risks

- CSS hot-reloading enabled (`watch_css=True`) can cause performance issues in fast typing
- No input debouncing; rapid keypresses could trigger race conditions in `check_match()`
- Pause state disables input but doesn't stop time progression on falling words—resume could surprise players with missed words
- Theme compilation errors silently fall back to dracula theme; user changes may not apply
