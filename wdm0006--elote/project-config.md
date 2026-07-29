---
trigger: always_on
description: how to implement new competitors in elote
---

 # Implementing New Competitors in Elote

This guide explains how to implement new rating system competitors in the Elote framework using the unified interface provided by the `BaseCompetitor` abstract base class.

## Overview

Elote provides a unified interface for implementing rating systems through the `BaseCompetitor` abstract base class. This ensures that all rating systems follow the same API, making them interchangeable and compatible with arenas and other components of the framework.

## Step 1: Inherit from BaseCompetitor

All new competitor classes must inherit from `BaseCompetitor`:

```python
from elote.competitors.base import BaseCompetitor, InvalidRatingValueException, InvalidParameterException
from typing import Dict, Any, ClassVar, Type, TypeVar

T = TypeVar('T', bound='MyNewCompetitor')

class MyNewCompetitor(BaseCompetitor):
    """My new rating system competitor.
    
    Provide a detailed description of the rating system here, including
    its origins, key characteristics, and any relevant references.
    
    Class Attributes:
        _some_parameter (float): Description of the parameter. Default: value.
    """
    
    _some_parameter: ClassVar[float] = 1.0
```

## Step 2: Implement Required Methods

The `BaseCompetitor` class defines several abstract methods that must be implemented:

### 2.1. Constructor

```python
def __init__(self, initial_rating: float = 1500):
    """Initialize a new competitor.
    
    Args:
        initial_rating (float, optional): The initial rating of this competitor. Default: 1500.
        
    Raises:
        InvalidRatingValueException: If the initial rating is below the minimum rating.
    """
    if initial_rating < self._minimum_rating:
        raise InvalidRatingValueException(f"Initial rating cannot be below the minimum rating of {self._minimum_rating}")
        
    self._initial_rating = initial_rating
    self._rating = initial_rating
    # Initialize any other instance variables needed for your rating system
```

### 2.2. Rating Property

```python
@property
def rating(self) -> float:
    """Get the current rating of this competitor.
    
    Returns:
        float: The current rating.
    """
    return self._rating

@rating.setter
def rating(self, value: float) -> None:
    """Set the current rating of this competitor.
    
    Args:
        value (float): The new rating value.
        
    Raises:
        InvalidRatingValueException: If the rating value is below the minimum rating.
    """
    if value < self._minimum_rating:
        raise InvalidRatingValueException(f"Rating cannot be below the minimum rating of {self._minimum_rating}")
    self._rating = value
```

### 2.3. Expected Score

```python
def expected_score(self, competitor: BaseCompetitor) -> float:
    """Calculate the expected score (probability of winning) against another competitor.
    
    Args:
        competitor (BaseCompetitor): The opponent competitor to compare against.
        
    Returns:
        float: The probability of winning (between 0 and 1).
        
    Raises:
        MissMatchedCompetitorTypesException: If the competitor types don't match.
    """
    self.verify_competitor_types(competitor)
    # Implement your rating system's expected score calculation
    # Return a value between 0 and 1
```

### 2.4. Beat Method

```python
def beat(self, competitor: BaseCompetitor) -> None:
    """Update ratings after this competitor has won against the given competitor.
    
    This method updates the ratings of both this competitor and the opponent
    based on the match outcome where this competitor won.
    
    Args:
        competitor (BaseCompetitor): The opponent competitor that lost.
        
    Raises:
        MissMatchedCompetitorTypesException: If the competitor types don't match.
    """
    self.verify_competitor_types(competitor)
    # Implement your rating system's update logic for a win
    # Make sure to update both this competitor and the opponent
```

### 2.5. Tied Method

```python
def tied(self, competitor: BaseCompetitor) -> None:
    """Update ratings after this competitor has tied with the given competitor.
    
    This method updates the ratings of both this competitor and the opponent
    based on a drawn match outcome.
    
    Args:
        competitor (BaseCompetitor): The opponent competitor that tied.
        
    Raises:
        MissMatchedCompetitorTypesException: If the competitor types don't match.
    """
    self.verify_competitor_types(competitor)
    # Implement your rating system's update logic for a tie
    # Make sure to update both this competitor and the opponent
```

### 2.6. Export State

```python
def export_state(self) -> Dict[str, Any]:
    """Export the current state of this competitor for serialization.
    
    Returns:
        dict: A dictionary containing all necessary information to recreate
             this competitor's current state.
    """
    return {
        "initial_rating": self._initial_rating,
        "current_rating": self._rating,
        # Include any other instance variables needed to recreate the state
        "class_vars": {
            "some_parameter": self._some_parameter,
            # Include any class variables that might be configured
        },
    }
```

### 2.7. From State (Class Method)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wdm0006/elote](https://github.com/wdm0006/elote) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
