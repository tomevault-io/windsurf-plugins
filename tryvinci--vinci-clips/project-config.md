---
trigger: always_on
description: The Clips is an AI-powered video clipping tool built on Google Cloud Platform (GCP) using a REST API server (Node.js/Express), Next.js for the frontend, MongoDB for data storage, and Google Gemini API (Flash 2.0) for LLM-based analysis. These rules ensure consistent, high-quality development, rapid iteration, and alignment with the Product Requirements Document (@PRD.md).
---

# Clips Development Rules

## Project Overview
The Clips is an AI-powered video clipping tool built on Google Cloud Platform (GCP) using a REST API server (Node.js/Express), Next.js for the frontend, MongoDB for data storage, and Google Gemini API (Flash 2.0) for LLM-based analysis. These rules ensure consistent, high-quality development, rapid iteration, and alignment with the Product Requirements Document (@PRD.md).

## Code Structure and Style
1. **File Organization**:
   - Organize the project into clear directories: `/frontend` (Next.js), `/backend` (Express API), and `/docs` (documentation).
   - Use a monorepo structure with subdirectories for each service (e.g., `/frontend/pages`, `/backend/routes`).
   - Example:
     ```
     /clips
       /frontend
         /pages
         /components
         /styles
       /backend
         /routes
         /controllers
         /models
       /docs
         /api
         /guides
     ```

2. **Naming Conventions**:
   - Use camelCase for JavaScript variables and functions (e.g., `processVideo`, `uploadHandler`).
   - Use PascalCase for React components (e.g., `VideoUpload`, `ClipPreview`).
   - Prefix API routes with `/clips` (e.g., `/clips/process-video`).

4. **File Paths**:
    - All frontend code must be placed in the `frontend/src` directory.
    - All file paths in the frontend code must be relative to the `frontend` directory. For example, to import a component from `frontend/src/components/ui/button.tsx`, use the path ` "@/components/ui/button"`.

3. **Code Style**:
   - Enforce ESLint with Airbnb style guide for JavaScript/Node.js.
   - Use Prettier for consistent formatting across JavaScript, TypeScript, and CSS.
   - Example `.eslintrc.json`:
     ```json
     {
       "extends": "airbnb",
       "rules": {
         "no-console": "warn",
         "react/prop-types": "off"
       }
     }
     ```

## Version Control
1. **Git Workflow**:
   - Use Git with a feature-branch workflow: `main` for production, `develop` for integration, and feature branches (e.g., `feature/video-upload`).
   - Prefix branch names: `feature/`, `bugfix/`, `hotfix/`.
   - Commit messages follow the format: `[type](scope): description` (e.g., `feat(upload): add video upload endpoint`).
     - Types: `feat`, `fix`, `docs`, `style`, `refactor`, `test`, `chore`.

2. **Pull Requests**:
   - Require at least one reviewer for PRs.
   - Include a description linking to the @PRD.md feature or issue.
   - Run CI checks (linting, tests) before merging.
   - Example PR template:
     ```
     **Description**: Implements video upload API as per @PRD.md section 2.1.
     **Changes**: Added /clips/process-video endpoint, integrated GCP Cloud Storage.
     **Related Issue**: #123
     ```

## Development Practices
1. **Modularity**:
   - Write reusable React components in `/frontend/components` (e.g., `VideoPlayer.jsx`, `UploadDropzone.jsx`).
   - Separate backend logic into controllers, services, and models (e.g., `/backend/controllers/videoController.js`).

2. **Type Safety**:
   - Use TypeScript for Next.js and Express to enforce type safety.
   - Define interfaces for API payloads and MongoDB schemas (e.g., `IVideo`, `IClip`).
   - Example:
     ```typescript
     interface IVideo {
       userId: string;
       filePath: string;
       transcript: { start: number; end: number; text: string }[];
     }
     ```

3. **Error Handling**:
   - Implement global error middleware in Express for consistent API responses.
   - Use try-catch blocks for async operations (e.g., Gemini API calls, FFmpeg processing).
   - Example:
     ```javascript
     app.use((err, req, res, next) => {
       res.status(500).json({ error: err.message });
     });
     ```

4. **Rule Updates**:
   - Any modifications or additions to these rules must be clearly documented in a pull request and reviewed by the team.
   - Update rules everytime a prompt is made by the user suggesting a change in approach

## Testing
1. **Unit Tests**:
   - Write unit tests for backend services using Jest (e.g., test `videoController` functions).
   - Test frontend components with React Testing Library.
   - Aim for >80% code coverage.

2. **Integration Tests**:
   - Test API endpoints with Supertest (e.g., `/clips/process-video`).
   - Mock GCP services (Cloud Storage, Gemini API) using `@google-cloud/google-cloud-sdk` stubs or similar.
   - Example:
     ```javascript
     it('processes video and returns clips', async () => {
       const res = await request(app).post('/clips/process-video').send({ videoPath: 'test.mp4' });
       expect(res.status).toBe(200);
       expect(res.body.clips).toBeDefined();
     });
     ```

3. **End-to-End Tests**:
   - Use Cypress for frontend E2E testing (e.g., video upload flow, clip preview).
   - Test full workflow: upload → transcription → clipping → download.

## API Integration
1. **Gemini API (Flash 2.0)**:
   - Use `@google/generative-ai` SDK for Gemini integration.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tryvinci/vinci-clips](https://github.com/tryvinci/vinci-clips) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
