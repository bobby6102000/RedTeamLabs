# RedTeamLabs

This is a minimal starter repo for an isolated, authorized red team training lab.

## Final goal

Build a simple CTF-style lab that interns can reproduce with Vagrant, update with scripts, and share through Git. The student-facing exercise will be accessed through CTFd and should stay small, testable, and easy to explain.

## Start here

1. Read AGENTS.md.
2. Keep the first pass simple.
3. Add one small lab note at a time.

## Intern Tasks

### Phase 1 (Complete)

- docs/intern-task.md: the base VM setup - Vagrant, system updates, Apache, and a simple CRUD website backed by a database.
- Status: Complete and accepted.

### Phase 2 (Complete)

- docs/college-website-task.md: college website build — student + admin portal on the existing VM.
- Status: Complete and accepted. Website is live.

### Phase 3 (Active) — CURRENT ASSIGNMENT

- docs/phase3-production-task.md: **START HERE** — production hardening, hidden admin, credential spraying, and full-DB exfiltration objective.
- docs/college-ctf-lab-plan.md: approved Phase 2 design (Phase 3 deltas are in the task doc above).
- docs/beginner-ctf-flow.md: intended learner journey (Phase 3 extends the final stage).

## Current shape

- docs/: planning notes and task briefs.
- infra/: future environment setup.
- scenarios/: future exercise notes.
- runbooks/: future maintenance notes.

## Guiding idea

Build only what we need now, then expand as the lab grows.
