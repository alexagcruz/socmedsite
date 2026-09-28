# Repository Status And Setup

- Status: Documentation-only repository; the application is not implemented.
- Last verified: 2026-09-28

This page describes the code that exists now. It does not describe planned
features as working functionality.

## Current Architecture

There is no application architecture in the current repository. It contains
Markdown guidance and planning documents only. There is no server, frontend,
API, database integration, or application component/interface.

The approved target architecture is a Django monolith with server-rendered
templates and PostgreSQL. This is a human-approved design, not implemented
code. See:

- [Approved MVP decisions](../canonical/mvp-decisions.md)
- [Feasibility study](../study/1790599824_social_media_mvp_feasibility.md)
- [Implementation plan](../plan/1790602725_social_media_mvp_implementation.md)

## Repository Structure

Current tracked project files:

```text
AGENTS.md
doc/
  canonical/mvp-decisions.md
  plan/1790602725_social_media_mvp_implementation.md
  study/1790599824_social_media_mvp_feasibility.md
  wiki/README.md
  wiki/footguns/plan-is-not-runtime.md
```

There are no application source files, templates, static assets, dependency
declarations, environment examples, `.gitignore`, migrations, seed-data files,
or test files.

## Application Components And Interfaces

None exist. There are no HTTP routes, forms, APIs, views, services, or user
interfaces to use.

The project workflow and approved future interfaces are described in
`AGENTS.md` and the study/plan documents. They are not callable interfaces.

## Data, Authentication, And Business Rules

The current repository has no data model, database schema, or migrations. It
does not store users, posts, follow relationships, shares, campaigns, or
purchase records.

There is no authentication or authorization implementation. There are no
regular-user or campaign-manager accounts, sessions, role checks, or manager
account provisioning path.

No social or advertising business rule is enforced by running code. In
particular, there is no post creation, image upload, follow/unfollow, feed,
share, advertisement creation, simulated purchase, eligibility check, or
sponsored-ad insertion feature.

The approved target rules are recorded in
[`doc/canonical/mvp-decisions.md`](../canonical/mvp-decisions.md). Do not treat
that document as evidence that the rules are implemented.

## Clone And Setup

Clone the repository to inspect its documentation:

```sh
git clone https://github.com/alexagcruz/socmedsite.git
cd socmedsite
git status --short --branch
```

No application dependency installation or initialization command exists.
There is no dependency manifest, environment configuration, database, or
migration command. The repository contains no sample or seed data.

## Run The Application

There is no application startup command. In particular, this repository does
not contain `manage.py`, so Django commands cannot run here. No server process
binds to `0.0.0.0`, a CodeRange port, localhost, or any network port. The
repository cannot currently accept CodeRange external requests.

Do not use a command from the approved plan as if it were already available.
The implementation must add and document the actual supported commands.

## Database And Sample Data

No database engine is configured. PostgreSQL is the approved target, subject
to confirming CodeRange availability. No PostgreSQL connection settings,
schema, migrations, administrator account, fixtures, or seed-data command
exist.

No default or sample user accounts exist. Do not expect a login to work.

## Automated Tests

There is no project test suite or test command because no application code or
test files exist. At the last check, `python3 -m pytest` could not run because
`pytest` was not installed, and `python3 manage.py test` could not run because
`manage.py` did not exist. These are environment/repository observations, not
passing tests.

Do not report application behavior as tested until an implementation and its
test suite exist and those tests pass.

## CodeRange And Exogenous Configuration

There is no CodeRange runtime configuration in this repository. No Python
version, start command, allowed-host setting, forwarded-host setting, port
configuration, health check, database service, or persistent media volume has
been selected or verified.

No external service or credential is required to use the current
documentation-only repository. Implementation and deployment may require:

- CodeRange Python/runtime, port, forwarded-host, and build details.
- A PostgreSQL service and credentials.
- A durable CodeRange media volume or an approved S3-compatible storage
  service, bucket, endpoint, and credentials.
- Other business decisions listed in the canonical document, such as the
  simulated campaign purchase terms and exact ad interval.

Real payment processing and email delivery are not part of the approved MVP.
Do not create provider accounts, add credentials, or claim support for them.

## Current Limitations

- The application cannot be installed, initialized, tested, or started because
  no application project exists.
- No authentication, social, image, feed, campaign, purchase, or advertising
  behavior is available.
- CodeRange deployment and external access are unverified.
- Exact campaign-manager provisioning, ad frequency, simulated purchase
  semantics/lifecycle, some post visibility behavior, and hosting inputs
  remain open as listed in the canonical decisions and implementation plan.

Update this page after implementation. Replace each absence statement with
verified current behavior and exact commands. Do not copy planned behavior here
as if it already exists.
