---
trigger: always_on
description: Security Best Practices and Validation - review after major feature added
---

# Security Validation - Go Security Best Practices

## Input Validation

### Parameter Validation Pattern
```go
func (s *ApplicationService) Deploy(ctx context.Context, name string, options DeployOptions) error {
    // 1. Basic validation
    if name == "" {
        return fmt.Errorf("application name cannot be empty")
    }
    
    // 2. Format validation
    if len(name) > 63 {
        return fmt.Errorf("application name cannot exceed 63 characters")
    }
    
    // 3. Character validation
    if !isValidAppName(name) {
        return fmt.Errorf("application name contains invalid characters")
    }
    
    // 4. Business rule validation
    exists, err := s.repo.Exists(ctx, name)
    if err != nil {
        return fmt.Errorf("failed to check application existence: %w", err)
    }
    if !exists {
        return fmt.Errorf("application %s does not exist", name)
    }
    
    return nil
}

// Helper for validation
func isValidAppName(name string) bool {
    // DNS-compatible naming: alphanumeric and hyphens only
    matched, _ := regexp.MatchString(`^[a-z0-9](mdc:[a-z0-9\-]*[a-z0-9])?$`, name)
    return matched
}
```

### Struct Validation with Tags
```go
type DeployOptions struct {
    GitRef     string `validate:"omitempty,min=1,max=100"`
    BuildPack  string `validate:"omitempty,oneof=nodejs python go ruby"`
    Force      bool   `validate:""`
    Timeout    int    `validate:"min=1,max=3600"` // 1 second to 1 hour
}

func validateStruct(s interface{}) error {
    validate := validator.New()
    return validate.Struct(s)
}
```

## SQL Injection Prevention (Following Go Security Guide)

### Secure Database Operations
```go
// NEVER: Query construction by concatenation
func getUser(db *sql.DB, userID string) error {
    query := "SELECT * FROM users WHERE id = '" + userID + "'" // DANGEROUS!
    return nil
}

// GOOD: Using prepared statements
func getUser(db *sql.DB, userID string) (*User, error) {
    query := "SELECT id, name, email FROM users WHERE id = ?"
    
    var user User
    err := db.QueryRow(query, userID).Scan(&user.ID, &user.Name, &user.Email)
    if err != nil {
        return nil, fmt.Errorf("failed to get user: %w", err)
    }
    
    return &user, nil
}

// Even better: Using reusable prepared statements
type UserRepository struct {
    db       *sql.DB
    getUserStmt *sql.Stmt
}

func NewUserRepository(db *sql.DB) (*UserRepository, error) {
    getUserStmt, err := db.Prepare("SELECT id, name, email FROM users WHERE id = ?")
    if err != nil {
        return nil, fmt.Errorf("failed to prepare query: %w", err)
    }
    
    return &UserRepository{
        db:          db,
        getUserStmt: getUserStmt,
    }, nil
}
```

### Transaction Security
```go
func (r *UserRepository) UpdateUserSecurely(ctx context.Context, userID string, updates UserUpdate) error {
    tx, err := r.db.BeginTx(ctx, nil)
    if err != nil {
        return fmt.Errorf("failed to start transaction: %w", err)
    }
    defer tx.Rollback() // Automatic rollback on error
    
    // Use prepared statements in transaction
    _, err = tx.ExecContext(ctx, 
        "UPDATE users SET name = ?, email = ? WHERE id = ?",
        updates.Name, updates.Email, userID)
    if err != nil {
        return fmt.Errorf("failed to update user: %w", err)
    }
    
    if err = tx.Commit(); err != nil {
        return fmt.Errorf("failed to commit transaction: %w", err)
    }
    
    return nil
}
```

## Command Injection Prevention

### Dokku Command Safety
```go
// WhitelistedCommands defines allowed Dokku operations
var WhitelistedCommands = map[string]bool{
    "apps:list":        true,
    "apps:info":        true,
    "apps:create":      true,
    "config:get":       true,
    "config:set":       true,
    "domains:add":      true,
    "domains:list":     true,
    "ps:scale":         true,
}

func (c *DokkuClient) ExecuteCommand(ctx context.Context, command string, args []string) ([]byte, error) {
    // 1. Verify command is allowed
    if !WhitelistedCommands[command] {
        return nil, fmt.Errorf("command not allowed: %s", command)
    }
    
    // 2. Validate each argument
    for i, arg := range args {
        if err := validateCommandArgument(arg); err != nil {
            return nil, fmt.Errorf("invalid argument %d: %w", i, err)
        }
    }
    
    // 3. Build command securely
    cmd := exec.CommandContext(ctx, "/usr/bin/dokku", append([]string{command}, args...)...)
    
    // 4. Configure secure environment
    cmd.Env = []string{
        "PATH=/usr/bin:/bin",
        "USER=dokku",
    }
    
    output, err := cmd.Output()
    if err != nil {
        return nil, fmt.Errorf("command execution failed %s: %w", command, err)
    }
    
    return output, nil
}

func validateCommandArgument(arg string) error {
    // Prohibit dangerous characters
    dangerous := []string{";", "|", "&", "$", "`", "(", ")", "{", "}", "[", "]", "<", ">", "\n", "\r"}
    for _, char := range dangerous {
        if strings.Contains(arg, char) {
            return fmt.Errorf("dangerous character detected: %s", char)
        }
    }
    
    // Limit length
    if len(arg) > 255 {
        return fmt.Errorf("argument too long (max 255 characters)")
    }
    
    return nil
}
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dokku-MCP/dokku-mcp](https://github.com/dokku-MCP/dokku-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
