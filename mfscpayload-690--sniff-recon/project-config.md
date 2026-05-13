---
trigger: always_on
description: Sniff Recon is a **Streamlit-based network packet analyzer** with AI-powered analysis capabilities. It parses PCAP, CSV, and TXT files to extract packet data and provides natural language querying through multiple AI providers (Groq, OpenAI, Anthropic).
---

# Sniff Recon - AI Copilot Instructions

## Project Overview
Sniff Recon is a **Streamlit-based network packet analyzer** with AI-powered analysis capabilities. It parses PCAP, CSV, and TXT files to extract packet data and provides natural language querying through multiple AI providers (Groq, OpenAI, Anthropic).

**Key Architecture:** Streamlit GUI → Parser Layer → AI Module (Multi-Agent) → Scapy Packet Analysis

## Critical Architectural Patterns

### 1. Multi-Agent AI System (`multi_agent_ai.py`)
- **Chunking Strategy**: Large packet captures are split into 5MB/5000-packet chunks to avoid token limits
- **Load Balancing**: Round-robin across multiple AI providers (Groq, OpenAI, Anthropic)
- **Provider Abstraction**: All providers implement `AIProvider` abstract base class with `query()`, `test_connection()`, `name`, and `max_tokens` properties
- **Fallback Pattern**: If multi-agent fails, falls back to single Groq API via `ai_module.py`

```python
# Example: Multi-agent automatically chunks large files
chunks = multi_agent.chunk_packets(packets)  # Splits if >5MB or >5000 packets
responses = await multi_agent.query(prompt, packets)  # Processes chunks in parallel
```

### 2. Parser Layer Pattern (`parsers/`)
All parsers must return **pandas DataFrames** with standardized columns. Each parser is a simple function (no classes):

```python
# parsers/pcap_parser.py
def parse_pcap(file_path: str) -> pd.DataFrame:
    packets = rdpcap(file_path)  # Loads entire file into memory!
    # Returns: Timestamp, Source IP, Destination IP, Protocol, Source Port, Destination Port
```

**Critical Memory Pattern**: PCAP parser uses `scapy.rdpcap()` which loads entire file into memory. For large files (>200MB), the GUI enforces size limits at `sniff_recon_gui.py:179`.

**CSV Flexibility**: CSV parser handles inconsistent column names via key mapping (see `sniff_recon_gui.py` where file type detection occurs in file uploader section).

### 3. Streamlit State Management & Rerun Patterns
**Session State Keys** (`ai_query_interface.py:190-191`):
- `ai_responses`: List of all AI query results with timestamp (never cleared except on "Clear History")
- `user_query`: Pre-filled query text from suggested queries

**Critical Rerun Pattern** to avoid infinite loops:
```python
st.session_state.user_query = query  # Set state
st.rerun()  # Force UI refresh

# Later: ALWAYS clear before next rerun
st.session_state.user_query = ""
st.rerun()
```

**Never store packet data in session state** - packets are parsed fresh on each file upload and passed as function arguments.

### 4. CSS Injection Pattern
Every UI module calls `inject_modern_css()` or `inject_ai_interface_css()` to apply the cyberpunk-themed dark mode. **Always preserve CSS `unsafe_allow_html=True` patterns** when modifying UI. Pattern used throughout:

```python
st.markdown('<div class="protocol-card">...</div>', unsafe_allow_html=True)
```

## Development Workflows

### Running the Application
```powershell
# Primary method - checks dependencies, sets up environment
python start_gui.py

# Direct Streamlit (if deps already installed)
streamlit run sniff_recon_gui.py

# Custom port or Docker
streamlit run sniff_recon_gui.py --server.port 8502
docker-compose up -d  # Runs on port 8501, mounts ./output and .env
```

### Setting Up AI Providers (Optional)
1. Create `.env` in project root: `GROQ_API_KEY=...`, `OPENAI_API_KEY=...`, `ANTHROPIC_API_KEY=...`
2. Multi-agent system auto-detects and tests providers on init (`_test_providers()` in `multi_agent_ai.py:init`)
3. **Fallback**: App works without AI - provides local statistical analysis via `_provide_fallback_analysis()`

### Testing Packet Analysis
```python
# Parser testing (returns DataFrame)
from parsers.pcap_parser import parse_pcap
df = parse_pcap("test.pcap")

# AI analysis testing (returns Scapy packets)
import scapy.all as scapy
packets = scapy.rdpcap("test.pcap")  # For ai_module.query_ai()
```

## Project-Specific Conventions

### Error Handling Philosophy
- **AI Failures**: Always provide fallback analysis (see `_provide_fallback_analysis()` in `ai_module.py:409`)
- **API Errors**: Log to console, show user-friendly Streamlit warnings (not exceptions)
- **File Parsing**: Return empty DataFrame on failure, never crash GUI
- **No User Interruption**: App gracefully degrades when AI unavailable

### Data Flow for AI Queries
```
User Query → ai_query_interface.py (Streamlit UI)
  ↓
ai_module.py → filter_suspicious_packets() [Layer 1: reduce by ~90%]
  ↓
cluster_packets_by_ip() [Group by (src, dst) tuple]
  ↓
summarize_clusters() [Generate stats for LLM context]
  ↓
multi_agent_ai.py → chunk_packets() → query_single_chunk() [Parallel API calls]
  ↓
combine_responses() → Display in Streamlit with timestamp
```

**Why Layered Filtering?** Large PCAPs (100k+ packets) exceed LLM context windows. The 3-stage process (filter suspicious → cluster → summarize) reduces input by ~90% while preserving security insights.

### Scapy Packet Access Patterns (Critical!)
```python

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mfscpayload-690/Sniff-Recon](https://github.com/mfscpayload-690/Sniff-Recon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
