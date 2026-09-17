# Intern Task — Phase 3: Production Hardening & CTF Integration

**Northbridge College Portal — Provisioning-Only Build + Hidden Admin + Full-DB Exfiltration**

> **Prerequisite:** Phase 2 is complete and accepted. The college website is functional on the Vagrant VM (student portal + admin portal, Apache, SQLite, seeded data). This phase does not rebuild the site — it hardens, polishes, and converts the existing site into a reproducible, production-style CTF target.

---

## 1. Goal

Make the lab **production-realistic and self-contained**:

1. The VM builds entirely from the **provision script** — no host folder sync, no manual copy, no local file dependency.
2. The public site looks like a real (fictional) college portal with proper error handling.
3. The admin portal is **truly hidden** — discoverable only by directory enumeration, not by hints.
4. Admin access requires **password spraying** against a decoy credential file, not a single leaked password.
5. Even after admin login, the student sees only a **limited view**; the final objective is to **exfiltrate the whole database**, and the flag is **not stored inside that database**.

This matches how the real CTF VM will be deployed: a fresh VM is created, one script runs, and the exercise is ready.

---

## 2. What you are changing (and what you are NOT)

| Keep from Phase 2 | Change in Phase 3 |
|---|---|
| Same fictional data (students, courses, marks, faculty) | Remove all `synced_folder` / shared-folder dependencies |
| Same routes (`/`, `/register`, `/login`, `/profile`, `/marks`, `/admin/*`) | Add proper `404` / `403` / `500` handling — no raw errors |
| Same stack (Apache + PHP or Python/Flask + SQLite) | Make admin undiscoverable without enumeration |
| Same single-VM, host-only networking | Move credential clue from `robots.txt` to a sprayable `.env` decoy |
| | Restrict admin to a limited record view; full dump requires exploitation of one documented search field |
| | Move flag out of the database to a filesystem location |

**Do NOT** add new VMs, cloud hosts, real credentials, email, payment flows, or host-level weaknesses. All weaknesses stay inside the fictional app and seeded data.

---

## 3. Requirement 1 — Provisioning-Only Deployment

The finished lab must satisfy: **`git clone <repo> && vagrant up` builds the complete CTF** with no extra steps.

### 3.1 Remove host sync

- In `Vagrantfile`, disable the default synced folder:

  ```ruby
  config.vm.synced_folder ".", "/vagrant", disabled: true
  # (and disable any other synced_folder lines)
  ```

- Verify that **no** application file is read from `/vagrant` at runtime. Every file must come from inside the VM after provisioning.

### 3.2 Provision script does everything

Create or update `infra/provision.sh` (or `scripts/provision.sh` — pick one and document it) so it alone:

1. Updates packages and installs dependencies (Apache, PHP or Python + `venv`/`pip`, `sqlite3`, `git`, `curl`).
2. Clones or pulls the portal source **from a GitHub repository** to a VM-local path (e.g., `/opt/northbridge` or `/var/www/html`). Use a variable at the top of the script:

   ```bash
   PORTAL_REPO="https://github.com/<org>/northbridge-portal.git"
   PORTAL_BRANCH="main"
   PORTAL_DIR="/var/www/html"
   ```

3. Sets permissions (`chown www-data`, `chmod` for DB and uploads) and enables Apache modules / virtual host (`a2enmod proxy proxy_http rewrite` for Flask, or `a2enmod php` for PHP).
4. Creates the SQLite database and runs the seed script **idempotently** (check for existing DB/tables before inserting; do not duplicate on re-run).
5. Places the decoy `.env` file, the flag file (Section 7), and any static assets.
6. Restarts Apache and verifies the site responds on `127.0.0.1:80` (or the forwarded host port).

> The provision script must be **idempotent**: running `vagrant provision` a second time does not error or corrupt data. Use guards like `if [ ! -f "$DB" ]; then ...; fi` and `CREATE TABLE IF NOT EXISTS`.

### 3.3 Reproducibility contract

- A reviewer must be able to run `vagrant destroy -f && vagrant up` from a **clean clone** on a different host and get an identical working lab.
- Do not commit the SQLite `.db` file if the provision script generates it. If you do commit a seed `.sql`, the script must still be the source of truth.
- Document the GitHub repo URL, branch, and VM-local path in both the script header and `README.md`.

---

