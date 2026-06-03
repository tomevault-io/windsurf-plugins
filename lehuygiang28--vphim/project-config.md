---
trigger: always_on
description: This guide defines the steps and best practices for understanding the codebase and for creating accurate and useful diagrams. Use the following rules to ensure consistency and clarity in your documentation.
---

# Rules Guide for Diagram Instructions

This guide defines the steps and best practices for understanding the codebase and for creating accurate and useful diagrams. Use the following rules to ensure consistency and clarity in your documentation.

---

## 1. General Diagram Instructions

- **Output Format & Location:**  
  When a user requests a diagram, write it as PlantUML code. Save the generated file in the `/documents` folder.

- **Determine Diagram Type:**  
  Analyze the user request carefully to decide which type of diagram is needed (e.g., sequence diagram, class diagram, component diagram). Research current best practices from reliable sources (such as the PlantUML website, Wikipedia, Creately, or industry blogs) to choose and format the correct diagram.

- **Codebase Research:**  
  This repository is organized as a monorepo using Nx and contains the following projects:
  - **`apps/api`:** API backend project (written with NestJS).
  - **`apps/fe`:** Front-end project for main users/members (written with Next.js).
  - **`apps/mnt`:** Front-end project for administrative management (written with Next.js).
  - **`apps/mobile`:** Mobile project (built with Expo and UI Kitten).

  Perform a deep analysis of the entire codebase to understand how these projects communicate and depend on each other. This understanding is critical for creating diagrams that accurately represent inter-project interactions and dependencies.

  Deeply analysis to know what components of front-end, what class/object/instance of api (nestjs backed) that will appear in diagrams
  The component/class/object/instance must get it from codebase, DO NOT generate it, must have it in codebase

---

## 2. Specific Instructions for Sequence Diagrams

When a user requests a **sequence diagram**, follow these steps:

### 2.1 Reference Documentation

Read `https://agilemodeling.com/style/sequenceDiagram.htm`.
Read `https://zenuml.com/blog/2024/02/12/2024/best-practices-for-effective-sequence-diagrams/#annotations-and-comments`.
Read `https://www.lucidchart.com/pages/uml-sequence-diagram`.

Include background context to guide the creation of the sequence diagram. For example, incorporate documentation similar to the following excerpt:

```txt
From Wikipedia.

In software engineering, a sequence diagram shows process interactions arranged in time sequence. This diagram depicts the processes and objects involved and the sequence of messages exchanged as needed to carry out the functionality. Sequence diagrams are typically associated with use case realizations in the 4+1 architectural view model of the system under development. Sequence diagrams are sometimes called event diagrams or event scenarios.

For a particular scenario of a use case, the diagrams show the events that external actors generate, their order, and possible inter-system events.[2] The diagram emphasizes events that cross the system boundary from actors to systems. A system sequence diagram should be done for the main success scenario of the use case, and frequent or complex alternative scenarios.

There are two kinds of sequence diagrams:

- Sequence Diagram (SD): A regular version of sequence diagram describes how the system operates, and every object within a system is described specifically.
- System Sequence Diagram (SSD): All systems are treated as a black box, where all classes owned by the system are not depicted. Instead, only an object named System is depicted.

# Key elements of sequence diagrams
A sequence diagram shows, as parallel vertical lines (lifelines), different processes or objects that live simultaneously, and, as horizontal arrows, the messages exchanged between them in the order in which they occur. This allows for the graphical specification of simple runtime scenarios.

A system sequence diagram should specify and show the following:

- External actors
- Messages (methods) invoked by these actors
- Return values (if any) associated with previous messages
- Indication of any loops or iteration area
```

### 2.2 Diagram Construction
- **PlantUML Code Generation:**  
  Write the diagram code using PlantUML syntax, enclosed within `@startuml` and `@enduml` tags.

- **Save Location:**  
  Always place the generated diagram file in the `/documents` folder.

- **Diagram Content:**  
  Ensure the diagram includes all key interactions by doing the following:
  - Identify all relevant actors (external users, systems) and label them clearly.
  - Include lifelines for each actor or object.
  - Use solid arrows (`->`) for synchronous messages and dotted arrows (`-->`) for return messages.
  - Label messages with descriptive method names.
  - Include conditional logic and self-messages (when applicable) using fragments like `alt`, `opt`, or `loop`.
  - Application Communication Pattern: Note that our application follows a communication pattern where interactions typically occur from the front-end component to a resolver. For example, in a sequence diagram, you should represent messages like: `:FrontEndComponent to :SomeResolver`

- **Detail Level:**  

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lehuygiang28/vphim](https://github.com/lehuygiang28/vphim) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-03 -->
