---
trigger: always_on
description: FastAPI endpoints, request/response patterns, authentication, API documentation, or backend service logic.
---

# Diet Whisperer - API Patterns & FastAPI Conventions

**Use this rule when working on:** FastAPI endpoints, request/response patterns, authentication, API documentation, or backend service logic.

## FastAPI Application Structure

### Main Application (`apps/api/app/main.py`)
```python
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware
from app.routers import auth, diet_plans, meals, ai_agent

app = FastAPI(
    title="Diet Whisperer API",
    description="AI-powered diet planning assistant",
    version="1.0.0"
)

# CORS configuration
app.add_middleware(
    CORSMiddleware,
    allow_origins=["http://localhost:3000", "https://ai-dietary.herokuapp.com"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

# Include routers
app.include_router(auth.router, prefix="/auth", tags=["authentication"])
app.include_router(diet_plans.router, prefix="/diet-plans", tags=["diet-plans"])
app.include_router(meals.router, prefix="/meals", tags=["meals"])
app.include_router(ai_agent.router, prefix="/ai", tags=["ai-agent"])
```

## Router Organization

### File Structure
```
apps/api/app/routers/
├── __init__.py
├── auth.py          # Authentication endpoints
├── diet_plans.py    # Diet plan CRUD operations
├── meals.py         # Meal management with nutrition
├── ai_agent.py      # AI agent interactions
└── calendar.py      # Calendar feed generation
```

### Router Pattern
```python
from fastapi import APIRouter, Depends, HTTPException, status
from sqlmodel import Session
from app.database import get_session
from app.models import User
from app.auth import get_current_user

router = APIRouter()

@router.get("/", response_model=List[DietPlanResponse])
async def get_diet_plans(
    current_user: User = Depends(get_current_user),
    session: Session = Depends(get_session),
    skip: int = 0,
    limit: int = 100
):
    """Get user's diet plans with pagination"""
    # Implementation here
```

## Auto-Generation Service Patterns

### AutoGenerationService Class Structure
```python
# services/auto_generation_service.py
class AutoGenerationService:
    """Service for handling automatic meal generation for diet plans"""
    
    def __init__(self):
        self.agent = None
    
    def _get_agent(self):
        """Get or create the auto-generation agent"""
        if self.agent is None:
            self.agent = create_diet_auto_generation_agent()
        return self.agent
    
    async def run_auto_generation_for_all_plans(self) -> Dict[str, any]:
        """Run auto-generation for all eligible diet plans (scheduled task)"""
        # Implementation for batch processing
    
    async def generate_meals_for_plan(self, plan_id: int, days_to_generate: int) -> Dict[str, any]:
        """Generate meals for a specific diet plan (manual trigger endpoint)"""
        try:
            # Get diet plan information
            diet_plan = get_diet_plan_with_meals(plan_id)
            if not diet_plan:
                return {
                    "success": False,
                    "message": "Diet plan not found",
                    "error": "Diet plan not found"
                }
            
            # Create a plan dict in the format expected by _process_single_diet_plan
            plan_dict = {
                "id": plan_id,
                "user_id": diet_plan["user_id"],
                "auto_generate_days": days_to_generate,
                "last_auto_generation": diet_plan.get("last_auto_generation"),
                "user_email": "manual_trigger"  # Not used for single plan processing
            }
            
            # Use the existing processing logic
            result = await self._process_single_diet_plan(plan_dict)
            
            if result["success"]:
                if result["generated"]:
                    update_last_auto_generation(plan_id)
                    return {
                        "success": True,
                        "message": f"Successfully generated {result['days_generated']} days of meals",
                        "days_generated": result["days_generated"],
                        "start_date": result["start_date"]
                    }
                else:
                    return {
                        "success": False,
                        "message": result["reason"],
                        "error": result["reason"]
                    }
            else:
                return {
                    "success": False,
                    "message": "Generation failed",
                    "error": result["error"]
                }
        
        except Exception as e:
            logger.error(f"Error in generate_meals_for_plan: {str(e)}")
            return {
                "success": False,
                "message": f"Generation failed: {str(e)}",
                "error": str(e)
            }
    
    async def _process_single_diet_plan(self, plan: Dict) -> Dict[str, any]:
        """Process auto-generation for a single diet plan"""
        # Implementation for processing individual plans
```

### Auto-Generation Router Pattern
```python
# routers/diet_plans/auto_generation.py
@router.post("/{diet_plan_id}/auto-generate")

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/federicoBetti) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
