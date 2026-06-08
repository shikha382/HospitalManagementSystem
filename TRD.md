# ⚙️ Technical Requirements Document (TRD)
## Hospital Management System — 24f2007_HMS
**Version:** 1.0  
**Date:** June 2026  
**Author:** 24f2007  
**Status:** ✅ Implemented

---

## 1. System Overview

The HMS is a **monolithic Flask web application** that serves a JavaScript Single Page Application (SPA). It exposes a RESTful JSON API consumed entirely by the frontend. Background task processing is handled by **Celery** with **Redis** as the message broker and result backend.

```
┌─────────────────────────────────────────────────┐
│                   Browser (SPA)                  │
│        Vanilla JS + HTML + CSS (index.html)      │
└──────────────────────┬──────────────────────────┘
                       │ HTTP / REST JSON
┌──────────────────────▼──────────────────────────┐
│               Flask Application                  │
│   ┌──────────┐ ┌──────────┐ ┌────────────────┐  │
│   │ auth_bp  │ │ admin_bp │ │  doctor_bp     │  │
│   └──────────┘ └──────────┘ │  patient_bp    │  │
│                             └────────────────┘  │
│   ┌──────────────────┐  ┌───────────────────┐   │
│   │   SQLAlchemy ORM │  │   Flask-JWT-Ext.  │   │
│   │   (SQLite / PG)  │  │   (JWT Auth)      │   │
│   └──────────────────┘  └───────────────────┘   │
│   ┌──────────────────┐  ┌───────────────────┐   │
│   │   Flask-Mail     │  │   In-Memory Cache │   │
│   │   (SMTP)         │  │   (Redis/dict)    │   │
│   └──────────────────┘  └───────────────────┘   │
└────────────────────────┬────────────────────────┘
                         │
┌────────────────────────▼────────────────────────┐
│              Celery Worker + Beat                 │
│   ┌────────────────┐  ┌───────────────────────┐  │
│   │ send_reminders │  │ generate_monthly_rpt  │  │
│   │ export_csv     │  └───────────────────────┘  │
│   └────────────────┘                             │
│              Broker: Redis                        │
└─────────────────────────────────────────────────┘
```

---

## 2. Technology Stack

### 2.1 Backend

| Component | Technology | Version |
|---|---|---|
| Web Framework | Flask | 3.0.0 |
| ORM | Flask-SQLAlchemy | 3.1.1 |
| Auth | Flask-JWT-Extended | 4.6.0 |
| CORS | Flask-CORS | 4.0.0 |
| Email | Flask-Mail | 0.9.1 |
| Task Queue | Celery | 5.3.4 |
| Message Broker | Redis | 5.0.1 (client) |
| Password Hashing | Werkzeug | 3.0.1 |
| Env Config | python-dotenv | 1.0.0 |
| Database (dev) | SQLite | built-in |
| Database (prod) | PostgreSQL | via `DATABASE_URL` |

### 2.2 Frontend

| Component | Technology |
|---|---|
| Structure | HTML5 (single `index.html`) |
| Logic | Vanilla JavaScript (ES6+) |
| Styling | Vanilla CSS |
| Routing | Hash-based client-side routing |
| HTTP Client | Native `fetch` API |
| Auth Token | `localStorage` (JWT) |

### 2.3 Infrastructure

| Component | Technology |
|---|---|
| Message Broker | Redis (Windows binary bundled in `redis_win/`) |
| Task Scheduler | Celery Beat |
| Mail Testing | MailHog (default, port 1025) |
| Process Runner | Python `flask run` / `celery worker` |

---

## 3. Project Structure

```
24f2007_HMS/
├── backend/
│   ├── app.py              # Flask app factory + initialization
│   ├── config.py           # Centralized config (env-driven)
│   ├── celery_app.py       # Celery instance + Beat schedule
│   ├── email_utills.py     # Flask-Mail initialization
│   ├── models/
│   │   └── __init__.py     # SQLAlchemy models (all 6 tables)
│   ├── routes/
│   │   ├── auth.py         # /api/auth/* endpoints
│   │   ├── admin.py        # /api/admin/* endpoints
│   │   ├── doctor.py       # /api/doctor/* endpoints
│   │   ├── patient.py      # /api/patient/* endpoints
│   │   └── utils.py        # Decorators: role guards, user resolution
│   ├── tasks/
│   │   ├── reminders.py    # send_daily_reminders()
│   │   ├── reports.py      # generate_monthly_reports()
│   │   └── exports.py      # export_patient_treatments()
│   └── cache/              # Redis/in-memory cache helpers
├── static/
│   ├── js/
│   │   ├── app.js          # SPA router + auth orchestration
│   │   └── components/
│   │       ├── LoginForm.js
│   │       ├── AdminDashboard.js
│   │       ├── DoctorDashboard.js
│   │       └── PatientDashboard.js
│   └── images/
├── templates/
│   └── index.html          # SPA shell (Flask serves this for all routes)
├── exports/                # Generated patient export files
├── instance/
│   └── hospital.db         # SQLite database (auto-created)
├── redis_win/              # Redis Windows binary
├── requirements.txt
├── trigger_tasks.py        # CLI helper to manually trigger Celery tasks
├── PRD.md
├── TRD.md
└── README.md
```

