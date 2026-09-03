# Intern Task — College Website Build

**Phase 2 of the Northbridge College Portal CTF Lab**

---

## Overview

You have already completed the base lab VM with Apache and a working CRUD application.
Now you will turn that foundation into a **fictional college website** that serves as the
entry point for the CTF exercise.

This document is your build guide. Follow it step by step. Ask questions when something
is unclear, but keep the build small and explainable.

---

## What you are building

The **Northbridge College Portal** is a fictional campus website with two sections:

| Section | Purpose | Visibility |
|---------|---------|------------|
| **Student Portal** | Registration, profile editing, and personal marks view | Public entry point, requires login |
| **Admin Portal** | Department-clerk marks entry and student-record search | Unlinked route (`/admin`), discoverable through one public clue |

Both sections live on the **same VM and same Apache server**. Use one web port for this
phase (port 80 is fine).

---

## Functional requirements

### Student Portal (the main site)

Build these features first. They must work correctly before you add any challenge logic.

1. **Registration** — A new student can create an account with a username, password, full
   name, email, and department.
2. **Login / Logout** — Registered students can sign in and sign out. Sessions must expire
   or be clearable.
3. **Profile page** — A logged-in student can view and update their contact details
   (phone, address, date of birth).
4. **Marks view** — A logged-in student can see their own marks for registered courses.
   They must NOT see other students' marks through the normal interface.

### Admin Portal (`/admin`)

Build this section after the student portal works.

1. **Admin login** — A separate login form (or the same form with a role check) for
   department clerks.
2. **Marks entry** — Clerks can enter or update marks for students assigned to their
   department.
3. **Student record search** — Clerks can search for a student by name or ID and see a
   limited record view (name, department, enrolled courses, and marks).

---

## What the site must look like

Keep the design simple and functional. You do NOT need a CSS framework or a professional
theme. Requirements:

- Clean HTML pages with basic inline or linked CSS.
- A clear navigation bar showing the site name, a link to Home, and Login/Register (or
  Profile/Logout when logged in).
- The admin portal must NOT appear in the student navigation bar.
- Forms must have labels and basic input validation.
- Error messages must be user-friendly (no raw SQL errors or stack traces visible to users).

Suggested page structure:

```
/                   — Homepage (public)
/register           — Student registration form
/login              — Student login form
/profile            — Student profile (requires login)
/marks              — Student marks view (requires login)
/admin              — Admin portal landing page
/admin/login        — Admin login form
/admin/marks        — Marks entry page (requires admin login)
/admin/students     — Student record search (requires admin login)
```

---

## Seed data

Create a SQL seed script (or equivalent) that populates the database with fictional
data. This data is only for the lab — it must be clearly fake.

### Required seed data

**Students (at least 5):**

| Full Name | Username | Department | Email |
|-----------|----------|------------|-------|
| Aarav Patel | aarav | Computer Science | aarav.patel@northbridge.lab |
| Sara Mitchell | sara | Mathematics | sara.mitchell@northbridge.lab |
| David Kim | david | Physics | david.kim@northbridge.lab |
| Priya Sharma | priya | Computer Science | priya.sharma@northbridge.lab |
| James Okafor | james | English | james.okafor@northbridge.lab |

Use a consistent password for all seeded accounts (e.g., `password123`). Document it
in your setup notes.

**Courses (at least 3):**

| Course Code | Course Name | Department |
|-------------|-------------|------------|
| CS101 | Intro to Programming | Computer Science |
| MATH201 | Linear Algebra | Mathematics |
| PHYS101 | Classical Mechanics | Physics |

**Marks** — Assign realistic marks (0–100) for each student–course pair. Use a mix of
passing and failing grades so the data looks natural.

**Admin accounts (at least 1):**

| Full Name | Username | Role |
|-----------|----------|------|
| Dr. Helen Carter | helen.carter | Department Clerk (Computer Science) |

---

## Database choice

Use **SQLite** for this build. Reasons:

- No separate database server to configure.
- Easy to seed with a SQL file.
- Simple to reset (delete the `.db` file and re-run the seed script).
- The same `.db` file can be committed to Git for reproducibility.

If you have a strong reason to use MySQL or PostgreSQL instead, document why and
provide the equivalent setup and seed scripts. The lab reset procedure must still work.

---

## Tech stack constraints

| Component | Requirement |
|-----------|-------------|
| Web server | Apache (already installed on the VM) |
| Language | PHP, Python (Flask/Django), or plain CGI scripts — your choice |
| Database | SQLite (preferred) or MySQL/PostgreSQL with documented setup |
| Templating | Server-side rendering (no JavaScript-heavy SPAs) |
| Authentication | Session-based, server-side. No JWT or OAuth for this phase. |
| Port | 80 (standard HTTP). No HTTPS needed for the lab. |

If you use PHP, place files in `/var/www/html/` (or the Apache document root).
If you use Python/Flask, configure Apache to proxy to your Flask app on a local port
(e.g., `127.0.0.1:5000`).

---

## File structure suggestion

Organize your code clearly. This is a suggestion — adapt it to your chosen stack.

```
college-portal/
├── app.py                  # (or index.php) Main application entry point
├── database.py             # Database setup and seed script
├── seed.sql                # SQL seed data (if using raw SQL)
├── templates/
│   ├── base.html           # Shared layout (nav, footer)
│   ├── index.html          # Homepage
│   ├── register.html
│   ├── login.html
│   ├── profile.html
│   ├── marks.html
│   └── admin/
│       ├── dashboard.html
│       ├── login.html
│       ├── marks.html
│       └── students.html
├── static/
│   └── style.css
├── requirements.txt        # (Python) pip dependencies
├── README.md               # Setup and usage notes
└── TEST.md                 # Your test results
```

