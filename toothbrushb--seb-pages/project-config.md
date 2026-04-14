---
trigger: always_on
description: A Flask web application for managing Safe Exam Browser (SEB) exam sessions. Teachers create exams with embedded web tools (calculators, documents) that students access through SEB's secure browser environment. The system validates SEB security keys (Browser Exam Key, Config Key, User Agent) to ensure exam integrity.
---

# SEB Pages - Copilot Instructions

## Project Overview
A Flask web application for managing Safe Exam Browser (SEB) exam sessions. Teachers create exams with embedded web tools (calculators, documents) that students access through SEB's secure browser environment. The system validates SEB security keys (Browser Exam Key, Config Key, User Agent) to ensure exam integrity.

## Architecture

### Core Components
- **`app.py`**: Flask server with role-based auth (admin/teacher/student), exam CRUD, SEB key validation, Google OAuth
- **`sebConfigUtils.py`**: Crypto module for encrypting/decrypting SEB config files (`.seb` format with RNCryptor)
- **`sebLock.js`**: Client-side SEB security validation - hashes keys with URL and compares against meta tags
- **JSON Datastores**: `exams.json`, `users.json`, `exam_apps.json`, `exam_attempts.json` (no database - direct file I/O)

### SEB Security Model
Exams store three security components:
1. **BEK (Browser Exam Key)**: Array of SHA256 hashes - validates SEB browser instance
2. **CK (Config Key)**: SHA256 hash - validates SEB config file matches expected settings
3. **UA (User Agent)**: String seed that's hashed with URL - identifies genuine SEB instances

JavaScript (`sebLock.js`) retrieves actual keys from `SafeExamBrowser.security` API, hashes them with current URL, and compares against expected values in page meta tags. All three must match or exam is locked.

## Data Flow

### Exam Registration Flow
1. Teacher submits form with BEKs (array), CK, UA, and tools list
2. System generates UUID exam ID + 4-word memorable exam key (e.g., `ALPHA-BRAVO-CHARLIE-DELTA`)
3. Exam saved to `exams.json` with structure:
   ```json
   {
     "exam_key": "ALPHA-BRAVO-CHARLIE-DELTA",
     "user_id": "teacher-uuid",
     "beks": ["hash1", "hash2"],
     "ck": "config-hash",
     "ua": "user-agent-seed",
     "tools": [{"id": "uuid", "name": "...", "url": "...", "icon": "bi-..."}]
   }
   ```

### Exam Access Flow
1. Student visits `/<exam_id>/exam` route (no login required for exam takers)
2. Student signs in with Google (optional but recommended for tracking)
3. Server renders `exam_launcher.html` with SEB keys in meta tags
4. `sebLock.js` validates keys every 1 second
5. If valid, student can access tools via `/<exam_id>/app/<app_id>` routes
6. Each access is logged to `exam_attempts.json` with student's Google account info

### Exam Attempts Tracking
All exam accesses are logged with:
- Timestamp (ISO 8601 UTC)
- Google ID (if signed in)
- Email address
- Name
- Access type (`exam_launch` or `tool_access:<tool_name>`)
- Unique attempt ID

Teachers can view attempts at `/<exam_id>/attempts` route.

## Key Conventions

### UUID Generation
- **Exam IDs**: Cryptographically secure `uuid.uuid4()` 
- **Exam Keys**: 4 random words from 100-word NATO/space-themed list using `secrets.choice()`
- **User IDs**: UUID for each user account

### Role-Based Access Control
- `@login_required`: Decorator for any authenticated route
- `@admin_required`: Extends login check with `role == 'admin'` validation
- Ownership checks: Teachers see only their `user_id` exams, admins see all

### File I/O Pattern
All persistence uses this pattern:
```python
def load_exams():
    if os.path.exists(EXAM_DB_PATH):
        with open(EXAM_DB_PATH, 'r') as f:
            return json.load(f)
    return {}

def save_exams(exams):
    with open(EXAM_DB_PATH, 'w') as f:
        json.dump(exams, f, indent=2)
```
Global `EXAMS`, `USERS`, `EXAM_APPS` dicts loaded at startup, modified in-memory, saved after mutations.

### SEB Config Encryption
`sebConfigUtils.py` uses RNCryptor format:
- **Unencrypted**: Prefix `plnd` + gzipped XML
- **Password-encrypted**: Prefix `pswd` + RNCryptor(gzipped XML) with PBKDF2-derived keys
- Entire structure then gzipped again for final `.seb` file

## Development Workflow

### Running the Application
```bash
# Install dependencies
pip install -r requirements.txt

# Generate initial user accounts (admin/admin, teacher/teacher)
python generate_users.py

# Run development server
python app.py  # Starts on http://0.0.0.0:5000 with debug=True
```

### Testing SEB Key Generation
Use `sebTest.py` utilities:
```python
from sebTest import generate_config_key
# Pass plist dict or XML bytes to get SHA256 config key
```

### Hash Generator Tool
`/hash_generator` route provides UI for generating SHA256 hashes of arbitrary strings - useful for creating BEK/CK/UA values during exam setup.

## Critical Patterns

### URL Hashing for Security
SEB keys are always hashed with the full URL before comparison:
```javascript
// In sebLock.js
const url = window.location.origin + window.location.pathname;
const hashedCK = await concatHash(url, ckExpected);
const hashedUA = await concatHash(url, userAgent);
```
Backend must use same URL construction: `url_for('exam', exam_id=exam_id, _external=True)`.

### Multiple BEK Support
Exams store `beks` as array to support multiple valid browser configurations:
```python

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ToothbrushB) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
