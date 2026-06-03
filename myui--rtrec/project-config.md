---
trigger: always_on
description: RTREC is a Python-based real-time recommendation system with the following features:
---

# RTREC Improvement Suggestions by Claude

## Overview of Current Status

RTREC is a Python-based real-time recommendation system with the following features:

- Supports online updates
- Fast implementation (training speed over 190k samples/sec on a laptop)
- Efficient sparse data support
- Time-based weighting of user-item interactions
- Supports two major algorithms: SLIM and LightFM

The codebase is structured as follows:
- `rtrec/models/`: SLIM and LightFM model implementations
- `rtrec/utils/`: Common utilities (interactions, identifiers, feature management)
- `rtrec/experiments/`: Dataset management and experiment utilities
- `rtrec/serving/`: Web serving functionality using FastAPI

## Improvement Recommendations

### 1. Code Quality and Maintainability

#### 1.1 Enhance Type Hints and Python 3.12 Compatibility

Current code has commented-out references to `@override` decorator:
```python
# require typing-extensions >= 4.5
# from typing import override
```

**Recommendation**: Enable the `@override` decorator now that Python 3.12 is supported, making method overrides explicit for better code readability and safety.

```python
from typing import override  # Available in standard library for Python 3.12+

@override
def fit(self, interactions: Iterable[Tuple[Any, Any, float, float]], update_interaction: bool=False, progress_bar: bool=True) -> None:
    # Implementation...
```

#### 1.2 Improved Error Handling

**Recommendation**: Use more specific exceptions with better error messages.

```python
class ModelNotFittedError(Exception):
    """Exception raised when the model hasn't been fitted yet"""
    pass

def predict(self, user_id: int, interaction_matrix: sp.csr_matrix, dense_output: bool=True) -> ndarray:
    if self.item_similarity is None:
        raise ModelNotFittedError("Model must be trained with fit() or partial_fit() before prediction")
    # Implementation...
```

#### 1.3 Reduce Code Duplication

Current implementations of `SLIM` and `LightFM` contain significant code duplication.

**Recommendation**: Extract common functionality to the base class, leaving only model-specific implementations in subclasses.

```python
# Example: Common _recommend_batch implementation for both models
def _recommend_batch(self, user_ids: List[int], candidate_item_ids: Optional[List[int]] = None, users_tags: Optional[List[List[str]]] = None, top_k: int = 10, filter_interacted: bool = True) -> List[List[int]]:
    # Common implementation
```

### 2. Performance Optimization

#### 2.1 Memory Usage Optimization

**Recommendation**: Optimize memory usage for large datasets.

```python
# Optimized CSCMatrixWrapper class example
def get_col(self, j: int, copy: bool = False) -> sp.spmatrix:
    """
    Only create copies when necessary to save memory
    """
    col = self.csc_matrix.getcol(j)
    return col.copy() if copy else col
```

#### 2.2 Enhanced Multiprocessing

Current multiprocessing implementation only supports CSC format:

```python
if isinstance(interaction_matrix, sp.csc_matrix):
    if parallel:
        return self.fit_in_parallel(interaction_matrix, progress_bar=progress_bar)
    # ...
elif isinstance(interaction_matrix, sp.csr_matrix):
    if parallel:
        logging.warning("Multiprocessing is only supported for CSC format. Fitting in single process.")
```

**Recommendation**: Extend multiprocessing support to CSR format.

```python
if parallel:
    if isinstance(interaction_matrix, sp.csr_matrix):
        # Convert CSR to CSC for multiprocessing
        interaction_matrix = interaction_matrix.tocsc()
    return self.fit_in_parallel(interaction_matrix, progress_bar=progress_bar)
```

#### 2.3 Batch Processing Optimization

**Recommendation**: Optimize batch processing for high user count scenarios to reduce memory consumption.

```python
# Example: Dynamically adjust batch size based on user count
def determine_batch_size(self, num_users: int) -> int:
    if num_users > 1_000_000:
        return 100
    elif num_users > 100_000:
        return 500
    else:
        return 1_000
```

### 3. Feature Enhancements

#### 3.1 Support for Additional Algorithms

**Recommendation**: Add support for these popular algorithms:

1. Neural Collaborative Filtering (NCF)
2. Matrix Factorization with Alternating Least Squares (ALS)
3. Neural Graph Collaborative Filtering (NGCF)

```python
# Example: ALS implementation skeleton
from rtrec.models.base import BaseModel
from implicit.als import AlternatingLeastSquares

class ImplicitALS(BaseModel):
    def __init__(self, **kwargs):
        super().__init__(**kwargs)
        self.model = AlternatingLeastSquares(**kwargs)
        # Initialization code
    
    def _record_interactions(self, user_id: int, item_id: int, tstamp: float, rating: float) -> None:
        # Implementation
        
    def _fit_recorded(self, parallel: bool=False, progress_bar: bool=True) -> None:
        # Implementation
```

#### 3.2 Enhanced Cold-Start Handling

**Recommendation**: Strengthen processing for new users and items.

```python
# Example: Add similar user search functionality
def find_similar_users(self, query_user_id: int, user_tags: Optional[List[str]] = None, top_k: int = 10) -> List[Tuple[int, float]]:
    """

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [myui/rtrec](https://github.com/myui/rtrec) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-03 -->
