---
title: Gravitype Game Core Service
type: service
source_repositories:
  - gravitype
generated_at: 2026-07-14T00:00:00Z
akashic_version: 1.0
---

## Purpose

The Game Core service manages game state, word generation, and scoring logic for the Gravitype typing game. It handles word list management, category selection, and scoring calculations.

## Public API

### Word Management
- **`get_words_by_category(category: str) -> List[str]`** — Retrieves word list for a category (tech, general, mixed)
- **`generate_word_batch(category: str, level: int) -> List[WordItem]`** — Generates falling words based on difficulty level

### Scoring
- **`calculate_score(word: str, level: int) -> int`** — Computes points for correctly typed word
- **`check_word_match(typed: str, active_words: List[str]) -> int`** — Validates user input against active words, returns score or 0

## Dependencies

- `gravitype.core.config` — Configuration management for game settings
- `gravitype.core.words` — Word list data and category management

## Data Ownership

Owns:
- Active word list state during gameplay
- Score and level progression
- Category-specific word pools

Delegates to:
- Config service — high scores, theme, sound settings

## Operational Notes

- Word selection is deterministic per level/category to ensure consistent difficulty progression
- Level increases every 150 points (configurable in config service)
- No external word API; all words are bundled in `gravitype.core.words`
- Category switching only allowed from main menu, not during gameplay

## Known Risks

- No word validation during generation; words list must be pre-vetted
- No rate limiting on word generation; fast systems could create cascading words
- Single word pool per category limits game variety over long sessions
