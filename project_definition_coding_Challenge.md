# Project Definition: Technology Stack and Architecture

## Project Purpose

An interactive web application for learning Python programming through gamified coding challenges. Users solve programming tasks, earn points and achievements, maintain daily streaks, and track their progress over time.

---

## Technology Stack Overview

### Backend Framework

#### Flask 3.1.0+
**What it is:** Lightweight Python web framework for building web applications.

**Why chosen:**
- Minimal boilerplate, quick to set up and iterate
- Excellent for educational projects with straightforward requirements
- Rich ecosystem of extensions (Flask-Login, Flask-Mail, Flask-SQLAlchemy)
- Built-in development server for rapid testing
- Simple routing system makes code easy to understand and maintain

**Project benefits:**
- Single `app.py` file contains all routes, making navigation straightforward
- Jinja2 templating engine integrates seamlessly for dynamic HTML generation
- Werkzeug utilities provide security features out of the box
- Easy to extend with custom utilities and middleware

---

### Database Layer

#### MySQL 8.0
**What it is:** Open-source relational database management system.

**Why chosen:**
- Strong ACID compliance ensures data integrity for user accounts and submissions
- Excellent performance for relational data (users, tasks, achievements, submissions)
- Wide adoption means good tooling and community support
- Full-text search capabilities for task filtering
- Reliable transaction handling for critical operations (points, streak updates)

**Project benefits:**
- Normalized schema prevents data duplication (Many-to-Many relationships)
- Foreign key constraints maintain referential integrity
- Supports complex queries for achievement checking and task filtering
- JSON column type for storing flexible test results in submissions

#### SQLAlchemy 2.0.40+ & Flask-SQLAlchemy 3.1.1+
**What it is:** Python ORM (Object-Relational Mapping) and Flask integration.

**Why chosen:**
- Abstracts raw SQL into Python objects and methods
- Prevents SQL injection through parameterized queries
- Database-agnostic (could switch from MySQL to PostgreSQL if needed)
- Relationship management handles complex associations automatically
- Session management integrated with Flask request lifecycle

**Project benefits:**
- Models in `models.py` are self-documenting (clear schema definition)
- Many-to-many relationships (user-achievements, user-tasks) simplified with association tables
- Query API is intuitive: `User.query.filter_by(username=username).first()`
- Migrations could be added easily with Flask-Migrate/Alembic
- Lazy loading optimizes performance for large datasets

#### mysql-connector-python & mysqlclient
**What it is:** Python drivers for MySQL database connectivity.

**Why chosen:**
- `mysql-connector-python`: Pure Python implementation, cross-platform compatibility
- `mysqlclient`: Native C bindings for better performance
- Both provide CONNECTION pooling and thread safety

**Project benefits:**
- Reliable connection handling through SQLAlchemy
- Support for MySQL-specific features (JSON columns, full-text indexes)
- Compatible with SQLAlchemy's dialect system

---

### Code Execution & Security

#### Docker 7.1.0+
**What it is:** Containerization platform for running applications in isolated environments.

**Why chosen:**
- **Security**: Isolates untrusted user code from host system
- **Sandboxing**: Prevents malicious code from accessing filesystem, network, or system resources
- **Resource Control**: CPU, memory, and process limits prevent DoS attacks
- **Consistency**: Same execution environment regardless of host OS
- **Portability**: Works on Windows, Linux, macOS

**Project benefits:**
- User code runs in `python:3.11-slim` container with:
  - `--network none` (no internet access)
  - `--pids-limit 64` (prevents fork bombs)
  - `--memory 64m` (memory limit)
  - `--cpus 0.5` (CPU throttling)
  - Read-only file system mount
- `safe_execute()` function in `utils/utils.py` implements complete sandboxing
- Test cases can include malicious code without risk
- Clean slate for each execution (no state persistence between runs)
- Image caching means fast subsequent executions

#### subprocess (Python standard library)
**What it is:** Module for spawning new processes and capturing output.

