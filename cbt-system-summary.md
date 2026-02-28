# OpenCBT — Computer-Based Testing System
### Architecture Blueprint Summary

> **Stack:** Laravel 11 · shadcn/ui · MySQL 8 · Redis · React 18  
> **Scale:** 100–10,000 concurrent users · Self-hosted · Zero licensing cost

---

## Top 5 Essential Features

---

### 1. 🗂️ Question Bank Manager

A centralized, versioned repository for all exam content — the foundation of the entire platform.

| Capability | Detail |
|---|---|
| **Question Types** | MCQ (single/multi), True/False, Essay, Fill-in-Blank, Matching, Ordering, Hotspot, Numeric |
| **Rich Authoring** | TipTap editor with LaTeX math (KaTeX), code highlighting, image/audio/video embedding |
| **Organization** | Hierarchical categories + multi-tag system (subject, topic, difficulty, Bloom's Taxonomy) |
| **Versioning** | Full edit history with diff view; exams lock to a specific version snapshot |
| **Import/Export** | QTI 2.1 XML, CSV, GIFT format import; export to QTI, PDF, CSV |
| **Quality Metrics** | Per-question usage stats, difficulty index (P-value), discrimination index (D-value) |
| **Workflow** | Draft → Review → Approved → Archived with collaborative authoring |

**Why it's essential:** Every exam depends on the question bank. Versioning prevents live exams from breaking when questions are edited. Tagging enables intelligent random exam generation.

---

### 2. 🔒 Secure Exam Session Engine

A hardened, browser-based exam delivery system that deters cheating while remaining accessible.

| Capability | Detail |
|---|---|
| **Anti-Cheating** | Tab-switch detection, fullscreen enforcement, copy-paste disable, right-click disable |
| **Proctoring** | Periodic screenshots, webcam snapshots, client-side face detection (face-api.js / TensorFlow.js) |
| **Session Security** | Unique UUID session token, IP logging, device fingerprinting, IP whitelist per exam |
| **Proctor Dashboard** | Real-time live view of all active sessions, violation alerts, ability to terminate sessions |
| **Violation Policy** | Configurable: warn-only → auto-flag → auto-terminate with threshold settings |
| **Integrity Report** | Post-exam report with timestamped violation log for instructor review |
| **Browser Lockdown** | Full-screen mode with exit detection; configurable violation tolerance |

**Why it's essential:** Academic integrity is the core value proposition of any CBT system. Without robust anti-cheating, the platform cannot be trusted for high-stakes assessments.

---

### 3. ⚡ Auto-Grading Engine + Manual Grading Interface

Instant scoring for objective questions combined with a structured workflow for subjective responses.

| Capability | Detail |
|---|---|
| **Auto-Grading** | Instant scoring on submission for MCQ, True/False, Fill-in-Blank, Matching, Ordering |
| **Answer Matching** | Exact match, case-insensitive, regex pattern, keyword list for short answers |
| **Partial Credit** | Configurable partial credit for multi-select MCQ; negative marking support |
| **Rubric Builder** | Create scoring rubrics with criteria, levels, and point values for essays |
| **Grading Interface** | Side-by-side view: student response + rubric; inline annotation and comment tools |
| **Blind Grading** | Hide student identity from grader to eliminate bias |
| **Re-grading** | Update answer key → automatically recalculate all affected submissions |
| **Grade Release** | Bulk or individual result release with configurable timing |

**Why it's essential:** Grading is the most time-consuming part of exam administration. Auto-grading eliminates manual work for objective questions (typically 80%+ of exam content), while the rubric interface ensures consistent, fair evaluation of subjective responses.

---

### 4. 📊 Analytics Dashboard

Comprehensive performance reporting that transforms raw scores into actionable educational insights.

| Capability | Detail |
|---|---|
| **Student View** | Score, percentage, pass/fail, time taken, question-by-question breakdown |
| **Exam Analytics** | Score distribution histogram, mean/median/mode, standard deviation, pass rate |
| **Question Analytics** | Difficulty index (P-value), discrimination index (D-value), distractor analysis |
| **Trend Analysis** | Student performance trends across multiple exams over time |
| **Cohort Reports** | Class/department comparison; Bloom's Taxonomy performance breakdown |
| **Export** | PDF, CSV, XLSX export; scheduled email delivery of reports |
| **Certificates** | Configurable certificate templates auto-issued to passing students |
| **Visualizations** | Interactive charts via Recharts; leaderboard (optional, per-exam toggle) |

**Why it's essential:** Data-driven insights allow instructors to identify struggling students early, improve question quality, and demonstrate learning outcomes to institutional stakeholders.

---

### 5. 🛡️ Role-Based Access Control (RBAC)

Granular, policy-driven permission system ensuring every user can only access what they're authorized for.

| Role | Core Permissions |
|---|---|
| **Admin** | Full system access: user management, institution settings, audit logs, all data |
| **Instructor** | Create/manage own exams and questions, view own students' results, grade submissions |
| **Student** | Take assigned exams, view own results and performance history only |
| **Proctor** | Monitor assigned live exam sessions, view violation logs, terminate sessions |

| Capability | Detail |
|---|---|
| **Implementation** | Spatie Laravel-Permission with database-driven roles and permissions |
| **Resource Policies** | Laravel Policies for object-level authorization (e.g., only exam owner can edit) |
| **Permission Overrides** | User-level overrides (e.g., extended time for accessibility accommodations) |
| **SSO Integration** | SAML 2.0 / OAuth2 for institutional identity providers (Google Workspace, Microsoft Entra) |
| **MFA Enforcement** | Required for Admin and Instructor; optional for Student |
| **Audit Trail** | All permission-sensitive actions logged with user, timestamp, IP, before/after values |

**Why it's essential:** Without RBAC, a student could access other students' answers, an instructor could modify another's exam, or a proctor could release grades. RBAC is the security backbone of the entire platform.

---

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                        CLIENT LAYER                             │
│  React 18 + shadcn/ui + Tailwind CSS + Inertia.js/SPA          │
│  TipTap Editor · KaTeX · Monaco Editor · face-api.js           │
│  Zustand (state) · React Query (server cache) · IndexedDB      │
└──────────────────────────┬──────────────────────────────────────┘
                           │ HTTPS / WSS
┌──────────────────────────▼──────────────────────────────────────┐
│                      WEB SERVER LAYER                           │
│              Nginx 1.24 (reverse proxy, SSL termination)        │
│              Rate limiting · Gzip · Security headers            │
└──────────────────────────┬──────────────────────────────────────┘
                           │
┌──────────────────────────▼──────────────────────────────────────┐
│                    APPLICATION LAYER                            │
│              Laravel 11 (PHP 8.3-FPM)                          │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────────────┐  │
│  │  Auth &  │ │ Question │ │  Exam    │ │    Grading &     │  │
│  │  RBAC    │ │  Bank    │ │ Delivery │ │    Analytics     │  │
│  └──────────┘ └──────────┘ └──────────┘ └──────────────────┘  │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────────────┐  │
│  │Proctoring│ │Scheduler │ │  Notif.  │ │   REST API       │  │
│  │ Engine   │ │& Queues  │ │  System  │ │  (OpenAPI 3.0)   │  │
│  └──────────┘ └──────────┘ └──────────┘ └──────────────────┘  │
└──────────────────────────┬──────────────────────────────────────┘
                           │
┌──────────────────────────▼──────────────────────────────────────┐
│                      DATA LAYER                                 │
│  ┌─────────────────┐  ┌──────────────┐  ┌───────────────────┐  │
│  │   MySQL 8.0     │  │   Redis 7    │  │  File Storage     │  │
│  │  (Primary DB)   │  │ Sessions     │  │  S3/MinIO         │  │
│  │  Read Replicas  │  │ Cache        │  │  (media, PDFs,    │  │
│  │  Binary Logging │  │ Queues       │  │   screenshots)    │  │
│  └─────────────────┘  │ Pub/Sub      │  └───────────────────┘  │
│                        └──────────────┘                         │
└─────────────────────────────────────────────────────────────────┘
                           │
┌──────────────────────────▼──────────────────────────────────────┐
│                   REAL-TIME LAYER                               │
│         Soketi (self-hosted WebSocket server)                   │
│    Laravel Echo · Live proctor dashboard · Notifications        │
└─────────────────────────────────────────────────────────────────┘
```

---

## Implementation Roadmap

### Phase 1 — MVP (Weeks 1–8)
**Goal:** Working end-to-end exam flow for objective questions

- [ ] Laravel 11 project setup with MySQL, Redis, Sanctum auth
- [ ] User management with RBAC (Spatie Laravel-Permission)
- [ ] Question bank: MCQ and True/False with basic rich text editor
- [ ] Exam builder: manual question selection, time limits, scheduling
- [ ] Student exam-taking interface with auto-save, timer, submission
- [ ] Auto-grading for objective questions
- [ ] Basic results view for students and instructors
- [ ] Admin user management panel
- [ ] Docker Compose development environment

### Phase 2 — Full Feature Set (Weeks 9–18)
**Goal:** Feature-complete platform ready for pilot testing

- [ ] Additional question types: Essay, Fill-in-Blank, Matching, Ordering, Hotspot
- [ ] Manual grading interface with rubric builder
- [ ] Anti-cheating: tab-switch, fullscreen, copy-paste, IP logging, device fingerprinting
- [ ] Webcam snapshot and screenshot capture
- [ ] Proctor live dashboard with real-time violation alerts (Laravel Echo + Soketi)
- [ ] Comprehensive analytics dashboard with Recharts
- [ ] Email + in-app notification system
- [ ] Question bank import/export (QTI, CSV)
- [ ] Exam randomization (question pools, shuffle)
- [ ] Certificate generation (DomPDF)

### Phase 3 — Production Hardening (Weeks 19–26)
**Goal:** Production-ready, documented, tested platform

- [ ] Performance optimization: query optimization, Redis caching, async queues
- [ ] Load testing with k6 (target: 10,000 concurrent users)
- [ ] Security audit: penetration testing, OWASP Top 10 review
- [ ] SSO integration (SAML 2.0 / OAuth2)
- [ ] LTI 1.3 provider for LMS integration
- [ ] REST API with OpenAPI 3.0 documentation
- [ ] GDPR compliance features
- [ ] Offline resilience (IndexedDB caching)
- [ ] CI/CD pipeline (GitHub Actions): PHPUnit/Pest, PHPStan, Playwright E2E
- [ ] Production deployment guide

---

## Database Entity Summary

| Entity Group | Tables |
|---|---|
| **Identity** | users, roles, permissions, departments, institutions |
| **Question Bank** | questions, question_options, question_versions, question_categories, question_tags, question_media |
| **Exam Management** | exams, exam_sections, exam_questions, exam_question_pools, exam_schedules, exam_assignments |
| **Exam Delivery** | exam_sessions, exam_answers |
| **Grading** | exam_results, question_scores, rubrics, rubric_criteria, rubric_levels |
| **Proctoring** | proctoring_events, proctoring_media |
| **Communication** | notifications, announcements |
| **Certificates** | certificates, certificate_templates |
| **System** | audit_logs, settings, jobs, failed_jobs, sessions |

**Total: ~35 tables** with full referential integrity and optimized indexes.

---

## Security Checklist

- [x] HTTPS/TLS 1.3 with HSTS
- [x] HttpOnly + Secure + SameSite=Strict cookies (Sanctum)
- [x] CSRF protection on all state-changing requests
- [x] SQL injection prevention (Eloquent ORM + PDO prepared statements)
- [x] XSS prevention (Blade escaping + CSP header + DOMPurify)
- [x] Rate limiting (login, API, exam submission)
- [x] Bcrypt password hashing (cost factor 12)
- [x] Multi-factor authentication (TOTP)
- [x] Role-Based Access Control + Laravel Policies
- [x] Exam session UUID token validation
- [x] IP address logging and whitelist enforcement
- [x] Device fingerprinting
- [x] AES-256 encryption for sensitive data at rest
- [x] Signed URLs for file downloads (15-min expiry)
- [x] File upload validation + virus scanning (ClamAV)
- [x] Security headers (X-Frame-Options, X-Content-Type-Options)
- [x] Comprehensive audit logging
- [x] Account lockout after failed login attempts
- [x] Automated encrypted database backups
- [x] Dependency vulnerability scanning in CI/CD
- [x] GDPR compliance (data export + deletion)

---

## Open-Source License Summary

All dependencies use **MIT**, **Apache 2.0**, or **ISC** licenses — fully compatible with open-source distribution and commercial use.

---

*Generated for OpenCBT v1.0 Architecture Blueprint · Laravel 11 · shadcn/ui · MySQL 8 · Redis 7*
