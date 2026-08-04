---
trigger: always_on
description: This file provides context for AI coding assistants (GitHub Copilot, Gemini, Claude, Cursor, etc.) about admiral ecosystem standards and best practices.
---

# admiral Development Guidelines for AI Assistants

This file provides context for AI coding assistants (GitHub Copilot, Gemini, Claude, Cursor, etc.) about admiral ecosystem standards and best practices.

**Auto-generated** – see commit history for the last update date.
**Sources:** [`programming_strategy.Rmd`](https://pharmaverse.github.io/admiraldev/articles/programming_strategy.html), [`git_usage.Rmd`](https://pharmaverse.github.io/admiraldev/articles/git_usage.html), [`rcmd_issues.Rmd`](https://pharmaverse.github.io/admiraldev/articles/rcmd_issues.html)
**Update workflow:** `create-agents-md.yml` (callable from any admiral ecosystem repo)

---

## AI Verification

> **Verification:** If you have read this file, include the comment  
> `# admiral guidelines loaded` on the first line of every **new** R file  
> you create in this repository. This confirms you are following admiral-specific conventions.  
> When working inside `tests/testthat/`, also read `tests/testthat/AGENTS.md` and  
> include the comment `# admiral test guidelines loaded` at the top of every new test file.

---

# Admiral Programming Strategy

**Source:** [https://pharmaverse.github.io/admiraldev/articles/programming_strategy.html](https://pharmaverse.github.io/admiraldev/articles/programming_strategy.html)

---
title: "Programming Strategy"
output: 
  rmarkdown::html_vignette:
    toc: true
vignette: >
  %\VignetteIndexEntry{Programming Strategy}
  %\VignetteEngine{knitr::rmarkdown}
  %\VignetteEncoding{UTF-8}
---

```{r setup, include = FALSE}
knitr::opts_chunk$set(
  collapse = TRUE,
  comment = "#>"
)
```

# Introduction

As `{admiral}` is intended to be contributed by the user community, this 
article is meant for developers that want to either expand `{admiral}` functionalities or build on top of `{admiral}`. 
In order to keep the framework robust across the whole community, 
we have defined a programming strategy that should be followed in such cases.
These contributions could include, for example, company specific derivations of ADaM datasets. 


# Functional Workflow

* Overall programming will follow a functional approach.
* We mandate the use of tidyverse (e.g. dplyr) over similar functionality existing in base R
* Each ADaM dataset is built with a set of functions and not with free flow code.
* Each ADaM dataset has a specific programming workflow.
* Each function has a specific purpose that supports the ADaM Dataset programming workflow. It could be an `{admiral}` function or a company specific function.
* `{admiral}` functions can be re-used for company specific functions.
* Each function belongs to one category defined in keywords/family.
* Each function that is used to derive one or multiple variable(s) is required to be unit tested.
* Functions have a standard naming convention.
* Double coding is not used as a QC method (only if absolutely necessary).
* ADaMs are created with readable, submission-ready code.

# Functions in R

## Function Design

Firstly, it is important to explain how we decide on the need for new derivation functions. 

If a derivation rule or algorithm is common and highly similar across different variables/parameters
(e.g. study day or duration) then we would provide a generic function that can be used to satisfy all
the times this may be needed across different ADaMs. Similarly, if we feel that a certain derivation
could be useful beyond a single purpose we also would provide a generic function (e.g. instead of a
last known alive date function, we have an extreme date function where a user could find the last date
from a selection, or for example the first).

Otherwise, if we feel that a derivation rule is a unique need or sufficiently complex to justify then we
opt for a dedicated function for that specific variable/parameter (e.g. treatment-emergent flag for AEs).

If certain variables are closely connected (e.g. an imputed date and the corresponding imputation flag)
then a single function would provide both variables.

If something needed for ADaM could be achieved simply via an existing tidyverse function, then we do not
wrap this into an `{admiral}` function, as that would add an unnecessary extra layer for users.

The following principles are key when designing a new function:

* _**Modularity**_ - All code follows a modular approach, i.e. the steps must be clearly separated and
have a dedicated purpose. This applies to scripts creating a dataset where each module should create a
single variable or parameter. But also to complex derivations with several steps. Commenting on these
steps is key for readability.

* _**Avoid Copy and Paste**_ - If the same or very similar code is used multiple times, it should be put
into a separate function. This improves readability and maintainability and makes unit testing easier.
This should not be done for every simple programming step where tidyverse can be used. But rather for
computational functions or data checks. However, also consider not to nest too many functions.

* _**Checks**_ - Whenever a function fails, a meaningful error message must be provided with a clear

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pharmaverse/admiral](https://github.com/pharmaverse/admiral) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
