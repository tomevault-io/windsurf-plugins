---
trigger: always_on
description: SnapURL is a full-stack URL shortener web application with React.js frontend, Node.js/Express.js backend, MongoDB database, and a Chrome extension. The application provides URL shortening, user authentication, analytics tracking, and API documentation.
---

# SnapURL: URL Shortener Application

SnapURL is a full-stack URL shortener web application with React.js frontend, Node.js/Express.js backend, MongoDB database, and a Chrome extension. The application provides URL shortening, user authentication, analytics tracking, and API documentation.

Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.

## Working Effectively

### Bootstrap and Install Dependencies
- Install all dependencies (NEVER CANCEL: Takes 2-3 minutes):
  ```bash
  npm install
  ```
  This command automatically installs both frontend and backend dependencies via the root package.json script.

### Environment Setup
- Create backend environment file:
  ```bash
  cd backend
  ```
  Create `.env` file with the following variables:
  ```bash
  DB_URL=<your-mongodb-database-url>
  JWT_SECRET=<your-jwt-secret>
  SESSION_SECRET=<your-session-secret>
  PORT=4001
  BASE_URL=http://localhost:4001
  SHORT_URL_PREFIX=http://localhost:4001/u
  EMAIL_HOST=<your-email-host>
  EMAIL_PORT=<your-email-port>
  EMAIL_HOST_USER=<your-email-host-user>
  EMAIL_HOST_PASSWORD=<your-email-host-password>
  FRONTEND_URL=http://localhost:5173
  ```

- Create frontend environment file:
  ```bash
  cd frontend
  ```
  Create `.env` file with:
  ```bash
  VITE_API_ENDPOINT=http://localhost:4001
  ```

### Build and Development
- Build frontend (NEVER CANCEL: Takes 6-10 seconds, set timeout to 60+ seconds):
  ```bash
  cd frontend && npm run build
  ```

- Start development servers from root directory (NEVER CANCEL: Can take 30+ seconds to fully initialize both servers):
  ```bash
  npm start
  ```
  This starts both backend (port 4001) and frontend (port 5173) in parallel using npm-run-all.

- Start backend only (requires MongoDB connection):
  ```bash
  cd backend && npm start
  ```

- Start frontend only (starts in ~1 second):
  ```bash
  cd frontend && npm start
  # or alternatively:
  cd frontend && npm run dev
  ```

### Testing
- Run backend tests (requires MongoDB connection):
  ```bash
  cd backend && npm test
  ```
  **Note**: Tests will fail without proper environment setup including MongoDB database.

- Backend tests use Mocha with Chai and test URL shortening functionality.

### Code Quality
- Format backend code:
  ```bash
  cd backend && npm run format
  ```

- Format frontend code:
  ```bash
  cd frontend && npm run format
  ```

- Lint frontend code:
  ```bash
  cd frontend && npm run lint
  ```

- **CRITICAL**: Always run `npm run format` in both backend and frontend directories before committing or the CI (.github/workflows/autoFormatCode.yml) will fail.

## Validation

### Manual Testing Scenarios
After making changes, ALWAYS test these complete end-to-end scenarios:

1. **URL Shortening Flow**:
   - Navigate to http://localhost:5173
   - Enter a valid URL in the shortening form
   - Verify a shortened URL is generated
   - Test that the shortened URL redirects to the original URL

2. **User Authentication Flow**:
   - Test user registration at /signup
   - Test user login at /login
   - Verify authenticated users can see their URL history
   - Test logout functionality

3. **API Documentation**:
   - Navigate to http://localhost:4001/doc
   - Verify Swagger UI loads and displays API endpoints

4. **Chrome Extension** (manual validation only):
   - Load the chrome-extension directory as an unpacked extension
   - Test URL shortening from the extension popup

### Build Validation
- Always run formatting commands before committing changes
- Frontend linting may have warnings - fix only new issues you introduce
- Backend has no separate linting but uses Prettier formatting

## Tech Stack and Architecture

### Backend (Node.js/Express.js)
- **Entry Point**: `backend/index.js`
- **Database**: MongoDB with Mongoose ODM
- **Authentication**: JWT with Passport.js
- **API Documentation**: Swagger UI at `/doc` endpoint
- **Rate Limiting**: Express-rate-limit configured
- **Email**: Nodemailer for transactional emails
- **Testing**: Mocha with Chai and Chai-HTTP

### Frontend (React.js)
- **Build Tool**: Vite for fast development and building
- **Routing**: React Router DOM
- **Styling**: React Bootstrap + Material-UI components
- **HTTP Client**: Axios for API calls
- **State Management**: React Context API
- **Notifications**: React Toastify

### Chrome Extension
- **Manifest Version**: 3
- **Files**: Located in `chrome-extension/` directory
- **Permissions**: activeTab only
- **No build process**: Ready to load as unpacked extension

### Database Schema
- **Users**: Authentication and profile data
- **URLs**: Original URLs, shortened URLs, analytics
- **Tokens**: Password reset and email verification
- **Feedback**: User feedback and reviews

## Common Tasks

### Frequently Used Commands
```bash
# Install all dependencies
npm install

# Start development (both frontend and backend)
npm start

# Format all code before committing
cd backend && npm run format && cd ../frontend && npm run format


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DhananjayThomble/URL-Shortener-App](https://github.com/DhananjayThomble/URL-Shortener-App) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
