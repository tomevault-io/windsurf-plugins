---
trigger: always_on
description: Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.
---

# SD5913 - Programming for Art and Design

Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.

This is a Python-based educational repository for Programming for Artists and Designers class, organized by weekly exercises. Each week contains Python scripts, Jupyter notebooks, and projects focusing on web scraping, data visualization, AI/ML, computer vision, and creative applications.

## Working Effectively

### Environment Setup
- Repository uses Python 3.12.3 with system packages already available
- Key available system packages: `requests`, `numpy`, `matplotlib`, `lxml`, `python-dotenv`
- Packages that INSTALL SUCCESSFULLY: `pandas`, `matplotlib`, `numpy`, basic Python libraries
- Docker 28.0.4 and Docker Compose v2.38.2 are available
- Network access is LIMITED - pip installs frequently fail due to SSL/timeout issues
- External API calls (web scraping) will fail due to network restrictions

### Bootstrap and Test Repository
1. **ALWAYS start with basic environment check:**
   ```bash
   python3 --version  # Should show Python 3.12.3
   docker --version   # Should show Docker 28.0.4
   docker compose version  # Should show v2.38.2
   ```

2. **For week-specific work:**
   ```bash
   cd /home/runner/work/pfad/pfad/week##/
   pip3 install -r requirements.txt  # EXPECT FAILURES due to network limitations
   ```

3. **Test basic functionality:**
   ```bash
   # Test matplotlib (works)
   python3 -c "import matplotlib.pyplot as plt; import numpy as np; x=np.linspace(0,10,100); plt.plot(x, np.sin(x)); plt.savefig('/tmp/test.png'); print('Success')"
   
   # Test local modules
   python3 -c "import sys; sys.path.append('.'); import main; print('Module loaded')"
   ```

### Package Installation - CRITICAL LIMITATIONS
- **NETWORK LIMITATIONS**: pip installs FREQUENTLY FAIL due to SSL certificate issues and timeouts
- **NEVER CANCEL**: pip install attempts take 2-5 minutes and often fail. WAIT for completion.
- **Available packages**: Use system packages when possible: `requests`, `numpy`, `matplotlib`, `lxml`, `python-dotenv`
- **Successfully install**: `pandas`, `matplotlib`, `numpy` - these typically work within ~30 seconds
- **Known failures**: `streamlit`, `coqui-tts`, `langgraph`, `langchain`, `weaviate`, AI/ML packages requiring internet
- **Workaround**: Focus on testing code logic with available packages; document expected requirements

### Docker Workflows
- **Docker Compose configuration validation:**
  ```bash
  cd /home/runner/work/pfad/pfad
  docker compose config  # Takes 1-2 seconds, validates YAML
  ```

- **Docker builds WILL FAIL** due to network restrictions during pip install steps:
  ```bash
  cd /home/runner/work/pfad/pfad/week08
  docker build -t test .  # EXPECTED TO FAIL after 20-30 seconds
  ```

- **NEVER CANCEL**: Docker builds take 20-30 seconds before failing. Document the failure pattern.

## Weekly Structure and Validation

### Week 01: Web Scraping & Data Collection
- **Files**: `main.py`, `.env`, `requirements.txt`
- **Dependencies**: `python-dotenv`, `requests`, `lxml` (available)
- **Test command**: `cd week01 && python3 main.py` (WILL FAIL due to network restrictions)
- **Expected behavior**: Network error when trying to fetch HKO tide data
- **Validation**: Code syntax and imports work; network calls fail as expected

### Week 02: Data Visualization & Utilities  
- **Files**: `plot_tides.py`, `scraping_utils.py`, `draw_svg.py`, `week02_notebook.ipynb`, `python_foundations/` subfolder
- **Dependencies**: `python-dotenv`, `requests`, `lxml` (same as week01)
- **Test commands**:
  ```bash
  cd week02
  python3 -c "import scraping_utils; print('Utils work')"
  python3 -c "import matplotlib.pyplot as plt; print('Matplotlib works')"
  ```
- **Validation**: Modules import successfully, plotting works (save to `/tmp/`)
- **Subfolder**: `python_foundations/` contains Jupyter notebooks covering flow control, functions, matplotlib installation, and variable types

### Week 03: Fractals & Mathematical Visualization
- **Files**: `run_examples.py`, fractal scripts (`koch_curve.py`, `mandelbrot.py`, `sierpinksi_matplot_animation.py`), `week03_notebook.ipynb`
- **Dependencies**: `numpy`, `matplotlib`, `pandas` (ALL WORK - install successfully)
- **Topics**: Koch snowflake, Mandelbrot set, recursive algorithms, complex number mathematics
- **Test commands**:
  ```bash
  cd week03
  pip3 install -r requirements.txt  # Installs successfully in ~30 seconds
  python3 run_examples.py  # Generates plots in week03/plots/
  ls plots/  # Should show line_plot.png, multi_series.png, rolling_mean.png, monthly_avg.png
  ```
- **Validation**: Full week03 functionality WORKS - script execution and plot generation succeed
- **Output**: Plots saved to `week03/plots/` directory

### Week 04: Interactive User Interfaces with Streamlit
- **Files**: `1_user_input.py`, `2_user_input_with_history.py`, `3_chat_with_response.py`, `ollama_chatbot.py`, `lmstudio_chatbot.py`, `display_graph.py`, `display_image.py`
- **Dependencies**: `streamlit`, `ollama`, `openai` (INSTALL FAILS)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [venetanji/pfad](https://github.com/venetanji/pfad) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-04 -->
