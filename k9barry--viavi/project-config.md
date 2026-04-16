---
trigger: always_on
description: This is a Docker Compose project that processes output files from a Viavi 8800SX service monitor, parses them, and stores the data in a MySQL database. The application stores test records with full BLOB storage of original files and provides a web interface for searching and managing data.
---

# GitHub Copilot Instructions for Viavi 8800SX Project

## Project Overview
This is a Docker Compose project that processes output files from a Viavi 8800SX service monitor, parses them, and stores the data in a MySQL database. The application stores test records with full BLOB storage of original files and provides a web interface for searching and managing data.

## Technology Stack
- **Backend**: PHP 8.3.2-FPM
- **Database**: MySQL
- **Web Server**: Nginx (via Docker Compose)
- **Frontend**: Bootstrap 4.5.0, jQuery 3.5.1
- **Containerization**: Docker & Docker Compose
- **Code Quality**: PHP CS Fixer (PSR-12), YAML linting, Markdown linting

## Configuration System

### Current Architecture
The project uses a **traditional PHP configuration** approach with simple variable assignments in `config.php`:

```php
// Database configuration
$db_server = 'db';
$db_name = 'viavi';
$db_user = 'viavi';
$db_password = trim(file_get_contents($passwordFile));
$link = mysqli_connect($db_server, $db_user, $db_password, $db_name);

// Application configuration
$appname = '8800SX';
$language = 'en';
$no_of_records_per_page = 10;
$protocol = (isset($_SERVER['HTTPS']) && $_SERVER['HTTPS'] == 'on') ? 'https' : 'http';
$domain = $protocol . '://' . ($_SERVER['HTTP_HOST'] ?? 'localhost');

// Upload configuration
$upload_max_size = 5000000;
$upload_target_dir = "uploads/";
$upload_disallowed_exts = array(/* ... */);

// Translations
$translations = include($localeFile);
```

**Important**: 
- All configuration variables are defined directly in `config.php`
- No Config class or OOP approach
- Helper functions use `global` keyword to access configuration
- Database connection `$link` is created directly in config.php

## Code Style and Conventions

### PHP Guidelines
1. **Security First**: Always use prepared statements for database queries
2. **Input Validation**: Validate and sanitize all user inputs
3. **File Uploads**: 
   - Always validate file types using MIME types, not just extensions
   - Check file sizes against configured limits
   - Use the `handleFileUpload()` function in helpers.php
4. **Error Handling**: Use try-catch blocks and log errors appropriately
5. **Documentation**: Add PHPDoc comments for functions
6. **Code Formatting**: Follow PSR-12 standards (auto-formatted by GitHub Actions)

### Database Interactions
1. Always use parameterized queries (prepared statements)
2. Never concatenate user input into SQL queries
3. Use appropriate data types in the schema
4. Handle database connection errors gracefully
5. Access database via `$link` variable from config.php

### Global Variables Usage
Functions that need configuration values should use the `global` keyword:

```php
function myFunction() {
    global $link, $upload_target_dir, $translations;
    // Use the variables
}
```

### File Structure
```
/data/web/
├── nginx.conf         # Nginx configuration
└── app/
    ├── config.php     # Configuration variables only (no classes)
    ├── helpers.php    # Helper functions (uses global variables)
    ├── main.php       # File upload handler
    ├── upload.php     # Upload UI
    ├── alignments-*.php  # CRUD operations
    ├── locales/       # Internationalization files
    └── security-headers.php  # Security headers
/data/db/init/
└── init-db.sql        # Database initialization script
```

## Security Considerations

### Critical Security Rules
1. **Never hardcode credentials** - Use environment variables and file-based secrets
2. **Validate all inputs** - Both client and server-side
3. **Use prepared statements** - Always for database queries
4. **Implement rate limiting** - Prevent abuse
5. **Keep dependencies updated** - Regular security patches
6. **Use HTTPS** - In production environments
7. **Set security headers** - CSP, X-Frame-Options, etc.

### File Upload Security
- Validate MIME types server-side
- Restrict file extensions using allowlist
- Set maximum file size limits
- Scan uploaded files for malware if possible
- Store files with unique names to prevent overwrites

## Development Workflow

### Before Making Changes
1. Review existing code patterns
2. Check for similar implementations
3. Consider security implications
4. Update documentation if needed

### Testing
1. Test file uploads with various file types
2. Verify database operations don't have SQL injection vulnerabilities
3. Test with invalid inputs to ensure proper error handling
4. Check that uploaded files are processed correctly

### Docker Commands
```bash
# Start services
docker compose up -d

# View logs
docker compose logs -f

# Rebuild after changes
docker compose down
docker compose build --no-cache
docker compose up -d

# Access PHP container
docker compose exec web bash
```

## Common Patterns

### Database Query Pattern
```php
// Correct - Using prepared statements
$stmt = $connection->prepare("SELECT * FROM alignments WHERE id = ?");
$stmt->bind_param("i", $id);
$stmt->execute();
$result = $stmt->get_result();
```

### Input Validation Pattern
```php
// Validate and sanitize input

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/k9barry) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
