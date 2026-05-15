---
trigger: always_on
description: カフェロット (Cafferot) は、ユーザーが自作のカフェロットを展示し、カフェ経営とコミュニティを融合させた2Dゲームです。
---

# Cafferot - 全体実装計画

## プロジェクト概要
カフェロット (Cafferot) は、ユーザーが自作のカフェロットを展示し、カフェ経営とコミュニティを融合させた2Dゲームです。

## 技術スタック
- **フロントエンド**: React 18 + TypeScript + Tailwind CSS
- **ビルドツール**: Vite
- **レイアウト**: Flexbox + Absolute Positioning (Grid不使用)
- **アニメーション**: Framer Motion
- **リアルタイム通信**: WebSocket → Nostr (段階的移行)
  - Phase 1: WebSocket (ws)
  - Phase 2: 抽象化レイヤー導入
  - Phase 3: Nostr Protocol
- **状態管理**: React Context API / Zustand (必要に応じて)
- **ファイル処理**: FileReader API (Base64変換)
- **ストレージ**: LocalStorage → IndexedDB (大容量対応時)
- **バックエンド**:
  - 初期: Node.js + Express + WebSocket
  - 将来: Nostrリレー（分散型、サーバーレス）

---

## フェーズ1: 基盤構築 (Week 1-2)

### 1.1 プロジェクト構造の整備（name-name準拠）

**プロジェクト全体構成:**
```
cafferot/                      # リポジトリルート
├── backend/                   # FastAPI バックエンド
│   ├── app/
│   │   ├── __init__.py
│   │   ├── main.py           # FastAPI エントリーポイント
│   │   ├── models.py         # Pydantic モデル
│   │   └── websocket.py      # WebSocket管理
│   ├── Dockerfile
│   ├── pyproject.toml        # uv/pip 依存関係
│   └── uv.lock
├── frontend/                  # React + Vite フロントエンド
│   ├── src/
│   │   ├── components/
│   │   ├── services/
│   │   ├── App.tsx
│   │   └── main.tsx
│   ├── Dockerfile.dev
│   ├── package.json
│   ├── vite.config.ts
│   └── tailwind.config.js
├── compose.yaml              # Docker Compose設定
├── .gitignore
├── README.md
└── CLAUDE.md
```

**`compose.yaml`（ルート）:**
```yaml
services:
  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile
    ports:
      - "8080:8080"
    volumes:
      - ./backend/app:/app/app
    environment:
      - PYTHONUNBUFFERED=1
    command: uvicorn app.main:app --host 0.0.0.0 --port 8080 --reload
    networks:
      - cafferot-network

  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile.dev
    ports:
      - "3000:3000"
    volumes:
      - ./frontend/src:/app/src
      - ./frontend/public:/app/public
      - ./frontend/index.html:/app/index.html
      - ./frontend/vite.config.ts:/app/vite.config.ts
    environment:
      - VITE_API_URL=http://localhost:8080
    command: npm run dev -- --host 0.0.0.0
    networks:
      - cafferot-network
    depends_on:
      - backend

networks:
  cafferot-network:
    driver: bridge
```

---

### 1.1.1 Frontend構成（name-name準拠）
**`frontend/`**
```
frontend/
├── src/
│   ├── components/          # Reactコンポーネント
│   │   ├── cafe/           # カフェ関連
│   │   ├── cafferot/       # カフェロット関連
│   │   ├── community/      # コミュニティ関連
│   │   └── ui/             # 共通UIコンポーネント
│   ├── hooks/              # カスタムフック
│   ├── services/           # ビジネスロジック
│   │   ├── websocketService.ts   # WebSocket実装
│   │   └── storage.ts            # LocalStorage管理
│   ├── types/              # TypeScript型定義
│   ├── App.tsx
│   ├── main.tsx
│   └── index.css
├── public/
├── index.html
├── Dockerfile.dev
├── package.json
├── vite.config.ts
├── tsconfig.json
├── tailwind.config.js
├── .prettierrc
└── eslint.config.js
```

**`frontend/package.json`:**
```json
{
  "name": "cafferot-frontend",
  "private": true,
  "version": "0.0.0",
  "type": "module",
  "scripts": {
    "dev": "vite",
    "build": "tsc && vite build",
    "preview": "vite preview",
    "lint": "eslint .",
    "lint:fix": "eslint . --fix",
    "format": "prettier --write \"src/**/*.{ts,tsx,css}\"",
    "format:check": "prettier --check \"src/**/*.{ts,tsx,css}\""
  },
  "dependencies": {
    "react": "^18.3.1",
    "react-dom": "^18.3.1",
    "react-router-dom": "^6.28.0",
    "framer-motion": "^11.0.0"
  },
  "devDependencies": {
    "@eslint/js": "^9.39.1",
    "@tailwindcss/postcss": "^4.1.17",
    "@types/react": "^18.3.18",
    "@types/react-dom": "^18.3.5",
    "@typescript-eslint/eslint-plugin": "^8.20.0",
    "@typescript-eslint/parser": "^8.20.0",
    "@vitejs/plugin-react": "^4.3.4",
    "autoprefixer": "^10.4.21",
    "eslint": "^9.18.0",
    "eslint-config-prettier": "^10.1.8",
    "globals": "^16.5.0",
    "prettier": "^3.4.2",
    "prettier-plugin-tailwindcss": "^0.7.1",
    "tailwindcss": "^4.1.17",
    "typescript": "^5.7.3",
    "vite": "^6.0.11"
  }
}
```

---

### 1.1.2 Backend構成（name-name準拠・FastAPI）
**`backend/`**
```
backend/
├── app/
│   ├── __init__.py
│   ├── main.py             # FastAPIエントリーポイント
│   ├── models.py           # Pydanticモデル（型定義）
│   └── websocket.py        # WebSocket管理
├── Dockerfile
├── pyproject.toml          # uv/pip依存関係
├── uv.lock
└── .gitignore
```

**`backend/pyproject.toml`:**
```toml
[project]
name = "cafferot-backend"
version = "0.1.0"
description = "カフェロット バックエンドAPI"
requires-python = ">=3.11"
dependencies = [
    "fastapi>=0.115.0",
    "uvicorn[standard]>=0.32.0",
    "pydantic>=2.9.2",
    "python-multipart>=0.0.12",
    "websockets>=14.0",
]

[project.optional-dependencies]
dev = [
    "pytest>=8.0.0",
    "httpx>=0.27.0",
]

[build-system]
requires = ["hatchling"]
build-backend = "hatchling.build"

[tool.hatch.build.targets.wheel]
packages = ["app"]

[tool.uv]
dev-dependencies = [
    "pytest>=8.0.0",
    "httpx>=0.27.0",
]
```

**`backend/app/main.py`:**
```python
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware
import logging

from .websocket import manager

# ログ設定
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kako-jun/cafferot](https://github.com/kako-jun/cafferot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
