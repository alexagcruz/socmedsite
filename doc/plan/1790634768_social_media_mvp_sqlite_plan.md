# Social Media MVP Implementation Plan (SQLite)

- Status: Reconstituted and ready for execution; not executed.
- Date: 2026-09-28
- Supersedes: `doc/plan/1790602725_social_media_mvp_implementation.md`
- Source study: `doc/study/1790599824_social_media_mvp_feasibility.md`
- Authoritative decisions: `doc/canonical/mvp-decisions.md`

**PLAN READY FOR EXECUTION: YES.** No unresolved product decision, account,
credential, external database, or object-storage service blocks application
implementation. The unknown CodeRange-assigned port/forwarded host and the
durability of the SQLite data file are runtime concerns. The application must
support a specified port without requiring `PORT` to exist during development.

Do not execute this plan until authorized. Keep every checklist editable and
mark an item complete only after its behavior or command is verified.

## Repository And Environment Baseline

At plan reconstitution, `main` is clean and the repository contains planning
and workflow documentation only. No application code, migrations, dependency
manifest, tests, seed data, or runtime command exists.

Environment observations:

- `python3` is Python 3.12.3. The `python` command is not available.
- Python `venv` and `ensurepip` modules are available.
- Python's SQLite module is available (SQLite 3.45.1).
- pip 24.0 is available through `python3 -m pip`.
- `uv`, Poetry, PDM, and `pip-compile` are not installed.
- No `PORT`, forwarded-host, PostgreSQL connection, or CodeRange port/host
  environment variable is exposed.
- `psql` is installed, but no PostgreSQL server responds on the local socket
  or `127.0.0.1:5432`; all checked PostgreSQL connection variables are unset.
- The workspace and `/tmp` use the root `overlay` filesystem. No dedicated
  persistent media/data mount was found.

Use these observations as implementation constraints. Do not assume an
external PostgreSQL service, storage service, persistent mount, assigned port,
or public forwarded host exists.

## OPEN QUESTIONS

These questions do not block application implementation. Resolve them before
the corresponding CodeRange runtime or data-retention claim:

- [ ] **CodeRange external port and host:** The externally assigned port and
  forwarded-host/proxy contract are unavailable. The application must accept
  a configured port, read `PORT` if present, and work with a documented local
  development default when it is absent. Obtain the actual CodeRange values
  before external-access verification. Do not invent them.
- [ ] **SQLite data-file durability:** The current filesystem is an overlay and
  no durable data mount was found. Decide whether accounts, posts, follows,
  campaigns, and purchases must survive redeployment. If yes, provide a
  durable local path and configure `SQLITE_PATH` to that mount. If not, record
  that database contents can be lost on redeployment. This does not block
  local implementation or tests.

The operator must create the initial Django superuser interactively. This is
an operational credential, not a committed value or a prerequisite for
implementing the application. The operator must provide it before using the
admin interface to provision campaign managers.

## Approved Requirements And Boundaries

- [ ] Support exclusive `regular` and `campaign_manager` roles. Administrators
  provision campaign managers; public users cannot select or assign that role.
- [ ] Regular users can register and authenticate, create text/image posts,
  follow and unfollow users, view their feed, view any individual post, and
  reshare eligible posts authored by other users.
- [ ] A user's normal feed contains their own posts and posts from accounts
  they follow. Authenticated regular users can view an individual post even
  when its author is not followed.
- [ ] A share is a new attributed feed item linked to the root original. It
  does not copy or mutate the original. Reject sharing when the root original
  belongs to the current user.
- [ ] Campaign managers can authenticate, create and manage only their own
  advertisements, and use a clearly identified simulated-purchase action.
- [ ] A successful simulated purchase activates a campaign immediately for
  seven days. A manager can end the campaign sooner. There is no real payment,
  real currency, or payment-provider integration.
- [ ] Insert one eligible active advertisement after every 10 organic feed
  posts. Do not count ads as organic posts. Keep insertion deterministic and
  label every ad `Sponsored` with a distinct visual treatment.
- [ ] Only a simulated-purchased, active, non-expired, non-ended campaign is
  eligible. Do not show ads to campaign managers or in an empty organic feed.
- [ ] Use local application media storage for uploaded images. Loss of images
  after CodeRange redeployment is an accepted MVP limitation. Do not add
  external object storage or credentials.
- [ ] Use SQLite through Django's standard database abstraction. Do not add
  PostgreSQL services, drivers, credentials, or SQLite-specific logic in
  application services.
