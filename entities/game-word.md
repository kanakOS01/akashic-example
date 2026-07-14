---
title: Game Word Entity
type: entity
source_repositories:
  - gravitype
generated_at: 2026-07-14T00:00:00Z
akashic_version: 1.0
---

## Definition

A Game Word is a text string displayed as a falling object in Gravitype gameplay. Words are selected from category-specific word lists and rendered with animation state.

## Fields

| Field | Type | Required | Constraints | Notes |
|-------|------|----------|-------------|-------|
| `text` | string | Yes | 2-20 chars | The word to type |
| `category` | enum | Yes | `tech`, `general`, `mixed` | Word source category |
| `y_position` | int | Yes | 0-board_height | Vertical position; increases downward |
| `level` | int | Yes | 1+ | Difficulty tier; affects spawn speed |
| `speed` | float | Yes | 0.1-2.0 | Fall velocity; derived from level |
| `is_active` | bool | Yes | N/A | Not yet matched or missed |
| `render_time` | float | Yes | 0+ | Frames since spawn; used for animation |

## Lifecycle

1. **Generation** — Word selected from category word pool; spawned at top of board
2. **Falling** — Word falls at speed determined by level; position increments each frame
3. **Match** — Player types word correctly; word removed; score awarded
4. **Miss** — Word reaches bottom of board without match; life lost; word removed
5. **Cleanup** — Word object destroyed after match or miss

## Relationships

- **Many-to-Game: Game State** — Multiple words exist per active game
- **One-to-Category: Word Pool** — Word selected from one category's list

## Constraints

- Text must be from approved word list for category; no arbitrary strings
- Y position is 0 (top) to board_height (bottom)
- Speed is calculated as `base_speed * (1 + 0.1 * (level - 1))` or similar
- Only one instance per word at a time (no duplicates in active set)
- Words are immutable during gameplay; cannot be edited or renamed

## Examples

### Active Word During Gameplay
```python
{
  "text": "Python",
  "category": "tech",
  "y_position": 150,
  "level": 3,
  "speed": 1.3,
  "is_active": True,
  "render_time": 42.5
}
```

### Word List Entry (from gravitype/core/words.py)
```json
{
  "tech": ["Python", "JavaScript", "Docker", "Kubernetes", "API", ...],
  "general": ["apple", "elephant", "mountain", "computer", ...],
  "mixed": [all words from tech and general combined]
}
```

## Known Gaps

- No word difficulty metadata (all words equally weighted)
- No word pronunciation or hints
- No replay/review of words used in past games
- No user-submitted words or custom word lists
- No translation to other languages
- Word list is hardcoded; no database or dynamic loading
