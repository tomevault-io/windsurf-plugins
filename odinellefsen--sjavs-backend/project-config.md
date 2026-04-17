---
trigger: always_on
description: This final step implements the authentic Sjavs Cross/Rubber scoring system where teams start with 24 points and count down. When a team reaches or passes zero, they win a "cross". This system determines the overall match winner across multiple games.
---

# Step 5: Cross/Rubber Management System

## Overview

This final step implements the authentic Sjavs Cross/Rubber scoring system where teams start with 24 points and count down. When a team reaches or passes zero, they win a "cross". This system determines the overall match winner across multiple games.

## Implementation Tasks

### 1. Create Cross/Rubber State Management

**File: `src/game/cross.rs`**

```rust
use serde::{Deserialize, Serialize};
use crate::game::scoring::GameResult;

/// Cross/Rubber state tracking for a match
#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct CrossState {
    /// Current score for trump team (starts at 24, counts down)
    pub trump_team_score: i8,
    /// Current score for opponent team (starts at 24, counts down)
    pub opponent_team_score: i8,
    /// Number of crosses won by trump team
    pub trump_team_crosses: u8,
    /// Number of crosses won by opponent team
    pub opponent_team_crosses: u8,
    /// Bonus points for next game (from ties)
    pub next_game_bonus: u8,
    /// Match ID this cross state belongs to
    pub match_id: String,
    /// Whether the rubber is complete
    pub rubber_complete: bool,
}

impl CrossState {
    /// Create new cross state for a match
    pub fn new(match_id: String) -> Self {
        Self {
            trump_team_score: 24,
            opponent_team_score: 24,
            trump_team_crosses: 0,
            opponent_team_crosses: 0,
            next_game_bonus: 0,
            match_id,
            rubber_complete: false,
        }
    }

    /// Apply game result to cross scores
    pub fn apply_game_result(&mut self, game_result: &GameResult) -> CrossResult {
        // Apply bonus points if any
        let trump_team_points = game_result.trump_team_score + self.next_game_bonus;
        let opponent_team_points = game_result.opponent_team_score;
        
        // Reset bonus after use
        self.next_game_bonus = 0;

        // Handle tie scenario (both teams get 60 points)
        if game_result.trump_team_score == 0 && game_result.opponent_team_score == 0 {
            // This was a tie - add 2 to next game bonus
            self.next_game_bonus = 2;
            return CrossResult {
                trump_team_old_score: self.trump_team_score,
                opponent_team_old_score: self.opponent_team_score,
                trump_team_new_score: self.trump_team_score,
                opponent_team_new_score: self.opponent_team_score,
                cross_won: None,
                bonus_applied: 0,
                next_game_bonus: self.next_game_bonus,
                rubber_complete: false,
            };
        }

        let old_trump_score = self.trump_team_score;
        let old_opponent_score = self.opponent_team_score;

        // Subtract points from respective teams
        self.trump_team_score -= trump_team_points as i8;
        self.opponent_team_score -= opponent_team_points as i8;

        // Check for cross completion
        let cross_won = self.check_cross_completion();

        CrossResult {
            trump_team_old_score: old_trump_score,
            opponent_team_old_score: old_opponent_score,
            trump_team_new_score: self.trump_team_score,
            opponent_team_new_score: self.opponent_team_score,
            cross_won,
            bonus_applied: if trump_team_points > game_result.trump_team_score { 
                trump_team_points - game_result.trump_team_score 
            } else { 0 },
            next_game_bonus: self.next_game_bonus,
            rubber_complete: self.rubber_complete,
        }
    }

    /// Check if a cross has been completed
    fn check_cross_completion(&mut self) -> Option<CrossWinner> {
        // Check if trump team won
        if self.trump_team_score <= 0 {
            // Check for double victory
            let double_victory = self.opponent_team_score == 24;
            
            self.trump_team_crosses += 1;
            self.rubber_complete = true; // For now, one cross = complete rubber
            
            return Some(CrossWinner {
                winning_team: CrossTeam::TrumpTeam,
                double_victory,
                final_score: (self.trump_team_score, self.opponent_team_score),
                crosses_won: self.trump_team_crosses,
            });
        }

        // Check if opponent team won
        if self.opponent_team_score <= 0 {
            // Check for double victory
            let double_victory = self.trump_team_score == 24;
            
            self.opponent_team_crosses += 1;
            self.rubber_complete = true; // For now, one cross = complete rubber
            
            return Some(CrossWinner {
                winning_team: CrossTeam::OpponentTeam,
                double_victory,
                final_score: (self.trump_team_score, self.opponent_team_score),
                crosses_won: self.opponent_team_crosses,
            });
        }

        None
    }

    /// Check if teams are "on the hook" (6 points remaining)
    pub fn get_hook_status(&self) -> (bool, bool) {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/odinellefsen) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
