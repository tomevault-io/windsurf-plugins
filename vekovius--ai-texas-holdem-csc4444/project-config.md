---
trigger: always_on
description: A trainable poker bot requires comprehensive testing to ensure:
---


# Testing Standards & Practices

## Testing Philosophy

A trainable poker bot requires comprehensive testing to ensure:
1. **Game mechanics work correctly** (hand evaluation, pot calculation, etc.)
2. **Strategies are testable** (can compare different approaches)
3. **Opponent modeling is accurate** (statistics calculated correctly)
4. **ML training produces improvements** (new models outperform old)
5. **Edge cases are handled** (invalid actions, disconnections, etc.)

---

## Testing Pyramid

### Unit Tests (Base Layer)
Test individual functions and methods in isolation:
- Hand evaluator logic
- Pot odds calculations
- Opponent statistics tracking
- Action validation
- Card parsing and formatting

### Integration Tests (Middle Layer)
Test module interactions:
- Bot + Hand Evaluator
- Bot + Decision Engine
- Bot + Opponent Tracker
- Full message flow (WebSocket simulation)

### System Tests (Top Layer)
Test complete system against real scenarios:
- Full game simulations (using test_server.py)
- Multi-hand performance testing
- Strategy evaluation against different opponents
- Stress testing (many hands, edge cases)

---

## Unit Testing Standards

### Hand Evaluator Tests

```python
import pytest
from hand_evaluator import HandEvaluator

class TestHandEvaluator:
    def setup_method(self):
        self.evaluator = HandEvaluator()

    def test_royal_flush_detection(self):
        """Test that royal flush is correctly identified."""
        cards = ["As", "Ks", "Qs", "Js", "Ts"]
        result = self.evaluator.classify_hand(cards)

        assert result["hand_type"] == "royal_flush"
        assert result["rank"] == 10  # Highest rank

    def test_straight_flush_detection(self):
        """Test straight flush identification."""
        cards = ["9h", "8h", "7h", "6h", "5h"]
        result = self.evaluator.classify_hand(cards)

        assert result["hand_type"] == "straight_flush"
        assert result["rank"] == 9

    def test_four_of_kind(self):
        """Test four of a kind detection."""
        cards = ["Kd", "Kh", "Kc", "Ks", "7d"]
        result = self.evaluator.classify_hand(cards)

        assert result["hand_type"] == "four_of_kind"
        assert result["primary_rank"] == "K"

    def test_full_house(self):
        """Test full house detection."""
        cards = ["Qd", "Qh", "Qc", "8s", "8d"]
        result = self.evaluator.classify_hand(cards)

        assert result["hand_type"] == "full_house"
        assert result["trips_rank"] == "Q"
        assert result["pair_rank"] == "8"

    def test_flush(self):
        """Test flush detection."""
        cards = ["Ad", "Jd", "9d", "6d", "2d"]
        result = self.evaluator.classify_hand(cards)

        assert result["hand_type"] == "flush"

    def test_straight(self):
        """Test straight detection."""
        cards = ["Td", "9h", "8s", "7c", "6d"]
        result = self.evaluator.classify_hand(cards)

        assert result["hand_type"] == "straight"

    def test_ace_low_straight(self):
        """Test A-2-3-4-5 straight (wheel)."""
        cards = ["5d", "4h", "3s", "2c", "Ad"]
        result = self.evaluator.classify_hand(cards)

        assert result["hand_type"] == "straight"
        assert result["high_card"] == "5"  # Not ace in wheel

    def test_three_of_kind(self):
        """Test trips detection."""
        cards = ["7h", "7d", "7c", "Ks", "4h"]
        result = self.evaluator.classify_hand(cards)

        assert result["hand_type"] == "three_of_kind"

    def test_two_pair(self):
        """Test two pair detection."""
        cards = ["Jh", "Jd", "5c", "5s", "Ah"]
        result = self.evaluator.classify_hand(cards)

        assert result["hand_type"] == "two_pair"

    def test_one_pair(self):
        """Test pair detection."""
        cards = ["9d", "9h", "Kc", "8s", "3d"]
        result = self.evaluator.classify_hand(cards)

        assert result["hand_type"] == "pair"

    def test_high_card(self):
        """Test high card (no made hand)."""
        cards = ["As", "Jh", "9d", "6c", "3s"]
        result = self.evaluator.classify_hand(cards)

        assert result["hand_type"] == "high_card"

    def test_best_5_from_7(self):
        """Test finding best 5-card hand from 7 cards."""
        hole_cards = ["As", "Kh"]
        board = ["Qd", "Jc", "Tc", "3s", "2h"]
        result = self.evaluator.evaluate_postflop(hole_cards, board)

        assert result["hand_type"] == "straight"
        assert result["strength"] > 0.8  # Strong hand

    @pytest.mark.parametrize("cards,expected_outs", [
        (["Ah", "Kh", "Qh", "5h"], 9),  # Flush draw
        (["9s", "8h", "7d", "6c"], 8),  # Open-ended straight draw
        (["Td", "9s", "7h", "6c"], 4),  # Gutshot straight draw
    ])
    def test_draw_potential(self, cards, expected_outs):
        """Test out calculation for various draws."""
        board_cards = cards
        result = self.evaluator.calculate_draw_potential([], board_cards)

        assert result["num_outs"] == expected_outs
```

### Opponent Tracker Tests

```python
from opponent_tracker import OpponentTracker

class TestOpponentTracker:
    def setup_method(self):
        self.tracker = OpponentTracker()

    def test_initial_stats(self):
        """New player should have zero stats."""

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/vekovius) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
