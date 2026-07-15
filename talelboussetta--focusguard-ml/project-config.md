---
trigger: always_on
description: FocusGuard is a full-stack productivity platform combining Pomodoro focus sessions with AI-powered webcam analysis, gamification, and team features.
---

# FocusGuard AI Coding Agent Instructions

## Project Overview
FocusGuard is a full-stack productivity platform combining Pomodoro focus sessions with AI-powered webcam analysis, gamification, and team features.

### Tech Stack
- **Backend**: FastAPI (Python 3.11+) with async PostgreSQL (asyncpg)
- **Frontend**: React 18 + TypeScript + Vite + Tailwind CSS
- **Database**: PostgreSQL 15+ with sequential SQL migrations
- **ML Runtime**: MediaPipe (Face + Pose Landmarker) - 100% browser-based, no backend ML
- **Auth**: JWT (HS256) with refresh token rotation
- **State**: React Context (AuthContext, SessionContext, NotificationContext)

## Architecture Patterns

### Backend: Service-Route-Model-Schema Layering
FastAPI endpoints follow strict separation:
```
routes/ → services/ → models/ → database
```
- **Routes** (`api/routes/*.py`): HTTP handlers only - no business logic. Use `Depends(get_current_user_id)` for auth.
- **Services** (`api/services/*_service.py`): All business logic, XP calculations, transaction handling.
- **Models** (`api/models/*.py`): SQLAlchemy ORM with UUID primary keys.
- **Schemas** (`api/schemas/*.py`): Pydantic for request/response validation.

**Never put business logic in routes.** Move calculations to services.

### Database Patterns
- All IDs are UUIDs (`UUID(as_uuid=True)`)
- Migrations are sequential SQL files: `001_extensions.sql`, `002_users.sql`, etc. in `serv/database/init/`
- Foreign keys cascade delete automatically
- Use async session: `AsyncSession = Depends(get_db)`

### Frontend Patterns
- **State**: React Context for auth/sessions (`contexts/AuthContext.tsx`, `SessionContext.tsx`)
- **API**: Centralized in `services/api.ts` with typed responses
- **Auth**: JWT in localStorage, header: `Authorization: Bearer <token>`
- **Styling**: Tailwind classes only - no custom CSS files except `index.css`

## Critical Workflows

### Running the Stack
```bash
# Backend (in serv/)
python main.py  # FastAPI on :8000

# Frontend (in client/focusguard-dashboard/)
npm run dev  # Vite on :5173

# Database
docker-compose up -d  # PostgreSQL on :5432
```

### Adding a New Endpoint
1. Create Pydantic schema in `api/schemas/`
2. Add business logic in `api/services/`
3. Create route in `api/routes/` - import service, use `Depends(get_current_user_id)`
4. Register router in `main.py` (already includes all routers)

### Database Migration
1. Create `serv/database/init/0XX_description.sql`
2. Run: `docker-compose down && docker-compose up -d` (migrations auto-run on init)
3. Or manual: `python run_migration.py` (for single-column changes)

### Testing Endpoints
- Swagger UI: http://localhost:8000/docs (auto-generated, live testing)
- See `serv/TESTS.md` for curl examples with real tokens

## Code Conventions

### Backend
- **Async everywhere**: All DB operations use `async`/`await`
- **Error handling**: Raise custom exceptions from `api/utils/exceptions.py` (e.g., `UserNotFoundException`, `InvalidCredentialsException`)
- **Rate limiting**: Apply `@limiter.limit("N/timeunit")` decorator and add `Request` param
- **Documentation**: Docstrings on all service functions with Args/Returns/Raises

Example route pattern:
```python
@router.post("/", response_model=TeamResponse, status_code=201)
@limiter.limit("5/hour")
async def create_team(
    request: Request,
    team_data: TeamCreate,
    user_id: str = Depends(get_current_user_id),
    db: AsyncSession = Depends(get_db)
):
    team = await team_service.create_team(db, user_id, team_data)
    return TeamResponse.model_validate(team)
```

### Frontend
- **Types**: Match backend schema exactly (`User`, `Session`, `Garden`)
- **API calls**: Always use `services/api.ts` functions, handle errors with `getErrorMessage()`
- **Auth context**: Access user with `const { user } = useAuth()`
- **Routing**: React Router v6 - use `useNavigate()` for programmatic navigation

## Project-Specific Quirks

1. **XP System**: Session completion awards XP (25min = 10 XP). XP → Level logic in `session_service.py`
2. **Garden**: 1-to-1 with sessions via `session_id` foreign key. Create garden entry when session completes.
3. **Teams**: Users can only be in ONE team at a time. Enforced in `team_service.join_team()`.
4. **Blink Rate**: Stored on `sessions.blink_rate` (nullable FLOAT). Calculated by frontend ML models.
5. **Authentication**: Access token expires in 15 min. Refresh token in 7 days (from `api/config.py`).
6. **Leaderboards**: Calculated in `stats_service.py` - no caching yet, pulls from DB each request.

## Key Files

- `serv/main.py`: FastAPI app entry, CORS config, lifespan events
- `serv/api/database.py`: Async engine setup, `get_db()` dependency
- `serv/api/middleware/auth_middleware.py`: JWT validation, `get_current_user_id()`
- `client/focusguard-dashboard/src/services/api.ts`: All backend calls, error handling
- `docker-compose.yml`: PostgreSQL with auto-init from `serv/database/init/`

## Common Tasks

**Add new model field:**
1. Update SQLAlchemy model in `api/models/`
2. Update Pydantic schema in `api/schemas/`
3. Create migration SQL in `database/init/0XX_add_field.sql`
4. Update frontend type in `services/api.ts`

**Add rate limit:**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [talelboussetta/FocusGuard-ML](https://github.com/talelboussetta/FocusGuard-ML) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-15 -->