---

## 4. Database Schema

### 4.1 Entity-Relationship Diagram

```
users (1) ──── (1) doctors ──── (M) appointments ──── (1) treatments
                │                      │
                │               (M) ───┤
users (1) ──── (1) patients ────────────┘
                
departments (1) ──── (M) doctors

doctors (1) ──── (M) doctor_availability
```

### 4.2 Table Definitions

#### `users`
| Column | Type | Constraints |
|---|---|---|
| id | INTEGER | PK, Auto-increment |
| username | VARCHAR(80) | UNIQUE, NOT NULL |
| email | VARCHAR(120) | UNIQUE, NOT NULL |
| password_hash | VARCHAR(255) | NOT NULL |
| role | VARCHAR(20) | NOT NULL (`admin`/`doctor`/`patient`) |
| active | BOOLEAN | DEFAULT TRUE |
| created_at | DATETIME | DEFAULT now() |

#### `departments`
| Column | Type | Constraints |
|---|---|---|
| id | INTEGER | PK |
| name | VARCHAR(100) | UNIQUE, NOT NULL |
| description | TEXT | |
| created_at | DATETIME | DEFAULT now() |

#### `doctors`
| Column | Type | Constraints |
|---|---|---|
| id | INTEGER | PK |
| user_id | INTEGER | FK → users.id, NOT NULL |
| name | VARCHAR(100) | NOT NULL |
| specialization_id | INTEGER | FK → departments.id, NOT NULL |
| experience | INTEGER | |
| qualification | VARCHAR(200) | |
| contact | VARCHAR(20) | |
| created_at | DATETIME | DEFAULT now() |

#### `patients`
| Column | Type | Constraints |
|---|---|---|
| id | INTEGER | PK |
| user_id | INTEGER | FK → users.id, NOT NULL |
| name | VARCHAR(100) | NOT NULL |
| age | INTEGER | |
| gender | VARCHAR(10) | |
| contact | VARCHAR(20) | |
| address | TEXT | |
| medical_history | TEXT | |
| created_at | DATETIME | DEFAULT now() |

#### `appointments`
| Column | Type | Constraints |
|---|---|---|
| id | INTEGER | PK |
| patient_id | INTEGER | FK → patients.id, NOT NULL |
| doctor_id | INTEGER | FK → doctors.id, NOT NULL |
| date | DATE | NOT NULL |
| time | TIME | NOT NULL |
| status | VARCHAR(20) | DEFAULT `booked` |
| notes | TEXT | |
| created_at | DATETIME | DEFAULT now() |
| — | UNIQUE | `(doctor_id, date, time)` — prevents double-booking |

> **Appointment status values:** `booked` → `completed` / `cancelled` / `expired`

#### `treatments`
| Column | Type | Constraints |
|---|---|---|
| id | INTEGER | PK |
| appointment_id | INTEGER | FK → appointments.id, NOT NULL |
| diagnosis | TEXT | NOT NULL |
| prescription | TEXT | |
| notes | TEXT | |
| next_visit_date | DATE | |
| created_at | DATETIME | DEFAULT now() |

#### `doctor_availability`
| Column | Type | Constraints |
|---|---|---|
| id | INTEGER | PK |
| doctor_id | INTEGER | FK → doctors.id, NOT NULL |
| date | DATE | NOT NULL |
| start_time | TIME | NOT NULL |
| end_time | TIME | NOT NULL |
| is_available | BOOLEAN | DEFAULT TRUE |
| created_at | DATETIME | DEFAULT now() |

---

## 5. API Reference

### 5.1 Authentication — `/api/auth`

| Method | Endpoint | Auth | Description |
|---|---|---|---|
| POST | `/api/auth/login` | None | Returns JWT access token |
| POST | `/api/auth/register` | None | Registers a new patient |
| GET | `/api/auth/me` | JWT | Returns current user info |

### 5.2 Admin — `/api/admin`

