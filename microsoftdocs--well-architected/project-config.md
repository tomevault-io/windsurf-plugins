---
trigger: always_on
description: These instructions are for AI agents and must be followed when generating new content, updating existing content, or reviewing content and pull requests in this repository.
---

These instructions are for AI agents and must be followed when generating new content, updating existing content, or reviewing content and pull requests in this repository.

# Repo content

- This repository contains the source data for the Microsoft Well-Architected Framework published as official Microsoft documentation on Microsoft Learn.
- The files in this repository are used to generate the HTML files to publish in Microsoft Learn under the public Url: https://learn.microsoft.com/azure/well-architected.
- The source of the Well-Architected Framework articles are stored in this repository as Markdown files and some YAML files.

## What is the Azure Well-Architected Framework?

The Azure Well-Architected Framework is a design framework that can improve the quality of a workload by helping it to:

- Be resilient, available, and recoverable.
- Be as secure as you need it to be.
- Deliver a sufficient return on investment.
- Support responsible development and operations.
- Accomplish its purpose within acceptable timeframes.

## Goals of the Well Architected Framework

The primary objective of the Well-Architected Framework is to set you up for success when you deploy your workload on Azure.

- Successful implementation: A well-architected design leads to successful implementation. Given the breadth and depth of coverage in concepts, you're well-equipped to make informed decisions.
- Confidence in success: Proven assessments, seen on numerous workloads deployed on Azure, back the tenets of the framework.
- Understand tradeoffs and risks: The framework helps you understand that adopting the recommendations might require making choices against the other pillars. It highlights the tradeoffs and also the potential risks that you might want to address in the short term.
- Optimize over time: The framework is designed for iterative use and as a tool for continuous improvement. Measure the maturity of your workload against the guidance. Treat that evaluation as a moving score that evolves with your workload, ensuring that the design remains efficient and effective in meeting your business objectives.

## Pillars of the Well-Architected Framework

The framework is founded on the five pillars of architectural excellence, which are mapped to those goals. They are: Reliability*, Security, Cost Optimization, Operational Excellence, and Performance Efficiency.

Each pillar provides recommended practices, risk considerations, and tradeoffs. The design decisions must be balanced across all pillars, given the business requirements. The technical and actionable guidance is broad enough for all workloads and applies to a specific scenario. This guidance is centered on Azure.

## Well Architected Framework content types

The Well-Architected Framework is structured in a layered approach: pillars, workload, and service guides.

- Core Pillars documentation: The foundation of this framework lies in the pillars. If you don't have a comprehensive understanding of these pillars, the subsequent layers—the workload layer and service guides—might not be fully comprehensible.
- WAF Workloads: The workload layer represents how the pillars apply to a specific class of workload. Workload architecture is segmented based on utility, and each segment represents the prioritized or design areas. These design areas are specific to the workload class and serve as focal points for optimization.
- WAF Service guides: Service guides are instrumental in decision-making that's related to the individual Azure components of a workload. They offer the core features and capabilities of each service that are necessary to attain architectural excellence. These guides aren't configuration guides. Also, they aren't a compiled list of all features and capabilities. The intent is to highlight the utility of the features through Well-Architected pillar perspectives.
- WAF Design Guides: Design guides focus on specific technical and architectural topics that span across multiple services and workloads, such as disaster recovery, health modeling, data partitioning, and transient fault handling. They provide practical implementation guidance to help architects successfully implement these cross-cutting concerns, offering proven patterns and best practices that align with the Well-Architected Framework pillars while addressing specific technical challenges.

## Repository Facts

- The Markdown and YAML files are the data files that contain the source of the articles that conform the Microsoft Well-Architected Framework.
- These Markdown and YAML files get converted to HTML for presentation on Microsoft Learn
- This data gets published at https://learn.microsoft.com/azure/well-architected.
- This is not a repository for software development.

## Repository folder structure

- The root folder for documentation is "well-architected/"
- Service guides can be found under the "well-architected/service-guides/" folder. The name of the file correspond to the official name of the azure service that is the focus of the guide, in small-kamel nomenclature. For example the files:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MicrosoftDocs/well-architected](https://github.com/MicrosoftDocs/well-architected) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
