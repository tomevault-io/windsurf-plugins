---
trigger: always_on
description: Tabbed layout: Header (Logo + MainTabBar + GO + Badge) | Main (Workspace/Mission/System/Tests/Team tabs) | BottomBar (contextual tabs, resizable). Overlay modals for gates, questions, skills, and completion.
---

# Frontend Components

Tabbed layout: Header (Logo + MainTabBar + GO + Badge) | Main (Workspace/Mission/System/Tests/Team tabs) | BottomBar (contextual tabs, resizable). Overlay modals for gates, questions, skills, and completion.

## Component Tree

```
App.tsx
  shared/MainTabBar.tsx              Workspace/Mission Control/System/Tests/Team tab switcher in header
  shared/ErrorBoundary.tsx           Class component error boundary (wraps App in main.tsx)
  shared/GoButton.tsx                Build trigger with ready/building/stop/disabled states
  shared/ReadinessBadge.tsx          Backend readiness indicator
  BlockCanvas/WorkspaceSidebar.tsx   Vertical icon toolbar (Folder/Open/Save/Skills/Portals/Examples/Help)
  BlockCanvas/BlockCanvas.tsx        Blockly editor wrapper. Read-only during build. Always mounted.
  Planning/PlanningModal.tsx          Full-screen Planning Mode overlay (conversation + plan state)
    Planning/PlanningPanel.tsx        Two-panel layout: ChatPanel left, PlanStatePanel right
    Planning/PlanStatePanel.tsx       Live-updating plan summary (6 primitives with decided/open status)
    Planning/PlanSummaryCard.tsx      Readiness review card ("Generate Canvas" / "Ask me more")
    Planning/TeachingAnnotation.tsx   Collapsible teaching annotations (why_asking, patterns, what_if)
    Planning/widgets/SingleSelectWidget.tsx   Clickable buttons for single-choice planning questions
    Planning/widgets/MultiSelectWidget.tsx    Checkboxes for multi-select planning questions
    Planning/widgets/RankPrioritiesWidget.tsx Drag-to-reorder priority ranking
    Planning/widgets/QuestionWidgetRenderer.tsx Dispatcher: renders correct widget by question.type
  AgentTeam/AgentTeamPanel.tsx       Full-width agent cards + comms feed (Agents tab)
  TaskMap/TaskMapPanel.tsx           Full-width interactive task DAG (Tasks tab)
  SystemPanel/SystemPanel.tsx        System main tab: architecture explorer (spec view pre-build, task list + detail during/post-build)
  MissionControl/MissionControlPanel.tsx  Main mission control layout with narrator feed + minion squad + fix banner above DAG when isFixing
  MissionControl/MinionSquadPanel.tsx     Minion cards with status badges and task assignments
  MissionControl/NarratorFeed.tsx         Scrolling narrator message feed with mood indicators
  MissionControl/PlanningIndicator.tsx    Planning phase status indicator
  MissionControl/TaskDAG.tsx         @xyflow/react graph of task dependencies (pulsing amber badge on meeting-blocked and fix tasks)
  MissionControl/CommsFeed.tsx       Scrolling agent message log
  MissionControl/MetricsPanel.tsx    Token usage bars per agent, cost display, budget percentage
  MissionControl/FeedbackLoopIndicator.tsx  Correction cycle animation + attempt counter for retrying tasks
  MissionControl/ConvergencePanel.tsx       Convergence tracking: attempt history, trends, teaching moments
  MissionControl/ContextFlowAnimation.tsx   Animated context flow dots between DAG nodes on task completion
  BottomBar/BottomBar.tsx            Resizable tabbed panel with contextual visibility (Trace/Board/Learn/Progress/Health/Tokens)
    TraceabilityView.tsx             Requirement-to-test traceability table with status badges
    HealthDashboard.tsx              System health vital signs (live score + post-build grade + breakdown + Architect-level trend chart)
    BoardOutput.tsx                  Serial output (conditional on serial data)
    TeachingSidebar.tsx              Learning moments list
    ProgressPanel.tsx                Build progress bar + phase text
  shared/ModalHost.tsx               Renders all overlay modals (gate, question, flash, skills, rules, portals, dir picker, board detected, examples, help)
  shared/HumanGateModal.tsx          Blocks pipeline, awaits user approve/reject
  shared/QuestionModal.tsx           Multi-choice from agent, user picks answer
  shared/TeachingToast.tsx           Floating notification for learning moments
  shared/AgentAvatar.tsx             Status dot + role icon
  shared/MinionAvatar.tsx            Animated avatar for narrator/minion characters
  shared/ProofMeter.tsx              Segmented progress bar for requirement verification (green/red/amber)
  shared/ExamplePickerModal.tsx      Card grid to choose bundled example nuggets (filters by availableDeviceIds when requiredDevices specified)
  shared/DirectoryPickerModal.tsx    Text input fallback for non-Electron workspace directory selection
  shared/BoardDetectedModal.tsx      Celebrates ESP32 connection, offers one-click Portal creation
  shared/FlashWizardModal.tsx        Multi-device flash wizard with progress bar for IoT deploy
  shared/MeetingInviteToast.tsx      Floating meeting invite notification with accept/decline + 30s auto-dismiss
  shared/MeetingInviteCard.tsx       Inline meeting invite card for embedding in done modal (non-positioned, flow layout)
  shared/LevelBadge.tsx             System level badge (Explorer/Builder/Architect) in header
  shared/ImpactPreview.tsx          Pre-execution impact preview card (task estimate, complexity, heaviest reqs)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zoidbergclawd/elisa](https://github.com/zoidbergclawd/elisa) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
