---
trigger: always_on
description: 本文件旨在建立一套涵蓋軟體開發全流程與 AGI 輔助技術的整體指引，幫助團隊與個人達成快速原型 (POC) 開發、知識積累與持續優化。文件整合了以下各部分內容：
---

# .cursorrules 文件：軟體開發與 AGI 融合指引 
 
本文件旨在建立一套涵蓋軟體開發全流程與 AGI 輔助技術的整體指引，幫助團隊與個人達成快速原型 (POC) 開發、知識積累與持續優化。文件整合了以下各部分內容： 
 
- 與使用者互動及內部記錄的基本指示   
- 開發過程中各角色的分工與職責   
- 軟體開發流程與運算思維方法   
- 快速 POC 開發的 SOP 指引   
- 持續學習與自我優化的機制   
- 套件與版本紀錄機制（含版本相依性守則）   
- 依賴查詢：透過官方文件 URL 搜尋套件相依性資訊 
 
--- 

## 一、基本指示 (Instructions) 
 
- **可重用資訊記錄**   
  在與使用者互動過程中，若發現專案中可重用的資訊（例如函式庫版本、模型名稱、錯誤修正或收到的糾正），請立即記錄於本文件的 **Lessons** 區塊，避免日後重複相同錯誤。 
 
- **Scratchpad 作為思考與記錄工具**   
  - 使用本文件作為 Scratchpad（工作筆記區），組織與記錄所有新任務的思考、規劃與進度。   
  - 開發流程規劃 - 任務內容 
  - 接到新任務時，首先回顧 Scratchpad 內容，若有與當前任務無關的舊任務，請先清除。   
  - 說明任務內容、規劃完成任務所需步驟，可使用 todo markers 表示進度，如：   
    - [X] 任務 1   
    - [ ] 任務 2   
  - 完成子任務時更新進度，並於每個里程碑後反思與記錄，確保全局規劃與細節追蹤兼備。 
 
--- 
 
## 二、Cursor Learned 
 
- 處理搜尋結果時，確保正確處理不同國際查詢的字符編碼（UTF-8）。 
- 在 stderr 中輸出除錯資訊，同時保持 stdout 輸出整潔，便於整合管道操作。 
- 使用 matplotlib 畫圖時，若需採用 seaborn 風格，請使用 seaborn-v0_8 而非傳統 `seaborn`（因近期版本變更）。 
- 使用 OpenAI 的 GPT-4 時，請以 gpt-4o 作為模型名稱，尤其在具備視覺功能時。 
 
--- 
 
## 三、資料夾結構規劃 
 
> 資料夾樹狀結構