- [ ] Keep the app a single Django monolith with server-rendered templates.
  Defer likes, comments, messages, notifications, search, private accounts,
  targeting, ad analytics, real payments, video, public APIs, and a separate
  frontend.

## Implementation Checklist

### 1. Project Setup And Runtime Configuration

- [ ] Read this plan and `doc/canonical/mvp-decisions.md` before coding.
- [ ] Check Git status and preserve existing work. Create a feature branch
  from the agreed base before implementation; use scoped Conventional
  Commits.
- [ ] Create one Django project package named `config` and cohesive apps for
  accounts, social content, and advertisements. Do not add a separate API or
  frontend build.
- [ ] Target the available Python 3.12.3 runtime and select a compatible,
  supported Django LTS version. Record the exact tested version in
  `requirements.txt`.
- [ ] Configure environment-driven settings for `SECRET_KEY`, `DEBUG`,
  `ALLOWED_HOSTS`, `CSRF_TRUSTED_ORIGINS`, `PORT`, `SQLITE_PATH`, and
  `MEDIA_ROOT`. Never put secret values in source control.
- [ ] Set a safe local development default for the port (8000). Use an
  explicit supplied port or `PORT` when present. Treat 8000 only as a local
  default, not as CodeRange's assigned external port.
- [ ] Configure trusted host and forwarded-proxy behavior only from actual
  CodeRange information. Do not trust arbitrary forwarded headers or use a
  wildcard allowed-host setting in deployment.
- [ ] Add semantic, responsive Django templates and static assets. Keep
  JavaScript small and progressive; do not add a SPA framework.
- [ ] Use Django static files and `collectstatic`. If the CodeRange runtime
  does not serve collected static assets, add and pin WhiteNoise as the
  minimal static-file solution. Do not use it as the uploaded-media store.

### 2. Pinned Dependencies And Fresh-Clone Setup

- [ ] Add `requirements.txt` with exact tested pins for all application
  dependencies. Include Django, Pillow for image decoding, and Gunicorn for
  the WSGI runtime. Include WhiteNoise only if selected for static-file
  serving. Do not include PostgreSQL, payment, email, or object-storage
  packages.
- [ ] Keep test dependencies at zero unless a concrete test requirement
  needs an additional package; Django's built-in test runner is approved.
- [ ] Verify the fresh-environment installation command:
  `python3 -m pip install -r requirements.txt`.
- [ ] Document the supported Python version, virtual-environment commands,
  exact dependency-install command, and configuration variables.
- [ ] Add `.gitignore` entries for `.venv/`, SQLite database files, local
  uploads, collected static output, and local secret/environment files.
- [ ] Confirm a fresh clone does not depend on global packages, untracked
  configuration, a database service, or credentials.

### 3. Accounts, Authentication, And Authorization

- [ ] Define a custom Django user model before the first migration. Include a
  unique username/handle and a role restricted to `regular` or
  `campaign_manager`.
- [ ] Keep Django staff/superuser flags separate from product roles.
- [ ] Implement username/password registration, login, logout, and Django
  session authentication. Email verification and email delivery are out of
  scope.
- [ ] Make public registration create only regular users. Ignore or reject
  submitted role, staff, and privilege fields.
- [ ] Use the Django admin to let administrators provision campaign-manager
  accounts. Configure the user admin so a staff administrator can set the
  manager role; do not expose this control to public registration or a
  campaign manager.
- [ ] Enforce role checks on every protected request, not only in templates.
  Regular users cannot access campaign-management or purchase actions.
  Campaign managers cannot use regular-user social actions.
- [ ] Scope campaign reads, updates, end actions, and purchases to the owning
  authenticated manager.
- [ ] Test authentication, role assignment boundaries, forged role fields,
  unauthenticated actions, and cross-manager access.

### 4. Data Models And Migrations

- [ ] Implement a follow relationship with follower, followed user, and
  creation time. Add a unique pair constraint and reject self-follow.
- [ ] Implement a post/feed-item model with author, optional body, optional
  image path, optional reference to the root original, and creation time.
  Originals must contain text or an image. Shares reference, but do not copy,
  original content.
- [ ] Enforce share rules in a server-side service: the root original must
  belong to another user; a reshare of a reshare resolves to the root; the
  original author and content remain unchanged.
- [ ] Implement an advertisement campaign owned by a campaign manager, with
  text-first creative, optional validated destination URL, explicit status,
  simulated-purchase time, activation time, expiration time, and optional
  ended time.
