---
trigger: always_on
description: This is a comprehensive **cryptocurrency trading platform** built with Node.js, Express, MongoDB, and Socket.IO. The platform provides features for crypto trading, mining, P2P transactions, wallet management, and user authentication with KYC support.
---

# GitHub Copilot Instructions for Crypto Trading Platform

## Project Overview
This is a comprehensive **cryptocurrency trading platform** built with Node.js, Express, MongoDB, and Socket.IO. The platform provides features for crypto trading, mining, P2P transactions, wallet management, and user authentication with KYC support.

## Technology Stack
- **Backend Framework**: Express.js (v4.18.2)
- **Database**: MongoDB with Mongoose ODM (v7.5.0)
- **Real-time Communication**: Socket.IO (v4.7.2)
- **Authentication**: JWT (JSON Web Tokens) with bcryptjs for password hashing
- **Validation**: express-validator (v7.0.1)
- **Security**: Helmet, CORS, express-rate-limit
- **File Uploads**: Multer (v2.0.2)
- **Email**: Nodemailer (v7.0.10)

## Project Structure
```
/
├── server.js              # Main application entry point
├── config/
│   └── initAdmin.js       # Admin user initialization
├── middleware/
│   └── auth.js            # Authentication middleware (protect, authorize, isAdmin, requireKYC)
├── models/
│   ├── User.js            # User schema with authentication
│   ├── Wallet.js          # Wallet schema with balance management
│   ├── Mining.js          # Mining operations
│   ├── Trade.js           # Trading records
│   ├── P2PTrade.js        # Peer-to-peer trading
│   └── Transaction.js     # Transaction history
├── routes/
│   ├── auth.js            # Authentication endpoints (register, login, profile)
│   ├── users.js           # User management
│   ├── admin.js           # Admin operations
│   ├── wallet.js          # Wallet operations
│   ├── mining.js          # Mining endpoints
│   ├── trading.js         # Trading endpoints
│   ├── p2p.js             # P2P trading endpoints
│   └── transactions.js    # Transaction history
└── utils/
    └── jwt.js             # JWT utilities (generateToken, sendTokenResponse)
```

## Coding Standards and Patterns

### 1. Route Structure
All routes follow a consistent pattern with JSDoc-style comments:
```javascript
// @route   METHOD /api/endpoint
// @desc    Description of what the endpoint does
// @access  Public/Private/Admin
router.method('/path', middleware, async (req, res) => {
  try {
    // Implementation
  } catch (error) {
    res.status(500).json({
      success: false,
      message: 'Error message',
      error: error.message
    });
  }
});
```

### 2. Response Format
All API responses follow a consistent structure:
```javascript
// Success response
{
  success: true,
  message: 'Operation successful',
  data: { /* response data */ }
}

// Error response
{
  success: false,
  message: 'Error description',
  error: error.message,  // Optional, only in development
  errors: []             // For validation errors
}
```

### 3. Mongoose Models
- Use `mongoose.Schema` with validation rules
- Include `timestamps: true` for createdAt/updatedAt
- Implement pre-save hooks for data transformation (e.g., password hashing)
- Add custom methods to schema for common operations
- Override `toJSON` to exclude sensitive fields

### 4. Authentication Middleware
- `protect`: Verifies JWT token, requires Bearer token in Authorization header
- `authorize(...roles)`: Role-based access control
- `isAdmin`: Checks for admin role
- `requireKYC`: Ensures user has approved KYC status

### 5. Validation
Use `express-validator` for input validation:
```javascript
[
  body('field').validation().withMessage('Error message'),
  body('field2').custom((value, { req }) => {
    // Custom validation
    return true;
  })
]
```

### 6. User Model Features
- Password hashing with bcrypt (10 salt rounds)
- Automatic referral code generation
- KYC status tracking: 'not_submitted', 'pending', 'approved', 'rejected'
- User roles: 'user', 'admin'
- Two-factor authentication support (twoFactorEnabled, twoFactorSecret)
- Login history tracking

### 7. Wallet System
- Multi-currency support (BTC, ETH, USDT, etc.)
- Each balance has: `available`, `locked`, `total` fields
- `available`: funds available for use
- `locked`: funds locked in pending transactions
- `total`: calculated as available + locked
- Methods: `getBalance(currency)`, `updateBalance(currency, amount, type)`

### 8. Security Practices
- Never send password in responses (use `select: false` in schema)
- Always use JWT for protected routes
- Check user `isActive` status before allowing access
- Implement rate limiting (100 requests per 15 minutes per IP)
- Use Helmet for security headers
- Validate all user inputs

### 9. Error Handling
- Always wrap async operations in try-catch blocks
- Log errors to console for debugging
- Return appropriate HTTP status codes:
  - 200: Success
  - 201: Created
  - 400: Bad Request (validation errors)
  - 401: Unauthorized (authentication failed)
  - 403: Forbidden (insufficient permissions)
  - 404: Not Found
  - 500: Internal Server Error

### 10. Socket.IO Integration
- Socket.IO instance is available via `req.app.get('io')`
- Use for real-time updates (price changes, trade notifications, etc.)
- Client connects from `CLIENT_URL` environment variable

## Environment Variables

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AGWU662/micro](https://github.com/AGWU662/micro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
