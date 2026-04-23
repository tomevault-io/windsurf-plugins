---
trigger: always_on
description: **Due:** April 17, 2025
---

# CineMatch — Product Requirements Document

**Version:** 1.0  
**Due:** April 17, 2025  
**Team Size:** 5 members  

---

## Table of Contents

1. [Overview](#overview)
2. [Problem & Goals](#problem--goals)
3. [User Stories](#user-stories)
4. [Features & Requirements](#features--requirements)
5. [Technical Stack](#technical-stack)
6. [Application Structure](#application-structure)
7. [Vue Component Tree](#vue-component-tree)
8. [Express API Routes](#express-api-routes)
9. [Database Schema](#database-schema)
10. [Recommendation Engine](#recommendation-engine)
11. [Rubric Mapping](#rubric-mapping)
12. [Submission Checklist](#submission-checklist)

---

## Overview

CineMatch is a personalized movie discovery and recommendation web application. Users search and can save films to watchlists, while an intelligent recommendation engine learns their taste and surfaces movies they are likely to love. Rich D3-powered visualizations give users insight into their own viewing habits and preferences.

---

## Problem & Goals

Deciding what to watch is overwhelming — streaming platforms surface algorithmic content that favors their own catalogue. CineMatch puts the user in control: a personal watchlist and viewing history feed a transparent recommendation engine, and dashboards reveal patterns in their own taste over time.

**Goals:**
- Let users discover movies through search, filters, and smart recommendations
- Provide a personal watchlist and watch history system
- Visualize user taste with interactive D3 charts
- Demonstrate all course technologies in a cohesive, production-ready app

---

## User Stories

| As a...       | I want to...                                      | So that...                                      |
|---------------|---------------------------------------------------|-------------------------------------------------|
| Guest user    | Search and browse movies                          | I can explore content without an account        |
| Registered user | Save movies to my watchlist                    | I can track what I want to watch                |
| Registered user | Mark movies as watched                         | My history feeds the recommendation engine      |
| Registered user | Get personalized recommendations               | I discover movies tailored to my taste          |
| Registered user | View my taste dashboard                        | I understand my own viewing patterns            |
| Any user      | View full movie details with trailers            | I can decide if a movie interests me            |

---

## Features & Requirements

### 1. Search & Browse

- Full-text movie search via TMDB API
- Filter by genre, release year, language, and minimum rating
- Infinite scroll results with poster cards
- Trending and new release sections on the home page

### 2. Watchlist

- Add/remove movies to a personal watchlist
- Mark movies as watched (moves to history)
- Drag-and-drop reordering of watchlist items
- Shareable watchlist link via generated URL token

### 3. Movie Detail Page

- Full metadata: title, overview, genres, release date, runtime, language
- Embedded YouTube trailer via TMDB video endpoint
- Cast carousel with headshots from TMDB
- Similar movie suggestions pulled from TMDB `/movie/{id}/similar`
- Link to full cast and crew page

### 4. Recommendation Engine *(new technology)*
 
- Semantic similarity powered by OpenAI `text-embedding-3-small` model
- Each movie's overview + genres + director are embedded into a vector on first watch and cached in the DB
- A user profile vector is computed by averaging all watched movie embeddings
- Candidate movies (trending + popular from TMDB, not yet watched) are scored by cosine similarity against the profile vector
- Recommendation feed shown on home page and dedicated `/recommendations` page

### 5. Taste Dashboard

- **Genre breakdown** — D3 pie/donut chart of watched genres
- **Watch history timeline** — D3 line chart of movies watched per month
- **Decade heatmap** — D3 heatmap of movies by decade and genre
- **Top directors** — D3 horizontal bar chart of most-watched directors

### 6. Authentication

- Register and login with email and password
- Passwords hashed with `bcrypt`
- Sessions managed via JWT stored in `localStorage`
- Protected routes redirect unauthenticated users to login

---

## Technical Stack

| Layer         | Technology                          |
|---------------|-------------------------------------|
| Frontend      | Vue 3 (Composition API)             |
| Routing       | Vue Router                          |
| State         | Pinia                               |
| Styling       | Tailwind CSS                        |
| Charts        | D3.js                               |
| Backend       | Node.js + Express.js                |
| Database      | SQLite (via `better-sqlite3`)       |
| External API  | TMDB API (free tier)                |
| Auth          | JWT + bcrypt                        |
| New Tech      | OpenAI Embeddings API (`text-embedding-3-small`)  |

> **Why SQLite?** Zero setup, file-based, works out of the box on any machine for demo and grading purposes.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jhadenn) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
