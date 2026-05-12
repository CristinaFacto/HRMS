# HR Management System (HRMS)

> A fully browser-based Human Resources Management System built with vanilla HTML, CSS, and JavaScript — deployed on Vercel with Supabase as the backend.

[![Status](https://img.shields.io/badge/Status-Live%20%26%20Operational-27ae60?style=flat-square)](https://hrms-chi-seven.vercel.app)
[![Deployment](https://img.shields.io/badge/Deployed%20on-Vercel-000000?style=flat-square&logo=vercel)](https://hrms-chi-seven.vercel.app)
[![Backend](https://img.shields.io/badge/Backend-Supabase-3ECF8E?style=flat-square&logo=supabase)](https://supabase.com)

---

## 🔗 Live URL

**https://hrms-chi-seven.vercel.app**

---

## 📋 Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Tech Stack](#tech-stack)
- [User Roles](#user-roles)
- [Project Structure](#project-structure)
- [Authentication Flow](#authentication-flow)
- [Database Schema](#database-schema)
- [Modules](#modules)
- [Deployment](#deployment)
- [Sprint Summary](#sprint-summary)

---

## Overview

HRMS is a role-based employee management platform. It allows organizations to manage employee records, departments, job positions, and employment history through a clean, browser-based interface. Access is controlled by a three-tier role system (Super Admin → Admin → User) enforced through Supabase's Row-Level Security.

No backend server is required — all logic runs in the browser, with Supabase handling authentication, data storage, and security policies.

---

## Features

- **Employee Management** — Add, edit, soft-delete, and recover employee records
- **Department & Job Position Management** — Maintain organizational structure
- **Job History Tracking** — Full employment history per employee
- **Role-Based Dashboards** — Separate interfaces for Super Admin, Admin, and User
- **Registration Approval Workflow** — New users wait for admin approval before gaining access
- **User Account Controls** — Block, deactivate, approve, or reject user accounts
- **Audit Logging** — In-session log of all create/update/delete actions
- **Deleted Items Recovery** — Soft-delete architecture with full recovery support
- **Google OAuth** — Sign in or register using a Google account
- **Password Reset** — Email-based reset via Supabase
- **Real-time Status Polling** — Waiting page auto-redirects upon admin approval (every 10 seconds)
- **Mid-session Block Detection** — Active sessions checked every 30 seconds; blocked accounts are automatically redirected

---

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Frontend | Vanilla HTML5, CSS3, JavaScript (ES2020+) |
| Backend / Auth | Supabase (PostgreSQL 15, GoTrue Auth) |
| Auth Providers | Email/Password + Google OAuth 2.0 |
| Hosting | Vercel (Static site, global CDN) |
| JS Client Library | @supabase/supabase-js v2 (via jsDelivr CDN) |

---

## User Roles

| Role | Dashboard | Key Permissions |
|------|-----------|----------------|
| **Super Admin** | `superAdmin.html` | Full CRUD, permanent delete, block/deactivate users, all audit logs |
| **Admin** | `admin.html` | Full CRUD (no permanent delete/deactivate), approve/reject registrations |
| **User** | `user.html` | Read-only — view employees, departments, jobs, job history, and profiles |

### Account Status Values

| Status | Description |
|--------|-------------|
| `active` | Approved — can log in normally |
| `inactive` | Newly registered — awaiting admin approval |
| `blocked` | Blocked by admin — redirected to `blocked.html` |
| `deactivated` | Permanently deactivated by Super Admin — redirected to `deactivated.html` |
| `rejected` | Registration rejected by admin |

---

## Project Structure

```
HRMS-main/
├── index.html           # Login & Registration (public)
├── auth-callback.html   # Google OAuth callback handler
├── waiting.html         # Pending-approval holding screen
├── blocked.html         # Blocked account notice
├── deactivated.html     # Deactivated account notice
├── admin.html           # Admin dashboard (83.8 KB)
├── superAdmin.html      # Super Admin dashboard (92.5 KB)
└── user.html            # Employee read-only dashboard (37.7 KB)
```

Each HTML file is self-contained — it includes all CSS styles, JavaScript logic, and Supabase client initialization inline.

---

## Authentication Flow

### Email / Password

1. User registers on `index.html` (First Name, Last Name, Username, Email, Password)
2. Supabase creates an auth user; a database trigger inserts a `profiles` row with `status = 'inactive'`
3. User receives an email confirmation link
4. After confirming, user logs in; `status = 'inactive'` redirects to `waiting.html`
5. Admin approves the registration → `status` updated to `'active'`
6. `waiting.html` detects approval (10-second poll) and redirects to the role's dashboard

### Google OAuth

1. User clicks **Register with Google** or **Sign in with Google**
2. Supabase OAuth redirects to Google, then back to `auth-callback.html`
3. `auth-callback.html` checks `profiles.status` and routes accordingly

### Password Reset

1. User enters email on the Login form and clicks **Forgot Password**
2. Supabase sends a reset link; user sets a new password via the Supabase-hosted UI

---

## Database Schema

### `profiles`
| Column | Type | Description |
|--------|------|-------------|
| `id` | UUID (PK) | Matches `auth.users.id` |
| `username` | TEXT | Display username |
| `email` | TEXT | User email |
| `full_name` | TEXT | Full name |
| `role` | TEXT | `superadmin` \| `admin` \| `user` |
| `status` | TEXT | `active` \| `inactive` \| `blocked` \| `deactivated` \| `rejected` |
| `approved_by` | TEXT | Username of approving admin |
| `approved_at` | TIMESTAMPTZ | Approval timestamp |

### `employee`
| Column | Type | Description |
|--------|------|-------------|
| `id` | SERIAL (PK) | Internal ID |
| `empno` | TEXT | 5-digit employee number |
| `username` | TEXT | Linked username |
| `lastname` / `firstname` | TEXT | Employee name |
| `gender` | TEXT | Male \| Female \| Other |
| `hiredate` / `sepdate` | DATE | Employment dates |
| `deptid` | INTEGER | FK → `department.id` |
| `jobid` | INTEGER | FK → `job.id` |
| `salary` | NUMERIC | Current salary |
| `email` / `phone` | TEXT | Contact details |
| `deleted` / `blocked` | BOOLEAN | Soft-delete and block flags |

### `department`
`id`, `code` (e.g. HR, IT), `name`, `deleted`

### `job`
`id`, `code`, `description`, `deptid`, `deleted`

### `jobhistory`
`id`, `empid`, `jobid`, `deptid`, `effdate`, `salary`, `deleted`

---

## Modules

### Employee Management
- Full CRUD (Super Admin & Admin); read-only view (User)
- Bulk delete via checkbox selection
- Search by text, filter by department/status, sort A–Z / Z–A
- Stats bar showing active / blocked / inactive counts
- Employee Profile modal with full details + job history (Admin/SA)

### Department Management
Add, edit, soft-delete, and recover department records with department codes.

### Job Position Management
Add, edit, soft-delete, and recover job positions linked to departments.

### Job History
Track each position an employee has held, with effective date and salary at the time.

### Deleted Items
Consolidated view of all soft-deleted records. Super Admin can permanently delete; all admins can recover.

### User Management
- **Users tab** — list all accounts with block/deactivate/edit-role controls
- **Pending tab** — approve or reject new registrations (with optional rejection note)
- **All Registrations tab** — full history; Reconsider moves rejected back to Pending

### Audit Log
Every Add, Edit, Delete, Restore, Block action is logged in-session with actor username, timestamp, entity type, and descriptive note. Viewable per section via modal.

---

## Deployment

The application is deployed as a **static site on Vercel** — no build step required.

| Item | Value |
|------|-------|
| Platform | Vercel |
| Live URL | https://hrms-chi-seven.vercel.app |
| Supabase Project | https://fpqmxvgehtyqivkuiltw.supabase.co |
| OAuth Redirect | https://hrms-chi-seven.vercel.app/auth-callback.html |
| HTTPS | Enforced (Let's Encrypt via Vercel) |

### Supabase Configuration
- **Auth Providers:** Email/Password (with email confirmation) + Google OAuth 2.0
- **Row-Level Security (RLS):** Enabled on all tables
- **Trigger:** `on_auth_user_created` → auto-inserts a `profiles` row with `status = 'inactive'`
- **Anon key:** Embedded in HTML files — safe to expose; RLS is the actual security layer

---

## Sprint Summary

### Sprint 1 — Foundation & Core Setup
- Set up Vercel project and Supabase backend (tables, auth, trigger)
- Built `index.html` with email/password registration and Google OAuth
- Implemented role-based routing post-login
- Created `waiting.html`, `blocked.html`, `deactivated.html`
- Initial Vercel deployment

### Sprint 2 — Role-Based Features & Admin Tools
- Built `admin.html` with full CRUD for Employees, Departments, Jobs, Job History
- Built `user.html` as a read-only employee dashboard with profile view
- Implemented soft-delete architecture and Deleted Items section
- Added registration approval/rejection workflow (Pending tab)
- Added `auth-callback.html` for Google OAuth routing
- Added search, filter, and sort functionality across all tables

### Sprint 3 — Super Admin Dashboard, Audit Log & Final Polish
- Built `superAdmin.html` with permanent delete, block/unblock, deactivate capabilities
- Implemented in-session audit logging for all CUD actions
- Added real-time mid-session block detection (30-second poll)
- Added All Registrations tab with Reconsider action
- Added Add User form for Super Admin
- Replaced `alert()` and `confirm()` with custom toast notifications and modal dialogs
- Fixed status-check bug (`'active'` vs `'approved'` discrepancy)
- Final Vercel production deployment

---

## License

This project is for internal / academic use. All rights reserved.
