---
trigger: always_on
description: Reglas de base de datos, migraciones y caché para la terminal de trading
---


# 🗄️ BASE DE DATOS Y PERSISTENCIA — TRADING TERMINAL

## PRINCIPIOS DE ACCESO A DATOS

- **Nunca** SQL raw en services o endpoints → usar SQLAlchemy ORM
- **Siempre** async para no bloquear el event loop
- **Decimal** para todos los valores monetarios (nunca float)
- **UUID** como primary key (no autoincrement entero)
- **Timestamps** en UTC siempre

---

## 🔌 CONEXIÓN Y SESIÓN

```python
# core/database.py

from sqlalchemy.ext.asyncio import AsyncSession, create_async_engine, async_sessionmaker
from sqlalchemy.orm import DeclarativeBase
from core.config import settings

# Motor async — una sola instancia global
engine = create_async_engine(
    settings.DATABASE_URL,
    echo=settings.DEBUG,          # Solo SQL logs en modo debug
    pool_size=10,                 # Conexiones en el pool
    max_overflow=20,              # Conexiones extra bajo demanda
    pool_pre_ping=True,           # Verificar conexiones antes de usar
)

AsyncSessionLocal = async_sessionmaker(
    engine,
    class_=AsyncSession,
    expire_on_commit=False        # Importante para async
)

class Base(DeclarativeBase):
    """Base para todos los modelos SQLAlchemy."""
    pass

# Dependency para FastAPI
async def get_db() -> AsyncSession:
    async with AsyncSessionLocal() as session:
        try:
            yield session
            await session.commit()
        except Exception:
            await session.rollback()
            raise
        finally:
            await session.close()
```

---

## 📋 PATRÓN REPOSITORY

```python
# repositories/base_repo.py — Repository base genérico

from typing import Generic, TypeVar, Type, Optional, List
from uuid import UUID
from sqlalchemy.ext.asyncio import AsyncSession
from sqlalchemy import select, update, delete
from core.database import Base

ModelType = TypeVar("ModelType", bound=Base)

class BaseRepository(Generic[ModelType]):
    """Repository base con operaciones CRUD comunes."""
    
    def __init__(self, model: Type[ModelType]):
        self.model = model
    
    async def get_by_id(self, db: AsyncSession, id: UUID) -> Optional[ModelType]:
        result = await db.execute(select(self.model).where(self.model.id == id))
        return result.scalar_one_or_none()
    
    async def get_all(
        self, 
        db: AsyncSession, 
        skip: int = 0, 
        limit: int = 100
    ) -> List[ModelType]:
        result = await db.execute(select(self.model).offset(skip).limit(limit))
        return list(result.scalars().all())
    
    async def create(self, db: AsyncSession, obj_data: dict) -> ModelType:
        db_obj = self.model(**obj_data)
        db.add(db_obj)
        await db.flush()   # flush, no commit (lo hace el middleware)
        await db.refresh(db_obj)
        return db_obj
    
    async def update(
        self, 
        db: AsyncSession, 
        id: UUID, 
        update_data: dict
    ) -> Optional[ModelType]:
        await db.execute(
            update(self.model)
            .where(self.model.id == id)
            .values(**update_data)
        )
        return await self.get_by_id(db, id)
    
    async def delete(self, db: AsyncSession, id: UUID) -> bool:
        result = await db.execute(
            delete(self.model).where(self.model.id == id)
        )
        return result.rowcount > 0


# repositories/order_repo.py — Específico para órdenes

from sqlalchemy import select, and_, desc
from typing import Optional, List
from uuid import UUID
from datetime import datetime, date

from repositories.base_repo import BaseRepository
from models.order import Order, OrderStatus

class OrderRepository(BaseRepository[Order]):
    
    def __init__(self):
        super().__init__(Order)
    
    async def get_open_orders(
        self, 
        db: AsyncSession, 
        user_id: UUID
    ) -> List[Order]:
        """Obtener todas las órdenes abiertas de un usuario."""
        result = await db.execute(
            select(Order)
            .where(and_(
                Order.user_id == user_id,
                Order.status.in_([OrderStatus.PENDING, OrderStatus.OPEN])
            ))
            .order_by(desc(Order.created_at))
        )
        return list(result.scalars().all())
    
    async def get_by_symbol(
        self,
        db: AsyncSession,
        user_id: UUID,
        symbol: str,
        limit: int = 50
    ) -> List[Order]:
        """Historial de órdenes por símbolo."""
        result = await db.execute(
            select(Order)
            .where(and_(
                Order.user_id == user_id,
                Order.symbol == symbol
            ))
            .order_by(desc(Order.created_at))
            .limit(limit)
        )
        return list(result.scalars().all())
    
    async def get_daily_volume_usd(
        self, 
        db: AsyncSession, 
        user_id: UUID, 
        date: date
    ) -> float:
        """
        Calcular volumen total operado en un día.
        Usado por RiskService para límites diarios.
        """
        from sqlalchemy import func, cast, Date
        
        result = await db.execute(
            select(func.sum(Order.quantity * Order.fill_price))
            .where(and_(
                Order.user_id == user_id,
                Order.status == OrderStatus.FILLED,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [juandoroteoflesiauni-lang/Market-options-stocks-Scanner](https://github.com/juandoroteoflesiauni-lang/Market-options-stocks-Scanner) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-09 -->
