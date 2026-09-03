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

### Phase 2 (Active)

- docs/college-website-task.md: CURRENT ASSIGNMENT - build the Northbridge College Portal (student and admin website) on the existing VM.
- docs/college-ctf-lab-plan.md: the approved Phase 2 design and CTF lab plan.
- docs/beginner-ctf-flow.md: the intended learner journey for the CTF exercise.

## Current shape

- docs/: planning notes and task briefs.
- infra/: future environment setup.
- scenarios/: future exercise notes.
- runbooks/: future maintenance notes.

## Guiding idea

Build only what we need now, then expand as the lab grows.