- [ ] Implement the lifecycle as `draft -> active -> ended`, with
  `expires_at = activated_at + 7 days`. A successful simulated purchase
  atomically sets the simulated-purchased state and activates the campaign.
  Ending is terminal for this campaign; a manager creates a new campaign to
  advertise again.
- [ ] Compute expiry from timestamps; do not require a background scheduler.
  Treat a campaign with `now >= expires_at` as ineligible even if its stored
  status remains `active`.
- [ ] Configure Django's SQLite backend with a configurable `SQLITE_PATH`,
  defaulting to a local `db.sqlite3` file under the project directory.
  Application code must use Django ORM and migrations, not SQLite-specific
  SQL for business operations.
- [ ] Add relevant foreign keys, uniqueness/check constraints, indexes, and
  timestamp fields. Use timezone-aware Django datetimes.
- [ ] Create and review initial migrations. Verify `migrate` creates an empty
  database from a fresh clone.

### 5. Text Posts, Images, And Local Media

- [ ] Add one authenticated server-rendered post form for text-only,
  image-only, and text-plus-image posts.
- [ ] Reject an original post with neither text nor image. Return clear form
  errors and create no partial record on validation or storage failure.
- [ ] Use Django's local filesystem media storage. Default `MEDIA_ROOT` to a
  local `media/` directory and allow a runtime path override.
- [ ] Use Pillow to decode images and verify their actual format. Accept
  JPEG, PNG, and WebP; reject SVG and unsupported formats. Do not trust the
  client filename, extension, or supplied MIME type.
- [ ] Set a documented server-side maximum image size of 10 MiB. Test the
  limit at and above the boundary.
- [ ] Generate opaque storage filenames. Do not use user-supplied paths.
- [ ] Provide a safe media response path compatible with local storage and
  Gunicorn. Require regular-user authentication, resolve files through the
  post/storage record rather than a raw user path, set the content type from
  validated image data, and prevent path traversal.
- [ ] Use temporary media storage for tests and ignore local uploads in Git.
- [ ] Document that local media files may be lost on CodeRange redeployment.
  Do not add an external bucket or imply persistence.

### 6. Follow And Unfollow

- [ ] Implement authenticated, CSRF-protected follow and unfollow actions for
  regular users.
- [ ] Reject self-follow. Enforce unique relationships in the database.
  Repeated follow/unfollow submissions must be safe and deterministic.
- [ ] Reject campaign managers from social follow actions.
- [ ] Test creation, removal, duplicate submissions, self-follow, role
  restrictions, and how feed membership changes afterward.

### 7. Sharing And Post Details

- [ ] Add a one-click reshare action without commentary.
- [ ] Permit sharing only when the root original was created by another user.
  If the selected feed item is already a reshare, resolve its root original
  and apply the same ownership rule.
- [ ] Create a new share feed item with the sharer, root-original reference,
  and share time. Do not modify or duplicate original content.
- [ ] Render the sharer's attribution and the root original author's identity
  and content together.
- [ ] Allow any authenticated regular user to view an individual post detail,
  including posts by users they do not follow. Do not extend this permission
  to campaign-manager accounts.
- [ ] Test self-share rejection, other-user shares, reshare-of-reshare root
  references, original immutability, attribution, and authenticated detail
  access by a non-follower.

### 8. Organic Feed

- [ ] Build the organic feed at request time from posts authored by the
  signed-in regular user and posts authored by accounts they follow.
- [ ] Keep post-detail access separate from feed membership. A non-follower
  may view a post detail but does not receive that author's posts in their
  ordinary feed unless they follow the author or see a reshare.
- [ ] Sort newest first with a stable secondary key and paginate organic
  results. Bound query work and avoid N+1 database queries.
- [ ] Keep the empty organic feed usable and do not insert advertisements into
  it.
- [ ] Test inclusion/exclusion, chronology, tie ordering, pagination, follow
  changes, and detail visibility independently.

### 9. Advertisement Creation And Lifecycle

- [ ] Add authenticated campaign-manager pages/forms to list, create, edit,
  and end only the manager's own campaigns.
- [ ] Create campaigns in `draft` status. Use text-first ad creative and an
  optional HTTP(S) destination URL; validate the URL and never fetch it from
  the server.
- [ ] Add the simulated-purchase POST action. Clearly label the flow as a
  simulation and do not display or store real currency or payment data.
- [ ] On a successful simulation, use one database transaction to record
  simulated purchase confirmation, activate the campaign immediately, and
  set a seven-day expiration timestamp.