## 4. Requirement 2 — Functional Polish & Error Handling

The public college site must feel complete, not like a skeleton.

### 4.1 Required polish

- Consistent header/nav and footer on every page (college name, Home, Login/Register or Profile/Logout when authenticated).
- Clean, readable forms with labels and server-side validation.
- No raw stack traces, SQL errors, or framework debug pages visible to unauthenticated users.

### 4.2 Custom error pages

| Condition | Expected behaviour |
|---|---|
| Unknown route (e.g., `/nope123`) | Custom `404` page — college-branded "Page not found", HTTP 404 status, no Apache default page |
| Unauthenticated access to `/profile`, `/marks`, `/admin/*` | Redirect to `/login` or render `403 Forbidden` with a friendly message and link back — HTTP 302 or 403, never 500 |
| Invalid form input | Inline validation message, form re-rendered, no 500 |
| Unhandled server error | Generic `500` page — "Something went wrong, please try again" — details written only to `/var/log/apache2/error.log` |

Configure this in Apache (`ErrorDocument 404 /404.html`) or in the app framework's error handlers. Test with `curl -i http://localhost/nonexistent` and confirm the status code.

---

## 5. Requirement 3 — Truly Hidden Admin Portal

### 5.1 Hide without hints

- **Remove every Phase 2 discovery clue** for `/admin`: no link in nav/footer/sitemap, no `robots.txt` entry, no HTML comment, no `sitemap.xml`, no reference in `README` or client JS.
- `robots.txt` (if present) must be either absent or generic (e.g., only `User-agent: *\nDisallow:`) — it must **not** mention `admin`, `.env`, or any CTF path.
- The admin route must still be reachable at `/admin` (and `/admin/login`) and return `200` when discovered, but be indistinguishable from a normal hidden path until enumerated.

### 5.2 Discovery method

- Students must discover `/admin` by **directory enumeration** (e.g., `ffuf`, `gobuster`, `dirb`, `feroxbuster` with common wordlists like `common.txt` / `directory-list-2.3-small.txt`). Ensure your path choice (`/admin`) is present in those default wordlists — do not invent an obscure name.
- Do not require port scanning. One web port (80) remains the whole attack surface.
- Verify by running `ffuf -u http://localhost/FUZZ -w /usr/share/wordlists/dirb/common.txt` (or equivalent) from the host and confirming `/admin` appears in results with `200`.

---

## 6. Requirement 4 — Decoy Credential File & Password Spraying

Replace the Phase 2 "single password in a README" pattern with a lab-safe **spraying exercise**.

### 6.1 The decoy file

- Place a file named `.env` (or `.env.bak` / `faculty-backup.env`) **inside the web root** so it is retrievable via enumeration or a subtle hint on the admin login page *after* discovery (e.g., an HTML comment `<!-- backup: /.env -->` visible only on `/admin/login` source — not on the public homepage).
- The file must contain **8–12 fictional faculty credential pairs**, for example:

  ```
  # Northbridge faculty training accounts — LAB ONLY, NOT REAL SECRETS
  # format: username:password
  helen.carter:Winter2026!
  james.okafor:Autumn2025!
  priya.sharma:Northbridge123
  david.kim:Faculty2026!
  sara.mitchell:Tr@ining2026
  aarav.patel:CollegeLab2026!
  robert.chen:Welcome2026!
  linda.park:Spring2026!
  # ... (only ONE of these is valid for /admin/login)
  ```

- Only **one** pair grants admin access (e.g., `helen.carter:Winter2026!`). All others must fail with a generic "Invalid credentials" message.
- Label the file clearly as **fictional training data** — never use real passwords, reused secrets, or host credentials.
- Ensure the file is served as plain text by Apache (add `AddType text/plain .env` if needed) and returns `200`, not `403`.

### 6.2 Teaching point

- Students learn that exposed configuration backups are dangerous and that password reuse / predictable patterns enable spraying. They must try each credential from the file against `/admin/login` (manual or with `hydra`/`ffuf`/`burp intruder` in the lab) until one succeeds.
- Do not implement CAPTCHA, lockout, or rate-limit that would block this lab exercise. A generous delay or no throttling is correct for the lab; note this explicitly in code comments as `LAB ONLY — no throttling`.
- Log spray attempts to Apache access logs for instructor review, but do not reveal validity via timing or verbose errors.

### 6.3 What NOT to do

