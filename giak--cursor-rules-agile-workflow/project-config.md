---
trigger: always_on
description: IMPLEMENT when CREATING or MANAGING epics and stories to ENSURE consistent structure, effective splitting, and proper tracking
---


# Epic & Story Management Framework

↹ foundational•principles [p=1] {
  ⊕ structure•standardization => consistent•documentation
  ⊕ value•first•approach => user•centric•stories
  ⊕ appropriate•granularity => manageable•increments
  ⊕ traceable•relationships => connected•artifacts
  ⊕ continuous•refinement => evolving•quality
  ⊕ measurable•outcomes => verifiable•success
}
Σ core•guidelines

↹ document•relationships [p=1] {
  ⊕ hierarchy•position => PFD → PRD → Architecture → Epics → Stories
  
  ⊕ pfd•relationship {
    consumes: strategic•vision+business•goals,
    provides: high•level•context+project•boundaries,
    ensures: business•alignment+strategic•fit
  }
  
  ⊕ prd•relationship {
    consumes: functional+non-functional•requirements,
    provides: feature•specifications+acceptance•criteria,
    transforms: requirements → implementable•stories,
    requires: bi-directional•traceability
  }
  
  ⊕ architecture•relationship {
    consumes: technical•constraints+design•decisions,
    provides: implementation•guidance+technical•boundaries,
    ensures: technical•feasibility+architectural•compliance,
    requires: consistent•technical•approach
  }
  
  ⊕ epic•to•prd•mapping {
    requirement: every•epic•maps•to•prd•feature,
    traceability: explicit•references•to•prd•sections,
    validation: confirm•coverage•of•requirements
  }
  
  ⊕ story•to•architecture•mapping {
    requirement: implementation•conforms•to•architecture,
    traceability: references•to•architectural•components,
    validation: technical•alignment•verification
  }
  
  ⊕ visual•representation: ```mermaid
    flowchart TB
      PFD[Project Foundation Document] --> PRD[Product Requirements Document]
      PFD --> ARCH[Architecture Document]
      
      PRD --> EPICS[Epics]
      ARCH --> EPICS
      
      EPICS --> STORIES[Stories]
      STORIES --> TASKS[Implementation Tasks]
      
      classDef pfd fill:#ff9900,stroke:#333,stroke-width:2px
      classDef prd fill:#66cc99,stroke:#333,stroke-width:2px
      classDef arch fill:#6699ff,stroke:#333,stroke-width:2px
      classDef epic fill:#cc99ff,stroke:#333,stroke-width:2px
      classDef story fill:#ff99cc,stroke:#333,stroke-width:2px
      classDef task fill:#ffcc66,stroke:#333,stroke-width:1px
      
      class PFD pfd
      class PRD prd
      class ARCH arch
      class EPICS epic
      class STORIES story
      class TASKS task
  ```
}
Σ documentation•hierarchy

↹ epic•organization [p=1] {
  ⊕ directory•structure => .ai/epic-{n}/
  ⊕ epic•metadata => _epic.md
  ⊕ story•placement => {m}-{code}.story.md
  ⊕ sequential•numbering => continuous•across•project
  ⊕ descriptive•naming => kebab-case•convention
  ⊕ status•tracking => [not-started|in-progress|complete]
}
Σ storage•convention

↹ epic•template [p=1] -> [
  ⊕ header {
    title: "Epic-{N}: {Epic Title}",
    format: markdown•h1
  },
  
  ⊕ traceability {
    prd: "Reference to PRD section: {Section}.{Subsection}",
    requirements: list•of•requirement•ids•from•prd
  },
  
  ⊕ description {
    content: comprehensive•overview,
    scope: clearly•defined•boundaries
  },
  
  ⊕ objectives {
    format: bullet•list,
    focus: business•value,
    specificity: measurable•when•possible
  },
  
  ⊕ stories {
    format: checkbox•list,
    naming: "Story-{N}.{M}: {Title}",
    status: progress•tracking
  },
  
  ⊕ success•criteria {
    format: bullet•list,
    qualities: specific+measurable+achievable
  },
  
  ⊕ risks {
    format: table,
    columns: [risk+impact+probability+mitigation],
    completeness: all•identified•risks
  },
  
  ⊕ dependencies {
    internal: cross•epic•references,
    external: third•party•dependencies,
    architectural: references•to•architecture•components
  },
  
  ⊕ status {
    values: [not-started|in-progress|complete],
    visibility: prominently•displayed
  }
]
Σ epic•document•structure

↹ story•lifecycle [p=1] {
  ⊕ creation => draft•initial•structure
  ⊕ refinement => complete•details•estimate
  ⊕ validation => verify•INVEST•criteria
  ⊕ prd•compliance => verify•alignment•with•requirements
  ⊕ architecture•compliance => verify•technical•feasibility
  ⊕ approval => ready•for•development
  ⊕ implementation => in•progress•status
  ⊕ verification => acceptance•criteria•check
  ⊕ completion => documentation•updates
}
Σ story•progression

↹ story•template [p=1] -> [
  ⊕ header {
    title: "Story: {Story Title}",
    meta: "Epic-{N}: {Epic Title}\nStory-{M}: {Story Title}"
  },
  
  ⊕ traceability {
    prd: "Reference to PRD requirement: {Requirement ID}",
    architecture: "Reference to Architecture component: {Component Path}"
  },
  
  ⊕ description {
    format: "**En tant que** {role}\n**Je veux** {action}\n**afin de** {benefit}",
    focus: value•to•user
  },
  
  ⊕ status {
    values: [draft|in-progress|complete|cancelled],
    visibility: always•current
  },
  
  ⊕ context {
    epic•relationship: connection•to•parent,
    dependencies: related•stories,
    constraints: limitations•assumptions,
    architectural•considerations: relevant•design•patterns+constraints
  },
  
  ⊕ estimation {
    format: "Story Points: {number}",
    scale: relative•complexity
  },
  
  ⊕ acceptance•criteria {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/giak) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
