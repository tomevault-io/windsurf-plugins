---
trigger: always_on
description: You are an agent tasked with helping the user build a new application.
---


# User Coding and Style preferences

You are an agent tasked with helping the user build a new application.

## 📏 Tools and Rules

This is a set of rules and conventions you should always follow - check to see if you are already working with these set up (e.g. in a monorepo)

- Prefered languages and tools:
    - `proto` and `moon` for monorepo dependency management
    - TypeScript for all Frontend and Backend applications
    - Vanilla CSS - no frameworks or additional tooling is needed for styling
    - Node, JavaScript React apps with ESM Module style code
    - Clean and portable bash code for shell scripts and other utilities (especially where installing a lot of dependencies via npm would not make sense )
    - For Frontend, unless asked use React as the frontend framework - set up with vite and vitest.  Here are the versions you should use:
```
     "dependencies": {
        "react": "18.3.1",
        "react-dom": "18.3.1",
        "react-router-dom": "7.1.5"
     }
     "devDependencies": {
            "@testing-library/jest-dom": "6.6.3",
            "@testing-library/react": "16.2.0",
            "@testing-library/user-event": "14.6.1",
            "@types/react": "18.3.18",
            "@types/react-dom": "18.3.5",
            "@vitejs/plugin-react": "4.3.4",
            "happy-dom": "16.8.1",
            "typescript": "5.7.3",
            "vite": "6.0.11",
            "vitest": "3.0.5"
     }
```
    - For Backend, unless asked used expressjs. Here are the versions you should use:
```
    "dependencies": {
        "body-parser": "1.20.3",
        "cors": "2.8.5",
        "express": "4.21.2",
        "express-rate-limit": "7.5.0"
    }
     "devDependencies": {
        "@types/body-parser": "1.19.5",
        "@types/cors": "2.8.17"
        "@types/express": "4.17.21",
        "@types/node": "22.13.10",
     }
```

## ✋ Your Behaviour

These are things you should do at all times to meet the users needs

- Do not hallucinate
- Do not ruminate or overexplain a feature - when building someting, keep it concise and neat and keep to a small number of features, do not keep adding more
- Loop no more than 3 times on a problem before asking for the users help, explain the problem and show them your suggested changes
- Avoid changing settings values like urls or names. For example if you set a port number, and the user changes it - assume the user knows what they are doing
- Be sure a dependency exists like library on npm exists - don't dream one up
- Where possible, when you do something complex you should create a markdown file to explain it
- Use mermaid diagrams to visualise the concept with a complete description


## 👍 Code preferences

- Always use 2 spaces for intentation of any code format
- Avoid rewriting configuration values, especially if a user has changed them 
- Always prefer smaller composable files over larger ones, but try keep the files logical and tidy.  If you see existing code not following these conventions feel free to refactor it. When writing and modularising code:

    - Keep files to a maximum of 300-400 lines of code - If you do end up writing a large file, immediatly split it down.
    - Focus on moving functions that can easily be put in a utility or library file (for example long handler functions in React components, or complex processing of data)
    - Break down large pieces of HTML or JSX into smaller leaf components that have boundaries of only taking data or return data via props (these components can have functionality but should be isolated and testable)
    - Take a Model-View-Controller approach to frontend components, breaking down routes and views into smaller parts with a controller, and connecting with a datasource model

## 🧪 Testing

- When writing a file, it should be accompanied by a test file in the same folder:  For example where a code file is <filename>.ts the unit test should always be <filename>.spec.ts

    - Write unit tests for vitest, both for backend and frontend code.
    - For frontend apps:
        - Use Playwrite for user story and integration testing
        - Tests should be practical UI tests or complex features of the app
        - Avoid testing DOM or library features
        - If a unit test requires JSDOM, instead write it to run in Playwrite
    - For backend apps:
        - Use Playwrite for automated testing for API test cases
        - Tests should be practical 

## 🎁 Monorepo Approach

- Check to see if you are already working in a monorepo

## `proto`

- Use `proto` for any repository that is set up, this sets up a local copy of system dependencies like `node` and `npm` for this repo only, giving consistancy.  Here is a starter `.protofile`:

```
jq = "1.7.1"
moon = "1.31.2"
node = "22.13.0"
npm = "11.0.0"

[plugins]
jq = "https://raw.githubusercontent.com/appthrust/proto-toml-plugins/main/jq/plugin.toml"
moon = "https://raw.githubusercontent.com/moonrepo/moon/master/proto-plugin.toml"
```

- Once created, run `proto use`

## `moon`

- Use `moon` for repo management and ensure you add a `moon.yml` file to each project, the stucture should be:

    - apps: Applications that have a frontend component
    - packages: libraries and other features
    - services: backend services that can be containerised

```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tanepiper/teskooano](https://github.com/tanepiper/teskooano) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-21 -->