- Do not put the working credential directly in `robots.txt`.
- Do not leave the `.env` linked from any public page.
- Do not commit real secrets or the flag into this file.

---

## 7. Requirement 5 — Limited Admin View & Database Exfiltration Objective

### 7.1 Limited admin view (intended behaviour)

After successful admin login, the clerk sees:

- Marks entry for their own department only.
- Student record search (`/admin/students?q=...`) that by design returns a **limited view**: e.g., `name`, `department`, `enrolled courses (count)`, and `marks (own department only)` — **not** full PII, not all departments, not internal notes.

Document this limit as fictional policy: *"Complete records require registrar approval — clerk view is intentionally restricted."* The limited view itself should contain a nearby note that yields **Flag 2** (per `college-ctf-lab-plan.md`), confirming the student understood the boundary.

### 7.2 Full database is the final objective

- One search field on `/admin/students` (the `q` parameter) must be **intentionally vulnerable to SQL injection over the seeded fictional data only** (document the field explicitly in an internal `docs/ctf-design.md` — not in student-facing pages).
- Using this field, a student can demonstrate that unsafe input handling exposes records beyond the intended limit and can eventually **dump all tables** (students, courses, marks, faculty). The dump itself is the success evidence for the final stage.
- The application must use SQLite or equivalent seeded data — no host files, no external DB.

### 7.3 Flag placement — NOT in the database

- The **final flag must NOT be stored in any database table** (not in `students`, `marks`, `flags`, or `notes`).
- Place the flag as a **filesystem file** on the VM, outside the database, for example:

  ```
  FLAG_PATH=/opt/northbridge/flag.txt
  # or /var/ctf/flag.txt
  # contents: NCC{...unique_value...}
  # permissions: root:root 0640 or root:www-data 0640 — readable by app user if the lab path requires it
  ```

- The flag file is created by the provision script (not by the app, not by hand). Use a unique value per deployment and keep the literal value out of Git and out of student-facing docs. Record only its path and format (`NCC{...}`) in internal docs.
- Student narrative: after proving they can dump the database (CTFd stage for data exposure), they submit the flag retrieved from the filesystem path that is hinted in the dumped data's fictional compliance note or in a lab-local file whose location is revealed only after the dump. The simplest valid pattern is: *dump shows a `compliance_notes` table containing a message "Full audit log archived at /opt/northbridge/flag.txt (lab host only)" — flag itself is in that file, not in the table.*
- This teaches: database compromise ≠ flag in DB; impact is full data exfiltration, and flag location simulates a separate host artifact.

---

## 8. Updated Challenge Flow (Phase 3)

| Stage | Student action | Lab component | Flag/evidence |
|---|---|---|---|
| 0. Discovery | Enumerate directories, find hidden `/admin` | Unlinked `/admin`, no `robots.txt` clue, custom 404 | `NCC{...}` on `/admin/login` page |
| 1. Credential spray | Find `.env` decoy, spray 8–12 faculty passwords against `/admin/login` | `.env` with one valid pair, generic error on failure | `NCC{...}` on admin dashboard after login |
| 2. Boundary check | Understand limited clerk view | Restricted `/admin/students` view + compliance note | `NCC{...}` in compliance note explaining limit |
| 3. Exfiltration | Exploit unsafe `q` search field to dump all fictional records | Single vulnerable search field, seeded data only | `NCC{...}` from filesystem path hinted after dump — **not** from any DB table |

CTFd should gate each stage (next objective revealed only after previous flag). Provide 2–3 staged hints per challenge without copy-paste payloads.

---

## 9. File & Repo Structure

Keep the structure consistent with `AGENTS.md`. Suggested layout after Phase 3:

```
.
├── Vagrantfile                 # synced_folder disabled, forwards 80 -> host
├── infra/
│   ├── provision.sh            # SINGLE source of truth — does everything
│   └── README.md               # How to run provision.sh standalone
├── docs/
│   ├── intern-task.md          # Phase 1 (frozen)
│   ├── college-website-task.md # Phase 2 (frozen, for reference)
│   ├── phase3-production-task.md # THIS FILE — Phase 3 assignment
│   ├── college-ctf-lab-plan.md # Design (update §Portal layout + §Challenge flow with Phase 3 notes)
│   └── beginner-ctf-flow.md    # Visual flow (add Phase 3 branch)
├── scenarios/
│   └── northbridge-ctf.md      # Student-facing walkthrough (update for Phase 3 flow)
├── runbooks/
│   └── reset.md                # `vagrant destroy -f && vagrant up` + `vagrant provision` + log locations
└── (portal source lives in GitHub repo referenced by provision.sh — NOT as a host-synced folder)
```

