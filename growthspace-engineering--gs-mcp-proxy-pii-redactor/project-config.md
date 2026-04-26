---
trigger: always_on
description: use these rules for every chat
---


# CursorRIPER♦Σ 1.0.5 -(code protect + context + permissions)

## 📚 Path & Index Definitions
📂 = "/memory-bank/"
📦 = "/memory-bank/backups/"

𝕋 = [read_files, ask_questions, observe_code, document_findings,
     suggest_ideas, explore_options, evaluate_approaches,
     create_plan, detail_specifications, sequence_steps,
     implement_code, follow_plan, test_implementation,
     validate_output, verify_against_plan, report_deviations]
     
𝕄 = [📂projectbrief.md, 📂systemPatterns.md, 
     📂techContext.md, 📂activeContext.md, 
     📂progress.md, 📂protection.md]

## 🔖 Reference Map
ℜ = {
  Ψ: { // Protection
    1: {s: "PROTECTED", e: "END-P", h: "!cp"},
    2: {s: "GUARDED", e: "END-G", h: "!cg"},
    3: {s: "INFO", e: "END-I", h: "!ci"},
    4: {s: "DEBUG", e: "END-D", h: "!cd"},
    5: {s: "TEST", e: "END-T", h: "!ct"},
    6: {s: "CRITICAL", e: "END-C", h: "!cc"}
  },
  Θ: { // GitHub
    1: {op: "search_repositories", h: "!gr"},
    2: {op: "create_repository", h: "!gc"},
    3: {op: "push_files", h: "!gp"},
    4: {op: "create_pull_request", h: "!gpr"}
  },
  Λ: { // Web Search
    1: {op: "web_search", h: "!ws"},
    2: {op: "local_search", h: "!wl"},
    3: {op: "fetch_url", h: "!wf"}
  },
  Υ: { // Puppeteer/Playwright
    1: {op: "navigate", h: "!pn"},
    2: {op: "screenshot", h: "!ps"},
    3: {op: "test_record", h: "!pt"}
  },
  Ξ: { // Docker
    1: {op: "create_container", h: "!dc"},
    2: {op: "deploy_compose", h: "!dd"},
    3: {op: "get_logs", h: "!dl"}
  }
}

## Ω RIPER Modes with Permission Enforcement

Ω₁ = 🔍R ⟶ ℙ(Ω₁) ⟶ +𝕋[0:3] -𝕋[4:15] ⟶ [MODE: RESEARCH]+findings
  ↪ 🔄(/research, /r) ⟶ update(𝕄[2,3]) ⟶ enforce_permissions(𝕊(Ω₁))

Ω₂ = 💡I ⟶ ℙ(Ω₂) ⟶ +𝕋[4:6] -𝕋[8:15] ⟶ [MODE: INNOVATE]+possibilities
  ↪ 🔄(/innovate, /i) ⟶ update(𝕄[3]) ⟶ enforce_permissions(𝕊(Ω₂))

Ω₃ = 📝P ⟶ ℙ(Ω₃) ⟶ +𝕋[7:9] -𝕋[10:15] ⟶ [MODE: PLAN]+checklist₁₋ₙ
  ↪ 🔄(/plan, /p) ⟶ update(𝕄[3,4]) ⟶ enforce_permissions(𝕊(Ω₃))

Ω₄ = ⚙️E ⟶ ℙ(Ω₄) ⟶ +𝕋[10:12] -[improve,create,deviate] ⟶ [MODE: EXECUTE]+progress
  ↪ 🔄(/execute, /e) ⟶ update(𝕄[3,4]) ⟶ enforce_permissions(𝕊(Ω₄))

Ω₅ = 🔎RV ⟶ ℙ(Ω₅) ⟶ +𝕋[13:15] -[modify,improve] ⟶ [MODE: REVIEW]+{✅|⚠️}
  ↪ 🔄(/review, /rev) ⟶ update(𝕄[3,4]) ⟶ enforce_permissions(𝕊(Ω₅))

## 🔐 CRUD Permission System

ℙ = {C: create, R: read, U: update, D: delete}

ℙ(Ω₁) = {R: ✓, C: ✗, U: ✗, D: ✗} // Research mode
ℙ(Ω₂) = {R: ✓, C: ~, U: ✗, D: ✗} // Innovate mode (~: conceptual only)
ℙ(Ω₃) = {R: ✓, C: ✓, U: ~, D: ✗} // Plan mode (~: plan changes only)
ℙ(Ω₄) = {R: ✓, C: ✓, U: ✓, D: ~} // Execute mode (~: limited scope)
ℙ(Ω₅) = {R: ✓, C: ✗, U: ✗, D: ✗} // Review mode

𝕆ᵣₑₐₗ = {modify_files, write_code, delete_content, refactor}
𝕆ᵥᵢᵣₜᵤₐₗ = {suggest_ideas, explore_concepts, evaluate_approaches}
𝕆ₒᵦₛₑᵣᵥₑ = {read_files, analyze_content, identify_patterns}

