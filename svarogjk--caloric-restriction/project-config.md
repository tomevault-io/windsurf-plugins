---
trigger: always_on
description: > These instructions apply to GitHub Copilot, Claude Code, and other AI coding assistants.
---

# AI Coding Assistant Instructions

> These instructions apply to GitHub Copilot, Claude Code, and other AI coding assistants.

## Project Overview

This is a **GEO Survival Analysis** application - a full-stack bioinformatics web app that analyzes gene expression data from the NCBI Gene Expression Omnibus (GEO) database to identify genes associated with survival outcomes. The project uses AI/LLM for intelligent dataset ranking and analysis.

## Technology Stack

### Backend (Python 3.13+)
- **Framework:** FastAPI with Uvicorn
- **Data Processing:** Pandas, NumPy, SciPy
- **Survival Analysis:** lifelines (Kaplan-Meier, Cox regression)
- **LLM Integration:** Pydantic-AI (Mistral, Anthropic Claude)
- **Validation:** Pydantic 2.0+
- **HTTP Client:** httpx (async)
- **Package Manager:** uv

### Frontend (TypeScript)
- **Framework:** React 18 with TypeScript
- **State Management:** Redux Toolkit
- **Build Tool:** Vite
- **Styling:** Tailwind CSS
- **Visualization:** Recharts
- **HTTP Client:** Axios

## Code Style Guidelines

### Exception Handling
- **Never** create bare `try/except` blocks
- Always catch **specific exceptions** or omit try/except entirely
- Only use exception handling where at least one particular exception type can be defined

```python
# BAD - Never do this
try:
    result = process_data()
except:
    pass

# BAD - Too broad
try:
    result = process_data()
except Exception as e:
    logger.error(e)

# GOOD - Specific exceptions
try:
    result = process_data()
except (ValueError, KeyError) as e:
    logger.error(f"Data processing failed: {e}")
    raise

# GOOD - If no specific exception applies, don't use try/except
result = process_data()  # Let it fail naturally
```

### Environment & Commands
- Use `uv` to manage Python environments and run commands
- Activate environment: `uv sync` then use `uv run` prefix
- Example: `uv run python -m uvicorn app.main:app --reload`

### File Creation Policy
- **Never** create standalone explanation/documentation `.md` files
- Keep documentation in code comments or existing README files
- Prefer editing existing files over creating new ones

### Python Conventions
- Use `snake_case` for functions, variables, and modules
- Use `CamelCase` for class names
- Use type hints for all function signatures
- Use `async/await` for I/O operations
- Use dataclasses or Pydantic models for data structures
- Follow the service layer pattern (routes → services → clients)

```python
# Example service function signature
async def analyze_survival(
    dataset_id: str,
    gene_symbol: str,
    time_column: str,
    event_column: str,
) -> GeneSurvivalResult:
    """Perform survival analysis for a gene in a dataset."""
    ...
```

### TypeScript/React Conventions
- Use `camelCase` for variables and functions
- Use `PascalCase` for components and interfaces
- Use functional components with hooks
- Keep components focused and composable
- Use Redux for global state, local state for UI-only concerns

```typescript
// Example component structure
interface GeneCardProps {
  gene: GeneSurvivalResponse;
  isExpanded: boolean;
  onToggle: () => void;
}

const GeneCard: React.FC<GeneCardProps> = ({ gene, isExpanded, onToggle }) => {
  // Component implementation
};
```

## Project Structure

```
backend/
├── app/
│   ├── main.py              # FastAPI entry point
│   ├── api/routes.py        # API endpoints
│   ├── models/              # Pydantic models
│   │   ├── request_models.py
│   │   ├── response_models.py
│   │   └── llm_models.py
│   └── services/            # Business logic
│       ├── geo_client.py              # GEO API client
│       ├── geo_loader_service.py      # Dataset loading
│       ├── survival_analysis_service.py
│       └── geo_survival_workflow_orchestrator.py

frontend/
├── src/
│   ├── components/          # React components
│   ├── store/               # Redux state
│   │   ├── store.ts
│   │   └── searchSlice.ts
│   └── services/api.ts      # API client
```

## Key Services & Patterns

### Backend Services
- **GEOClient:** Handles NCBI GEO API requests with rate limiting
- **GEODataLoaderService:** AI-powered format detection for expression data
- **SurvivalAnalysisService:** Kaplan-Meier and Cox regression analysis
- **GEODatasetRankingService:** LLM-based dataset quality ranking

### Common Patterns
```python
# Service instantiation pattern
class SurvivalAnalysisService:
    def __init__(self, config: Optional[AnalysisConfig] = None):
        self.config = config or AnalysisConfig()
        self.logger = logging.getLogger(__name__)

    async def analyze(self, data: pd.DataFrame) -> AnalysisResult:
        ...

# Route handler pattern
@router.post("/search", response_model=AnalysisResponse)
async def search_datasets(request: AnalysisRequest) -> AnalysisResponse:
    orchestrator = GEOSurvivalWorkflowOrchestrator()
    return await orchestrator.execute(request)
```

## API Reference

### Endpoints
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/health` | Health check |
| GET | `/api/models` | List available LLM models |
| POST | `/api/search` | Search and analyze datasets |

### Request/Response Models
```python
# Request
class AnalysisRequest(BaseModel):
    query: str

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/svarogjk) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
