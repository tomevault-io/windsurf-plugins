---
trigger: always_on
description: OEN-Project_MUx1 is a comprehensive client-server authentication system with email verification, supporting both HTTP REST API and Socket.IO real-time communication. The application includes SSH tunnel support for secure remote access and Docker integration for seamless deployment.
---

# Copilot Instructions for OEN-Project_MUx1

## Project Overview

OEN-Project_MUx1 is a comprehensive client-server authentication system with email verification, supporting both HTTP REST API and Socket.IO real-time communication. The application includes SSH tunnel support for secure remote access and Docker integration for seamless deployment.

**Tech Stack:**
- **Backend**: Flask 3.1.2, Flask-SocketIO 5.5.1, SQLAlchemy 3.1.1
- **Database**: SQLite (dev) / PostgreSQL (prod)
- **Authentication**: Werkzeug password hashing, email verification codes
- **Real-time**: Socket.IO with bidirectional messaging
- **Deployment**: Docker Compose, native Python 3.11+

## Architecture Overview

Flask authentication server with **dual communication protocols**:
- **REST API** (`/api/*`) - HTTP endpoints for signup, login, email verification
- **Socket.IO** - Real-time bidirectional messaging for authenticated users

**Client interfaces**: 
- CLI (`client/client.py`) - Python command-line client
- Web GUI (`server/static/index.html` served at `/`) - Form-based interface
- Web Terminal (`server/static/terminal.html` served at `/terminal`) - MUD-style xterm.js interface

**Key files**: 
- `server/app.py` - Flask+SocketIO server with API endpoints and database models
- `client/client.py` - Python CLI client with Socket.IO support
- `server/email_service.py` - Email sending (SMTP/console fallback)
- `server/static/` - Web interfaces (index.html, terminal.html)

## Critical Development Knowledge

### Email Verification Gotcha
When SMTP credentials are not configured, verification codes print to the **server console only** (not the client). Check `docker-compose logs` or server terminal output during testing.

### Authentication Flow (Must Understand)
1. `/api/signup` → creates user with `is_verified=False`, generates 6-digit code (24hr expiry)
2. `/api/verify-email` → validates code, sets `is_verified=True`
3. `/api/login` → **rejects unverified users with HTTP 403**
4. Socket.IO `authenticate` → same verification requirement

### Database Behavior
- **No migrations configured** - schema changes require deleting `auth.db`
- Database file created in working directory where server runs (typically `server/`)
- Password hashing: `user.set_password()` / `user.check_password()` via Werkzeug

## Build, Run, and Test Commands

### Development Setup
```bash
# Create virtual environment
python3 -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt
```

### Running the Application
```bash
# Server (auto-creates venv on Unix)
./start_server.sh  # or: cd server && python app.py

# Client  
./start_client.sh  # or: cd client && python client.py

# Docker (recommended for production)
docker-compose up -d              # Start services
docker-compose logs -f            # View logs
docker-compose down               # Stop services
docker-compose run --rm client    # Run client interactively
```

### Testing
```bash
# Health check
curl http://localhost:5000/api/health

# Sign up new user
curl -X POST http://localhost:5000/api/signup \
  -H "Content-Type: application/json" \
  -d '{"username":"test","email":"test@example.com","password":"pass123"}'

# Verify email (check server console for code)
curl -X POST http://localhost:5000/api/verify-email \
  -H "Content-Type: application/json" \
  -d '{"email":"test@example.com","code":"123456"}'

# Login
curl -X POST http://localhost:5000/api/login \
  -H "Content-Type: application/json" \
  -d '{"username":"test","password":"pass123"}'
```

**Note**: No automated test suite currently exists. Manual testing required for all changes.

## Development Workflows

### Adding New API Endpoints
```python
# In server/app.py - follow existing JSON response pattern
@app.route('/api/new-endpoint', methods=['POST'])
def new_endpoint():
    """Brief description of endpoint purpose"""
    data = request.get_json()
    if not data or not data.get('required_field'):
        return jsonify({'error': 'Missing required fields'}), 400
    
    try:
        # Your logic here
        result = process_data(data)
        return jsonify({'message': 'Success', 'data': result}), 200
    except Exception as e:
        return jsonify({'error': str(e)}), 500
```

### Adding Socket.IO Events
```python
# In server/app.py
@socketio.on('event_name')
def handle_event(data):
    """Handle Socket.IO event"""
    # Validate input
    if not data:
        emit('error', {'message': 'Invalid data'})
        return
    
    # Process and respond
    emit('response_event', {'result': data})
```

### Database Model Changes
```python
# Add column to User model in server/app.py
class User(db.Model):
    # ... existing fields ...
    new_field = db.Column(db.String(100))  # Add new column

# Then delete database and restart (no migrations configured)
# rm server/auth.db && cd server && python app.py
```

**Important**: Schema changes require deleting `auth.db` - no migrations exist.

## Environment Variables

| Variable | Default | Notes |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ULFHEDNAR-JAKE) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
