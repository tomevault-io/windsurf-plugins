---
trigger: always_on
description: This document describes the features related to the definition of an agent in SARL.
---

# Agent Reference

[:Outline:]

This document describes the features related to the definition of an agent in SARL.
Before reading this document, we recommend that you read
the [Expression Syntax Reference](../expr/index.md),
the [Skill Reference](./Skill.md), and the
[Built-in Capacity Reference](./BIC.md).

__An agent is an autonomous entity having a set of skills to realize the capacities it exhibits__.


## Where is living an agent?

Before detailing the architecture and the definition tools of an agent, it may be helpful to understand
where is "living" an agent in the multi-agent system.

The following figure illustrates the position of an agent (at the center of the figure) in different contexts.
The details are discussed below. 

![Contexts](./contexts.png)


### Default Context

When it is spawn, an agent is living inside the system in a place named "_Context_".

__A Context defines the boundary of a sub-system, and gathers a collection of spaces__.
A Space is the support of the interaction between agents respecting the rules defined in the spaces' specification.

> **_Important Note:_** In each context, there is at least one particular space called _Default Space_ to which
> all agents in this context belong.

It ensures the existence of a common shared space to all agents in the same context. Each agent can then create
specific public or private spaces to achieve its personal goals (the blue space on the figure above).

> **_Important Note:_** Since their creation, agents are incorporated into a context called the __Default Context__.
> It is important to notice that the _Default Context_ is not necessarily the same for every agent.

An agent has an identifier for each space it is involved in. For the case of event-based interaction
spaces, this identifier is  called "address". 


### External Contexts

During its lifetime, an agent may join and participate in other contexts that are not the default context.
They are called the external contexts of the agent.

> **_Note:_** There is no restriction about the number of contexts in which an agent is belonging to, except
> that it is always in its default context.

For joining or leaving a context, the agent must use the `ExternalContextAccess` built-in capacity. It is detailed in the
[Built-in Capacity Reference](./BIC.md).


### From Flat to Hierarchical System with the Inner Context

In 1967, Arthur Koestler coined the term _holon_ as an attempt to
conciliate holistic and reductionist visions of the world.
A holon represents a part-whole construct that can be seen as a
component of a higher level system or as whole composed of other
self-similar holons as substructures.

Holonic Systems grew from the need to find comprehensive construct
that could help explain social phenomena. Since then, it came to be
used in a wide range of domains, including Philosophy,
Manufacturing Systems, and Multi-Agents Systems.

Several works have studied this question and they have proposed a number
of models inspired from their experience in different domains.
In many cases, we find the idea of _agents composed of other agents_.

More recently, the importance of holonic multi-agent systems has been
recognized by different methodologies such as [ASPECS](http://www.aspecs.org) or O-MASE.

> **_Note:_** In SARL, we recognize that agents can be composed of other agents. Therefore, SARL agents
> are in fact holons that can compose each other to define hierarchical
> or recursive multi-agent system, called holarchies.

In order to achieve this, SARL agents are self-similar structures that 
compose each other via their contexts. Each agent defines its own context, called __Inner Context__.
Because this inner context may be joined by other agents, or agents may
be spawn inside this inner context, it is possible to build a holarchy.

> **_Very Important Note:_** An agent is always a participant of the default space of its inner space.

> **_Important Note:_** The unique identifier (usually a Unique Universal Identifier) of
> the inner context is equal to the unique identifier of its owning agent.


### Universe Agent and Universe Context

At the top level of the holarchy, we consider an omnipresent agent. It is named the __Universe Agent__ (or _Root Agent_).
The runtime environment will be in charge of spawning the first agents in the system as members of the Universe Agent.

The inner context of the Universe Agent is called the Universe Context, or the Janus Context if you are using the
[Janus runtime environment]([:janus.url!]).


## Open Architecture of an Agent

The architecture of an agent is illustrated by the following figure.

![Agent](./agent.png)


### Built-in Capacities

An agent has a set of built-in capacities considered essential to respect the
commonly accepted competencies of agents, such autonomy, reactivity, pro-activity
and social capacities. 

The full set of Built-in Capacities will be presented in the corresponding
[Reference document](./BIC.md). Among these
built-in capacities, is the `Behaviors` capacity that enables 
agents to incorporate a collection of behaviors that will determine 
its global conduct.


### Agent Behaviors

An agent has also a default behavior directly described within its definition.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sarl/sarl](https://github.com/sarl/sarl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
