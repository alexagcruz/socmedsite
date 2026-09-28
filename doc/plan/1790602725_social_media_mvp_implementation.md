# Social Media MVP Implementation Plan

- Status: Plan prepared; implementation has not started.
- Date: 2026-09-28
- Source study: `doc/study/1790599824_social_media_mvp_feasibility.md`
- Authoritative decisions: `doc/canonical/mvp-decisions.md`

This is an executable and editable checklist for a later implementation
session. The approved canonical decisions are requirements. Do not expand the
MVP beyond them. Resolve the blocking items in [OPEN QUESTIONS](#open-questions)
before implementing behavior that depends on them. Do not substitute a guess
for an unavailable service, credential, or human decision.

## Repository Baseline

At plan creation, the current branch is `main`. The repository has no
application code, dependency manifest, test suite, database schema, or
CodeRange deployment configuration. It contains the workflow guide, approved
study, and canonical decision record. These documentation files are staged
and there is no existing commit history. Preserve this work. Before execution,
check the actual Git state again because commits or other work may have changed
it.

## OPEN QUESTIONS

Record each answer in `doc/canonical/mvp-decisions.md` before using it to
finalize dependent behavior. An unresolved infrastructure item can block
deployment or image handling even if unrelated code can proceed.

- [ ] **CodeRange contract:** Confirm supported Python version, dependency
  install/build command, start command, health-check behavior, required port,
  forwarded host/HTTPS headers, and deployment environment-variable names.
- [ ] **PostgreSQL:** Confirm CodeRange PostgreSQL service availability and
  how the application receives its connection settings. PostgreSQL is
  approved. If unavailable, stop and request approval for an alternative; do
  not silently switch production to SQLite.
- [ ] **Durable image storage:** Confirm that CodeRange has a media volume that
  survives restarts and deployments. If not, provide an approved S3-compatible
  storage service, bucket/endpoint, owner, and secure credential-delivery path.
- [ ] **Advertisement interval:** Select the numeric number of organic posts
  between sponsored items. The approved policy is a fixed deterministic
  interval, but no numeric frequency is approved yet.
- [ ] **Simulated purchase and campaign lifecycle:** Define what a simulated
  purchase represents and any displayed demo price/currency, campaign period,
  activation timing, expiration, and pause behavior. Do not claim a real
  charge or invent pricing.
- [ ] **Campaign-manager provisioning:** Confirm whether an operator creates
  manager accounts or sends invitations. Public registration must not grant
  this role.
- [ ] **Sharing own posts:** Confirm whether a user may reshare their own post.
  Reshares of reshares must point to the root original.
- [ ] **Post detail visibility:** Confirm whether users outside the feed's
  author-and-followed-account set may access posts through profile or detail
  pages. Do not add private-account features without approval.
- [ ] **Dependency installation:** Confirm whether CodeRange supports the
  selected `pyproject.toml` lock/install workflow. If not, use the approved
  pip-compatible pinned requirements approach and document it.

The approved MVP uses username/password authentication without email
verification or password-reset email. Adding email delivery is out of scope
unless a later decision supplies an email provider and credentials.

## Approved Scope And Architecture

- [ ] Keep one Django monolith with one PostgreSQL database and server-rendered
  templates. Do not add a separate SPA/API, microservices, queue, cache, or
  recommendation service.
- [ ] Use the CodeRange-supported Python version and a compatible, supported
  Django LTS release.
- [ ] Use Django sessions, password hashing, forms, CSRF protection, ORM,
  migrations, and the built-in test runner/test client.
- [ ] Bind the deployed service to `0.0.0.0` and the CodeRange-provided port.
- [ ] Use Django's storage interface. Use local media storage in development;
  use only a verified durable CodeRange volume or approved object storage in
  deployment.
- [ ] Keep product roles exclusive: `regular` and `campaign_manager`.
  Administrative staff/superuser access is separate.
- [ ] Regular users can register, authenticate, create text/image posts,
  follow/unfollow, view their feed, and reshare existing posts.
- [ ] Campaign managers can authenticate, create and manage their own
  advertisements, and complete a clearly labeled simulated purchase.
- [ ] Insert only confirmed simulated-purchase, active, in-window campaigns.
  Show ads to all regular users without targeting, clearly label them
  `Sponsored`, use the approved fixed interval, and show no ad in an empty
  organic feed.
- [ ] Do not implement likes, comments, messaging, notifications, search,
  ranking, targeting, analytics, auctions, budgets, real payment processing,
  video, native clients, or a public API.

## Implementation Checklist

### 1. Pre-Execution And Project Setup

- [ ] Read this plan and `doc/canonical/mvp-decisions.md` before coding.
- [ ] Check `git status` and preserve all existing staged and unstaged work.
  Do not reset, clean, overwrite, or stage unrelated changes.
- [ ] Resolve or explicitly record the OPEN QUESTIONS that block the
  dependent work. Obtain external inputs before the code needs them.
- [ ] Create a feature branch from the agreed base branch before application
  work. Keep commits scoped and use Conventional Commit messages.
- [ ] Verify CodeRange supports the selected Python, Django, PostgreSQL, and
  dependency installation versions. Record exact versions in project files.
- [ ] Create the Django project and a small set of cohesive apps for accounts,
  social content, and advertisements. Keep a single deployable application.
- [ ] Establish environment-based settings for the secret key, debug mode,
  allowed hosts, CSRF trusted origins, database connection, media root/storage,
  and port. Do not commit real credentials.
- [ ] Add a safe environment example with placeholders and ensure required
  files are not excluded by `.gitignore`.
- [ ] Configure project templates, responsive base styles, static assets, and
  URL routing without adding a JavaScript application framework.
- [ ] Select CodeRange-compatible static-file serving. Document the selected
  method; add a package such as WhiteNoise only if the platform requires it.

### 2. Dependencies And Reproducible Project Definition

- [ ] Declare runtime and development dependencies in `pyproject.toml` and
  commit a lock file if CodeRange supports that workflow. Otherwise use a
  pinned pip-compatible requirements file as the canonical install input.
- [ ] Include Django, the PostgreSQL driver, Pillow for decoded image
  validation and Django image fields, and Gunicorn for the WSGI runtime.
- [ ] Add an object-storage package only if the approved storage path requires
  it. Do not add a payment SDK or email package for the approved MVP.
- [ ] Pin supported versions and verify installation from a clean environment.
- [ ] Document the exact dependency install command and supported Python
  version once CodeRange requirements are confirmed.

### 3. Accounts, Authentication, And Authorization

- [ ] Define a custom Django user model before creating the first migration.
  Include the unique login/handle, password-managed-by-Django fields, role,
  and timestamps needed by the MVP.
- [ ] Restrict role choices to `regular` and `campaign_manager`; keep staff
  flags separate from product roles.
- [ ] Make public signup create regular users only. Ignore or reject any
  submitted role or privilege field.
- [ ] Implement username/password registration, login, logout, and session
  authentication. Do not require email verification or email delivery.
- [ ] Add server-side role checks for every protected action. Hiding a link or
  button is not authorization.
- [ ] Scope campaign queries and mutations to the authenticated manager who
  owns the campaign. Return a safe denied/not-found response for another
  manager's campaign.
- [ ] Add the approved manager-account provisioning path after the OPEN
  QUESTION is resolved. Do not provide public manager self-registration.
- [ ] Test unauthenticated access, role boundaries, forged role submissions,
  and cross-owner campaign access.

### 4. Data Models And Migrations

- [ ] Implement a `Follow` relationship with follower and followed user,
  creation time, a database uniqueness constraint on the pair, and a
  self-follow prevention rule.
- [ ] Implement a `Post`/feed-item model with author, optional text, optional
  image key, optional root-original reference for a reshare, and creation
  time. An original must contain text or an image; a reshare references its
  original and does not copy its content.
- [ ] Enforce in application validation that a reshare points to a root
  original rather than creating a chain. Apply database constraints where
  supported and reliable.
- [ ] Implement an `AdCampaign` owned by a campaign manager, with a simple
  text-first creative, optional validated destination URL, lifecycle state,
  and approved time bounds.
- [ ] Implement a purchase record related to a campaign. Represent simulated
  confirmation explicitly; do not store fabricated provider references or
  card data.
- [ ] Apply the campaign activation, pause, and expiration transitions only
  after the relevant OPEN QUESTIONS are answered. Represent the approved
  states explicitly and reject invalid transitions.
- [ ] Add foreign keys, unique constraints, and indexes for follow lookups,
  post chronology, share originals, campaign owners, and eligibility queries.
- [ ] Create and review initial migrations after the custom user model and
  relationships are settled. Verify migrations apply to an empty PostgreSQL
  database and do not rely on local developer data.

### 5. Text And Image Posts

- [ ] Add a server-rendered post form for text-only, image-only, and
  text-plus-image posts.
- [ ] Validate that an original post has text or a valid image. Reject empty
  posts and invalid field combinations with clear form errors.
- [ ] Add image upload handling using Django's storage interface and Pillow.
  Decode image content; do not trust filename extensions or browser MIME
  values. Reject SVG and any format not explicitly supported.
- [ ] Select and document a configurable image-size limit and explicit
  supported raster formats. Enforce the limit on the server and test both
  accepted and rejected boundary cases.
- [ ] Generate opaque storage names. Do not use a user-provided path as a
  storage key. Keep media separate from executable application code.
- [ ] Configure development uploads under a local media directory that is
  ignored by Git. Use a temporary media directory in automated tests.
- [ ] Configure the deployment media location from environment/platform
  settings. Do not serve production uploads from ephemeral disk. Block
  deployment of image functionality until a durable approved storage target
  is available.
- [ ] Handle invalid, oversized, missing, and failed-storage uploads without
  creating a partial post or exposing internal errors.

### 6. Follow And Unfollow

- [ ] Implement authenticated follow and unfollow actions using CSRF-protected
  requests.
- [ ] Prevent self-follow and duplicate relationship rows. Make repeated
  follow/unfollow submissions safe and deterministic.
- [ ] Allow only the approved regular-user capabilities; campaign managers
  remain separate from social actions.
- [ ] Provide clear success and invalid-action feedback without disclosing
  private data.

### 7. Post Sharing

- [ ] Add a one-click reshare action with no commentary.
- [ ] Create a new feed item linked to the root original. Do not modify the
  original author, body, image, or timestamp, and do not duplicate original
  content into the share record.
- [ ] Render both the sharer's identity/time and the original author's
  identity/content in the feed card.
- [ ] Enforce the approved root-original behavior when a user reshares a
  reshare. Resolve the own-post sharing OPEN QUESTION before setting its
  allowed behavior.
- [ ] Reject missing, invalid, or unauthorized share targets safely.

### 8. Social Feed Generation

- [ ] Build the organic feed at request time from the signed-in user's posts
  and posts by accounts they follow. Do not add ranking or a precomputed feed.
- [ ] Sort newest first with a stable secondary key. Paginate the organic feed
  and bound database work; avoid loading all history or creating N+1 queries.
- [ ] Apply the approved post-visibility decision to feed and post-detail
  queries. Do not expose posts through a detail/profile route beyond the
  approved visibility policy.
- [ ] Render original posts and reshares with the required attribution.
- [ ] Provide an empty-feed state without advertisement insertion.
- [ ] Test feed inclusion, exclusion, ordering, pagination, and visibility
  after follow/unfollow changes.

### 9. Campaign Creation And Lifecycle

- [ ] Add campaign-manager pages/forms to list, create, and edit only the
  manager's own campaigns.
- [ ] Keep advertisement creative text-first with an optional destination URL.
  Validate URL scheme and output safely; never fetch the URL from the server.
- [ ] Create draft campaigns by default. Protect all manager routes with
  role checks and all campaign queries with owner scoping.
- [ ] Implement explicit purchase and campaign states according to the
  approved lifecycle answers. A draft, pending, failed, paused, or expired
  campaign must not qualify for insertion.
- [ ] Keep ownership, purchase confirmation, activation, and date eligibility
  server-controlled. Do not accept those values from browser-submitted fields.

### 10. Simulated Purchase

- [ ] Add an explicit simulated-purchase action for the owning campaign
  manager. Clearly label the page and confirmation as a simulation.
- [ ] Use a CSRF-protected POST. Use a database transaction and safe
  duplicate-submission behavior so repeated requests do not create conflicting
  purchase records.
- [ ] Apply the approved purchase-unit and lifecycle rules. Do not display an
  unapproved price/currency or imply a real charge.
- [ ] Transition eligibility only after the simulated purchase is confirmed
  and the campaign is active under the approved rules.
- [ ] Do not collect card data, call a payment provider, store provider
  credentials, or create real-payment endpoints.
- [ ] Test successful, repeated, invalid, non-owner, and wrong-role purchase
  actions. Test that unsuccessful/unconfirmed states never become eligible.

### 11. Advertisement Eligibility And Feed Insertion

- [ ] Create one server-side eligibility query/service that requires a
  confirmed simulated purchase, active state, and valid approved time window.
- [ ] Return no advertisements to a campaign manager's feed or to an empty
  organic feed.
- [ ] After the approved number of organic posts, insert an eligible sponsored
  item. Do not count sponsored items as organic items when applying the
  interval.
- [ ] Select campaigns in a deterministic, non-personalized order and cycle
  through the eligible set. Do not randomize, target users, rank campaigns,
  or add impression/click tracking.
- [ ] Preserve organic post ordering and pagination while composing each feed
  page with advertisements.
- [ ] Render a clear `Sponsored` label and a visual treatment distinct from a
  normal social post. Keep the campaign identity and approved creative visible.
- [ ] Test every ineligible campaign state, active eligible campaigns,
  deterministic interval boundaries, rotation order, empty-feed behavior,
  regular-user audience, and sponsorship labeling.

### 12. Validation, Errors, And Security

- [ ] Validate all inputs server-side. Provide safe field errors for expected
  invalid requests and do not expose stack traces, secret values, or internal
  storage/database details.
- [ ] Use Django password hashing, sessions, CSRF protection, template
  escaping, and ORM parameterization. Do not add custom authentication
  cryptography.
- [ ] Apply authentication, role, and ownership checks to direct HTTP
  requests, not only the rendered interface.
- [ ] Prevent over-posting of role, author, owner, purchase, and activation
  fields. Derive them from the authenticated user and server-side workflow.
- [ ] Configure allowed hosts, CSRF trusted origins, HTTPS/security cookies,
  and proxy headers for confirmed CodeRange behavior. Trust only documented
  platform proxy headers.
- [ ] Keep secret key, database credentials, and optional object-storage
  credentials in runtime configuration. Commit placeholders only.
- [ ] Ensure safe URL handling and safe media serving. Do not fetch advertiser
  destinations from the application server.
- [ ] Add reasonable limits for signup and uploads using platform controls or
  a justified small dependency. Do not add a distributed abuse-control
  service.

### 13. Automated Tests

- [ ] Use Django's built-in test runner and test client. Keep tests independent
  of external payment, email, or storage services.
- [ ] Run database integration tests against PostgreSQL when available. Use
  isolated temporary media storage and deterministic time/eligibility inputs.
- [ ] Test registration, login, logout, and session behavior.
- [ ] Test that public signup cannot create or promote a campaign manager.
- [ ] Test role-based authorization, campaign ownership, and unauthorized
  direct requests.
- [ ] Test text-only, image-only, and text-plus-image post creation, plus
  empty, malformed, unsupported, and oversized image inputs.
- [ ] Test follow, duplicate follow, self-follow rejection, unfollow, and
  repeated unfollow behavior.
- [ ] Test feed visibility, chronology, pagination, stable ordering, and
  membership changes after follow/unfollow.
- [ ] Test sharing preserves source data and attribution, points to the root
  original, and handles invalid targets and the approved own-post policy.
- [ ] Test campaign creation/edit authorization and safe URL validation.
- [ ] Test simulated purchase confirmation, duplicate submission, invalid
  state transitions, and campaign eligibility.
- [ ] Test that unpaid/unconfirmed, draft, pending, failed, paused, expired,
  and out-of-window advertisements never appear.
- [ ] Test that eligible ads appear only for regular users at the approved
  deterministic interval, without disrupting organic order, and are visibly
  labeled `Sponsored`.
- [ ] Test CSRF and expected validation/permission error behavior.
- [ ] Run the complete suite after each major feature group and before
  rendezvous. Fix failures; do not mark behavior complete because code exists.

### 14. Runtime, CodeRange, And Reproducibility

- [ ] Implement a production start command using Gunicorn and Django WSGI.
  Bind to `0.0.0.0` and the CodeRange port (normally `PORT`); do not bind only
  to localhost or hard-code a production port.
- [ ] Configure `DEBUG=False`, a non-default secret, allowed hosts, CSRF
  origins, PostgreSQL connection settings, static assets, and durable media
  storage using the confirmed platform contract.
- [ ] Document database provisioning, environment setup, migrations,
  static-file collection/serving, manager bootstrap, optional seed data,
  tests, and startup commands exactly.
- [ ] Provide optional development sample data only if useful. Generate it
  with a documented command, never commit real accounts, secrets, user images,
  or production credentials, and never seed a default production password.
- [ ] Confirm a clean clone can install declared dependencies, initialize an
  empty PostgreSQL database, run migrations/tests, and start the application.
- [ ] Verify access through the CodeRange forwarded host and port. Confirm
  uploaded media survives a restart/redeployment if a platform volume is
  selected.
- [ ] Verify required source, dependency, and configuration-example files are
  not ignored and that no local-only state is required.

### 15. Documentation And Sync

- [ ] Update `doc/wiki/` with only implemented behavior, including accurate
  install, configuration, migration, seed, test, and runtime instructions.
- [ ] Add a `doc/wiki/footguns/` entry for non-obvious role, simulated
  purchase, feed eligibility, share-root, or media-persistence rules that
  future maintainers could otherwise break.
- [ ] Keep `doc/canonical/mvp-decisions.md` aligned with approved decisions.
  Record new human decisions there before implementing changes that depend on
  them.
- [ ] Update this plan's checklist as tasks complete. Do not mark unchecked
  behavior complete without verification.

### 16. Rendezvous And Acceptance

- [ ] Verify every acceptance criterion below against the running application
  and automated tests.
- [ ] Run all relevant tests, migrations, dependency checks, and CodeRange
  runtime checks. Resolve integration issues where appropriate.
- [ ] Confirm the application is in a workable state and document any
  remaining limitation or external blocker.
- [ ] Review Git status and scoped Conventional Commits. Preserve unrelated
  changes. Merge the completed feature branch into `main` after validation.
- [ ] Report test results, branch, Git status, remaining limitations, and
  push outcome. Verify remote state rather than assuming a local commit was
  pushed.

## Acceptance Criteria

- [ ] A regular user can register and authenticate, and cannot grant
  themselves the campaign-manager role.
- [ ] A campaign manager can authenticate and access campaign functions, but
  cannot use regular-user social actions under the approved exclusive-role
  policy.
- [ ] A regular user can create valid text and image posts. Invalid or
  oversized images do not create partial records.
- [ ] A user can follow and unfollow another eligible user. Self-follow is
  rejected, duplicate follows do not occur, and feed membership changes
  accordingly.
- [ ] The feed contains only the approved organic posts, uses stable
  newest-first ordering, and is paginated.
- [ ] Sharing creates a separate attributed feed item linked to the root
  original and leaves original content unchanged.
- [ ] A campaign manager can create and manage only their own advertisement.
- [ ] The simulated purchase is clearly identified as simulated, does not
  collect payment data, and cannot be falsely confirmed by client-supplied
  state.
- [ ] Only purchased/confirmed, active, in-window advertisements are eligible.
  Draft, pending, failed, paused, expired, or unpurchased advertisements are
  absent from feeds.
- [ ] Eligible advertisements are inserted at the approved deterministic
  interval for regular users, are marked `Sponsored`, and do not reorder
  social posts. Empty organic feeds contain no ads.
- [ ] Automated tests cover authentication, authorization, post creation and
  image validation, follows, feed visibility, shares, manager restrictions,
  campaign creation, simulated purchase, ad eligibility/placement/labeling,
  and invalid or unauthorized actions.
- [ ] The app runs through CodeRange on `0.0.0.0:$PORT`, uses PostgreSQL and
  durable media storage, and has documented exact setup and runtime commands.
- [ ] A fresh clone can install dependencies, configure an empty database,
  apply migrations, run tests, and start the app without untracked local
  state or committed secrets.
- [ ] Living documentation describes only behavior that exists, and
  noteworthy footguns are recorded.

## Execution Status

- [ ] Open questions resolved or recorded as explicit implementation gates.
- [ ] Implementation started on a feature branch.
- [ ] All implementation, test, runtime, reproducibility, and documentation
  checklist items completed.
- [ ] Rendezvous, merge, and remote verification completed.

No implementation tasks in this plan have been executed.