**Why chosen:**
- Standard library means no extra dependencies
- Direct process control with timeout support
- Captures stdout, stderr, and exit codes
- Used for both Docker CLI invocation and fallback execution

**Project benefits:**
- Timeout parameter prevents infinite loops in user code
- Input piping supports test cases with stdin requirements
- Error handling captures Python exceptions and displays to users
- Fallback `unsafe_execute()` for development without Docker

---

### Authentication & Security

#### Flask-Login 0.6.3+
**What it is:** User session management extension for Flask.

**Why chosen:**
- Handles login/logout state automatically
- Session cookie management with security best practices
- `@login_required` decorator for route protection
- `current_user` proxy for accessing logged-in user context
- Remember-me functionality built-in

**Project benefits:**
- Simple integration: `LoginManager` initialization in `app.py`
- `User` model inherits from `UserMixin` for required methods
- Protected routes ensure only authenticated users can submit solutions
- Automatic redirection to login page for unauthorized access
- Session persistence across browser restarts

#### Werkzeug 3.1.3+
**What it is:** WSGI utility library (comes with Flask, explicitly pinned for security).

**Why chosen:**
- `generate_password_hash()` and `check_password_hash()` use PBKDF2
- Secure random token generation
- Request parsing and validation
- Development server with auto-reload

**Project benefits:**
- Passwords never stored in plain text (hashed with salt)
- Timing-safe comparison prevents timing attacks
- User model methods (`set_password()`, `check_password()`) abstract security details
- Default PBKDF2 iterations provide strong protection against brute force

#### itsdangerous 2.2.0+
**What it is:** Cryptographically signed token generation library.

**Why chosen:**
- Time-limited tokens for password reset links
- Signed tokens prevent tampering
- Integrates with Flask's SECRET_KEY
- Built-in expiration handling

**Project benefits:**
- `generate_reset_token()` creates secure 1-hour password reset links
- `verify_reset_token()` validates tokens and extracts email safely
- Tokens invalidate after expiration or use
- URL-safe encoding works in email links

#### python-dotenv 1.1.0+
**What it is:** Loads environment variables from `.env` files.

**Why chosen:**
- Separates sensitive configuration from code
- Different environments (dev, staging, prod) use different `.env` files
- Prevents accidental secret commits to version control
- Standard practice in Python web applications

**Project benefits:**
- `SECRET_KEY`, `DATABASE_URL`, email credentials stay out of source code
- `.env` file in `.gitignore` prevents credential leaks
- Easy to change configuration without code changes
- Local development uses local `.env`, production uses environment variables

#### Google OAuth 2.0 (google-auth-oauthlib, google-api-python-client)
**What it is:** Google authentication and API integration libraries.

**Why chosen (imported but not fully implemented):**
- Social login reduces friction for users
- Eliminates password management burden
- Trusted authentication provider
- Access to Google APIs if needed later

**Current status:** Libraries installed but OAuth flow not active in current routes

---

### Email Functionality

#### Flask-Mail 0.10.0+
**What it is:** Flask extension for sending emails via SMTP.

**Why chosen:**
- Simple API for sending emails from Flask app
- SMTP configuration through Flask config system
- Support for attachments, HTML emails, and bulk sending
- Integration with Flask application context

**Project benefits:**
- Password reset emails via Gmail SMTP (app.py:28-37)
- `Message` class simplifies email composition
- Asynchronous sending possible with threading
- Configurable sender, recipients, subject, body
- Error handling for failed email delivery

**Configuration:**
```python
MAIL_SERVER: smtp.gmail.com
MAIL_PORT: 587
MAIL_USE_TLS: True
```

---

### User Interface & Task Creation

#### Gradio (Python library)
**What it is:** Fast UI framework for building machine learning and data science interfaces.

**Why chosen:**
- Rapid prototyping of data entry interfaces
- No HTML/CSS/JavaScript knowledge required
- Built-in components (Textbox, Dataframe, Dropdown, Code editor)
- Auto-generates web interface from Python code
- File upload/download built-in

