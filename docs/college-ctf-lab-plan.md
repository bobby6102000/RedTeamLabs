# College Portal CTF Lab Plan

## Status and purpose

This is a design and intern handoff brief only. It does not authorize building or publishing the vulnerable lab yet.

The target is an isolated, single-VM, CTFd-backed exercise lasting about 2 to 2.5 hours. It is for students seeing these vulnerability classes for the first time, so it must teach one idea at a time before asking students to apply it.

The first release is intentionally narrower than the long-term plan: it teaches discovery, exposed secrets, authorization boundaries, and SQL injection. XSS, LFI, and RFI-style inclusion are later exercises after this beginner lab has been tested with a cohort.

## Scenario

**Northbridge College Portal** is a fictional campus system used by students and faculty.

- Students can register, sign in, update their contact details, and view their own marks.
- College administrators manage student records and are the only role intended to change marks.
- All student, staff, and mark data is fictional and seeded solely for this lab.

The student narrative is: investigate reports that the portal has exposed data and determine the extent of the problem. Each completed stage yields one flag for CTFd and a plain clue to the next stage.

## Fixed scope and safety boundaries

- One Vagrant VM only; do not use bridged networking or expose the lab to the public internet.
- Use host-only access or deliberate local port forwarding. Keep the VM's outbound access limited to build-time package installation; the finished lab must not depend on external resources.
- Use fabricated identities, marks, messages, credentials, and flags. Do not use company data or reusable secrets.
- Every intended weakness must be confined to the fictional application and seeded lab data. Do not weaken the host, Vagrant provider, SSH, or unrelated system services.
- The RFI-style exercise must point only to a static, lab-local service on the same VM. It must not fetch arbitrary internet content or execute a student-supplied remote program.
- Provide a reset path that restores the initial data and flags between student sessions.
- Do not include persistence, stealth, credential theft, lateral movement, or real-target instructions.

## Beginner student journey

Plan for a nominal 120–135-minute route, including a short instructor introduction, hints, notes, and final submission. CTFd should reveal the next objective only after a student submits the current flag.

| Stage | Time | Simple student goal | Lab component | Completion evidence |
| --- | ---: | --- | --- | --- |
| 0. Find the admin portal | 15 min | Inspect the college site and find the unlinked `/admin` route. | A public, non-sensitive discovery clue such as a `robots.txt` entry or an HTML comment. | Flag 0 on the admin login page. |
| 1. Find the lab password | 20 min | Follow the clue on the admin page to a deliberately exposed training file. | An unlinked static `README` or backup note containing a fake, lab-only administrator password. | Flag 1 next to the fake password and a reminder that it is not a real secret. |
| 2. Explore restricted records | 15–20 min | Sign in as the fictional department clerk and understand the normal limits of the records page. | Administrator marks entry and student-record search. The clerk may update marks for assigned students but sees only a limited record view. | Flag 2 in a fictional compliance note explaining that complete records require a separate approval. |
| 3. Find the data-exposure flaw | 30–35 min | Test the limited student-record search and demonstrate that unsafe input handling exposes additional fictional records. | One deliberately unsafe search field using seeded data only. | Flag 3 in a training-only complete-record result. |
| 4. Explain and close | 20–25 min | Submit the four flags and explain the impact and remediation in plain language. | CTFd response field or instructor worksheet. | Completion confirmation. |

The intended teaching point is not that students should obtain administrator access in real systems. It is that hidden routes are not access control, secrets must not be left in web-accessible files, and database queries must handle input safely.

## Portal layout

| Service | Audience and purpose | Discovery method | Notes |
| --- | --- | --- | --- |
| Student portal | Registration, profile changes, and personal marks. | Advertised entry point. | The primary CRUD application. |
| Admin portal | Department-clerk marks entry and limited student-record search. | Unlinked `/admin` route with one public discovery clue. | Hosts the exposed-secret and SQL injection learning stages. |

Use one main web port for this first beginner release. A later expansion may move a secondary portal to another local port. Exact port assignments, flag values, and seeded accounts must live in implementation-only configuration, not in the student-facing CTFd descriptions. Use a consistent flag format such as `NCC{stage_specific_value}` and ensure every stage has a distinct value.