```text
crewai-agentic-course/               ← Git 倉庫根
│
├─ README.md                         ← 一頁式快速啟動
├─ requirements.txt                  ← Python 依賴
├─ pyproject.toml                    ← Poetry (可選)
├─ .env.example                      ← API 金鑰範例
├─ .gitignore
│
├─ docs/                             ← 教材／說明文件 (Why‧What)
│  ├─ syllabus.md
│  ├─ patterns/
│  │   ├─ reflection.md
│  │   ├─ planning.md
│  │   ├─ tool_use.md
│  │   └─ multi_agent.md
│  ├─ guides/
│  │   ├─ setup_guide.md
│  │   ├─ faq.md
│  │   └─ style_guide.md
│  ├─ rubrics.md                     ← 評量指標 (5★/3★/1★)
│  └─ refs.bib                       ← BibTeX 資料庫
│
├─ src/                              ← 核心程式碼 (How)
│  ├─ __init__.py
│  │
│  ├─ core/                          ← CrewAI 基礎薄封裝
│  │   ├─ agents/
│  │   │   ├─ __init__.py
│  │   │   ├─ agent_base.py
│  │   │   ├─ researcher.py
│  │   │   ├─ writer.py
│  │   │   └─ planner.py
│  │   ├─ tasks/
│  │   │   ├─ __init__.py
│  │   │   ├─ task_base.py
│  │   │   └─ evaluation_task.py
│  │   ├─ tools/
│  │   │   ├─ __init__.py
│  │   │   ├─ code_interpreter_tool.py
│  │   │   ├─ forex_api_tool.py
│  │   │   └─ website_search_tool.py
│  │   ├─ flows/
│  │   │   ├─ __init__.py
│  │   │   └─ flow_base.py
│  │   ├─ crews/
│  │   │   ├─ __init__.py
│  │   │   └─ crew_factory.py
│  │   ├─ memory/
│  │   │   ├─ __init__.py
│  │   │   ├─ memory_manager.py
│  │   │   ├─ chromadb_client.py
│  │   │   ├─ sqlite_client.py
│  │   │   └─ entity_memory.py
│  │   └─ knowledge/
│  │       ├─ __init__.py
│  │       ├─ base_source.py
│  │       ├─ pdf_source.py
│  │       ├─ csv_source.py
│  │       └─ web_source.py
│  │
│  ├─ patterns/                      ← 可插拔四大 Agentic 模式
│  │   ├─ reflection/
│  │   │   ├─ __init__.py
│  │   │   ├─ self_critique.py
│  │   │   └─ templates/
│  │   │       └─ critique_prompt.txt
│  │   ├─ planning/
│  │   │   ├─ __init__.py
│  │   │   └─ wbs_planner.py
│  │   ├─ tool_use/
│  │   │   ├─ __init__.py
│  │   │   └─ robust_tool_wrapper.py
│  │   └─ multi_agent/
│  │       ├─ __init__.py
│  │       └─ delegation_manager.py
│  │
│  ├─ pipelines/                     ← 高階工作流範例
│  │   ├─ __init__.py
│  │   ├─ self_refine/
│  │   │   ├─ __init__.py
│  │   │   ├─ self_refine_pipeline.py
│  │   │   └─ README.md
│  │   ├─ rag_reflect/
│  │   │   ├─ __init__.py
│  │   │   ├─ rag_loop.py
│  │   │   └─ README.md
│  │   └─ aqi_alert_flow/
│  │       ├─ __init__.py
│  │       ├─ aqi_flow.py
│  │       └─ README.md
│  │
│  ├─ data/                          ← 內建示範資料集
│  │   ├─ forex_sample.csv
│  │   └─ aqi_sample.json
│  │
│  └─ templates/                     ← 程式／報告範本
│      ├─ agent_template.py
│      ├─ task_template.py
│      ├─ flow_template.py
│      ├─ dockerfile_template      (Dockerfile 範本)
│      └─ lab_report_template.md
│
├─ work/                             ← 學生實作 (Do)
│  ├─ labs/
│  │   ├─ week01_reflection/
│  │   │   ├─ README.md
│  │   │   ├─ solution.py
│  │   │   └─ sample_output.json
│  │   ├─ week02_reflection/
│  │   ├─ week03_planning/
│  │   ├─ …                         （week04~12 依序）
│  │   └─ week12_training/
│  │
│  └─ projects/
│      └─ capstone_teamX/
│          ├─ docs/
│          │   ├─ design_doc.md
│          │   └─ TODO_future_work.md
│          ├─ src/
│          │   ├─ app.py
│          │   └─ requirements.txt
│          └─ evaluation/
│              ├─ rubric.xlsx
│              └─ demo_video.mp4
│
├─ infra/                            ← 執行／部署 (Run)
│  ├─ docker-compose.yml
│  ├─ Dockerfile
│  ├─ k8s/
│  │   ├─ crewai-deployment.yaml
│  │   ├─ chroma-deployment.yaml
│  │   └─ grafana-deployment.yaml
│  ├─ ci/
│  │   └─ github-actions.yml
│  └─ observability/
│      ├─ grafana/
│      │   └─ crewai_dashboard.json
│      └─ prometheus/
│          └─ prometheus.yml
│
├─ tests/                            ← pytest 單元／整合測試
│  ├─ conftest.py
│  ├─ test_agents.py
│  ├─ test_tools.py
│  ├─ test_memory.py
│  └─ test_pipelines.py
│
└─ notebooks/                        ← 可選：教學 Jupyter
    ├─ 01_reflection_demo.ipynb
    └─ 07_tool_use_async.ipynb

```

### 資料夾規劃說明


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Zenobia000/iSpan_LLM-Agent-Crewai-cookbooks](https://github.com/Zenobia000/iSpan_LLM-Agent-Crewai-cookbooks) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-01 -->