If your portal source is in this same repo, the provision script must still **clone from GitHub** (or copy from a `git archive`) rather than relying on the shared folder — the host folder must not be mounted at runtime.

---

## 10. Git Workflow

1. Branch from the accepted Phase 2 branch:

   ```bash
   git checkout -b feature/production-hardening
   ```

2. Commit in small, reviewable steps:

   - "Disable Vagrant synced folders, add provision-only clone"
   - "Add custom 404/403/500 pages and error handling"
   - "Hide admin route, remove robots.txt clues"
   - "Add decoy .env with 10 faculty creds (one valid)"
   - "Restrict admin view to limited records"
   - "Add vulnerable search field (documented, seeded-data only)"
   - "Move flag to filesystem path outside DB, provision creates it"
   - "Update docs and runbooks for Phase 3 flow"

3. Each commit message explains *what* and *why*. No secrets or flags in commit history.

---

## 11. Testing Checklist

Record every row in `infra/TESTING.md` or `TEST.md` with command, expected result, and pass/fail. Include the exact `curl`/`ffuf` commands you ran.

### 11.1 Provisioning

| # | Test | Command / Action | Expected |
|---|---|---|---|
| 1 | Clean clone + up | `git clone <url> && vagrant destroy -f && vagrant up` on a fresh host | VM provisions with no errors, site reachable at `http://localhost:8080` (or forwarded port) |
| 2 | No host sync | `vagrant ssh -c "mount | grep vagrant; ls /vagrant"` | No `/vagrant` mount, or empty; site still works |
| 3 | Idempotent re-provision | `vagrant provision` (second run) | Succeeds, no duplicate seed data, site still works |
| 4 | Repo fetch | `vagrant ssh -c "ls -la /var/www/html; cat /var/www/html/.env | head"` | Files present from GitHub clone, not from host |

### 11.2 Functional & error handling

| # | Test | Expected |
|---|---|---|
| 5 | `curl -i http://localhost/` | `200`, homepage renders |
| 6 | `curl -i http://localhost/nonexistent123` | `404`, custom college 404 page, no Apache default |
| 7 | Visit `/profile` unauthenticated | Redirect to `/login` or `403` with friendly message |
| 8 | Submit invalid registration / login | Inline error, no stack trace or SQL error leaked |
| 9 | Trigger app error (e.g., malformed query) | Generic `500` page, details only in `/var/log/apache2/error.log` |

### 11.3 Hidden admin & credential spray

| # | Test | Expected |
|---|---|---|
| 10 | View source of `/`, check `robots.txt`, `sitemap.xml` | No mention of `admin` or `.env` |
| 11 | `ffuf -u http://localhost/FUZZ -w common.txt` | `/admin` appears as `200` (discoverable) |
| 12 | `curl -i http://localhost/.env` | `200`, 8–12 `user:password` lines, labelled LAB ONLY |
| 13 | Try 3 wrong passwords at `/admin/login` | Generic "Invalid credentials" each time |
| 14 | Try the one valid pair from `.env` | Login succeeds, admin dashboard loads |

### 11.4 Limited view & exfiltration

| # | Test | Expected |
|---|---|---|
| 15 | As admin, search for own-department student | Limited view (name, dept, course count, own-dept marks) |
| 16 | As admin, search for other-department student via normal UI | Limited or no results (boundary holds for normal use) |
| 17 | Exploit `q` parameter with crafted input (lab payload) | Returns records beyond intended limit — demonstrates flaw over seeded data |
| 18 | Dump all tables via search field | All fictional student/course/mark rows retrievable |
| 19 | Check `sqlite3 /var/www/html/app.db "SELECT * FROM flags"` (if table exists) | **No flag row** — flag is not in DB |
| 20 | `cat /opt/northbridge/flag.txt` on VM | Flag file exists, format `NCC{...}`, not world-readable by unintended users |
| 21 | Full reset | `vagrant destroy -f && vagrant up` restores site, `.env`, DB, and flag to known state |

---

## 12. Deliverables