---

## Provisioning

You already have a working Vagrant setup from Phase 1. **Extend it** — do not start over.

1. Update the provisioning script to install any new dependencies (e.g., `php-sqlite3`,
   `python3-flask`, `python3-pip`).
2. Add the college portal files to the VM (either directly or via a shared folder).
3. Configure Apache to serve the college portal.
4. Run the database seed script on first provision.
5. Verify the site loads at `http://localhost` (or the configured port) from the host.

---

## Git workflow

1. Create a new branch from your Phase 1 work:
   ```bash
   git checkout -b feature/college-portal
   ```
2. Commit in small, focused steps:
   - "Add database schema and seed script"
   - "Build student registration and login"
   - "Build profile page and marks view"
   - "Add admin portal with marks entry"
   - "Add student record search"
   - "Update provisioning for college portal"
3. Write a short commit message for each change that explains *what* and *why*.
4. Push the branch and open a pull request (or share the branch for review).

---

## Testing checklist

Test every item below and record results in `TEST.md`. If something fails, note the
error and how you fixed it.

### Student Portal tests

| # | Test | Expected result | Pass/Fail |
|---|------|-----------------|-----------|
| 1 | Visit `/` | Homepage loads with navigation bar | |
| 2 | Click Register | Registration form appears | |
| 3 | Submit registration with valid data | Account created, redirected to login | |
| 4 | Submit registration with duplicate username | Error message, no duplicate created | |
| 5 | Log in with valid credentials | Redirected to profile or marks page | |
| 6 | Log in with invalid credentials | Error message, not logged in | |
| 7 | Visit `/profile` when logged in | Shows own profile details | |
| 8 | Update phone number on profile | Changes saved and visible on reload | |
| 9 | Visit `/marks` when logged in | Shows own marks only | |
| 10 | Visit `/marks` when NOT logged in | Redirected to login page | |
| 11 | Log out | Session ended, cannot access profile or marks | |

### Admin Portal tests

| # | Test | Expected result | Pass/Fail |
|---|------|-----------------|-----------|
| 12 | Visit `/admin` | Admin landing page loads | |
| 13 | Log in as department clerk | Redirected to admin dashboard | |
| 14 | Log in with wrong credentials | Error message, not logged in | |
| 15 | Enter marks for a student | Marks saved and visible | |
| 16 | Search for a student by name | Limited record view shown | |
| 17 | Try to access admin pages as a student | Access denied or redirected | |

### Reproducibility tests

| # | Test | Expected result | Pass/Fail |
|---|------|-----------------|-----------|
| 18 | Run `vagrant destroy -f && vagrant up` | VM provisions successfully | |
| 19 | After re-provision, student portal works | All student tests pass | |
| 20 | After re-provision, admin portal works | All admin tests pass | |
| 21 | Seed data is present after re-provision | 5+ students, 3+ courses, marks, 1+ admin | |

---

## Deliverables

Your submission must include all of the following:

| Deliverable | Format | Location |
|-------------|--------|----------|
| College portal source code | Git branch | `feature/college-portal` |
| Database seed script | SQL file or Python function | `seed.sql` or `database.py` |
| Updated provisioning script | Shell script | `infra/` or `scripts/` |
| Setup and usage notes | Markdown | `README.md` in the portal directory |
| Test results | Markdown | `TEST.md` |
| Git history | Clean commit log | At least 6 focused commits |

---

## Acceptance criteria

A reviewer will check all of these:

- [ ] `vagrant up` provisions a working VM with the college portal running.
- [ ] The student portal supports registration, login, profile editing, and marks view.
- [ ] The admin portal supports marks entry and student record search.
- [ ] Seeded data is present and looks like a real (fictional) college.
- [ ] The admin portal is not linked from the student navigation (but is reachable at `/admin`).
- [ ] SQL errors and stack traces are not visible to users.
- [ ] Reprovisioning restores the site to a clean state.
- [ ] All 21 test cases are documented with pass/fail results.
- [ ] Git history is clean with focused, descriptive commits.
- [ ] You can explain every decision you made.

---

## What this is NOT (yet)

This phase does **not** include the CTF challenge logic. You are building the normal
college website first. The intentionally vulnerable elements (exposed secrets, unsafe
search, hidden flags) will be added in a later phase after this build is reviewed and
accepted.

**Do not weaken authentication, add backdoors, or introduce vulnerability logic at
this stage.** Focus on a clean, working college website.

---

## Hints if you get stuck

- **Choosing a stack:** PHP + SQLite is the fastest path if you are comfortable with it.
  Python Flask + SQLite is equally valid and has better session handling out of the box.
- **Apache config:** Look up `mod_proxy` for Python/Flask apps, or `mod_php` for PHP.
- **Session management:** Use framework-provided session support (e.g., `flask.session`,
  PHP `session_start()`). Do not build your own cookie system.
- **Database schema:** Start with three tables: `students`, `courses`, `marks`.
  Add an `admins` table for the admin portal.
- **Seed script:** Run it once during provisioning, not on every page load. Check if data
  exists before inserting.

---

## Questions

If anything in this document is unclear, ask your instructor before making assumptions.
Write down any assumptions you do make — that is part of the deliverable.

Good luck, and keep it simple.

