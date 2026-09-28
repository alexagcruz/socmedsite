# Agent Workflow

Use this workflow for application work:

**STUDY => PLAN => EXECUTE PLAN => RENDEZVOUS => SYNC DOCS**

Do not skip a stage. Keep important project intent and human decisions in
repository documentation, not only in conversation. Do not ask the human to
implement code that the coding harness can implement. Express implementation
intent through outcomes, constraints, interfaces, acceptance criteria, and tests.

## STUDY

- Run `date +%s` before creating the study. Use its Unix timestamp in the file
  name.
- Write the study to `doc/study/{unix_timestamp}_{topic}.md`.
- Study the requested outcome before implementation.
- Analyze requirements, feasibility, architecture, technology choices,
  tradeoffs, constraints, risks, testing, and exogenous inputs.
- Recommend an appropriate technology stack and architecture. Explain why each
  major choice fits the requirements, identify reasonable alternatives and
  important tradeoffs, and avoid unnecessary complexity.
- Identify decisions that require human input. Do not silently make major
  technology or architecture decisions when meaningful tradeoffs require
  human judgment. Ask for required external inputs before they become a
  blocker.
- Do not change application code during study.
- Commit the study with a `docs:` Conventional Commit.

## PLAN

- Run `date +%s` before creating the plan. Use its Unix timestamp in the file
  name.
- Write the plan to `doc/plan/{unix_timestamp}_{topic}.md`.
- Base the plan on the approved study and the current repository.
- Structure the plan as an executable, editable checklist. Include
  implementation, automated testing, validation, runtime, reproducibility,
  documentation, and acceptance criteria.
- Put unresolved human decisions in a prominent `OPEN QUESTIONS` section.
- Do not implement during planning.
- Commit the plan with a `docs:` Conventional Commit.

## EXECUTE PLAN

- Read the approved plan before execution.
- Check Git status before execution. Do not overwrite or discard work that is
  already present.
- Normally create a feature branch.
- Execute the approved checklist and update it as work is completed.
- Use automated tests as feedback during implementation. Continue until the
  plan is complete, or a genuine external blocker requires human input.
- Use appropriately scoped Conventional Commits. Do not combine unrelated
  changes in one commit.

## RENDEZVOUS

- Verify the implementation against the approved plan.
- Run all relevant automated tests. Confirm that the application is in a
  workable state and resolve integration problems where appropriate.
- Merge the completed feature branch into `main`.
- Report test results, the current branch, Git status, and remaining
  limitations.

## SYNC DOCS

- Update `doc/wiki/` so that living documentation reflects the actual current
  state of the codebase.
- Record noteworthy or unintuitive behavior in `doc/wiki/footguns/`.
- Document only functionality that actually exists.
- Include accurate setup and runtime instructions.

## TECHNOLOGY AND ARCHITECTURE DECISIONS

- Make technology and architecture recommendations during STUDY.
- Prefer established technologies with strong automated testing support unless
  the requirements justify another choice.
- Explain major choices, reasonable alternatives, and tradeoffs. Identify
  major decisions that need human approval before implementation.
- Record approved decisions and their reasoning in project documentation.

## TESTING

- Treat automated testing as an important part of implementation.
- Define important behavior through testable outcomes rather than unnecessary
  implementation details.
- Where appropriate, test major user flows, business rules, validation, and
  important failure cases.
- Run relevant tests during execution and before rendezvous.
- Do not consider important behavior complete merely because code exists.

## EXOGENOUS INPUTS

- Clearly identify every application input that cannot be created inside the
  codebase. Examples include API keys, credentials, accounts, external
  services, proprietary or domain information, datasets, and required assets.
- Never invent unavailable exogenous inputs.
- Ask the human for required external inputs before they become a blocker.

## REPRODUCIBILITY

- Keep the repository reproducible from a fresh clone.
- Declare dependencies with the standard dependency declaration mechanism for
  the selected stack. Do not assume packages installed in the current
  CodeRange environment will exist elsewhere.
- Do not depend on untracked local state or commit secrets or API keys.
- Ensure required project files are not accidentally excluded by
  `.gitignore`.
- Document exact installation, configuration, migration, initialization,
  seed-data, testing, and startup commands.
- Before submission, verify the project from a fresh clone.

## CODERANGE

- Applications that must accept CodeRange external requests must bind to
  `0.0.0.0` and the required port.
- Account for the CodeRange forwarded-host environment when applicable.
- Do not assume that working on localhost proves that the application works
  through CodeRange.

## VERSION CONTROL

- Use Git throughout the project.
- Scope meaningful changes into clean Conventional Commits. Use prefixes such
  as `feat:`, `fix:`, `docs:`, `test:`, `refactor:`, `build:`, and `chore:`.
- Use `TODO.md` as the editable file-based todo list where appropriate.
- Treat `doc/canonical/` as authoritative human-approved project information.
- Use `doc/memory/` for project memory when needed and `doc/roadmap/` for
  roadmap material.

## REMOTE VERSION CONTROL

- When `origin` is configured and authentication is available, push completed,
  committed workflow changes to the remote repository.
- After rendezvous and SYNC DOCS, ensure completed work is committed and push
  `main`. Push again after later completed corrective changes.
- Never assume that a local commit exists on the remote. Verify local and
  remote state before claiming that GitHub is current.
- Report whether pushes succeed.

## DOCUMENTATION STYLE

Use ASD-STE100 as the base documentation style unless important nuance would
otherwise be lost.