| Deliverable | Location | Must contain |
|---|---|---|
| Updated `Vagrantfile` | repo root | `synced_folder disabled`, correct port forwarding, `provision.sh` path |
| Provision script | `infra/provision.sh` | Full build from GitHub, idempotent, variables for repo/branch/path |
| Portal source on GitHub | `PORTAL_REPO` | Functional site, custom error pages, hidden admin, decoy `.env`, limited admin view, single vulnerable search field, flag file creation (via provision) |
| Decoy credential file | Deployed to `/.env` by provision script | 8–12 fictional pairs, one valid, labelled LAB ONLY |
| Flag file | Deployed to `/opt/northbridge/flag.txt` (or equivalent) by provision script | Unique `NCC{...}`, not in DB, not in Git |
| Test evidence | `infra/TESTING.md` or `TEST.md` | All 21 tests with commands, outputs, pass/fail, fixes |
| Updated docs | `docs/`, `scenarios/`, `runbooks/` | Reflect Phase 3 flow, no secrets in docs |
| Git history | branch `feature/production-hardening` | ≥8 focused commits, clean messages, no secrets |

---

## 13. Acceptance Criteria

A reviewer will run the checks below from a **clean clone** — all must pass:

- [ ] `vagrant up` from clean clone builds the full lab with no host file sync.
- [ ] `vagrant provision` is idempotent — second run succeeds with no duplication.
- [ ] Public site is polished: nav/footer consistent, forms validated, no raw errors.
- [ ] Unknown routes return a custom `404` page with HTTP 404 (not Apache default).
- [ ] `/admin` is unlinked from every public surface and absent from `robots.txt`; it is discoverable only by directory enumeration with a common wordlist.
- [ ] `/.env` contains 8–12 fictional credential pairs, only one grants admin login; file is labelled LAB ONLY and is not linked from public pages.
- [ ] Brute-force / spraying the `.env` list against `/admin/login` yields exactly one success with a generic failure message otherwise.
- [ ] Admin login shows a deliberately limited record view; a compliance note explains the limit.
- [ ] The vulnerable search field is the **only** intended SQL injection point and operates only over seeded fictional data.
- [ ] Full database dump is achievable via that field; flag is **not** present in any DB table and lives at a filesystem path created by the provision script.
- [ ] `vagrant destroy -f && vagrant up` restores site, DB, `.env`, and flag to a known-good state.
- [ ] Test evidence covers all 21 cases with commands and outcomes.
- [ ] No real credentials, personal data, or flag values are committed to Git or docs.

---

## 14. Safety & Scope Reminders

- This is an **isolated, authorized training lab** on a single Vagrant VM with host-only networking. Never expose it to the public internet or bridged networks.
- All identities, marks, credentials, and flags are **fabricated**. Do not reuse company or personal secrets.
- Every weakness is confined to the fictional app and seeded data. Do not weaken SSH, the Vagrant provider, or host services.
- The flag file must be lab-local and must not require persistence, lateral movement, or host compromise to retrieve.
- If you use AI tools, verify every generated change manually and note AI assistance in your pull-request summary.
- When in doubt, ask the instructor before changing scope.

---

## 15. Hints if you get stuck

- **Disabling sync:** Search `Vagrantfile` for `synced_folder` — set `disabled: true` and test with `mount | grep vagrant` inside the VM.
- **GitHub fetch:** `git clone --branch "$PORTAL_BRANCH" "$PORTAL_REPO" "$PORTAL_DIR"` — handle the "already exists" case with `git pull` or `rm -rf` guard.
- **Custom 404:** For Flask, use `@app.errorhandler(404)`; for PHP/Apache, use `ErrorDocument 404 /404.html` in the vhost.
- **Serving `.env`:** Apache may block dotfiles via `Require all denied` — adjust the `<FilesMatch "^\.env">` block to `Require all granted` for this lab file only, with a comment.
- **Seeding:** Run seed only if DB empty — `SELECT COUNT(*) FROM students` guard.
- **Flag creation:** `echo "NCC{phase3_$(openssl rand -hex 8)}" > /opt/northbridge/flag.txt` in provision script (example — use your own generator, ensure CTFd flag is updated to match).

---

## 16. Questions

Ask the instructor before assuming. Document every assumption you make — that is part of the deliverable.

Keep it small, reproducible, and explainable.
