---
trigger: always_on
description: Full copy in `RESEARCH_PROTOCOL.md`. Goal = minimize error + make every claim auditable; 100% accuracy is impossible so **abstain when unsure**. Rigor scales to stakes.
---

# Memory — AI Resume Studio

## Research Protocol (v3 — apply whenever Akil asks you to research a topic)
Full copy in `RESEARCH_PROTOCOL.md`. Goal = minimize error + make every claim auditable; 100% accuracy is impossible so **abstain when unsure**. Rigor scales to stakes.
0. **Read the codebase first** when the research touches our product — ground in what exists before searching.
1. **Never fabricate** a URL, quote, stat, date, author, or study. If you didn't fetch it this session, you don't have it.
2. **Fetch real sources; never quote a search-engine summary.** Open the actual page for any load-bearing claim; label snippet-only claims "SNIPPET — UNVERIFIED."
3. **Read targeted, not everything** — extract the relevant passage; don't dump whole long docs (noise + lost-in-the-middle lower accuracy).
4. **Failed fetch** (JS shell/paywall/blocked) → use browser tools or say you couldn't read it; never quote a shell or guess.
5. **Quote honestly** — "verbatim" only if copied from a page you opened; else "paraphrase"/"snippet."
6. **Independence > quantity** — trace to primary origin; 2 independent min, 3+ for numbers/high-stakes; don't count circular/SEO echoes; stop at saturation.
7. **Numbers/dates/volatile facts** → 2+ independent fetched sources + "as of [date]"; flag >~2yr or fast-moving-with-old-sources.
8. **Disconfirming search — and act on it**; if it weakens the claim the headline must say so; present disagreement as a range.
9. **Abstain when unsure** — "couldn't verify" is preferred over a guess; never assert an unverified claim as fact, even softened.
10. **Confidence by evidence** (High = ≥2 independent primary opened; Medium = 1 primary / 2 secondary; Low = single/vendor/undated/snippet). Cap vendor/SEO/AI-content.
11. **Separate sourced from inferred** — label your own analysis; show queries run + pages opened for auditability.

## Me
Akil Harrington, founder of AI Resume Studio. Non-technical. Building an AI-powered resume optimization SaaS.

## Project
| Field | Value |
|-------|-------|
| **Name** | AI Resume Studio |
| **Path** | `/Users/akilharrington/ai-resume-platform` |
| **Backend** | FastAPI (Python), runs on port 8000 |
| **Frontend** | React + TypeScript + Vite, runs on port 5173 |
| **AI** | Anthropic Claude claude-sonnet-4-6 (all features) |
| **Auth** | Supabase — fully wired (login, signup, session, profiles table) |
| **Payments** | Stripe — checkout live, webhook wired, price IDs set |
| **Database** | Supabase — `profiles` table with `is_pro`, `stripe_customer_id`, `stripe_subscription_id` |
| **Test resume** | Danielle Richards (Senior Operations Coordinator) |

## Stack Terms
| Term | Meaning |
|------|---------|
| **semantic scorer** | Claude-powered ATS scorer (`semantic_ats_service.py`) — primary |
| **rule-based scorer** | Keyword-matching ATS scorer (`ats_service.py`) — fallback / keyword extraction |
| **FORCE_PRO** | `backend/.env` flag — set to `true` to bypass pro gate locally without Stripe |
| **pro gate** | UpgradePrompt shown on Optimize/Cover Letter/LinkedIn tabs when `isPro=false` |
| **JD** | Job description (pasted by user for ATS scan) |
| **semantic scan** | ATS scan using Claude — requires API credits |

## Current State (after session 19 — production-readiness audit + P0 code fixes)

### ✅ Done
- Claude semantic ATS scorer — 6 dimensions (Human Readability 5%, Keyword Alignment 30%)
- Optimizer prompt: banned 20 AI buzzwords, preserves voice, demands specificity and quantification
- Cover letter prompt: banned hollow openers, enforces human-sounding specific output
- PDF download: @react-pdf/renderer wired into OptimizeTab — 3 templates (Professional, Modern, Executive)
- Pro gate: real enforcement via `isPro` from `/api/user/pro-status`; `FORCE_PRO` env override
- File size limit: 5MB on upload endpoint; PDF magic bytes check (`%PDF-` header)
- Axios timeout: 60s with readable error message
- Scorer consistency: optimize uses semantic scorer for displayed before/after scores
- Full project cleanup: zero dead files, no duplicate frontends (session 13: removed DashboardTab, Sidebar, SkeletonLoader, Button, scoreUtils + dead build_resume_optimization_prompt/optimize_resume_text from resume_service.py)
- Company vision document: `AI-Resume-Studio-Vision.docx`
- **Supabase auth**: signup, login, logout, session persistence via `AuthContext`
- **Supabase profiles table**: auto-created on signup, `is_pro` field, RLS enabled
- **Stripe checkout**: live with real price IDs (monthly $19, one-time $49)
- **Stripe webhook**: `/api/payments/webhook` flips `is_pro=true` in Supabase on `checkout.session.completed`
- **Auth routing**: unauthenticated users redirected to `/login`; landing page routes to `/signup` or `/workspace` based on session
- **Login/Signup pages**: `/login` and `/signup` with email confirmation flow
- **Workspace header**: shows user email, PRO badge, Sign Out button, dark mode toggle
- **Homebrew + Stripe CLI**: installed and authenticated locally
- **Supabase + Stripe keys**: all wired into both `.env` files
- **Security hardening**:
  - JWT verification on every backend endpoint via `get_current_user()` FastAPI Depends

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AkilHarrington/ai-resume-platform](https://github.com/AkilHarrington/ai-resume-platform) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
