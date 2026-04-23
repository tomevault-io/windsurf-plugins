---
trigger: always_on
description: AI-specific coding standards and patterns for ML implementations on the Keyera Data Platform
---


# AI Code Standards for Keyera Data Platform

## Core Principles

### Data Platform Integration
- All AI code must integrate seamlessly with the medallion architecture (Bronze-Silver-Gold)
- Use existing data quality frameworks and extend them for ML-specific validations
- Follow established solution architecture patterns with AI/ML-specific adaptations
- Leverage Unity Catalog for ML asset management and governance

### AI-Specific Code Organization
- Organize AI code following the established solution structure:
  ```
  solutions/ai_solution_name/
  ├── 01_bronze_data_extraction.py
  ├── 02_silver_feature_engineering.py
  ├── 03_gold_model_training.py
  ├── 04_model_inference.py
  ├── solution_code/
  │   ├── feature_engineering.py
  │   ├── model_training.py
  │   ├── model_evaluation.py
  │   ├── model_deployment.py
  │   └── ai_utils.py
  ├── solution_resources/
  │   ├── model_configs/
  │   ├── feature_configs/
  │   └── deployment_configs/
  └── solution_tests/
      ├── test_feature_engineering.py
      ├── test_model_training.py
      └── test_model_evaluation.py
  ```

## AI Code Standards

### Model Development Standards
- Use type hints for all AI/ML functions and classes
- Implement proper error handling for ML-specific exceptions
- Follow functional programming principles for feature engineering
- Use dataclasses for ML configuration objects
- Implement comprehensive logging for model training and inference

### Feature Engineering Standards
```python
from typing import Dict, List, Optional, Union
from dataclasses import dataclass
from pyspark.sql import DataFrame
from pyspark.sql.functions import col

@dataclass
class FeatureConfig:
    """Configuration for feature engineering pipeline."""
    feature_name: str
    source_columns: List[str]
    transformation_type: str
    parameters: Dict[str, Union[str, int, float]]
    validation_rules: Dict[str, float]

class BaseFeatureEngineer:
    """Base class for feature engineering implementations."""
    
    def __init__(self, config: FeatureConfig):
        self.config = config
        self.logger = self._setup_logger()
    
    def transform(self, df: DataFrame) -> DataFrame:
        """Transform input DataFrame with feature engineering."""
        # Validate input data quality
        self._validate_input_data(df)
        
        # Apply transformations
        transformed_df = self._apply_transformations(df)
        
        # Validate output features
        self._validate_output_features(transformed_df)
        
        return transformed_df
    
    def _validate_input_data(self, df: DataFrame) -> None:
        """Validate input data quality for feature engineering."""
        # Use existing data quality framework
        pass
    
    def _apply_transformations(self, df: DataFrame) -> DataFrame:
        """Apply feature transformations."""
        raise NotImplementedError("Subclasses must implement transformation logic")
    
    def _validate_output_features(self, df: DataFrame) -> None:
        """Validate output feature quality."""
        pass
```

### Model Training Standards
```python
from typing import Dict, Any, Optional, Tuple
import mlflow
from mlflow.models import ModelSignature

class BaseModelTrainer:
    """Base class for model training implementations."""
    
    def __init__(self, model_config: Dict[str, Any]):
        self.config = model_config
        self.logger = self._setup_logger()
        self.mlflow_client = mlflow.tracking.MlflowClient()
    
    def train(self, train_df: DataFrame, val_df: Optional[DataFrame] = None) -> Dict[str, Any]:
        """Train model with comprehensive tracking and validation."""
        with mlflow.start_run() as run:
            # Log model configuration
            mlflow.log_params(self.config)
            
            # Prepare data
            X_train, y_train = self._prepare_training_data(train_df)
            
            # Train model
            model = self._train_model(X_train, y_train)
            
            # Evaluate model
            metrics = self._evaluate_model(model, val_df)
            mlflow.log_metrics(metrics)
            
            # Log model artifacts
            self._log_model_artifacts(model, run.info.run_id)
            
            return {
                'model': model,
                'metrics': metrics,
                'run_id': run.info.run_id
            }
    
    def _prepare_training_data(self, df: DataFrame) -> Tuple[Any, Any]:
        """Prepare training data from DataFrame."""
        raise NotImplementedError("Subclasses must implement data preparation")
    
    def _train_model(self, X: Any, y: Any) -> Any:
        """Train the actual model."""
        raise NotImplementedError("Subclasses must implement model training")
    
    def _evaluate_model(self, model: Any, val_df: Optional[DataFrame]) -> Dict[str, float]:
        """Evaluate model performance."""
        raise NotImplementedError("Subclasses must implement model evaluation")
    
    def _log_model_artifacts(self, model: Any, run_id: str) -> None:
        """Log model and related artifacts to MLflow."""
        # Register model in Unity Catalog
        # Save model artifacts
        # Log feature importance if applicable
        pass

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/IvanWeissVanDerPol) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
