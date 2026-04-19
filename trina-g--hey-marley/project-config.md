---
trigger: always_on
description: Create a simple web application prototype that connects to two Langflow pipelines for Iteration 1 of the onboarding workflow:
---

# Cursor Rules: Simple App Prototype with Langflow Integration

## Project Overview

Create a simple web application prototype that connects to two Langflow pipelines for Iteration 1 of the onboarding workflow:
- **Flow 1:** Scenario Generation (creates personalized writing prompts)
- **Flow 2:** Assessment + Plan (evaluates writing and generates learning focus areas)

## Architecture Requirements

### Tech Stack Recommendations

**Frontend:**
- Use a simple, modern framework: **React** (with Vite) or **Next.js** for quick setup
- UI Library: **Tailwind CSS** + **shadcn/ui** or **Material-UI** for rapid prototyping
- State Management: **React Context** or **Zustand** for simple state (no Redux needed)

**Backend:**
- **FastAPI** (Python) or **Express.js** (Node.js) - choose based on team preference
- REST API endpoints to handle onboarding workflow
- Load Langflow configuration from JSON file (with embedded API keys)
- Simple in-memory or file-based session storage for prototype (upgrade to PostgreSQL later)

**Langflow Configuration:**
- Configuration provided as JSON file with embedded API keys and flow endpoints
- Backend loads configuration at startup
- No direct Langflow API calls - configuration contains all necessary connection details

## Project Structure

```
app-prototype/
├── frontend/
│   ├── src/
│   │   ├── components/
│   │   │   ├── OnboardingForm.jsx      # Fixed Q&A form
│   │   │   ├── WritingExercise.jsx     # Exercise submission UI
│   │   │   └── ResultsDisplay.jsx      # Assessment results
│   │   ├── services/
│   │   │   └── api.js                  # API client for backend
│   │   ├── context/
│   │   │   └── SessionContext.jsx      # Session state management
│   │   └── App.jsx
│   ├── package.json
│   └── vite.config.js
├── backend/
│   ├── app/
│   │   ├── main.py                     # FastAPI app entry
│   │   ├── routes/
│   │   │   └── onboarding.py           # Onboarding endpoints
│   │   ├── services/
│   │   │   └── config_loader.py        # Load Langflow config from JSON
│   │   └── models/
│   │       └── session.py              # Session data models
│   ├── config/
│   │   └── langflow_config.json        # JSON config with API keys
│   ├── requirements.txt
│   └── .env                            # Environment variables (optional)
└── README.md
```

## Implementation Guidelines

### 1. Frontend UI Components

**OnboardingForm Component:**
- Simple form with fixed questions (lifestyle + mindset)
- No LLM calls - just collect user input
- Submit button triggers Flow 1 call
- Show loading state while waiting for scenario generation

**WritingExercise Component:**
- Display generated scenario and 3 exercise prompts from Flow 1
- Text areas for each exercise (5-10 min writing time)
- Submit button triggers Flow 2 call
- Show loading state during assessment

**ResultsDisplay Component:**
- Display skill assessment results
- Show 3 recommended learning focus areas
- Simple, clean presentation

### 2. Backend API Endpoints

**POST /api/onboarding/scenario**
- Input: `{ lifestyle: string, mindset: string }`
- Uses Langflow Flow 1 configuration (from JSON config)
- Returns: `{ scenario: string, exercises: [string, string, string] }`
- Store session state (in-memory for prototype)

**POST /api/onboarding/assess**
- Input: `{ sessionId: string, exercises: [string, string, string] }`
- Uses Langflow Flow 2 configuration (from JSON config)
- Returns: `{ assessment: object, focusAreas: [string, string, string] }`
- Update session state

**GET /api/session/:sessionId**
- Retrieve current session state
- Used for page refresh recovery

### 3. Langflow Configuration Loading

**Configuration File Structure:**
The backend will receive a JSON configuration file (`config/langflow_config.json`) with embedded API keys and flow details:

```json
{
  "flow_1": {
    "name": "scenario_generation",
    "endpoint": "https://langflow-instance.com/api/v1/run/{flow_id}",
    "flow_id": "scenario-generation-flow-id",
    "api_key": "embedded_api_key_here",
    "input_type": "json",
    "output_type": "json"
  },
  "flow_2": {
    "name": "assessment_plan",
    "endpoint": "https://langflow-instance.com/api/v1/run/{flow_id}",
    "flow_id": "assessment-plan-flow-id",
    "api_key": "embedded_api_key_here",
    "input_type": "json",
    "output_type": "json"
  },
  "langflow_base_url": "https://langflow-instance.com"
}
```

**Config Loader Service:**
```python
# Example structure
import json
from pathlib import Path
from typing import Dict

class LangflowConfigLoader:
    def __init__(self, config_path: str = "config/langflow_config.json"):
        self.config_path = Path(config_path)
        self.config: Dict = {}
        self.load_config()
    
    def load_config(self):
        """Load configuration from JSON file at startup"""
        with open(self.config_path, 'r') as f:
            self.config = json.load(f)
    
    def get_flow_config(self, flow_name: str) -> Dict:
        """Get configuration for a specific flow"""
        return self.config.get(f"flow_{flow_name}", {})
    
    def get_api_key(self, flow_name: str) -> str:
        """Get API key for a specific flow"""
        flow_config = self.get_flow_config(flow_name)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Trina-G) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
