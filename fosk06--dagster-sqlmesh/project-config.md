---
trigger: always_on
description: Architecture Patterns for dg-sqlmesh module
---


# Architecture Patterns for dg-sqlmesh

## Individual Asset Pattern

### Overview

Each SQLMesh model becomes a separate Dagster asset for granular control and better UI experience.

### Implementation

```python
# In [src/dg_sqlmesh/factory.py](mdc:src/dg_sqlmesh/factory.py)
def create_model_asset(current_model_name, current_asset_spec, current_model_checks):
    @asset(
        key=current_asset_spec.key,
        description=f"SQLMesh model: {current_model_name}",
        group_name=current_asset_spec.group_name,
        metadata=current_asset_spec.metadata,
        deps=current_asset_spec.deps,
        check_specs=current_model_checks,
        tags={
            **(current_asset_spec.tags or {}),
            "dagster/max_retries": "0",
            "dagster/retry_on_asset_or_op_failure": "false"
        },
    )
    def model_asset(context: AssetExecutionContext, sqlmesh: SQLMeshResource, sqlmesh_results: SQLMeshResultsResource):
        # Shared execution logic
        pass
```

### Benefits

- **Granular Control**: Each model can succeed/fail independently
- **Better UI**: Individual assets visible in Dagster UI
- **Selective Materialization**: Can materialize specific models
- **Proper Dependencies**: Dagster's native dependency resolution

## Shared SQLMesh Execution Pattern

### Overview

Single SQLMesh execution per Dagster run, shared between all selected assets via `SQLMeshResultsResource`.

### Implementation

```python
class SQLMeshResultsResource(ConfigurableResource):
    """Resource pour partager les résultats SQLMesh entre les assets d'un même run."""

    def store_results(self, run_id: str, results: Dict[str, Any]) -> None:
        """Stocke les résultats SQLMesh pour un run donné."""
        self._results[run_id] = results

    def get_results(self, run_id: str) -> Optional[Dict[str, Any]]:
        """Récupère les résultats SQLMesh pour un run donné."""
        return self._results.get(run_id)

    def has_results(self, run_id: str) -> bool:
        """Vérifie si des résultats existent pour un run donné."""
        return run_id in self._results
```

### Execution Logic

```python
# Check if SQLMesh already executed for this run
if not sqlmesh_results.has_results(context.run_id):
    # First asset in run - execute SQLMesh for all selected assets
    selected_asset_keys = context.selected_asset_keys
    models_to_materialize = get_models_to_materialize(selected_asset_keys, ...)
    plan = sqlmesh.materialize_assets_threaded(models_to_materialize)

    # Store results for other assets
    results = {
        "failed_check_results": sqlmesh._process_failed_models_events(),
        "skipped_models_events": sqlmesh._console.get_skipped_models_events(),
        "evaluation_events": sqlmesh._console.get_evaluation_events(),
    }
    sqlmesh_results.store_results(context.run_id, results)
else:
    # Use existing results from this run
    results = sqlmesh_results.get_results(context.run_id)
```

### Benefits

- **Respects SQLMesh Dependencies**: Natural upstream/downstream skipping
- **Single Execution**: One SQLMesh run per Dagster run
- **Consistent State**: All assets see same SQLMesh results
- **Better Performance**: Single SQLMesh context and execution

## Event-Driven Status Pattern

### Overview

Asset and check status determined from SQLMesh events captured by custom console.

### Implementation

```python
# In [src/dg_sqlmesh/sqlmesh_event_console.py](mdc:src/dg_sqlmesh/sqlmesh_event_console.py)
class SQLMeshEventCaptureConsole(IntrospectingConsole):
    def _handle_log_failed_models(self, event: LogFailedModels) -> None:
        """Capture failed model events for asset check creation."""
        self.failed_models_events.append(event)

    def _handle_log_skipped_models(self, event: LogSkippedModels) -> None:
        """Capture skipped model events for dependency logic."""
        self.skipped_models_events.append(event)

    def _handle_update_snapshot_evaluation(self, event: UpdateSnapshotEvaluationProgress) -> None:
        """Capture evaluation progress for audit results."""
        self.evaluation_events.append(event)
```

### Status Determination

```python
# Check if model was skipped due to upstream failure
if model_was_skipped:
    raise Exception(f"Model {model_name} was skipped due to upstream failures")

# Check if model materialized but audits failed
elif model_has_audit_failures:
    # Asset materializes successfully but with failed checks
    return MaterializeResult(
        asset_key=asset_key,
        check_results=[AssetCheckResult(passed=False, ...)]
    )
else:
    # Full success - materialization and audits passed
    return MaterializeResult(
        asset_key=asset_key,
        check_results=[AssetCheckResult(passed=True, ...)]
    )
```

## Tag Convention Pattern

### Overview

Use `dagster:property_name:value` convention to pass SQLMesh model properties to Dagster assets.

### Implementation

```python
# In [src/dg_sqlmesh/translator.py](mdc:src/dg_sqlmesh/translator.py)
def _get_dagster_property_from_tags(self, model, property_name: str) -> Optional[str]:
    """
    Parse SQLMesh tags to extract Dagster properties.
    Convention: "dagster:property_name:value"
    """
    tags = getattr(model, "tags", set())

    for tag in tags:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/fosk06) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