| Method | Endpoint | Description |
|---|---|---|
| GET | `/api/admin/dashboard` | System-wide stats (cached) |
| GET | `/api/admin/doctors` | List all doctors |
| POST | `/api/admin/doctors` | Add new doctor |
| PUT | `/api/admin/doctors/<id>` | Update doctor profile |
| PUT | `/api/admin/doctors/<id>/toggle` | Toggle active status |
| GET | `/api/admin/patients` | List all patients |
| PUT | `/api/admin/patients/<id>` | Update patient profile |
| PUT | `/api/admin/patients/<id>/toggle` | Toggle active status |
| GET | `/api/admin/appointments` | All appointments |
| GET | `/api/admin/departments` | All departments |
| GET | `/api/admin/search/doctors?q=` | Search doctors |
| GET | `/api/admin/search/patients?q=` | Search patients |
| POST | `/api/admin/reminders/trigger` | Manually fire daily reminders |
| POST | `/api/admin/reports/monthly` | Generate monthly report |

### 5.3 Doctor — `/api/doctor`

| Method | Endpoint | Description |
|---|---|---|
| GET | `/api/doctor/dashboard` | Doctor stats + 7-day appointments (cached 60s) |
| GET | `/api/doctor/appointments` | All appointments (filterable: status, date range) |
| PUT | `/api/doctor/appointments/<id>` | Update status/notes |
| GET | `/api/doctor/availability` | Current 7-day availability |
| POST | `/api/doctor/availability` | Set/replace 7-day availability |
| POST | `/api/doctor/treatment` | Add treatment to completed appointment |
| PUT | `/api/doctor/treatment/<id>` | Update treatment record |
| GET | `/api/doctor/patients/<id>/history` | Patient's completed appointment history |
| GET | `/api/doctor/report/monthly` | Download HTML monthly report |

### 5.4 Patient — `/api/patient`

| Method | Endpoint | Description |
|---|---|---|
| GET | `/api/patient/dashboard` | Patient stats + upcoming appointments |
| GET | `/api/patient/profile` | Get patient profile |
| PUT | `/api/patient/profile` | Update patient profile |
| GET | `/api/patient/departments` | List departments (cached) |
| GET | `/api/patient/doctors?specialization_id=` | List active doctors (filterable) |
| GET | `/api/patient/doctors/<id>/availability` | Doctor slots + booked slots |
| POST | `/api/patient/appointments` | Book appointment |
| GET | `/api/patient/appointments` | All patient appointments (auto-expires past) |
| DELETE | `/api/patient/appointments/<id>` | Cancel appointment |
| GET | `/api/patient/treatments` | Full treatment history |
| POST | `/api/patient/export` | Export treatment history to file |
| GET | `/api/patient/export/<filename>/download_direct` | Download export (JWT via query param) |

---

## 6. Authentication & Security

### JWT Flow
```
Client                          Server
  │── POST /api/auth/login ────▶│
  │                             │ validate credentials
  │◀── { access_token } ────────│
  │                             │
  │── GET /api/... ─────────────│
  │   Authorization: Bearer <token>
  │                             │ @jwt_required() validates token
  │                             │ @role_required('doctor') checks role
  │◀── { data } ────────────────│
```

### Role Guard Decorators (`routes/utils.py`)
- `@jwt_required()` — verifies JWT signature and expiry
- `@admin_required` — checks `role == 'admin'`
- `@doctor_required` — checks `role == 'doctor'`
- `@patient_required` — checks `role == 'patient'`
- `@role_required(*roles)` — generic multi-role check

### Password Security
- Hashed using **Werkzeug's `generate_password_hash`** (PBKDF2-SHA256)
- Plain text is never stored or logged

### Token Configuration
```python
JWT_TOKEN_LOCATION = ["headers", "query_string"]
JWT_QUERY_STRING_NAME = "token"   # Allows file downloads with token in URL
```

---

## 7. Caching Strategy

| Cache Key | Data | TTL |
|---|---|---|
| `admin:stats` | Admin dashboard counts | 300s (default) |
| `doctor:<id>:stats` | Doctor dashboard data | 60s |
| `departments` | Department list | 300s |

**Implementation:** In-memory Python dict (Redis-ready interface). Cache is invalidated via `invalidate_doctor_cache()` and `invalidate_appointment_cache()` on any write operation.

---

## 8. Background Tasks (Celery)

### 8.1 Task: `send_daily_reminders`
- **Schedule:** Configurable via Celery Beat (`crontab(minute=0)` — hourly in dev)
- **Logic:** Queries all appointments for tomorrow with status `booked`, sends email to each patient
- **Trigger (manual):** `POST /api/admin/reminders/trigger`

### 8.2 Task: `generate_monthly_reports`
- **Schedule:** Monthly via Celery Beat
- **Logic:** Aggregates appointment data by month, emails report to admin
- **Trigger (manual):** `POST /api/admin/reports/monthly` with optional `{ year, month }`