**Project benefits:**
- `task_designer.py` creates full CRUD interface for tasks in ~100 lines
- Dataframe component perfect for test case entry (input/expected/hidden columns)
- Code editor with Python syntax highlighting for code templates
- Export button generates JSON with proper structure
- Runs on separate port (7860) from main Flask app
- Non-technical users can create tasks without editing JSON manually

**Key features used:**
- `gr.Blocks()` for custom layout
- `gr.Dataframe()` with dynamic rows for test cases
- `gr.Code()` with language="python" for code template
- `gr.File()` for JSON export
- Button click events trigger Python functions

---

### Utilities & Data Handling

#### python-dateutil 2.9.0+
**What it is:** Extensions to Python's datetime module.

**Why chosen:**
- Robust date parsing from various formats
- Timezone handling and conversions
- Relative date calculations (useful for streaks)

**Project benefits:**
- Handles ISO format datetime strings from JSON
- Simplifies streak calculations (checking consecutive days)
- Timezone-aware datetime operations prevent DST bugs

#### JSON (Python standard library)
**What it is:** JavaScript Object Notation parsing and generation.

**Why chosen:**
- Human-readable data format for tasks and achievements
- Easy to edit manually if needed
- Standard format for web APIs
- Native Python support

**Project benefits:**
- Task definitions in `data/tasks/*.json` are version-controllable
- Achievement definitions in `data/achievements/*.json`
- Non-developers can create tasks by copying JSON templates
- Test results stored as JSON in database for flexibility
- Task Designer exports JSON directly

---

## Architectural Patterns & Design Decisions

### Application Structure

#### Modular Organization
```
app.py          → Routes and request handling
models/         → Data models and database schema
utils/          → Business logic and utilities
templates/      → View layer (Jinja2 HTML)
static/         → CSS, images, client-side assets
data/           → JSON-based content (tasks, achievements)
```

**Benefits:**
- Clear separation of concerns
- Easy to locate functionality
- New developers can navigate codebase quickly
- Testing can be done at module level

#### Blueprint-Free Design
**Decision:** All routes in single `app.py` file instead of Flask Blueprints.

**Rationale:**
- Small application (~400 lines)
- Routes are logically cohesive
- No need for route namespacing
- Simpler for educational purposes

**Trade-off:** Would need refactoring if app grows significantly

---

### Data Loading Strategy

#### JSON-Based Content Management
Tasks and achievements stored as JSON files, loaded at startup.

**Benefits:**
- Content changes don't require database migrations
- Version control for content (Git tracks JSON changes)
- Easy bulk import/export
- Non-developers can add content
- Idempotent loading (can restart app safely)

**Implementation:**
```python
# app.py:47-50
with app.app_context():
    db.create_all()
    load_tasks()        # Loads *.json from data/tasks/
    load_achievements() # Loads *.json from data/achievements/
```

**Idempotency:** Functions check if title/name exists before inserting.

---

### Security Model

#### Defense in Depth
Multiple layers protect against malicious code:

1. **Input Validation** (Flask forms)
2. **SQL Injection Prevention** (SQLAlchemy ORM)
3. **XSS Prevention** (Jinja2 autoescaping)
4. **Code Sandboxing** (Docker containers)
5. **Resource Limits** (CPU, memory, PIDs, timeout)
6. **Network Isolation** (`--network none`)
7. **Filesystem Protection** (read-only mounts)

**Example threat mitigation:**
```python
# User submits: import os; os.system('rm -rf /')
# Result: Docker container has no write access, isolated from host
```

---

### Database Design

#### Many-to-Many Relationships
Association tables enable complex relationships:

```
user_achievement: User ↔ Achievement
user_task:        User ↔ Task (completed tasks)
```

**Benefits:**
- One user has many achievements/completed tasks
- One achievement/task belongs to many users
- `unlocked_at` timestamp tracked in association table
- SQLAlchemy handles join queries automatically

#### JSON Column for Test Results
`Submission.test_results` stores flexible structured data:

```json
[
  {
    "input": "3",
    "expected": "6",
    "actual": "6",
    "passed": true,
    "hidden": false
  }
]
```

