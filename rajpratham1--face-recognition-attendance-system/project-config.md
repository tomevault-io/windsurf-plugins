---
trigger: always_on
description: This is an **Academic Face Recognition Attendance System** for Invertis University, built with Flask (Python) backend and vanilla JavaScript frontend with face-api.js for biometric verification.
---

# GitHub Copilot Instructions - Face Recognition Attendance System

## Project Context
This is an **Academic Face Recognition Attendance System** for Invertis University, built with Flask (Python) backend and vanilla JavaScript frontend with face-api.js for biometric verification.

## Architecture Overview
- **Backend**: Flask (Python 3.9+) with SQLAlchemy ORM
- **Frontend**: Jinja2 templates + Bootstrap 5 + face-api.js
- **Database**: SQLite (dev) / PostgreSQL (production)
- **Authentication**: Flask-Login with role-based access control
- **Face Recognition**: face-api.js (client-side) with 128-D face descriptors
- **Security**: CSRF protection, rate limiting, geofencing, device fingerprinting

## Key Design Patterns

### 1. Role-Based Architecture
```python
# Three distinct user roles with different workflows:
- Admin: System-wide management and analytics
- Teacher: Course creation, session management, attendance reports
- Student: Face registration, session attendance marking
```

### 2. Dual Attendance Models (IMPORTANT)
```python
# Attendance (Legacy) - Generic daily check-in for staff/teachers
class Attendance:
    user_id, date, time, status, latitude, longitude

# SessionAttendance (Primary) - Class session-based for students
class SessionAttendance:
    session_id, student_id, marked_at, face_distance, device_hash, ip_address
    
# AttendanceAttempt - Fraud detection audit log
class AttendanceAttempt:
    session_id, student_id, success, reason, latitude, longitude, face_distance
```

### 3. Security-First Approach
- All face verification happens client-side (privacy)
- Server validates face distance threshold (< 0.6)
- Geofencing enforced (100m radius from Invertis University)
- Rate limiting on all auth and attendance endpoints
- Device fingerprinting to track duplicate attempts
- CSRF tokens on all forms

## Coding Standards

### Python (Backend)
```python
# Follow these conventions:
- Use type hints for function parameters and returns
- All database queries should use SQLAlchemy ORM (no raw SQL unless necessary)
- UTC timestamps in database, convert to Asia/Kolkata for display
- Error handling: try-except with flash messages for user-facing errors
- Rate limiting: Use @limiter.limit() decorator on sensitive endpoints
- Validation: Check user role, enrollment status, session active status before operations
```

### JavaScript (Frontend)
```javascript
// Follow these conventions:
- Use async/await for face-api.js operations
- Always check face detection before sending to server
- Use fetch() with CSRF token from window.getCsrfToken()
- Handle camera permissions gracefully with user-friendly errors
- Implement proper loading states during face processing
```

### Database Migrations
```bash
# Always use Flask-Migrate for schema changes:
flask db migrate -m "descriptive message"
flask db upgrade
# Never modify models.py without creating migration
```

## Common Patterns to Follow

### 1. Creating New Routes
```python
@app.route("/teacher/new-feature", methods=["GET", "POST"])
@login_required
@limiter.limit("20 per hour")  # Add rate limiting
def new_feature():
    # 1. Check user role
    if current_user.role != "teacher":
        flash("Access denied", "danger")
        return redirect(url_for("dashboard"))
    
    # 2. Handle POST request
    if request.method == "POST":
        # Validate CSRF (automatic with Flask-WTF)
        # Validate input
        # Perform operation
        # Flash success message
        # Redirect
        pass
    
    # 3. Render template with context
    return render_template("new_feature.html")
```

### 2. Face Verification Pattern
```javascript
// Client-side face verification template
const detections = await faceapi.detectAllFaces(video)
    .withFaceLandmarks()
    .withFaceDescriptors();

// Check for single face
if (!detections || detections.length !== 1) {
    statusText.innerText = 'Exactly one face required';
    return;
}

// Send to server
const response = await fetch('/api/endpoint', {
    method: 'POST',
    headers: {
        'Content-Type': 'application/json',
        'X-CSRFToken': csrfToken
    },
    body: JSON.stringify({
        descriptor: Array.from(detections[0].descriptor),
        lat: userCoords.lat,
        lng: userCoords.lng,
        device_id: deviceId
    })
});
```

### 3. Attendance Recording Pattern
```python
def record_attendance_with_fraud_check(session_id, student_id, descriptor, lat, lng, device_hash):
    # 1. Validate session is active
    session = ClassSession.query.filter_by(id=session_id).first()
    if not session or not session.is_active:
        record_attempt(session_id, student_id, False, "Session inactive", lat, lng)
        return {"success": False, "message": "Session not active"}
    
    # 2. Check geofencing
    if not is_within_invertis(lat, lng):
        record_attempt(session_id, student_id, False, "Outside campus", lat, lng)
        return {"success": False, "message": "Outside campus boundary"}
    
    # 3. Check duplicate
    if SessionAttendance.query.filter_by(session_id=session_id, student_id=student_id).first():

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rajpratham1/Face-Recognition-Attendance-System](https://github.com/rajpratham1/Face-Recognition-Attendance-System) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
