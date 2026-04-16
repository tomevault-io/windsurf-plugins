---
trigger: always_on
description: StudyMate AI is an intelligent learning platform for UIT students, combining AI-powered assistance with comprehensive course management. The project emphasizes gamification, progress tracking, and personalized learning experiences similar to Duolingo.
---

# StudyMate AI - Cursor Rules
# Inspired by Duolingo's gamification and learning experience principles

## Project Overview
StudyMate AI is an intelligent learning platform for UIT students, combining AI-powered assistance with comprehensive course management. The project emphasizes gamification, progress tracking, and personalized learning experiences similar to Duolingo.

## Core Principles (Duolingo-Inspired)

### 1. Gamification & Engagement
- **Streak System**: Track daily learning streaks to encourage consistent study habits
- **Achievements & Badges**: Reward users for milestones (courses completed, perfect scores, daily goals)
- **Progress Visualization**: Clear progress bars, completion percentages, and visual feedback
- **Points & XP**: Award experience points for completing lessons, quizzes, and activities
- **Leaderboards**: Optional competitive elements for motivation (respecting privacy)

### 2. Learning Experience Design
- **Bite-sized Lessons**: Break content into small, digestible chunks (5-15 minutes)
- **Spaced Repetition**: Implement review cycles to reinforce learning
- **Adaptive Difficulty**: Adjust content difficulty based on user performance
- **Immediate Feedback**: Provide instant feedback on answers and progress
- **Practice Mode**: Allow users to review and practice previous content

### 3. User Interface & UX
- **Clean, Minimal Design**: Focus on content, reduce cognitive load
- **Mobile-First**: Optimize for mobile devices (primary learning platform)
- **Accessibility**: WCAG 2.1 AA compliance, keyboard navigation, screen reader support
- **Fast Loading**: Optimize assets, lazy loading, efficient database queries
- **Offline Capability**: Cache content for offline learning when possible

### 4. AI Integration Best Practices
- **Contextual Help**: AI assistant provides hints, explanations, not direct answers
- **Personalized Recommendations**: Suggest courses/content based on learning history
- **Learning Path Optimization**: AI suggests optimal learning sequences
- **Question Generation**: AI creates practice questions based on user weaknesses
- **Transparency**: Always indicate when AI is providing assistance

## Code Standards

### Backend (Node.js/Express) - MVC Pattern
```javascript
// Routes: Define endpoints and delegate to controllers
const controller = require('../controllers/controllerName');
router.get('/path', controller.methodName);

// Controllers: Handle business logic
const { applicationLogger } = require('../config/logger');

exports.methodName = async (req, res) => {
  try {
    // Use async/await for all database operations
    // Always handle errors with try-catch
    // Validate input data before processing
    // Use Sequelize models with proper associations
    // Implement proper error logging to Kibana
    const data = await Model.findAll();
    res.render('view', { data });
  } catch (error) {
    applicationLogger.error('Error in methodName', error, {
      type: 'controller',
      operation: 'methodName',
      userId: req.user?.id,
      path: req.path
    });
    req.flash('error', 'User-friendly error message');
    res.redirect('/fallback');
  }
};
```

### Frontend (EJS/Tailwind)
```html
<!-- Use semantic HTML5 elements -->
<!-- Implement responsive design with Tailwind utilities -->
<!-- Add loading states for async operations -->
<!-- Provide clear error messages -->
<!-- Use progressive enhancement -->
```

### Database (PostgreSQL/Sequelize)
- Use UUIDs for primary keys
- Implement soft deletes where appropriate
- Add indexes for frequently queried fields
- Use transactions for multi-step operations
- Normalize data structure but optimize for read performance

### API Design
- RESTful endpoints with clear naming
- Consistent response format: `{ success: boolean, message: string, data: object }`
- Proper HTTP status codes
- Rate limiting for public endpoints
- Authentication via JWT or session

## File Structure Conventions

```
/models          - Sequelize models with associations
/routes          - Express route definitions (thin layer, delegates to controllers)
/controllers     - Business logic and request handling (MVC pattern)
  /admin         - Admin-specific controllers
    - adminCourseController.js
    - adminCategoryController.js
    - adminContactController.js
    - adminUserController.js
    - adminStatisticsController.js
  - courseController.js
  - authController.js
  - dashboardController.js
  - profileController.js
  - chatController.js
  - contentController.js
  - fileController.js
  - userController.js
  - statisticsController.js
  - infoController.js
  - aiController.js
/middleware      - Authentication, validation, error handling
/services        - External services (AI, email, etc.)
/views           - EJS templates
  /pages         - Full page templates
  /partials      - Reusable components
/public          - Static assets
  /css           - Custom styles
  /js            - Client-side scripts
  /images        - Images and assets
/config          - Configuration files
/utils           - Helper functions
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/HieuNM2809) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