**Benefits:**
- Variable number of test cases per task
- Rich error information without schema changes
- Easy to display results in UI
- Queryable with MySQL JSON functions if needed

---

### Gamification System

#### Points, Streaks, and Achievements
**Motivation mechanics:**
- **Points:** Awarded for completing tasks (difficulty-based)
- **Streaks:** Consecutive days solving tasks
- **Achievements:** Unlocked at thresholds (points, tasks solved)

**Technical implementation:**
```python
# On successful task completion:
1. Award points (task.points)
2. Update streak (update_streak)
3. Mark task completed (check_tasks)
4. Check achievement unlocks (check_achievements)
5. Commit transaction atomically
```

**Benefits:**
- Encourages daily practice (streak system)
- Clear progression path (achievements)
- Visible progress (points, profile page)
- Retention mechanism (don't break the streak!)

---

## Development Workflow

### Local Development
1. Start MySQL container: `docker-compose up -d`
2. Activate virtualenv: `venv\Scripts\activate`
3. Run Flask app: `python app.py`
4. Access at `http://localhost:5000`

### Creating Content
1. Run Task Designer: `python utils/task_designer.py`
2. Fill in task details and test cases
3. Export JSON to `data/tasks/`
4. Restart Flask app to load new tasks

### Database Management
- Schema changes: Edit `models/models.py`, restart app
- Data reset: `docker-compose down -v` (deletes volumes)
- Migrations: Could add Flask-Migrate for production

---

## Deployment Considerations

### Current State
Configured for local development with:
- Debug mode disabled (`app.run(debug=False)`)
- Hardcoded localhost database connection
- Gmail SMTP for emails

### Production Readiness Checklist
**Would need for production:**
- [ ] WSGI server (Gunicorn/uWSGI) instead of Flask dev server
- [ ] Reverse proxy (Nginx) for static files and HTTPS
- [ ] Environment-based configuration (staging/production)
- [ ] Database connection pooling tuning
- [ ] Docker registry for Python execution image
- [ ] Proper logging (not just print statements)
- [ ] Error monitoring (Sentry)
- [ ] Database backups
- [ ] Rate limiting for submissions
- [ ] CDN for static assets

---

## Technology Trade-offs

### Chosen Approach vs. Alternatives

| Technology | Alternative | Why Current Choice |
|------------|------------|-------------------|
| Flask | Django | Lightweight, less boilerplate, easier to learn |
| MySQL | PostgreSQL | More common in educational contexts, slightly simpler setup |
| Docker | Direct subprocess | Security critical - Docker provides true isolation |
| Gradio | Custom HTML forms | Rapid development, 1/10th the code |
| SQLAlchemy | Raw SQL | Prevents SQL injection, cleaner code |
| Flask-Login | Custom sessions | Battle-tested, handles edge cases |
| Gmail SMTP | SendGrid/AWS SES | Free tier sufficient, no API key management |

---

## Scalability Considerations

### Current Bottlenecks
1. **Docker container startup:** ~1-2 seconds per execution
2. **Sequential test execution:** Tasks with many test cases take longer
3. **Single-threaded Flask dev server:** Can't handle concurrent requests
4. **No caching:** Database queries on every page load

### Scaling Strategies (if needed)
1. **Container pooling:** Pre-warmed containers ready for execution
2. **Parallel test execution:** Run test cases concurrently
3. **Production WSGI server:** Gunicorn with multiple workers
4. **Redis caching:** Cache task lists, achievement data
5. **Background job queue:** Celery for async code execution
6. **Database read replicas:** Separate read/write connections

---

## Summary

This technology stack prioritizes:
1. **Security**: Docker sandboxing, password hashing, SQL injection prevention
2. **Simplicity**: Flask's minimalism, JSON-based content, single-file app
3. **Developer Experience**: Hot reload, clear error messages, modular structure
4. **User Experience**: Fast feedback, gamification, progress tracking
5. **Maintainability**: Standard patterns, well-documented libraries, type hints possible

The architecture supports the project's educational mission by making Python programming practice safe, engaging, and accessible.
