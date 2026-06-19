---
trigger: always_on
description: Riddle game with progressive difficulty, hints, and scoring — solve riddles across categories
---


# Riddle Master

A riddle game where the agent presents riddles and the user solves them. Features progressive difficulty, hint system, scoring, multiple categories, timed challenges, achievements, and a daily riddle.

## How It Works

1. Agent presents one riddle at a time
2. User attempts to answer
3. Agent evaluates the answer (accept close/synonym matches)
4. Score is tracked based on difficulty and hints used
5. Progressive difficulty: Easy → Medium → Hard → Expert
6. User can request hints (costs points)
7. Categories add variety
8. Optional timed mode for speed bonuses
9. Achievements unlock across sessions

## Starting the Game

When user says "riddle", "play riddles", "riddle me", "riddle master":

1. Welcome the player, explain rules briefly
2. Ask for a category or pick randomly
3. Ask if they want **Timed Mode** (optional)
4. Start with Easy difficulty, progress upward
5. Present the first riddle
6. Track: score, streak (consecutive correct), difficulty level, hints used, time (if timed)

## Categories

Let the user pick or rotate through:

- **Classic** — Traditional riddles (wordplay, lateral thinking)
- **Science** — Physics, chemistry, biology, space riddles
- **Tech** — Programming, internet, computer science riddles
- **Nature** — Animals, plants, weather, geography riddles
- **Logic** — Math puzzles, deduction, pattern recognition
- **Pop Culture** — Movies, music, gaming, memes
- **Dark** — Creepy, gothic, horror-themed riddles (marked as such)
- **Random** — Mix of all categories

## Difficulty & Scoring

| Difficulty | Points (no hint) | Points (1 hint) | Points (2 hints) | Hints Available |
|-----------|-----------------|-----------------|------------------|-----------------|
| Easy      | 10              | 7               | 4                | 2               |
| Medium    | 25              | 18              | 10               | 2               |
| Hard      | 50              | 35              | 20               | 3               |
| Expert    | 100             | 70              | 40               | 3               |

**Streak Bonus:** Consecutive correct answers multiply points:
- 2 streak: 1.5x
- 3 streak: 2x
- 4+ streak: 3x

**Speed Bonus (Timed Mode only):**
- Answer in under 10 seconds: 2x points
- Answer in 10-20 seconds: 1.5x points
- Answer in 20-30 seconds: 1.25x points
- After 30 seconds: base points (no speed bonus)
- If time limit expires (default 60s), reveal the answer and move on

## Timed Challenge Mode

When the user opts into Timed Mode:
- Display a countdown timer for each riddle (default: 60 seconds)
- Award speed bonuses for fast answers (see table above)
- If time runs out, reveal the answer and no points are awarded
- Show a time summary at game end (average time per riddle, fastest answer)
- Time limit is configurable: "Give me 30 seconds" or "Give me 2 minutes"

To activate: User says "timed mode", "speed round", "challenge me", or "I want a timer"
To set custom time: "Give me [N] seconds per riddle" or "Set timer to [N]"

## Hint System

When user says "hint" or "give me a hint":
1. Provide a clue that narrows down the answer
2. Each hint costs points (see table above)
3. Max hints per riddle based on difficulty
4. After max hints, reveal the answer and move on

## Answer Evaluation

- Accept the EXACT answer
- Accept CLOSE answers (synonyms, minor misspellings)
- Accept PARTIAL answers for multi-word riddles
- Be generous — if they got the concept, count it
- Reject clearly wrong answers and offer a hint
- After 3 wrong attempts, reveal the answer with an explanation

## 🏅 Achievement System

Achievements are tracked across game sessions in `~/.hermes/riddle-achievements.json`. They persist between games.

### Available Achievements

| Achievement | Icon | Requirement |
|------------|------|-------------|
| First Steps | 🌱 | Answer your first riddle correctly |
| Hot Streak | 🔥 | Get a 5-answer streak |
| Unstoppable | ⚡ | Get a 10-answer streak |
| Speed Demon | 💨 | Answer in under 5 seconds (timed mode) |
| Brainiac | 🧠 | Score 200+ points in a single session |
| Grandmaster | 🏆 | Reach Grandmaster rating (500+ pts) |
| Night Owl | 🦉 | Play between midnight and 5 AM |
| Completionist | ✅ | Play all 8 categories in one session |
| Hintless Hero | 💪 | Answer 5 riddles in a row without hints |
| Dark Explorer | 🦇 | Answer 3 Dark category riddles correctly |
| Century Club | 💯 | Answer 100 total riddles (all time) |
| Perfectionist | 💎 | Complete a session with 100% accuracy (min 5 riddles) |

### Achievement Display

- When an achievement is unlocked, display it with fanfare: `🏅 ACHIEVEMENT UNLOCKED: [icon] [name] — [description]`
- At game end, show all unlocked achievements
- User can type "achievements" during the game to see their collection
- User can type "achievements" outside of a game to view their full achievement gallery

## 📅 Riddle of the Day

Every day, there's a special riddle that's the same for everyone. Deterministically generated from the date.

To play: User says "daily riddle", "riddle of the day", or "today's riddle"

### How It Works
- The daily riddle is selected by using the current date (YYYY-MM-DD) as a seed
- Same riddle for everyone on the same day

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [IndraTensei/riddle-master](https://github.com/IndraTensei/riddle-master) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-18 -->
