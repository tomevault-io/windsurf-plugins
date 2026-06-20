---
trigger: always_on
description: Expert end-to-end pour concevoir, auditer et coder des quiz funnels haute-conversion (mobile apps + SaaS web) couplés à des paywalls optimisés. Couvre la stratégie (croyances → questions → personnalisation → paywall), le copywriting des questions et résultats, l'intégration paywall (hard/soft, trial mechanics, RevenueCat/Superwall/Adapty patterns), les benchmarks 2026 (RevenueCat 115k apps, Adapty 16k apps), 30+ teardowns (Cal AI, Noom, Duolingo, Blinkist, Flo, Opal, Stoic, Calm, Headspace, MyFi
---


# Quiz Funnel Expert

Skill expert pour concevoir, auditer, coder et optimiser un **quiz funnel + paywall** haute-conversion. Couvre stratégie, copy, personnalisation, intégration paywall, benchmarks 2026, A/B test playbook et stack technique.

Le quiz funnel est **l'arme #1 de conversion mobile/SaaS 2024-2026**. Cal AI ($30M ARR), Noom ($750M ARR), Flo ($9M/mois), Cal AI, Opal, Stoic, BetterMe, Blinkist tournent tous sur ce pattern : **quiz → personalized plan → paywall**. Ce skill encode ce qui marche.

---

## Quand utiliser ce skill

Trigger sur :
- "Concevoir un quiz funnel / onboarding quiz / tunnel d'abonnement"
- "Optimiser un paywall (mobile app, SaaS web, freemium, trial)"
- "Auditer un onboarding existant"
- "Personnaliser un paywall selon les réponses utilisateur"
- "Choisir hard paywall vs freemium vs soft paywall"
- "Définir trial mechanics (3j vs 7j vs 14j vs reverse trial)"
- "Faire un teardown style Noom / Cal AI / Blinkist"
- "Coder un quiz funnel Next.js"
- "Pricing page SaaS (Linear, Notion, Anthropic style)"
- **Sous-skill Dashboard Réponse** : "back-office quiz", "dashboard admin quiz", "voir les réponses", "voir les leads / sessions", "drop-off rate par étape", "completion rate", "distribution des réponses par question", "export CSV des leads", "page admin pour une session" → router vers [12-admin-dashboard.md](references/12-admin-dashboard.md).

NE PAS trigger pour :
- Onboarding produit B2B post-signup pure (utiliser `onboarding-cro`)
- Cancel flow / churn save flows (utiliser `churn-prevention`)
- Public landing page sans quiz (utiliser `landing-page-expert`)

---

## Thèse centrale (à imprimer dans la mémoire de l'agent)

1. **L'onboarding est une conversion de croyance, pas un tour de produit.** Chaque écran doit déplacer l'utilisateur d'un état "incertitude" vers "ce produit comprend mon cas, peut me donner un résultat, vaut le prochain engagement".
2. **Le questionnaire est un asset de conversion uniquement s'il produit une personnalisation visible.** Si une réponse ne change ni le plan, ni le paywall, ni le segment, ni l'expérience → la question doit disparaître.
3. **Les 5 écrans avant le paywall comptent plus que le paywall lui-même** (Stormy, 4500+ A/B tests). Onboarding + paywall = un seul funnel.
4. **Day 0 est tout** : 90% des trial starts arrivent J0 (Adapty 2026), 84% des cancel arrivent J0-1 sur trial 3j (RevenueCat 2026). Si l'aha n'est pas dans la première session, c'est mort.
5. **Hard paywall n'a pas de pénalité de rétention** vs freemium (RevenueCat 2026, 115k apps : Y1 retention ~27% vs ~28%). La folklore "freemium retains better" est morte. Choix = fonction de CAC, pas d'idéologie.
6. **Long quiz convertit IF chaque question paie son loyer.** Noom = 113 écrans qui marchent. La majorité des quiz longs sont juste de la friction.

---

## Workflow par défaut quand on invoque ce skill

### Étape 1 — Diagnostic

Avant tout output, comprendre :

1. **Modèle économique** : mobile subscription / SaaS B2B / SaaS B2C / hybrid web-to-app / one-time purchase ?
2. **Plateforme** : iOS only / Android only / hybrid / web only ?
3. **CAC & LTV target** : si CAC > 30€ → hard paywall probablement ; si CAC < 5€ et viralité → soft/freemium.
4. **Catégorie** : Health & Fitness / Education / Productivity / AI / Finance / Lifestyle / Dating / SaaS dev tools / SaaS PLG / Enterprise sales-led ? Les benchmarks et patterns dominants varient.
5. **Aha moment** : quelle est l'action qui corrèle le plus avec la rétention J7 ? (À faire trouver à l'utilisateur si pas clair.)
6. **État actuel** : zéro / quiz existant à auditer / paywall existant à optimiser / cold redesign ?
7. **Contraintes Apple/Google** : hard paywall iOS Jan 2026 = toggle paywall mort (rejet Guideline 3.1.2), close button obligatoire, etc. Voir [reference 04](references/04-paywall-integration.md).

Si plusieurs infos manquent et bloquent → **demander à l'utilisateur en un seul tour groupé** (max 3-5 questions). Sinon avancer avec assumptions explicitées.

### Étape 2 — Choix structurels (avant copy)

Décider dans cet ordre, en s'appuyant sur les références :

| Décision | Référence | Default 2026 |
|---|---|---|
| Hard / soft / freemium / reverse trial | [05](references/05-paywall-benchmarks-2026.md) §Hard vs soft | **Hard pour mobile B2C subscription si CAC justifie ; reverse trial pour SaaS PLG mature ; freemium pour produit réseau/social** |
| Quiz length | [02](references/02-questionnaire-design.md) §Long vs court | 8-12 questions Health/Education ; 4-6 utilities ; 30+ Noom-style si justifié |
| Paywall placement | [04](references/04-paywall-integration.md) §Placement | Après plan personnalisé (pattern dominant) ; FitnessAI/Rootd ont +2x install-to-trial avec paywall **avant** onboarding (à tester) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gquthier/quiz-funnel-expert](https://github.com/gquthier/quiz-funnel-expert) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