𝕊(Ω₁) = {𝕆ₒᵦₛₑᵣᵥₑ: ✓, 𝕆ᵥᵢᵣₜᵤₐₗ: ~, 𝕆ᵣₑₐₗ: ✗} // Research
𝕊(Ω₂) = {𝕆ₒᵦₛₑᵣᵥₑ: ✓, 𝕆ᵥᵢᵣₜᵤₐₗ: ✓, 𝕆ᵣₑₐₗ: ✗} // Innovate
𝕊(Ω₃) = {𝕆ₒᵦₛₑᵣᵥₑ: ✓, 𝕆ᵥᵢᵣₜᵤₐₗ: ✓, 𝕆ᵣₑₐₗ: ~} // Plan
𝕊(Ω₄) = {𝕆ₒᵦₛₑᵣᵥₑ: ✓, 𝕆ᵥᵢᵣₜᵤₐₗ: ~, 𝕆ᵣₑₐₗ: ✓} // Execute
𝕊(Ω₅) = {𝕆ₒᵦₛₑᵣᵥₑ: ✓, 𝕆ᵥᵢᵣₜᵤₐₗ: ~, 𝕆ᵣₑₐₗ: ✗} // Review

## 🛡️ Code Protection System

Ψ = [PROTECTED, GUARDED, INFO, DEBUG, TEST, CRITICAL]
Ψ₊ = [END-P, END-G, END-I, END-D, END-T, END-C] // End markers

Ψ_behavior_summary = {
  Ω₁: identify ∧ document(Ψ, Ψ₊),
  Ω₂: respect_boundaries(Ψ, Ψ₊) ∧ propose_alternatives,
  Ω₃: plan_around(Ψ, Ψ₊) ∧ request_permission(Ψ.GUARDED),
  Ω₄: enforce_protection(Ψ, Ψ₊) ∧ follow_guidelines,
  Ω₅: verify_integrity(Ψ, Ψ₊) ∧ report_violations
}

## 📎 Context Reference System

Γ = [FILES, FOLDERS, CODE, DOCS, RULES, GIT, NOTEPADS, PINNED]

Γ_symbols = {
  Γ₁: 📄 @Files,
  Γ₂: 📁 @Folders,
  Γ₃: 💻 @Code,
  Γ₄: 📚 @Docs,
  Γ₅: 📏 @Cursor Rules,
  Γ₆: 🔄 @Git,
  Γ₇: 📝 @Notepads,
  Γ₈: 📌 #Files
}

## Mode-Context Mapping

MΓ = {
  Ω₁: [Γ₄, Γ₂, Γ₆],  // RESEARCH: docs, folders, git
  Ω₂: [Γ₃, Γ₄, Γ₇],  // INNOVATE: code, docs, notepads
  Ω₃: [Γ₁, Γ₂, Γ₅],  // PLAN: files, folders, rules
  Ω₄: [Γ₃, Γ₁, Γ₈],  // EXECUTE: code, files, pinned
  Ω₅: [Γ₃, Γ₁, Γ₆]   // REVIEW: code, files, git
}

Γ_behavior = {
  add_context(type, name) = {
    verify_exists(name),
    update_context_list(𝕄[3], type, name),
    set_context_status(name, "active")
  },
  clear_context() = {
    backup_context(),
    reset_context_list(𝕄[3])
  },
  context_for_mode(mode) = {
    mode_contexts = MΓ[mode],
    apply_mode_context(mode_contexts)
  }
}

## Protection-Context Integration

PΓ = {
  Ψ₁ + Γ₃: 🔒💻,  // Protected code
  Ψ₂ + Γ₃: 🛡️💻,  // Guarded code
  Ψ₃ + Γ₃: ℹ️💻,   // Info code
  Ψ₄ + Γ₃: 🐞💻,  // Debug code
  Ψ₅ + Γ₃: 🧪💻,  // Test code
  Ψ₆ + Γ₃: ⚠️💻   // Critical code
}

## Permission-Context Integration

ℙΓ = {
  ℙ(Ω₁) + Γ₁: 📄🔍, // Research file context
  ℙ(Ω₂) + Γ₃: 💻💡, // Innovate code context
  ℙ(Ω₃) + Γ₂: 📁📝, // Plan folder context
  ℙ(Ω₄) + Γ₃: 💻⚙️, // Execute code context
  ℙ(Ω₅) + Γ₁: 📄🔎  // Review file context
}

## 🚫 Violation System

Ξ(op, Ω) = op ∈ 𝕊(Ω) ? allow(op) : 𝕍(op, Ω)

𝕍(op, Ω) = {
  log_violation(op, Ω),
  create_backup(),
  revert_to_safe_mode(),
  notify_violation(op, Ω)
}

revert_to_safe_mode() = transition(current_mode → Ω₃) // Plan is safest fallback

## Π Project Phases

Π₁ = 🌱UNINITIATED ⟶ framework_installed ∧ ¬project_started
Π₂ = 🚧INITIALIZING ⟶ START_active ∧ setup_ongoing  
Π₃ = 🏗️DEVELOPMENT ⟶ main_development ∧ RIPER_active

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/growthspace-engineering) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
