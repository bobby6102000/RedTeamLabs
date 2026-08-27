# RedTeamLabs

This is a minimal starter repo for an isolated, authorized red team training lab.

## Final goal

Build a simple CTF-style lab that interns can reproduce with Vagrant, update with scripts, and share through Git. The student-facing exercise will be accessed through CTFd and should stay small, testable, and easy to explain.

## Start here

1. Read [`AGENTS.md`](./AGENTS.md).
2. Keep the first pass simple.
3. Add one small lab note at a time.

## Intern Task

- [`docs/intern-task.md`](./docs/intern-task.md): the current handoff brief for one intern.
- The current assignment is a single VM with updates, Apache, and a simple CRUD website backed by a database.

## Current shape

- `docs/`: small planning notes.
- `infra/`: future environment setup.
- `scenarios/`: future exercise notes.
- `runbooks/`: future maintenance notes.

## Guiding idea

Build only what we need now, then expand as the lab grows.
