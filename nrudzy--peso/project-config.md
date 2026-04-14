---
trigger: always_on
description: - Respecter strictement la structure monorepo : `backend/`, `frontend/`, `mobile-app/`, `infra/`
---

# Cursor Rules - Peso SaaS Monorepo
# Règles spécifiques par namespace technique pour maintenir la cohérence du projet

## 🏗️ ARCHITECTURE GÉNÉRALE

### Structure Monorepo
- Respecter strictement la structure monorepo : `backend/`, `frontend/`, `mobile-app/`, `infra/`
- Chaque module doit être autonome avec ses propres dépendances
- Utiliser des chemins relatifs pour les imports entre modules
- Maintenir la séparation claire des responsabilités

### Conventions de Nommage
- **Fichiers Python** : snake_case (ex: `user_service.py`, `weight_entry.py`)
- **Fichiers JavaScript/Vue** : camelCase ou PascalCase (ex: `userService.js`, `WeightEntry.vue`)
- **Dossiers** : kebab-case (ex: `mobile-app/`, `weight-entries/`)
- **Modales** : Toujours dans le dossier `components/Modal/` (ex: `EditWeightEntryModal.vue`)
- **Variables** : snake_case en Python, camelCase en JavaScript
- **Constantes** : UPPER_SNAKE_CASE
- **Classes** : PascalCase dans tous les langages

### Documentation
- Tous les commentaires en **anglais** (comme spécifié dans les règles utilisateur)
- Documenter toutes les fonctions publiques avec docstrings
- Maintenir les README.md à jour dans chaque dossier
- Utiliser des emojis dans la documentation pour la lisibilité

---

## 🐍 BACKEND (FastAPI/Python)

### Structure du Code
```python
# Imports dans cet ordre :
# 1. Standard library
import os
from typing import Optional

# 2. Third-party packages
from fastapi import APIRouter, Depends, HTTPException
from sqlalchemy.orm import Session

# 3. Local imports
from app.core.database import get_db
from app.models.user import User
from app.schemas.user import UserCreate, UserResponse
```

### Modèles SQLAlchemy
```python
# Toujours hériter de Base
from sqlalchemy import Column, Integer, String, DateTime, ForeignKey
from sqlalchemy.orm import relationship
from app.core.database import Base

class User(Base):
    __tablename__ = "users"
    
    id = Column(Integer, primary_key=True, index=True)
    email = Column(String, unique=True, index=True, nullable=False)
    # Toujours ajouter des index sur les champs de recherche fréquente
    # Toujours définir nullable=False pour les champs obligatoires
```

### Schémas Pydantic
```python
# Utiliser des schémas séparés pour Create, Update, Response
from pydantic import BaseModel, EmailStr
from datetime import datetime
from typing import Optional

class UserCreate(BaseModel):
    email: EmailStr
    password: str
    name: str

class UserUpdate(BaseModel):
    name: Optional[str] = None
    # Pour les updates, tous les champs doivent être Optional

class UserResponse(BaseModel):
    id: int
    email: str
    name: str
    created_at: datetime
    
    class Config:
        from_attributes = True  # Pour SQLAlchemy 2.0
```

### Services
```python
# Logique métier dans les services, pas dans les routes
class UserService:
    def __init__(self, db: Session):
        self.db = db
    
    def create_user(self, user_data: UserCreate) -> User:
        # Validation métier ici
        # Logique de création
        pass
    
    def get_user_by_email(self, email: str) -> Optional[User]:
        return self.db.query(User).filter(User.email == email).first()
```

### Routes API
```python
# Utiliser des préfixes cohérents
router = APIRouter(prefix="/api/v1/users", tags=["users"])

@router.post("/", response_model=UserResponse, status_code=201)
async def create_user(
    user_data: UserCreate,
    db: Session = Depends(get_db)
):
    # Validation et création via service
    pass

@router.get("/me", response_model=UserResponse)
async def get_current_user(
    current_user: User = Depends(get_current_user)
):
    # Utiliser les dépendances pour l'authentification
    pass
```

### Gestion d'Erreurs
```python
# Toujours utiliser des exceptions HTTP appropriées
from fastapi import HTTPException

def get_user_or_404(user_id: int, db: Session) -> User:
    user = db.query(User).filter(User.id == user_id).first()
    if not user:
        raise HTTPException(status_code=404, detail="User not found")
    return user
```

### Tests
```python
# Structure des tests
import pytest
from fastapi.testclient import TestClient
from app.main import app

class TestUserAPI:
    def test_create_user_success(self, client: TestClient):
        # Arrange
        user_data = {"email": "test@example.com", "password": "password123"}
        
        # Act
        response = client.post("/api/v1/users/", json=user_data)
        
        # Assert
        assert response.status_code == 201
        assert response.json()["email"] == user_data["email"]
```

### Migrations Alembic
```python
# Toujours créer des migrations pour les changements de schéma
# Utiliser des noms descriptifs
# alembic revision --autogenerate -m "add_user_email_verification"
```

---

## 🎨 FRONTEND (Vue.js/JavaScript)

### Structure des Composants Vue
```vue
<template>
  <!-- Template en premier -->
  <div class="user-profile">
    <h1>{{ user.name }}</h1>
    <p>{{ user.email }}</p>
  </div>
</template>

<script>
// Script en second, avec Composition API
import { ref, onMounted } from 'vue'
import { useUserStore } from '@/stores/user'

export default {
  name: 'UserProfile',
  setup() {
    const user = ref(null)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nRudzy) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
