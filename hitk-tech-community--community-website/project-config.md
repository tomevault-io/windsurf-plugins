---
trigger: always_on
description: 1. [Folder Structure](#folder-structure)
---

# Conventions to be followed while working on the project (frontend)

**Index**

1. [Folder Structure](#folder-structure)
2. [React Component Conventions](#react-component-conventions)
3. [CSS Conventions](#css-conventions)
4. [Steps to refactor the codebase](#refactor)

## Folder Structure

```
frontend
└── src
    ├── components
    |    ├── ErrorHandlerPages
    |    |    ├── 404
    |    |    |   ├── NotFound.jsx
    |    |    |   ├── index.js
    |    |    |   ├── not-found.module.scss
    |    |    ├── LoggedIn
    |    |    |   ├── LoggedIn.jsx
    |    |    |   ├── index.js
    |    |    |   ├── logged-in.module.scss
    |    ├── Footer
    |    |    ├── Footer.jsx
    |    |    |── footer.module.scss
    |    |    |── index.js
    |    ├── Navbar
    |    |    ├── Navbar.jsx
    |    |    |── navbar.module.scss
    |    |    |── index.js
    |    ├── util
    |    |    ├── Button
    |    |    |   ├── Button.jsx
    |    |    |   ├── index.js
    |    |    |   ├── button.module.scss
    |    |    ├── DropMenu
    |    |    |   ├── DropMenu.jsx
    |    |    |   ├── index.js
    |    |    |   ├── drop-menu.module.scss
    |    |    ├── Dropdown
    |    |    |   ├── Dropdown.jsx
    |    |    |   ├── MenuItems.js
    |    |    |   ├── index.js
    |    |    |   ├── dropdown.module.scss
    |    |    ├── ScrollToTop
    |    |    |   ├── LoggedIn.jsx
    |    |    |   ├── index.js
    |    |    |   ├── logged-in.module.scss
    |    |    ├── Toast
    |    |    |   ├── Toast.jsx
    |    |    |   ├── index.js
    |    |    |   ├── toast.module.scss
    |    |    ├── Toggle
    |    |    |   ├── Toggle.jsx
    |    |    |   ├── index.js
    |    |    |   ├── toggle.module.scss
    ├── config
    |    ├── api.js
    ├── fonts
    |    ├── FuturaLT-Book.ttf
    ├── test_data
    |    ├── broadcast_text.json
    |    ├── superadmin_credential.json
    |    ├── team-roles.json
    |    ├── teams.js
    ├── store
    |    ├── actions
    |    |    ├── actions.js
    |    |    ├── auth.js
    |    ├── reducers
    |    |    ├── reducer.js
    ├── pages
    │    ├── Home
    |    │    ├── components
    |    |    |    ├── JoinUs
    |    |    |    |   ├── JoinUs.jsx
    |    |    |    |   ├── index.js
    |    |    |    |   ├── join-us.module.scss
    |    |    |    ├── Carousel
    |    |    |    |   ├── Carousel.jsx
    |    |    |    |   ├── index.js
    |    |    |    |   ├── carousel.module.scss
    |    |    |    |   ├── owl-carousel-styles.css
    |    |    |    ├── JoinUsForm
    |    |    |    |   ├── Form.jsx
    |    |    |    |   ├── index.js
    |    |    |    |   ├── form.module.scss
    |    |    |    ├── Motive
    |    |    |    |   ├── Motive.jsx
    |    |    |    |   ├── index.js
    |    |    |    |   ├── motive.module.scss
    |    |    |    ├── Overview
    |    |    |    |   ├── Overview.jsx
    |    |    |    |   ├── index.js
    |    |    |    |   ├── overview.module.scss
    |    │    ├── index.js
    |    │    ├── Home.jsx
    |    │    └── home.module.scss
    |    │    └── nav-icon.png
    │    ├── About
    │    │    ├── About.jsx
    │    │    ├── index.js
    │    │    ├── about.module.scss
    │    │    ├── about.scss
    │    └── Login
    │    |     ├── Login.jsx
    │    |     ├── index.js
    │    |     └── login.module.scss
    │    ├── Admin
    |    │    ├── components
    |    |    |    ├── JoinUs
    |    |    |    |   ├── JoinUs.jsx
    |    |    |    |   ├── index.js
    |    |    |    ├── About
    |    |    |    |   ├── About.jsx
    |    |    |    |   ├── index.js
    |    |    |    |   ├── about-us.module.scss
    |    |    |    ├── AddTeamMember
    |    |    |    |   ├── AddTeamMember.jsx
    |    |    |    |   ├── index.js
    |    |    |    |   ├── add-team-member.module.scss
    |    |    |    ├── Broadcast
    |    |    |    |   ├── AddBroadcasts
    |    |    |    |   |   ├── AddBroadcasts.jsx
    |    |    |    |   |   ├── index.js
    |    |    |    |   |   ├── add-broadcasts.module.scss
    |    |    |    |   ├── Broadcast.jsx
    |    |    |    |   ├── index.js
    |    |    |    |   ├── broadcast.module.scss
    |    |    |    ├── Contact
    |    |    |    |   ├── Contact.jsx
    |    |    |    |   ├── index.js
    |    |    |    ├── Dashboard
    |    |    |    |   ├── Dashboard.jsx
    |    |    |    |   ├── index.js
    |    |    |    |   ├── dashboard.module.scss
    |    |    |    ├── Faq
    |    |    |    |   ├── AddFaq
    |    |    |    |   |   ├── AddFaq.jsx
    |    |    |    |   |   ├── index.js
    |    |    |    |   |   ├── add-faq.module.scss
    |    |    |    |   ├── Faq.jsx
    |    |    |    |   ├── index.js
    |    |    |    |   ├── faq.module.scss
    |    |    |    ├── MangeTeams
    |    |    |    |   ├── ManageTeams.jsx
    |    |    |    |   ├── index.js
    |    |    |    |   ├── manage-teams.module.scss
    |    |    |    ├── Profile
    |    |    |    |   ├── Profile.jsx
    |    |    |    |   ├── index.js
    |    |    |    |   ├── profile.module.scss
    |    |    |    ├── Resource
    |    |    |    |   ├── Resource.jsx
    |    |    |    |   ├── index.js\
    |    |    |    ├── Setting
    |    |    |    |   ├── Invite
    |    |    |    |   |   ├── Invite.jsx
    |    |    |    |   |   ├── index.js

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [HITK-TECH-Community/Community-Website](https://github.com/HITK-TECH-Community/Community-Website) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