## CTFd design

- Create four flag-submission challenges, one per technical stage, with progressive point values and one unique flag each. Add a final written reflection or instructor review rather than another technical challenge.
- Give each challenge a short plain-language objective, one success condition, and two or three staged hints. The first hint should explain where to look; later hints should guide observation without providing copy-and-paste exploit strings.
- Make later hints cost points or have a deliberate delay, according to the course policy.
- Include a final feedback question asking students to state the affected component, impact, and recommended remediation for one challenge.
- Do not place live credentials, implementation source, or reset details in CTFd.

## Intern task: build the beginner lab

Build the lab only after the Phase 1 base-VM assignment is reviewed and accepted. Keep all changes in small, reviewable commits and preserve the Vagrant-based, reproducible approach.

### Deliverables

1. **Design record**
   - Document the chosen stack, VM networking, service-to-port map, threat boundaries, and reset approach.
   - Specify each flag's intended location, prerequisite stage, expected student evidence, and remediation lesson. Keep literal flags out of committed public-facing documentation.

2. **Reproducible environment**
   - Extend the existing Vagrant VM and provisioning so a clean checkout creates the complete lab consistently.
   - Automate package installation, application setup, seed data, service configuration, and test data reset.
   - Keep services bound only to the lab-accessible interfaces required for the exercise.

3. **College portal**
   - Implement the normal student and administrator workflows first: registration, profile update, personal marks view, and administrator-only mark management.
   - Seed enough fictional students, courses, marks, and clerk notes for the story to feel credible.
   - Add only the planned, documented training weaknesses. Each must have an explicit scope and safe fixture data.

4. **Challenge flow**
   - Implement the four technical stages in the table above, including distinct flags and simple in-app clues.
   - Make `/admin` unlinked but easy to discover through exactly one public clue. Do not make directory guessing or port scanning the main lesson.
   - Use an obviously fake, lab-only password in a deliberately exposed static training file. Do not use real credentials, system passwords, or secrets from environment configuration.
   - Limit the normal clerk view to a believable subset of fictional records, then place the intended SQL injection lesson in one clearly documented search field.

5. **Tests and validation**
   - Test a clean provision from scratch and a reset followed by re-test.
   - Test intended normal workflows for both student and administrator roles.
   - Test every challenge's intended path in an isolated student account, confirming its unique flag and that it reveals no non-fixture host data.
   - Verify all intended ports and confirm no unintended network listeners are exposed.
   - Record exact commands, expected results, failures found, and fixes in a validation note.

6. **Git handoff**
   - Use a dedicated branch, focused commits, and clear commit messages.
   - Add a concise pull-request summary covering setup, scenario flow, test evidence, reset instructions, and known limitations.
   - If AI tools are used, note which parts they assisted with and verify every generated change manually.

## Acceptance criteria

- A reviewer can provision the single VM from a clean clone using the documented instructions.
- The normal college workflows behave as described before students begin the challenges.
- All four flags are unique, CTFd-ready, and recoverable through their intended stages.
- The required beginner route fits comfortably within 2–2.5 hours for the target cohort, with hints available for students who stall.
- The `/admin` route requires discovery but is accessible only within the local lab boundary.
- The exposed password and all accessible records are fictional fixture data.
- Reset returns the application, seed data, and flags to a known-good state.
- The intern provides test evidence and a remediation note for hidden-route discovery, exposed secrets, and SQL injection.

## Review checkpoints

1. Approve the service map, safety boundaries, and challenge order before vulnerabilities are added.
2. Review the normal application and reproducible provisioning before challenge logic is added.
3. Review each challenge against its documented flag, scope, reset behavior, and remediation lesson.
4. Run an independent end-to-end student test and a clean rebuild before CTFd publication.

## Out of scope for this phase

- Multiple VMs, containers, cloud deployment, or public hosting.
- Real college or company data, live integrations, email delivery, or payment flows.
- Authentication bypasses outside the fictional application or any host-level exploitation.
- XSS, LFI, RFI-style inclusion, advanced exploitation topics, and a mandatory access-control challenge in the first cohort.
