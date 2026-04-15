---
trigger: always_on
description: You are working with the CoreUI Free React Admin Template, a professional admin dashboard built with React 19, CoreUI React components, and modern build tools. This project uses Vite for development and building, React Router for navigation, Redux for state management, and Sass for styling.
---

# CoreUI Free React Admin Template - AI Assistant Rules

You are working with the CoreUI Free React Admin Template, a professional admin dashboard built with React 19, CoreUI React components, and modern build tools. This project uses Vite for development and building, React Router for navigation, Redux for state management, and Sass for styling.

## Critical Rules

**Component Library**: ALWAYS use CoreUI React components from https://coreui.io/react/docs/. NEVER use Tailwind CSS, Material-UI, or other component libraries. This project is built on Bootstrap 5 and CoreUI React components exclusively.

**Technology Stack**: This project uses:
- React 19 with functional components and Hooks
- JSX for component markup
- CoreUI React 5.x and @coreui/coreui 5.x
- React Router 7.x for client-side routing
- Redux 5.x with React-Redux for state management
- Vite 8.x for development server and building
- Sass/SCSS for styling with Bootstrap 5 variables
- Chart.js 4.x with @coreui/react-chartjs for data visualization

## Code Conventions

**JavaScript/React Standards**:
- Use functional components with Hooks (useState, useEffect, useSelector, etc.)
- Follow React Hooks rules (only call at top level, only in React functions)
- Use Prettier formatting: no semicolons, single quotes, 2-space indentation
- Enforce ESLint rules with React and Prettier plugins
- Use PropTypes for component prop validation
- Prefer const arrow functions: `const Component = () => { }`
- Use destructuring for props: `const { prop1, prop2 } = props`

**File Organization**:
- `src/` - All source code
  - `components/` - Reusable UI components (AppHeader, AppSidebar, etc.)
  - `views/` - Page components organized by feature (dashboard, forms, charts, etc.)
  - `layout/` - Layout wrapper components (DefaultLayout)
  - `assets/` - Static assets (images, brand logos)
  - `scss/` - Global styles and theme customization
  - `routes.js` - Route definitions
  - `_nav.js` - Navigation/sidebar menu configuration
  - `store.js` - Redux store configuration
  - `App.js` - Main application component with routing
  - `index.js` - Application entry point

**React/JSX Practices**:
- Use React.lazy() for code splitting and lazy loading
- Wrap lazy components in <Suspense> with fallback UI
- Use semantic component names (e.g., DefaultLayout, AppHeader)
- Keep components focused and single-responsibility
- Extract reusable logic into custom hooks when appropriate
- Use React.memo() for performance optimization when needed

**CSS/Sass Practices**:
- Import global styles in App.js: `import './scss/style.scss'`
- Use Bootstrap utilities first before custom CSS
- Leverage CoreUI CSS custom properties for theming
- Support dark mode through CoreUI's color mode system
- File: `src/scss/style.scss` - main stylesheet importing CoreUI and Bootstrap
- File: `src/scss/_custom.scss` - custom style overrides
- Use SCSS variables from Bootstrap and CoreUI when possible

**Routing Conventions**:
- Use HashRouter for client-side routing (GitHub Pages compatible)
- Define routes in routes.js as array of objects
- Lazy load route components for better performance
- Use exact path matching where needed
- Public routes (login, register, 404, 500) defined in App.js
- Protected routes handled in DefaultLayout with AppContent

**State Management**:
- Use Redux for global state (theme, sidebar visibility)
- Use Redux Toolkit pattern with actions and reducers
- Connect components with useSelector and useDispatch hooks
- Keep component-level state in useState when state is local
- File: `src/store.js` - Redux store configuration

**Naming Conventions**:
- PascalCase for component files and component names (AppHeader.js, DefaultLayout.js)
- camelCase for variables, functions, and hooks (useState, useEffect)
- UPPER_SNAKE_CASE for constants (API_URL, MAX_ITEMS)
- kebab-case for CSS classes (following Bootstrap/CoreUI conventions)
- Descriptive names that indicate purpose (AppHeaderDropdown vs Dropdown)

## Project Structure

```
coreui-free-react-admin-template/
├── public/              # Static assets served directly
├── src/
│   ├── assets/         # Images, logos, icons
│   │   ├── brand/     # Logo components
│   │   └── images/    # Image files
│   ├── components/     # Reusable UI components
│   │   ├── AppHeader.js
│   │   ├── AppSidebar.js
│   │   ├── AppFooter.js
│   │   ├── AppContent.js
│   │   ├── AppBreadcrumb.js
│   │   └── header/    # Header sub-components
│   ├── layout/         # Layout components
│   │   └── DefaultLayout.js
│   ├── views/          # Page components
│   │   ├── dashboard/ # Dashboard page
│   │   ├── base/      # Base UI components examples
│   │   ├── buttons/   # Button examples
│   │   ├── forms/     # Form examples
│   │   ├── charts/    # Chart examples
│   │   ├── icons/     # Icon examples
│   │   ├── notifications/ # Notification examples
│   │   ├── widgets/   # Widget examples
│   │   └── pages/     # Auth & error pages
│   ├── scss/           # Stylesheets
│   │   ├── style.scss # Main stylesheet

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/coreui) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
