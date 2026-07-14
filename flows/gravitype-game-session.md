---
title: Gravitype Game Session Flow
type: flow
source_repositories:
  - gravitype
generated_at: 2026-07-14T00:00:00Z
akashic_version: 1.0
---

## Trigger

User selects "START GAME" from welcome screen with a chosen category.

## Actors

- **User** — Keyboard input
- **GravitypeApp** — Main application state machine
- **GameScreen** — Active game display and input handling
- **GameBoard** — Word rendering and physics simulation
- **Config Service** — Persistent settings

## Ordered Steps

1. **Game Initialization**
   - GameScreen created, mounted
   - Reset input field, clear board
   - Initialize score=0, level=1, lives=3 (or from config)
   - Load word pool for selected category

2. **Display Game UI**
   - Render HeaderWidget (score, level, lives)
   - Render GameBoard (empty, ready for words)
   - Focus input field (keyboard ready)
   - Set up bindings: ESC to pause, Ctrl+Q to exit

3. **Word Generation Loop** (runs every frame, ~60fps)
   - Calculate words needed based on level
   - Spawn new words at top of board with:
     - Random word from category pool
     - Random x position
     - Speed based on level
   - Add to active_words list

4. **Rendering Loop** (every frame)
   - Update word y_position by speed (fall animation)
   - Re-render game board with current positions
   - Highlight words near bottom (visual warning)

5. **Input Processing** (on keystroke)
   - User types character
   - Input field fires `on_input_changed` event
   - Get current typed text (stripped)
   - Check if exact match with any active word

6. **Word Match** (if typed text matches)
   - Calculate score: base_score * level_multiplier
   - Remove matched word from board
   - Add score to total
   - Recalculate level: 1 + (total_score // 150)
   - Clear input field (triggers next on_input_changed)
   - Flash input container green (0.15s animation)
   - Sync header display

7. **Partial Match** (if typed is valid prefix)
   - Remove 'typo' CSS class from input
   - Keep input focused
   - Wait for more characters or clear

8. **No Match** (if typed is not prefix of any active word)
   - Add 'typo' CSS class to input (red highlight)
   - User must delete characters to fix

9. **Word Miss** (if word reaches bottom without match)
   - Remove word from active_words
   - Decrement lives by 1
   - Flash input container red (0.15s animation)
   - Ring bell/beep (if sound_enabled in config)
   - Sync header display
   - Check if lives <= 0

10. **Game Over** (if lives reach 0)
    - Store final score
    - Check if score > high_score
    - If new high score:
      - Update high_score in config
      - Save to `.gravitype_config.json`
    - Transition to GameOverScreen
    - Show stats: final score, level, high score (if new)

## Branch Cases

### User Pauses Game
- **Trigger:** User presses ESC
- **Action:** `action_toggle_pause()` called
- **Behavior:**
  - Set `board.is_paused = True`
  - Disable input field
  - Words continue falling (gap: should they?)
  - Show "PAUSED - ESC to Resume | Ctrl+Q to Exit"
- **Resume:** User presses ESC again, resume gameplay

### User Exits to Menu
- **Trigger:** User presses Ctrl+Q during game
- **Action:** `action_exit_to_menu()` called
- **Behavior:**
  - Discard current game state (score not saved)
  - Transition to MainScreen
  - Reset app state to defaults

### User Reaches New High Score
- **Trigger:** Final score > previous high_score
- **Behavior:**
  - Update app.high_score reactive property
  - Call `config.set("high_score", new_score)`
  - Config saves to filesystem
  - Display "[NEW HIGH SCORE!]" in game over screen

### Sound Disabled
- **Trigger:** `sound_enabled` is False in config
- **Behavior:** Beep skipped on word miss (no sound output)

## External Calls

- **Filesystem:** Read word list from `gravitype.core.words.py`
- **Filesystem:** Read config from `.gravitype_config.json` at startup
- **Filesystem:** Write config after game over if new high score
- **System:** Call `app.bell()` to ring terminal beep (if sound_enabled)

## Failure Modes

| Scenario | Impact | Recovery |
|----------|--------|----------|
| Word list empty or missing | Game crashes on word generation | Restart app; check words.py |
| Config file corrupted | Defaults used; new file created | User loses saved config |
| Theme CSS fails to compile | Falls back to dracula theme | User changes theme in settings |
| Input focus lost | Keyboard input doesn't register | Click input field to regain focus |
| Pause keeps words falling | Player loses words while paused | Current behavior; gap noted |

## Resulting State

### Game Won (Score High)
- Score persisted to high_score if new
- Config file updated
- User can view high score in welcome screen
- User can replay to try higher score

### Game Lost (Lives Exhausted)
- Final score displayed
- High score shown (with [NEW HIGH SCORE!] if applicable)
- User can:
  - PLAY AGAIN — return to step 1 with fresh state
  - MAIN MENU — return to welcome screen
  - QUIT GAME — exit application

## Success Criteria

- Game starts without errors
- Words fall and render correctly
- User input correctly matches/mismatches words
- Score increases on correct matches
- Level increases every 150 points
- Lives decrease on misses
- Game ends when lives reach 0
- High score persists across sessions
