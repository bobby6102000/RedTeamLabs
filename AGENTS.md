# AGENTS.md

This repository is for building an isolated, authorized red team training lab for company interns.

## Operating rules

- Treat safety, authorization, and containment as first-class requirements.
- Keep the work documentation-first and reproducible.
- Do not add instructions that would help with real-world compromise, persistence, stealth, credential theft, or unauthorized access.
- Prefer lab-safe examples, mocked assets, and clearly bounded simulations.
- If a request could affect scope or safety, state the assumption before changing files.
- Keep changes small, reviewable, and easy to roll back.
- Update the docs whenever the repo structure or operating model changes.

## Expected repository shape

- `README.md`: project overview and onboarding.
- `docs/`: scope, architecture, intern workflow, and validation notes.
- `infra/`: infrastructure-as-code, build scripts, and environment definitions.
- `scenarios/`: safe exercise briefs and lab walkthroughs.
- `runbooks/`: build, reset, and maintenance procedures.

## Content standards

- Write for interns, reviewers, and future maintainers.
- Use plain language and concrete outcomes.
- For every lab scenario, capture the objective, prerequisites, expected signals, cleanup steps, and verification method.
- Favor repeatable environments with snapshots, resets, and clear logging.

## When editing this repo

- Read `README.md` and the relevant docs before making changes.
- Preserve the isolated-lab assumption unless the user explicitly expands scope.
- Keep file names and headings consistent across the repo.