- [ ] Let the owner end an active campaign early. Ended or expired campaigns
  cannot re-enter the active state or be purchased again; create a new
  campaign instead.
- [ ] Define eligibility in one server-side service: purchase confirmed,
  status active, `now < expires_at`, and not ended. Ignore all eligibility
  values submitted by the browser.
- [ ] Test manager ownership, invalid transitions, duplicate purchase, early
  end, the seven-day boundary, expired campaigns, and absent real-payment
  behavior.

### 10. Deterministic Advertisement Insertion

- [ ] Compose the paginated feed from organic items plus eligible sponsored
  items. Do not persist or precompute feed copies.
- [ ] Insert exactly one eligible ad immediately after each 10th organic item
  in the complete ordered timeline. Keep the organic ordinal across page
  boundaries; ads never increment the organic count.
- [ ] Select campaigns in a stable order (activation time, then primary key)
  and cycle through that ordered set for successive ad slots. Do not randomize
  or target users.
- [ ] Insert no ad when there are no organic posts or no eligible campaigns.
- [ ] Render a prominent `Sponsored` label and visual treatment distinct from
  normal posts. Preserve the campaign identity and creative.
- [ ] Test slots after organic posts 10, 20, and 30; page boundaries;
  deterministic campaign cycling; no-eligible-ad cases; empty feeds; regular
  users versus managers; and the sponsorship label.

### 11. Validation, Security, And Error Handling

- [ ] Validate forms and uploads server-side. Show safe field errors for
  invalid input and do not expose stack traces, secret values, or filesystem
  paths.
- [ ] Use Django password hashing, sessions, CSRF protection, template
  escaping, and ORM parameterization. Do not write custom authentication or
  password cryptography.
- [ ] Protect every write route against CSRF and enforce authentication,
  role, and ownership checks on direct requests.
- [ ] Derive author, role, owner, purchase, activation, and campaign status
  from authenticated server-side state; reject over-posted fields.
- [ ] Configure non-default `SECRET_KEY`, `DEBUG=False`, allowed hosts,
  CSRF origins, and secure cookies for the actual runtime. Do not commit
  secrets.
- [ ] Keep local database and media paths out of Git. Handle SQLite and media
  write failures with safe errors.
- [ ] Test invalid and unauthorized actions, direct object-reference changes,
  CSRF behavior, malformed/oversized uploads, and URL validation.

### 12. Automated Tests

- [ ] Use Django's built-in test runner: `python3 manage.py test`.
- [ ] Use SQLite for the test database and a temporary media directory. Tests
  must not require PostgreSQL, external storage, email, or payment services.
- [ ] Test registration, login, logout, sessions, and public role assignment
  restrictions.
- [ ] Test campaign-manager authentication, role authorization, admin
  provisioning rules, manager ownership, and rejection of social actions.
- [ ] Test text, image-only, and text-plus-image creation, supported image
  formats, unsupported/corrupt files, size boundaries, and storage errors.
- [ ] Test follow/unfollow, self-follow, duplicates, and feed changes.
- [ ] Test organic feed membership/order/pagination separately from detail
  access by an authenticated non-follower.
- [ ] Test sharing only other users' root posts, root relationship and
  attribution, reshare-of-reshare behavior, and source immutability.
- [ ] Test advertisement creation/edit/end, simulated purchase, immediate
  activation, seven-day expiration, and early termination.
- [ ] Test that only simulated-purchased, active, non-expired, non-ended ads
  are eligible; test each ineligible state.
- [ ] Test exactly one ad after every 10 organic items, across page
  boundaries; test deterministic cycling, audience restrictions, no ads on an
  empty feed, and clear `Sponsored` labeling.
- [ ] Run the complete test suite after each major implementation group and
  before rendezvous. Fix failures; do not claim coverage from code inspection
  alone.

### 13. Runtime, Initialization, And CodeRange

- [ ] Support a configurable port and bind Gunicorn to `0.0.0.0`. Read
  `PORT` when set. When it is absent, allow a documented local development
  default (8000) or an explicitly supplied port so implementation/testing
  does not block.
- [ ] Do not claim that 8000 is CodeRange's assigned port. Obtain the actual
  external port and forwarded-host contract before verifying CodeRange
  external access.
- [ ] Configure SQLite at `SQLITE_PATH` (default `db.sqlite3` under the
  project directory) and media under `MEDIA_ROOT` (default `media/`).