### 8.3 Task: `export_patient_treatments`
- **Trigger:** `POST /api/patient/export` (synchronous execution)
- **Output:** HTML file saved to `exports/` directory; download URL returned
- **Download:** `GET /api/patient/export/<filename>/download_direct?token=<jwt>`

### 8.4 Celery Configuration
```python
celery.conf.timezone = "Asia/Kolkata"
broker  = Redis (REDIS_URL env var, default: redis://localhost:6379/0)
backend = Redis
```

---

## 9. Configuration Reference

All config is in `backend/config.py`, driven by environment variables:

| Variable | Default | Description |
|---|---|---|
| `SECRET_KEY` | `dev-secret-key` | Flask session secret |
| `JWT_SECRET_KEY` | `jwt-secret-key` | JWT signing key |
| `DATABASE_URL` | `sqlite:///instance/hospital.db` | SQLAlchemy DB URI |
| `REDIS_URL` | `redis://localhost:6379/0` | Redis connection |
| `CACHE_EXPIRY` | `300` | Cache TTL in seconds |
| `MAIL_SERVER` | `localhost` | SMTP host |
| `MAIL_PORT` | `1025` | SMTP port |
| `MAIL_USE_TLS` | `False` | TLS toggle |
| `MAIL_USERNAME` | `None` | SMTP username |
| `MAIL_PASSWORD` | `None` | SMTP password |
| `MAIL_DEFAULT_SENDER` | `noreply@hospital.com` | From address |

> ⚠️ **Production:** Always override `SECRET_KEY`, `JWT_SECRET_KEY`, and `DATABASE_URL` via environment variables or a `.env` file.

---

## 10. Data Seeding (Auto-run on startup)

On first `create_app()` call, `initialize_data()` seeds:

**Admin user:**
```
username: admin
email:    admin@hospital.com
password: admin123
role:     admin
```

**Departments (6):**
`Cardiology`, `Neurology`, `Orthopedics`, `Pediatrics`, `Dermatology`, `General Medicine`

---

## 11. Error Handling

| HTTP Code | Meaning | When returned |
|---|---|---|
| 400 | Bad Request | Missing fields, invalid data, slot conflict |
| 401 | Unauthorized | Missing/invalid/expired JWT |
| 403 | Forbidden | Role mismatch (e.g. patient accessing admin route) |
| 404 | Not Found | Doctor/patient/appointment/treatment not found |
| 500 | Internal Server Error | Unhandled exceptions |

All errors return JSON: `{ "error": "<message>" }`

---

## 12. Frontend Architecture

### SPA Routing (Hash-based)
```
/#/login         → LoginForm component
/#/admin         → AdminDashboard component
/#/doctor        → DoctorDashboard component
/#/patient       → PatientDashboard component
```

### Token Storage
- JWT stored in `localStorage` on login
- Attached as `Authorization: Bearer <token>` on every API call
- Cleared on logout or 401 response

### Component Structure
```
app.js                  ← Router, auth state, API helper
components/
  LoginForm.js          ← Login + Register forms
  AdminDashboard.js     ← Admin tabbed interface
  DoctorDashboard.js    ← Doctor tabbed interface
  PatientDashboard.js   ← Patient tabbed interface
```

---

## 13. Deployment Checklist

### Development
```bash
# 1. Install dependencies
pip install -r requirements.txt

# 2. Start Redis (Windows)
.\redis_win\redis-server.exe

# 3. Start Flask
python -m flask --app backend.app run --debug

# 4. Start Celery worker (optional, for background tasks)
celery -A backend.celery_app.celery worker --loglevel=info

# 5. Start Celery Beat (optional, for scheduled tasks)
celery -A backend.celery_app.celery beat --loglevel=info
```

### Production Considerations
- Set all secrets via environment variables (`.env` or host config)
- Switch `DATABASE_URL` to PostgreSQL
- Use Gunicorn: `gunicorn -w 4 "backend.app:app"`
- Use a production SMTP server (SendGrid, SES, etc.)
- Deploy Redis via managed service (Redis Cloud, ElastiCache)
- Run Celery worker as a system service (systemd / supervisor)

---

## 14. Known Limitations (v1.0)

| Limitation | Impact | Future Fix |
|---|---|---|
| SQLite in dev | Not concurrent-write safe | Switch to PostgreSQL for production |
| In-memory cache | Resets on server restart | Switch to Redis cache |
| No refresh tokens | Users must re-login after token expiry | Implement refresh token rotation |
| Celery Beat schedule is coarse | Tasks may fire more often than intended | Tune crontab expressions |
| No input sanitization middleware | XSS risk on text fields | Add bleach/sanitization layer |
| Single-server deployment | Not horizontally scalable as-is | Containerize with Docker + load balancer |

---

*Document prepared for academic and professional review — 24f2007 HMS Project*
