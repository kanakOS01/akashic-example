---
title: Gravitype Game Architecture
type: system
source_repositories:
  - gravitype
generated_at: 2026-07-14T00:00:00Z
akashic_version: 1.0
---

## Boundaries

Gravitype is a single-player terminal-based typing game. It does not include:
- Multiplayer or networking
- Cloud saves or remote sync
- Sound/audio generation (only system beep)
- Graphics rendering (terminal only)
- Mobile platforms or web version

Boundary APIs:
- **Inbound:** Keyboard input (readline), terminal display (Textual)
- **Outbound:** Filesystem (`.gravitype_config.json`), system clipboard (copy high score)

## Invariants

1. **Single Instance** — Only one game instance per terminal session
2. **Lives Enforcement** — Game ends when lives reach 0
3. **Score Progression** — Levels determined by score (1 + score // 150)
4. **Category Immutability** — Category fixed per game; cannot switch during gameplay
5. **Word Pool Consistency** — Same category always uses same word pool
6. **State Isolation** — Game state not shared between sessions

## Runtime Topology

```
Terminal (Textual Framework)
    ↓
GravitypeApp (Main application)
    ├→ Reactive properties (score, level, lives, category, high_score)
    ├→ Screen stack (main, game, game_over)
    │   ├→ MainScreen (welcome, settings, help, about)
    │   ├→ GameScreen (header, game board, input field)
    │   └→ GameOverScreen (stats, restart options)
    ├→ Widgets
    │   ├→ GameBoard (falling words, collision detection)
    │   ├→ HeaderWidget (score, level, lives display)
    │   ├→ MainHeader (navigation)
    │   └→ SettingsScreen (theme, sound, lives config)
    └→ Config service (persistent state)
        ↓
    Filesystem (.gravitype_config.json)
```

### Game Loop (Simplified)

1. **Boot:** Load config, compile theme CSS, initialize app
2. **Menu:** Display welcome screen with category selector
3. **Start Game:** Create GameScreen, reset state, focus input
4. **Render:** GameBoard updates word positions each frame
5. **Input:** User types; matches checked against active words
6. **Score:** On match, score and level updated; state synced to header
7. **Falling:** Words fall; on miss, life lost; on 0 lives, game ends
8. **Game Over:** Display final stats, update high score if new
9. **Repeat:** Return to menu for next game

## Integration Points

### Filesystem
- **Config file:** `.gravitype_config.json` in current directory
  - Stores: high_score, theme, sound_enabled, starting_lives
  - Persistence: Saved after game over if new high score
  - Error handling: Silently reverts to defaults on read failure

### Textual Framework
- **CSS engine:** Compiles theme + base styles to `theme_active.tcss`
  - Watch mode enabled for hot reload during development
  - Theme fallback to Dracula if file not found
- **Event system:** Keyboard input routed to Input widget's `on_input_changed` handler
- **Timers:** Used for animation (flash effects) with `set_timer(0.15, callback)`
- **Reactive properties:** Score, level, lives trigger widget updates

### System Features
- **Bell/Audio:** `self.app.bell()` rings system beep on missed word
- **Hyperlink:** `open_github()` action opens GitHub URL in default browser

## Data Flow Diagram

```
Game Initialization
    ├─ Load config from filesystem
    ├─ Generate theme CSS
    ├─ Initialize reactive state (score=0, level=1, lives=3)
    └─ Display main menu

Category Selection
    ├─ User selects tech/general/mixed
    ├─ Category stored in app.category
    └─ Ready to start

Game Start
    ├─ Reset score, level, lives
    ├─ Load word pool for category
    ├─ Create falling word objects with random y, speed
    ├─ Display game screen
    └─ Focus input field

Word Matching Loop
    ├─ User types characters
    ├─ Each keystroke checked against active words
    ├─ On match:
    │   ├─ Remove word from board
    │   ├─ Increment score
    │   ├─ Sync state to header
    │   └─ Clear input
    ├─ On non-match prefix:
    │   ├─ Add 'typo' CSS class (visual feedback)
    │   └─ Keep input active
    └─ On invalid prefix:
        └─ Display typo class, clear on valid prefix

Falling Physics
    ├─ Each frame: y_position += speed
    ├─ Speed = base_speed * (1 + 0.1 * (level - 1))
    ├─ When y_position >= board_height:
    │   ├─ Word missed
    │   ├─ Decrement lives
    │   ├─ Ring bell (if sound_enabled)
    │   ├─ Flash input field red
    │   └─ Check if lives <= 0
    └─ If lives <= 0:
        └─ Transition to game_over screen

Game End
    ├─ Check if new high score
    ├─ Save high score to config
    ├─ Display game over stats
    ├─ Offer replay, menu, or quit
    └─ Return to menu or exit
```

## Known Risks & Constraints

- **No persistence for game history:** Only high score saved; no replay/stats
- **Single word pool per category:** Limited variety; repetition over long sessions
- **Deterministic word selection:** Same seed → same word order (predictable)
- **No difficulty scaling beyond score:** Could add harder categories or boss words
- **No pausing time progression:** Pause disables input but words still fall
- **CSS hot reload enabled:** Can cause performance glitches during gameplay
- **Theme compilation errors silent:** Falls back to dracula; user changes may not apply
- **No network features:** Cannot upload high scores or compete with others
- **No accessibility features:** Terminal-based; limited support for screen readers
