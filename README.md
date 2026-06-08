# 🏥 Hospital Management System
### 24f2007_HMS — Full-Stack Flask + Vanilla JS Web Application

[![Python](https://img.shields.io/badge/Python-3.10+-3776AB?logo=python&logoColor=white)](https://python.org)
[![Flask](https://img.shields.io/badge/Flask-3.0.0-000000?logo=flask)](https://flask.palletsprojects.com)
[![SQLite](https://img.shields.io/badge/Database-SQLite%20%2F%20PostgreSQL-003B57?logo=sqlite)](https://sqlite.org)
[![Redis](https://img.shields.io/badge/Cache%20%26%20Broker-Redis-DC382D?logo=redis)](https://redis.io)
[![Celery](https://img.shields.io/badge/Tasks-Celery-37814A?logo=celery)](https://docs.celeryq.dev)
[![JWT](https://img.shields.io/badge/Auth-JWT-000000?logo=jsonwebtokens)](https://jwt.io)

---

## 📖 Overview

A production-grade **Hospital Management System** built as a Single Page Application (SPA). It provides role-based dashboards for **Administrators**, **Doctors**, and **Patients** — enabling seamless appointment management, digital treatment records, automated email reminders, and monthly reporting.

> **Course Project:** IIT Madras BS Degree — Application Development II (24f2007)

---

## ✨ Features at a Glance

| Role | Capabilities |
|---|---|
| 🔑 **Admin** | Manage doctors & patients, view all appointments, trigger reminders, generate reports |
| 🩺 **Doctor** | View schedule, set availability, record diagnoses & prescriptions, download monthly report |
| 🏥 **Patient** | Book appointments, view treatment history, cancel visits, export health records |

### Core Highlights
- ✅ **JWT Authentication** — stateless, role-aware, token-based auth
- ✅ **Real-time Slot Management** — DB-level unique constraint prevents double-booking
- ✅ **Automated Reminders** — Celery + Redis sends daily appointment emails
- ✅ **Monthly Reports** — auto-generated and emailed to admin on schedule
- ✅ **Smart Caching** — Redis-backed cache for dashboard stats (auto-invalidated on writes)
- ✅ **Auto-expiry** — past `booked` appointments automatically transition to `expired`
- ✅ **Export** — Patients can export their treatment history as downloadable files
- ✅ **SPA Frontend** — Zero-reload experience via hash-based client-side routing

---

## 🚀 Quick Start

### Prerequisites

| Tool | Version |
|---|---|
| Python | 3.10+ |
| pip | Latest |
| Redis | 7.x (bundled for Windows in `redis_win/`) |

### 1. Clone & Install

```bash
git clone <your-repo-url>
cd 24f2007_HMS

pip install -r requirements.txt
```

### 2. Configure Environment (Optional)

Create a `.env` file in the project root:

```env
# Flask
SECRET_KEY=your-super-secret-key
DATABASE_URL=sqlite:///instance/hospital.db   # or PostgreSQL URI

# JWT
JWT_SECRET_KEY=your-jwt-secret

# Redis
REDIS_URL=redis://localhost:6379/0

# Email (default: MailHog on port 1025 for dev)
MAIL_SERVER=localhost
MAIL_PORT=1025
MAIL_USE_TLS=False
MAIL_USERNAME=
MAIL_PASSWORD=
MAIL_DEFAULT_SENDER=noreply@hospital.com
```

### 3. Start Redis

**Windows (bundled binary):**
```bash
.\redis_win\redis-server.exe
```

**Linux / macOS:**
```bash
redis-server
```

### 4. Run the Application

```bash
python -m flask --app backend.app run --debug
```

🌐 Open your browser at: **http://localhost:5000**

### 5. Start Celery (for background tasks)

```bash
# Worker
celery -A backend.celery_app.celery worker --loglevel=info

# Beat scheduler (for timed/recurring tasks)
celery -A backend.celery_app.celery beat --loglevel=info
```

---

## 🔐 Default Credentials

> ⚠️ Change these immediately in a production environment!

| Role | Username | Password |
|---|---|---|
| Admin | `admin` | `admin123` |
| Doctor | Created by Admin | Set by Admin |
| Patient | Register via UI | Set by user |

---

## 📂 Project Structure

```
24f2007_HMS/
├── backend/
│   ├── app.py              # Flask app factory
│   ├── config.py           # Env-driven configuration
│   ├── celery_app.py       # Celery + Beat schedule
│   ├── email_utills.py     # Flask-Mail setup
│   ├── models/
│   │   └── __init__.py     # All SQLAlchemy models (6 tables)
│   ├── routes/
│   │   ├── auth.py         # /api/auth/*
│   │   ├── admin.py        # /api/admin/*
│   │   ├── doctor.py       # /api/doctor/*
│   │   ├── patient.py      # /api/patient/*
│   │   └── utils.py        # Role guard decorators
│   ├── tasks/
│   │   ├── reminders.py    # Email reminder task
│   │   ├── reports.py      # Monthly report task
│   │   └── exports.py      # Patient export task
│   └── cache/              # Caching utilities
├── static/
│   ├── js/
│   │   ├── app.js          # SPA router + fetch helper
│   │   └── components/     # Dashboard components (JS)
│   └── images/
├── templates/
│   └── index.html          # SPA shell
├── exports/                # Generated export files
├── instance/
│   └── hospital.db         # SQLite DB (auto-created)
├── redis_win/              # Redis for Windows
├── requirements.txt
├── trigger_tasks.py        # Manual task trigger CLI
├── PRD.md                  # Product Requirements Document
├── TRD.md                  # Technical Requirements Document
└── README.md
```

---

## 🗄️ Database Schema (ER Summary)

```
users ──1:1── doctors ──M:1── departments
users ──1:1── patients
doctors ──1:M── appointments ──1:1── treatments
patients ──1:M── appointments
doctors ──1:M── doctor_availability
```

**Tables:** `users`, `departments`, `doctors`, `patients`, `appointments`, `treatments`, `doctor_availability`

**Key constraint:** `UNIQUE(doctor_id, date, time)` on appointments — prevents double-booking at the DB level.

---

## 🌐 API Endpoints Summary

### Auth (`/api/auth`)
| Method | Endpoint | Description |
|---|---|---|
| POST | `/login` | Login → returns JWT |
| POST | `/register` | Register new patient |
| GET | `/me` | Current user info |

### Admin (`/api/admin`) — Requires `admin` role
| Method | Endpoint | Description |
|---|---|---|
| GET | `/dashboard` | Hospital-wide stats |
| GET/POST | `/doctors` | List / Add doctors |
| PUT | `/doctors/<id>` | Update doctor |
| PUT | `/doctors/<id>/toggle` | Activate/deactivate |
| GET | `/patients` | List all patients |
| PUT | `/patients/<id>` | Update patient |
| GET | `/appointments` | All appointments |
| POST | `/reminders/trigger` | Manually fire reminders |
| POST | `/reports/monthly` | Generate monthly report |

### Doctor (`/api/doctor`) — Requires `doctor` role
| Method | Endpoint | Description |
|---|---|---|
| GET | `/dashboard` | Doctor stats + schedule |
| GET | `/appointments` | Own appointments (filterable) |
| PUT | `/appointments/<id>` | Update status/notes |
| GET/POST | `/availability` | Get / Set weekly slots |
| POST | `/treatment` | Add treatment record |
| PUT | `/treatment/<id>` | Edit treatment |
| GET | `/patients/<id>/history` | Patient history |
| GET | `/report/monthly` | Download HTML report |

### Patient (`/api/patient`) — Requires `patient` role
| Method | Endpoint | Description |
|---|---|---|
| GET | `/dashboard` | Patient stats |
| GET/PUT | `/profile` | View / Update profile |
| GET | `/departments` | Browse departments |
| GET | `/doctors` | List doctors (filter by dept) |
| GET | `/doctors/<id>/availability` | Available slots |
| POST | `/appointments` | Book appointment |
| GET | `/appointments` | Appointment history |
| DELETE | `/appointments/<id>` | Cancel appointment |
| GET | `/treatments` | Full treatment history |
| POST | `/export` | Export health records |

---

## ⚙️ Background Tasks

| Task | Trigger | Description |
|---|---|---|
| `send_daily_reminders` | Celery Beat / Manual API | Emails patients with next-day appointments |
| `generate_monthly_reports` | Celery Beat / Manual API | Sends monthly stats report to admin |
| `export_patient_treatments` | On patient request | Generates downloadable treatment history file |

**Manually trigger tasks (dev helper):**
```bash
python trigger_tasks.py
```

---

## 🔒 Security Notes

- Passwords hashed with **PBKDF2-SHA256** (Werkzeug)
- JWT signed and validated server-side on every protected request
- Role-based access enforced via Python decorators — no frontend bypass possible
- JWT also accepted via query param (`?token=`) for file download endpoints

---

## 🧪 Testing the Application

### 1. Login as Admin
- URL: `http://localhost:5000`
- Credentials: `admin` / `admin123`

### 2. Add a Doctor
- Admin Dashboard → Doctors tab → Add Doctor form

### 3. Register a Patient
- Click "Register" on login page → complete form

### 4. Book an Appointment
- Login as Patient → Select department → Choose doctor → Pick available slot

### 5. Complete a Consultation
- Login as Doctor → Appointments → Mark appointment completed → Add treatment

### 6. Trigger Reminders
- Login as Admin → Dashboard → "Trigger Reminders" button

---

## 📦 Dependencies

```
Flask==3.0.0
Flask-SQLAlchemy==3.1.1
Flask-JWT-Extended==4.6.0
Flask-CORS==4.0.0
Flask-Mail==0.9.1
redis==5.0.1
celery==5.3.4
python-dotenv==1.0.0
werkzeug==3.0.1
```

Install all with:
```bash
pip install -r requirements.txt
```

---

## 🏭 Production Deployment

```bash
# 1. Use PostgreSQL
export DATABASE_URL=postgresql://user:password@host/dbname

# 2. Set strong secrets
export SECRET_KEY=<strong-random-string>
export JWT_SECRET_KEY=<strong-random-string>

# 3. Use production SMTP
export MAIL_SERVER=smtp.sendgrid.net
export MAIL_PORT=587
export MAIL_USE_TLS=True

# 4. Run with Gunicorn
pip install gunicorn
gunicorn -w 4 -b 0.0.0.0:8000 "backend.app:app"

# 5. Run Celery as a service
celery -A backend.celery_app.celery worker -D
celery -A backend.celery_app.celery beat -D
```

---

## 📄 Documentation

| Document | Description |
|---|---|
| [PRD.md](./PRD.md) | Product Requirements — features, personas, acceptance criteria |
| [TRD.md](./TRD.md) | Technical Requirements — architecture, schema, API reference, security |
| [README.md](./README.md) | This file — setup guide and project overview |

---

## 👤 Author

**Student ID:** 24f2007  
**Project:** IIT Madras BS — Application Development II  
**Stack:** Flask · SQLAlchemy · JWT · Celery · Redis · Vanilla JS

---

*Built with ❤️ for the IIT Madras BS Program*
