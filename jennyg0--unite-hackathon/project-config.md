---
trigger: always_on
description: Building a DeFi savings application that abstracts crypto complexity while using 1inch API extensively for the hackathon. Target: crypto-native users who don't use DeFi.
---

# DeFi Savings App - Cursor Rules

## 🎯 Project Overview
Building a DeFi savings application that abstracts crypto complexity while using 1inch API extensively for the hackathon. Target: crypto-native users who don't use DeFi.

## 🏗 Architecture Guidelines

### Tech Stack
- **Frontend**: Next.js 14 with TypeScript
- **Styling**: Tailwind CSS with custom design system
- **Wallet**: RainbowKit + Wagmi
- **APIs**: 1inch API (primary), Base network
- **Animations**: Framer Motion
- **Icons**: Lucide React

### Design Principles
- **Modern & Clean**: Don't look like typical crypto apps
- **User-Friendly**: Abstract away crypto complexity
- **Mobile-First**: Responsive design
- **Educational**: Help users learn while they save

## 📝 Git History Rules

### Commit Structure
- **Prefix commits** with type: `feat:`, `fix:`, `docs:`, `refactor:`, `style:`, `test:`
- **Clear descriptions**: What was changed and why
- **Atomic commits**: One logical change per commit
- **No large commits**: Keep changes focused and reviewable

### Branch Strategy
- `main`: Production-ready code
- `develop`: Integration branch
- `feature/*`: New features
- `fix/*`: Bug fixes

### Commit Examples
```
feat: Add 1inch API integration for token swaps
feat: Implement financial freedom calculator
fix: Resolve TypeScript errors in TokenSwap component
refactor: Extract wallet connection logic to custom hook
docs: Update README with setup instructions
```

## 🧩 Component Guidelines

### File Structure
```
components/
├── ui/           # Reusable UI components
├── forms/        # Form components
├── layout/       # Layout components
└── features/     # Feature-specific components
```

### Component Standards
- **TypeScript**: Strict typing for all components
- **Props Interface**: Define clear prop interfaces
- **Error Handling**: Graceful error states
- **Loading States**: Show loading indicators
- **Accessibility**: ARIA labels, keyboard navigation

### Naming Conventions
- **Components**: PascalCase (`FinancialFreedomCalculator`)
- **Files**: PascalCase for components, camelCase for utilities
- **Variables**: camelCase
- **Constants**: UPPER_SNAKE_CASE
- **Types**: PascalCase with descriptive names

## 🔧 Code Quality Standards

### TypeScript
- **Strict mode**: Enable all strict options
- **No any types**: Use proper typing
- **Interface over type**: Prefer interfaces for objects
- **Generic types**: Use when appropriate

### Error Handling
- **Try-catch blocks**: For async operations
- **User-friendly messages**: Don't expose technical errors
- **Fallback UI**: Graceful degradation
- **Error boundaries**: React error boundaries

### Performance
- **Lazy loading**: For heavy components
- **Memoization**: Use React.memo and useMemo
- **Bundle size**: Keep dependencies minimal
- **Image optimization**: Use Next.js Image component

## 🎨 Styling Guidelines

### Tailwind CSS
- **Custom classes**: Use component classes for consistency
- **Responsive design**: Mobile-first approach
- **Dark mode**: Consider future dark mode support
- **Design tokens**: Use consistent colors and spacing

### Component Classes
```css
.btn-primary: Primary action buttons
.btn-secondary: Secondary action buttons
.card: Standard card containers
.input-field: Form input fields
```

## 🔌 API Integration

### 1inch API Usage
- **Extensive integration**: Use as many endpoints as possible
- **Error handling**: Graceful API error handling
- **Rate limiting**: Respect API limits
- **Caching**: Cache responses when appropriate

### Network Support
- **Primary**: Base mainnet (cost-effective)
- **Secondary**: Ethereum mainnet (if needed)
- **Testnets**: Only for development

## 🧪 Testing Guidelines

### Test Structure
- **Unit tests**: For utility functions
- **Component tests**: For React components
- **Integration tests**: For API interactions
- **E2E tests**: For critical user flows

### Testing Standards
- **Coverage**: Aim for 80%+ coverage
- **Mocking**: Mock external dependencies
- **User scenarios**: Test real user flows
- **Error cases**: Test error conditions

## 📚 Documentation

### Code Documentation
- **JSDoc**: For complex functions
- **README**: Comprehensive setup guide
- **API docs**: Document custom hooks and utilities
- **Component docs**: Document component props

### User Documentation
- **Onboarding**: Clear user guides
- **Educational content**: Built into the app
- **Help system**: Contextual help

## 🚀 Deployment

### Environment Variables
- **Required**: 1inch API key
- **Optional**: WalletConnect project ID
- **Security**: Never commit API keys

### Build Process
- **Type checking**: Run before build
- **Linting**: ESLint with strict rules
- **Optimization**: Next.js optimizations
- **Bundle analysis**: Monitor bundle size

## 🔒 Security

### Best Practices
- **API keys**: Environment variables only
- **Input validation**: Validate all user inputs
- **XSS prevention**: Sanitize user content
- **CORS**: Proper CORS configuration

### Wallet Security
- **Connection**: Secure wallet connections
- **Transaction signing**: Clear transaction details
- **Error handling**: Secure error messages

## 🎯 Feature Priorities

### MVP (Current)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jennyg0) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