- [ ] Document and test fresh-clone setup. Expected command sequence after
  implementation is:

  ```sh
  git clone https://github.com/alexagcruz/socmedsite.git
  cd socmedsite
  python3 -m venv .venv
  . .venv/bin/activate
  python3 -m pip install -r requirements.txt
  python3 manage.py migrate
  python3 manage.py createsuperuser
  python3 manage.py collectstatic --noinput
  python3 manage.py test
  ```

- [ ] After testing the implemented runtime, document its exact startup
  command. The intended Gunicorn form is
  `gunicorn config.wsgi:application --bind "0.0.0.0:${PORT:-8000}"`; 8000 is
  a local fallback only. Use CodeRange's assigned value when provided.
- [ ] Document that the first superuser is created interactively. Do not
  commit a default admin username/password or sample credentials.
- [ ] Do not add seed data by default. Tests create their own fixtures. If a
  development demo command is useful, keep it explicit, development-only, and
  free of known passwords or external assets.
- [ ] Run migrations on an empty SQLite file and verify the application
  starts without an external database.
- [ ] Document SQLite data-file and local-media persistence limitations.
  Images may be lost on redeployment by approval. Do not claim account/post
  durability until `SQLITE_PATH` points to storage with verified persistence.
- [ ] Test local access at the configured port. Test external CodeRange access
  only after the assigned port and forwarded-host details are available.

### 14. Documentation, Reproducibility, And Rendezvous

- [ ] Update `doc/wiki/` after implementation so it describes only working
  behavior, exact setup/migration/test/start commands, SQLite and media
  limitations, and actual CodeRange configuration.
- [ ] Add `doc/wiki/footguns/` notes for the 10-organic-item cadence, root
  reshare restrictions, simulated campaign expiration, role boundaries, and
  local-file durability where needed.
- [ ] Keep `doc/canonical/mvp-decisions.md` aligned with approved decisions.
- [ ] Verify a fresh clone can install from pinned requirements, initialize
  SQLite, run migrations and tests, create an admin interactively, and start
  without untracked local state or external service credentials.
- [ ] Verify the running app against every acceptance criterion and resolve
  integration problems.
- [ ] Confirm a workable local application state. Record CodeRange external
  access as unverified if port/forwarded-host values remain unavailable.
- [ ] Review Git status and scoped Conventional Commits. Merge the completed
  feature branch into `main` only after validation. Push and verify the remote
  state.
- [ ] Report test results, current branch, Git status, push result, and
  limitations.

## Acceptance Criteria

- [ ] A regular user can register, authenticate, and create text, image-only,
  or combined posts. Public registration cannot grant campaign-manager role.
- [ ] An administrator can provision campaign-manager accounts. A manager can
  authenticate and manage only their campaigns, and cannot perform regular
  social actions.
- [ ] A user can follow and unfollow another user; self-follow is rejected
  and duplicate relationships cannot be stored.
- [ ] The organic feed includes only the user's own posts and posts from
  followed accounts, uses stable newest-first ordering, and is paginated.
- [ ] Any authenticated regular user can view an individual post detail,
  including a non-followed user's post.
- [ ] A regular user can share an eligible post created by another user.
  Shares link to the root original, preserve its author/content, and never
  modify it. Self-owned root posts cannot be shared.
- [ ] A campaign manager can create an ad, simulate its purchase without real
  currency/payment data, activate it immediately for seven days, and end it
  early.
- [ ] Only active, simulated-purchased, non-expired, non-ended campaigns are
  eligible. All other states are excluded.
- [ ] One ad appears after every 10 organic posts, counting across pages, in a
  deterministic order. Ads are regular-user-only, labeled `Sponsored`, do
  not change organic order, and do not appear in an empty feed.
- [ ] Automated tests cover authentication, authorization, posts/images,
  follow/unfollow, feed visibility, sharing, manager restrictions, campaign
  lifecycle, ad eligibility/insertion/labeling, and invalid/unauthorized
  behavior.
- [ ] A fresh clone installs with `python3 -m pip install -r requirements.txt`,
  migrates SQLite, runs tests, creates an admin interactively, and starts on
  `0.0.0.0` using a specified port or local fallback.
- [ ] Runtime docs state the actual tested startup command, CodeRange port
  limitation, SQLite data durability status, and approved local-media
  redeploy limitation.

## Execution Status

- [ ] Open runtime questions recorded and assigned to the correct stage.
- [ ] Implementation started on a feature branch.
- [ ] All implementation, test, initialization, reproducibility, and
  documentation checklist items completed.
- [ ] Rendezvous, merge, and remote verification completed.

No implementation tasks in this plan have been executed.
