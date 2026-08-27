# Intern Task

## Goal

Set up a single, isolated lab VM with Vagrant, install all available updates, install Apache, and build a simple website with CRUD database functionality for a basic user page.

This task is meant to check how you work, how you document your process, and how reproducible your setup is.

## Requirements

- Use Vagrant to spin up the VM.
- Provide a script that installs and configures all required components.
- Make the setup reproducible from a clean checkout.
- Use Git properly so the codebase can be shared and reviewed.
- You may use AI agents if helpful, but you must understand and be able to explain what was done.
- Test the setup and include the test steps you used.

## Scope

Keep this simple.

- One VM only.
- One website only.
- One small database-backed CRUD feature set.
- Prefer the least complex stack that still meets the goal.

## Suggested outcome

The website should support a simple user page with the ability to create, view, update, and delete records.

## Deliverables

- `Vagrantfile`
- Setup script or scripts
- Git history with clear commits
- Short setup notes
- Test notes or verification steps

## Acceptance criteria

- `vagrant up` brings the VM online successfully.
- The setup script can be run more than once without breaking the environment.
- Apache is installed and serving the site.
- The CRUD feature works against the database.
- The result can be reproduced by following the instructions from scratch.
- The work is tested and the test outcome is documented.
- The code is committed and shared in a clean Git workflow.

## What we will look at

- Whether you can follow requirements carefully.
- Whether your work is organized and easy to repeat.
- Whether you can use Git to share code changes clearly.
- Whether your setup is simple and well explained.
- Whether you test what you build.

## Notes

- If you make assumptions, write them down.
- If something fails, note the error and how you handled it.
- Keep the build small and readable.
