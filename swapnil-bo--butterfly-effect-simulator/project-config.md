---
trigger: always_on
description: You are my senior engineering partner for a 15-day portfolio project called the
---

You are my senior engineering partner for a 15-day portfolio project called the
"Butterfly Effect Simulator." I am Swapnil, targeting AI/ML Product Management
internships. This project needs to demonstrate mastery over structured AI data
generation, interactive UI, and graph visualization.

---

## THE CONCEPT

A user inputs a small decision they made (e.g., "I bought a cheap guitar at a
thrift store"). The AI generates a logical but wild butterfly effect chain of
5-7 escalating life events over 10 years, rendered as an interactive visual
timeline graph with an AI-generated image on the final node.

---

## MY HARDWARE

- Windows Desktop: Intel i3-12100, RTX 3050 6GB VRAM, 8GB DDR4 RAM (runs full stack: Frontend + Backend + Ollama)

---

## FINAL TECH STACK

- Frontend: Next.js (App Router) + TypeScript + Tailwind CSS
- Graph: React Flow (built-in useNodesState / useEdgesState hooks — NO Zustand)
- Backend: Python + FastAPI
- Local AI: Ollama running Qwen 2.5 3B on Windows desktop
- Production AI: Groq API free tier (Llama 3) — used when deployed
- Image Generation: Together.ai or fal.ai free API (NOT local Stable Diffusion)
- Frontend Deploy: Vercel
- Backend Deploy: Render

---

## COMPLETE DIRECTORY STRUCTURE

butterfly-effect-simulator/
│
├── frontend/
│   ├── src/
│   │   ├── app/
│   │   │   ├── page.tsx
│   │   │   ├── layout.tsx
│   │   │   └── globals.css
│   │   ├── components/
│   │   │   ├── TimelineGraph.tsx
│   │   │   ├── NodeCard.tsx
│   │   │   └── LoadingState.tsx
│   │   └── lib/
│   │       ├── api.ts
│   │       └── types.ts
│   ├── package.json
│   └── tailwind.config.ts
│
├── backend/
│   ├── main.py
│   ├── models.py
│   ├── ai_service.py
│   ├── prompt_builder.py
│   ├── graph_math.py
│   ├── image_service.py
│   └── requirements.txt
│
└── README.md

---

## THE API CONTRACT (NON-NEGOTIABLE)

GET /health
Response: { "status": "online", "engine": "qwen2.5:3b" }

POST /generate
Request:  { "user_decision": "I bought a cheap acoustic guitar at a thrift store." }
Response:
{
  "status": "success",
  "data": {
    "nodes": [
      {
        "id": "node-1",
        "position": { "x": 250, "y": 0 },
        "data": { "year": "Year 1", "event": "You learn three chords.", "impact": "low" }
      },
      {
        "id": "node-2",
        "position": { "x": 250, "y": 200 },
        "data": { "year": "Year 3", "event": "A TikTok goes viral.", "impact": "medium" }
      }
    ],
    "edges": [
      { "id": "e1-2", "source": "node-1", "target": "node-2" }
    ]
  }
}

POST /generate-image
Request:  { "final_event": "Playing a sold-out stadium in 2034" }
Response: { "status": "success", "image_url": "data:image/png;base64,..." }

---

## KEY BACKEND MODULES

### backend/models.py — Full nested Pydantic models (NOT data: dict)
from pydantic import BaseModel

class GenerateRequest(BaseModel):
    user_decision: str

class NodePosition(BaseModel):
    x: float
    y: float

class NodeData(BaseModel):
    year: str
    event: str
    impact: str  # "low" | "medium" | "high" | "life-changing"

class TimelineNode(BaseModel):
    id: str
    position: NodePosition
    data: NodeData

class TimelineEdge(BaseModel):
    id: str
    source: str
    target: str

class GraphData(BaseModel):
    nodes: list[TimelineNode]
    edges: list[TimelineEdge]

class GenerateResponse(BaseModel):
    status: str
    data: GraphData

---

### backend/graph_math.py
def calculate_positions(nodes: list, spacing_y: int = 200) -> list:
    for i, node in enumerate(nodes):
        node["position"] = {"x": 250, "y": i * spacing_y}
    return nodes

def generate_edges(nodes: list) -> list:
    if len(nodes) < 2:
        raise ValueError("Need at least 2 nodes to generate edges.")
    return [
        {"id": f"e{i+1}-{i+2}", "source": nodes[i]["id"], "target": nodes[i+1]["id"]}
        for i in range(len(nodes) - 1)
    ]

---

### backend/ai_service.py — Unified OpenAI SDK for both Ollama and Groq
import re, json, os
from openai import OpenAI

def get_client():
    provider = os.getenv("LLM_PROVIDER", "ollama")
    if provider == "groq":
        return OpenAI(
            base_url="https://api.groq.com/openai/v1",
            api_key=os.getenv("GROQ_API_KEY")
        )
    return OpenAI(base_url="http://localhost:11434/v1", api_key="ollama")

def extract_valid_json(raw_response: str) -> dict:
    cleaned = re.sub(r'```(?:json)?|```', '', raw_response).strip()
    try:
        return json.loads(cleaned)
    except json.JSONDecodeError:
        raise ValueError("LLM failed to produce valid JSON after cleaning.")

---

### backend/prompt_builder.py — Soul of the app
SYSTEM_PROMPT = """
You are a highly logical butterfly effect simulator.
Given a mundane life decision, generate exactly 5 to 7 escalating timeline
events spanning 10 years.

CRITICAL: Return ONLY a raw valid JSON array.
No markdown. No backticks. No explanation. No preamble. Just the array.

Schema:
[
  {"id": "node-1", "year": "Year 1", "event": "...", "impact": "low"},
  {"id": "node-2", "year": "Year 3", "event": "...", "impact": "medium"},
  {"id": "node-3", "year": "Year 5", "event": "...", "impact": "high"},
  {"id": "node-4", "year": "Year 10", "event": "...", "impact": "life-changing"}
]


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Swapnil-bo/Butterfly-Effect-Simulator](https://github.com/Swapnil-bo/Butterfly-Effect-Simulator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-26 -->
