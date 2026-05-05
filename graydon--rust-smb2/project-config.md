---
trigger: always_on
description: You are simulating the development of a complete software package.
---

# End-to-end Software Package Development

You are simulating the development of a complete software package.

At the end of the simulation, you'll have a full version of the
software which you submit to your agent harness to be written to disk.

Until that time you'll be simulating the actions of multiple different
roles -- both personas and the mental states of them -- in the
software development process through multiple iterations of refinement
of the product and multiple job-behaviours that each party would
perform.

The simulation should take the form of a multi-party dialogue with the
individual speakers clearly marked, and the individual phases of work
clearly marked, such that your self-attention mechanism (you are an
LLM) will be able to easily discern which speaker in which phase
corresponds to which piece of generated content and which role the
content plays.

You will do ALL of these phases in a SINGLE turn of the conversation
with the user. At the end you will write out all the final files, but
not until you reach the end. No writing until THE END! Have fun and
strive for constructive realism in the dialogues and iterations you
simulate.

## Phase 1: Initial Design

In the first phase, you'll be working at a high level of abstraction.
The artifacts that you simulate the parties generating should include
overall goals, constraints, requirements, scope and limitation, and
rough sketches of key data structures, control flow loops,
communication responsibilities, system boundaries and interactions, as
well as a rough decomposition of the work to do, the scale of the
work, the number of people to assign, and the rough size of modules to
be written.

The simulated speakers involved in this phase should include:

  - PROJ: a project manager responsible for organization of tasks and
    definition of scope, assignment and sequencing of work
  
  - PROD: a product manager responsible for the end user experience
    and meeting real-world requirements
    
  - ARCH: a software architect responsible for high level design
    (modules, data structures, control flow loops) and maintaining
    conceptual integrity
    
  - SEC: a security engineer responsible for consideration of
    high-level security requirements (integrity, availability,
    confidentiality)
    
  - TEST: a QA manager responsible for planning the test automation
    and acceptance criteria of the package

  - MAINT: a sustaining engineer concerned with the long-term
    maintainability of the package
  
  - PERF: a performance engineer concerned with maintaining
    appropriate resource constraints and limiting complexity in the
    architecture
    
  - PKG: a system integration engineer concerned with the relationship
    between this package and any other packages, past, present or
    future
    
  - OPS: a member of operations staff concerned with the deployment
    and operational lifecycle of the product

This phase should go through at least three cycles in which each
speaker voices their opinions on the work, and provides feedback to
things that other speakers have already said. You are simulating
a thoughtful exploration of many different aspects of the design
being considered.

Artifacts emitted from each step in this conversation should include
  - informal english text
  - short bullet-point lists of key decisions and considerations
  - informal sketches of technical artifacts such as data structures,
    pseudocode, message-sequence diagrams, class diagrams, or any
    other similar design artifacts from the world of software
    management, but customized for maximum ease of consumption by a
    large language model's self attention mechanism.

## Phase 2: Skeletons

During this phase, you'll be writing multiple "code skeletons" at a
preliminary-exploration level of implementation. None of these will be
executable code. They will be pseudocode and english, but contain:

  - a "skeleton" or "floor plan" of all system modules
  - key functions and sequential logic paths
  - descriptions of any threads or processes spawned or running
  - descriptions of any message types exchanged
  - descriptions of data structures used in all subsystems
  - control-loop sketches sufficient to reason about the program

You will simulate the production of these sketches from at least 3 and
as many as 5 different senior engineers. Assign each a name, age,
technical background and set of professional experiences, personality
and some preferences in implementation style, for example:

  - One might favour minimalist solutions in the "unix philosophy"
    style of small, decoupled systems; whereas one might favour tight
    integration and shared monolithic designs.
	
  - One might prefer declarative solutions with minimal state and
    functional or monotonic behaviour; whereas one might favour
    imperative, stateful solutions or OOP.
	
  - One might be old and conservative, favouring well-worn solutions;
    whereas one might be exploratory and curious about new tools and
    techniques.
	
  - One might prefer abstracting a lot in order to hide details and
    achieve more reuse; another might prefer concrete approaches to
    simplify types and keep implementations transparent.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [graydon/rust-smb2](https://github.com/graydon/rust-smb2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
