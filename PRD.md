# 📋 Product Requirements Document (PRD)
## Hospital Management System — 24f2007_HMS
**Version:** 1.0  
**Date:** June 2026  
**Author:** 24f2007  
**Status:** ✅ Implemented

---

## 1. Executive Summary

The **Hospital Management System (HMS)** is a full-stack web application that digitizes and streamlines day-to-day hospital operations. It provides a unified, role-based platform for **Administrators**, **Doctors**, and **Patients** to manage appointments, treatments, scheduling, and reporting — replacing fragmented manual workflows with an automated, real-time system.

---

## 2. Problem Statement

Traditional hospital workflows suffer from:

| Pain Point | Impact |
|---|---|
| Paper-based appointment booking | Scheduling conflicts, no real-time visibility |
| No centralized patient records | Doctors lack history context at consultation time |
| Manual reminders | High no-show rates, administrative overhead |
| No automated reporting | Management lacks data to make informed decisions |
| Siloed role operations | Poor coordination between Admin, Doctor, and Patient |

---

## 3. Goals & Objectives

### Primary Goals
- ✅ Provide a **role-based web portal** for Admin, Doctor, and Patient
- ✅ Enable **online appointment booking** with real-time slot management
- ✅ Maintain a **complete medical history** per patient
- ✅ Automate **appointment reminders** via email
- ✅ Generate **monthly activity reports** for doctors and administrators

### Success Metrics
| Metric | Target |
|---|---|
| Appointment booking time | < 60 seconds end-to-end |
| No scheduling conflicts | 0 double-bookings (DB constraint enforced) |
| Email reminder delivery | Daily automated dispatch |
| Report generation | On-demand + monthly scheduled |
| System uptime | 99%+ during business hours |

---

## 4. User Personas

### 🔑 Admin
- **Role:** Hospital administrator managing staff and operations
- **Goals:** Add/remove doctors, monitor all appointments, trigger reports, manage departments
- **Frustrations:** No single view of hospital-wide activity

### 🩺 Doctor
- **Role:** Medical professional consulting patients
- **Goals:** View schedule, record diagnosis & prescriptions, set availability, download monthly reports
- **Frustrations:** Wasted time on paperwork; no quick access to patient history

### 🏥 Patient
- **Role:** Hospital client seeking medical care
- **Goals:** Book appointments, track upcoming visits, view treatment history, export records
- **Frustrations:** Phone-based booking, lost prescription slips, no appointment reminders

---

## 5. Feature Requirements

### 5.1 Authentication & Authorization
| ID | Requirement | Priority |
|---|---|---|
| AUTH-01 | Users can register with email, username, and password | Must Have |
| AUTH-02 | JWT-based login with role detection (admin/doctor/patient) | Must Have |
| AUTH-03 | Role-based route guards; unauthorized access returns 403 | Must Have |
| AUTH-04 | Token expiry handling with user-friendly redirect to login | Must Have |

### 5.2 Admin Portal
| ID | Requirement | Priority |
|---|---|---|
| ADMIN-01 | Dashboard showing total doctors, patients, appointments, upcoming & completed counts | Must Have |
| ADMIN-02 | Add new doctor with specialization, qualifications, contact, and credentials | Must Have |
| ADMIN-03 | Edit doctor profile (name, email, username, specialization, contact) | Must Have |
| ADMIN-04 | Toggle doctor active/inactive status | Must Have |
| ADMIN-05 | View all patients and edit their profiles | Must Have |
| ADMIN-06 | Toggle patient active/inactive status | Must Have |
| ADMIN-07 | View all appointments system-wide (sorted by date descending) | Must Have |
| ADMIN-08 | Search doctors by name or specialization | Should Have |
| ADMIN-09 | Search patients by name, contact, or ID | Should Have |
| ADMIN-10 | Trigger daily appointment reminders manually | Should Have |
| ADMIN-11 | Generate and email monthly activity reports | Should Have |
| ADMIN-12 | View all departments | Must Have |

### 5.3 Doctor Portal
| ID | Requirement | Priority |
|---|---|---|
| DOC-01 | Personal dashboard: upcoming appointments (7-day window), total patients, completed today | Must Have |
| DOC-02 | View, filter appointments by status and date range | Must Have |
| DOC-03 | Mark appointment as completed or cancelled | Must Have |
| DOC-04 | Add treatment record (diagnosis, prescription, notes, next visit date) | Must Have |
| DOC-05 | Edit existing treatment records | Must Have |
| DOC-06 | Set weekly availability (date + start/end time blocks) | Must Have |
| DOC-07 | View patient appointment history | Must Have |
| DOC-08 | Download HTML monthly activity report | Should Have |

### 5.4 Patient Portal
| ID | Requirement | Priority |
|---|---|---|
| PAT-01 | Personal dashboard: upcoming appointments, last treatment summary, total appointment count | Must Have |
| PAT-02 | Browse departments and filter doctors by specialization | Must Have |
| PAT-03 | View doctor availability slots for the next 7 days | Must Have |
| PAT-04 | Book appointment from available slot (past dates blocked) | Must Have |
| PAT-05 | Cancel upcoming (booked) appointments | Must Have |
| PAT-06 | View full appointment history | Must Have |
| PAT-07 | View complete treatment history (diagnosis, prescription, next visit) | Must Have |
| PAT-08 | Export treatment history as downloadable file | Should Have |
| PAT-09 | Update personal profile (age, gender, contact, address, medical history) | Must Have |
| PAT-10 | Auto-expiry of past booked appointments to "expired" status | Must Have |

### 5.5 Background & Automation
| ID | Requirement | Priority |
|---|---|---|
| BG-01 | Daily email reminders to patients with appointments the following day | Must Have |
| BG-02 | Monthly report generation emailed to admin | Should Have |
| BG-03 | Patient treatment history CSV/HTML export | Should Have |
| BG-04 | Celery task queue with Redis broker | Must Have |

---

## 6. Out of Scope (v1.0)
- Online payment / billing module
- Video consultation / telemedicine
- Insurance claim management
- Mobile native app (iOS/Android)
- Multi-hospital / multi-branch support
- Lab test ordering and results management

---

## 7. Constraints
- Must run on Python 3.x + Flask backend
- Frontend must be a Single Page Application (SPA) served by Flask
- Database: SQLite for development (PostgreSQL-ready via `DATABASE_URL` env var)
- Email: Configurable via SMTP (default: local MailHog on port 1025)
- Task queue: Redis + Celery (Redis must be running separately)

---

## 8. Assumptions
- Admin account is pre-seeded (`admin` / `admin123`) on first run
- Six default departments are seeded: Cardiology, Neurology, Orthopedics, Pediatrics, Dermatology, General Medicine
- Doctors are created **only** by Admin (no self-registration for doctors)
- Patients self-register via the signup flow
- Availability is managed in a rolling 7-day window

---

## 9. Acceptance Criteria Summary

| Feature | Acceptance Criteria |
|---|---|
| Login | User is redirected to correct role dashboard on success |
| Book Appointment | Slot is locked; duplicate booking returns 400 error |
| Cancel Appointment | Status changes to "cancelled"; slot is freed |
| Treatment Record | Only the treating doctor can add/edit; appointment auto-marks "completed" |
| Reminders | Email sent to all patients with next-day appointments |
| Reports | Report contains total, completed, and cancelled appointment counts with table |
| Export | Patient receives downloadable treatment history file |

---

*Document prepared for academic and professional review — 24f2007 HMS Project*
