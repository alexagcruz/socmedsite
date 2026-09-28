# Social Media MVP Feasibility Study

- Status: Study approved on 2026-09-28. See `doc/canonical/mvp-decisions.md` for
  authoritative decisions and remaining open items.
- Date: 2026-09-28
- Scope: Feasibility and recommended approach only. No application code is included.

## Executive Recommendation

The requested MVP is feasible as a small, single-deployment web application.
It does not need microservices, a separate frontend application, a recommendation
engine, or an advertising network.

Approved starting architecture:

- Use Django with server-rendered HTML templates and ordinary forms.
- Use one PostgreSQL relational database for accounts, follows,
  posts, shares, campaigns, and purchase state.
- Use Django's authentication, forms, ORM, migrations, CSRF protection, and
  test client rather than replacing these features with custom systems.
- Keep feed and campaign logic in small application services. Keep a single
  deployable application and a single database.
- Start with a chronological, read-time feed of the signed-in regular user's
  posts and the posts of users they follow.
- Represent a share as a new feed item that references the original post. Never
  edit the original post when another user shares it.
- Clearly label every inserted campaign item as sponsored.
- Use an explicitly simulated purchase flow for the MVP. Do not collect card
  data or represent a simulation as a real charge.
- Use a persistent local media volume for images only if CodeRange guarantees
  that the volume persists across restarts and deployments. Otherwise, approve
  an S3-compatible object store and its credentials before implementation.

The product owner approved the recommended technology stack, architecture,
MVP scope, and technical approach on 2026-09-28. The canonical decision record
is `doc/canonical/mvp-decisions.md`. The remaining details in
[Open Questions](#open-questions) are not implied approvals.

## Repository And Evidence

At the time of this study, the repository contains `AGENTS.md` and no
application code, dependency declaration, tests, or CodeRange deployment
configuration. The remote is configured as `origin`, but no remote refs were
reported during inspection. There is therefore no existing framework,
database, or runtime setup to preserve. Exact CodeRange support for Python,
PostgreSQL, persistent disks, forwarded-host headers, and build commands must
be confirmed before deployment decisions are final.

This study uses the requirements in the request and the workflow in
`AGENTS.md`. It does not assume that any external service or credential is
available.

## Product Scope

### MVP Outcomes

Regular users can:

- Create an account and sign in and out.
- Create a text post or an image post. A post may contain text and an image if
  that is useful; it should not require a separate image-post subsystem.
- Follow and unfollow another regular user.
- View their own posts and posts from accounts they follow in reverse
  chronological order.
- Reshare a post while retaining attribution and a durable relationship to its
  original.

Campaign managers can:

- Authenticate with a distinct campaign-manager role.
- Create a simple advertisement creative.
- Complete the approved purchase flow.
- See their own campaign and its purchase/activation state.

The feed can insert eligible advertisements into the regular-user feed. Each
advertisement must be visually distinct from a social post and marked as
`Sponsored` (or equivalent). The feed must not select a draft, unpaid,
unsuccessfully purchased, paused, or expired advertisement.

### Excluded From The Basic MVP

Unless a later approved requirement changes scope, defer:

- Likes, comments, direct messages, notifications, and user search.
- Hashtags, trending, recommendations, ranking, and machine learning.
- Private accounts, follow requests, blocking, and complex feed privacy rules.
- Video, image galleries, image editing, and in-app content moderation tools.
- Demographic or behavioral advertising targeting, bidding, budgets, auctions,
  impression/click analytics, and A/B testing.
- Multiple ad formats, campaign groups, and multi-creative campaigns.
- Real payment processing if the simulated-purchase option is approved.
- Native mobile clients, a public API, microservices, background queues, and
  a separate frontend deployment.
- Email verification and password-reset email unless those are explicitly
  required. These features need an email service and its credentials.

Keep the deployment maintainable and secure, but do not add features that do
not support the stated user flows or required business rules.

## Feasibility And Main Risks

Feasibility is high. The core operations are standard relational web
application behavior. Django provides established support for account
authentication, permissions, forms, relational persistence, migrations, and
automated request testing. The main uncertainties are product and hosting
choices, not technical feasibility.

Important risks are:

- A campaign-manager role that users can assign to themselves would permit
  privilege escalation. Public registration must not trust a submitted role.
- A simulated purchase can be mistaken for a real charge unless the UI,
  database state, and documentation label it clearly.
- A local media directory may be ephemeral on CodeRange. Image posts would
  break after restart or redeployment if storage is not persistent.
- Ad creative and purchase records must be scoped to their owner. Guessing a
  campaign identifier must not allow one manager to edit or inspect another
  manager's campaign.
- Feed queries can become slow if they load every post or follow relationship.
  Paginate and index the required relationships from the start.
- Real payment processing adds webhook authenticity, retries, duplicate
  events, refunds, reconciliation, and financial support obligations. A
  browser redirect alone must never confirm a real payment.
- The CodeRange port and forwarded host must be configured correctly. A local
  `localhost` test does not verify external CodeRange access.

## Roles And Permissions

Approved role model:

- Store an explicit role on a custom Django user model, with at least
  `regular` and `campaign_manager` values.
- Keep staff/superuser administration separate from these user-facing roles.
- Make public registration create only regular users. Provide campaign-manager
  accounts through an administrator-approved process, invitation, or another
  process that the product owner approves. Do not expose a trusted role picker
  in public registration.
- Require server-side role checks for every protected action. Hiding a button
  in a template is not authorization.
- Scope campaign reads and writes to the authenticated manager who owns them.
- Define behavior for unknown or missing roles as denied, not as regular or
  privileged by accident.

The custom user model should be created before the first database migration.
Changing the user model later is disruptive. Django groups and permissions are
an alternative, but are more flexible than this MVP needs if every account has
exactly one product role.

Whether a campaign manager can also act as a normal social user is unresolved.
Do not infer this from the shared authentication system. A single account can
authenticate while having an exclusive product role.

## Major User Flows

### Regular User

1. Register a regular account with a unique handle and password.
2. Authenticate using Django's session authentication.
3. Submit a text post or an image post through a CSRF-protected form.
4. Follow or unfollow another eligible user. Reject self-follow. Repeating a
   follow or unfollow should have a defined, safe result and must not create
   duplicate relationship rows.
5. Request a paginated feed. See the user's own posts and followed accounts'
   posts in reverse chronological order, with reshares retaining attribution.
6. Reshare an existing post. Create a new relationship/feed item; do not alter
   the original content, author, or timestamp.

### Campaign Manager

1. Authenticate with an account whose role was provisioned through the
   approved process.
2. Create and edit only the manager's own draft advertisement.
3. Submit the campaign to the purchase flow.
4. On a confirmed simulated purchase, transition the campaign to the
   approved eligible state. A failed or pending purchase must not make it
   eligible.
5. Pause or end the campaign if the approved lifecycle includes those actions.

### Feed Advertisement

1. Build the organic feed from the authenticated regular user and follow
   relationships.
2. Retrieve only campaigns that satisfy every approved eligibility condition.
3. Insert sponsored feed items at the approved fixed interval or placement
   policy.
4. Render a sponsorship label and campaign creative in a layout that cannot
   be confused with a user's post.

## Data And Relationships

The following model is sufficient for the approved MVP. Names are
conceptual, not implementation requirements.

| Entity | Main data and relationships | Important rules |
| --- | --- | --- |
| User | Authentication fields, unique handle, role, created time | Public signup creates regular users only. Manager role is not user-editable. |
| Follow | Follower user, followed user, created time | Unique follower/followed pair; no self-follow. |
| Post | Author, optional text, optional image key, optional original-post reference, created time | Original posts contain text and/or an image. Reshares reference an original; they do not copy or mutate its content. |
| AdCampaign | Owning campaign manager, simple creative, optional destination URL, state, approved schedule | Only the owner or authorized staff can manage it. Validate destination URLs. |
| AdPurchase | Campaign, purchase mode/state, confirmation time, provider reference if real payments are approved | A campaign is eligible only when its purchase state is confirmed by the selected flow. Never record card numbers or security codes. |

For reshares, use one feed-item/post model with a nullable reference to the
original post, or an equivalent explicit share record. If one model is used,
validate that a reshare points to the root original rather than building an
unbounded chain. Do not duplicate the original body or image into the share.
The feed card should identify the person who shared it and the original
author. The open question below recommends a one-click reshare without added
commentary for the smallest MVP.

Add database uniqueness and foreign-key constraints for follows and ownership
relationships. Use indexes for post chronology, follow lookups, and campaign
eligibility. Add database checks where they can reliably enforce content
invariants; keep cross-row rules in a tested service when the database cannot
enforce them directly.

## Architecture And Interfaces

### Recommended Architecture

Use a conventional Django monolith with a single relational database:

- Django views accept HTML form submissions and render server-generated
  templates.
- Django forms validate user input and uploaded files.
- Small service functions own important transitions such as follow/unfollow,
  share creation, purchase confirmation, and feed assembly. Use database
  transactions for state changes that must be atomic.
- Django ORM models express relationships and constraints.
- Django templates render social posts, shares, and sponsored ads as distinct
  feed item types.
- Static assets are served by the platform or a small static-file solution.
- Media files use Django's storage interface so the approved storage backend
  can be configured without changing post behavior.

No separate JSON API is required for the listed browser flows. A REST API,
React/Vue frontend, task queue, cache, and service split would add deployment
and testing costs without a stated requirement. Add them only if an approved
future client or measured workload requires them.

### Major Technology Recommendations

| Area | Recommendation | Why it fits | Alternatives and tradeoffs |
| --- | --- | --- | --- |
| Application framework | Python and the currently supported Django LTS release at implementation time | Django includes authentication, sessions, ORM, migrations, forms, CSRF controls, and a test client. One team can implement the complete server-rendered MVP in one deployable unit. | Flask or FastAPI are viable but require more decisions and packages for server-rendered forms, authentication, and permissions. Rails is similarly capable but changes the language/runtime choice. Confirm CodeRange's supported Python version before approval. |
| Frontend | Django templates, semantic HTML, responsive CSS, and only small progressive JavaScript where needed | Required flows are form and feed pages. Server rendering avoids a second application, API contract, token flow, and frontend build pipeline. | React/Vue or another SPA is reasonable for richer client interactions, but is unnecessary for this scope and increases deployment and end-to-end test complexity. |
| Database | PostgreSQL | It provides mature transactions, constraints, indexes, and concurrency behavior for shared social and purchase state. It is a sound production database before concurrent feed and purchase writes become important. | SQLite is simpler for local development and can work for a small single-process demo, but has weaker concurrency characteristics and may differ from production behavior. PostgreSQL is approved, subject to confirming CodeRange availability. If it is unavailable, request approval before using a fallback. |
| Image handling | Django's built-in upload and storage interfaces; validate image content and store opaque generated names. Prefer a CodeRange persistent media volume for the MVP if available. | A durable mounted directory has few dependencies and is easy to inspect. The storage interface leaves a path to object storage if needed. | An S3-compatible object store is better for durable, scalable media and multiple application instances, but needs a bucket, endpoint, access credentials, network access, and likely an additional storage package. Ephemeral local disk is not acceptable for user images in a deployed app. |
| Test tools | Django `TestCase`, Django test client, temporary/fake media storage, and the selected database in CI | These tools cover models, forms, permissions, views, and core user flows without a browser stack. Django's runner avoids a pytest dependency for this modest suite. | `pytest` with `pytest-django` is well established and useful for larger suites. Playwright is appropriate for a small number of true browser checks if front-end interaction justifies its installation and runtime. Neither is necessary to begin. |
| Runtime server | Gunicorn with Django WSGI for a conventional synchronous application | It is established and simple to bind to the CodeRange-provided port. | ASGI/Uvicorn is suitable if async or long-lived connections become requirements. Those are not present in this MVP. Do not introduce an async server solely for trend or speculative scale. |
| Dependency management | Declare Python project metadata in `pyproject.toml`; use a committed lock file and a CodeRange-compatible, documented install command | This makes direct dependencies and reproducible versions explicit. | A pinned `requirements.txt` is broadly supported by Python hosts. If CodeRange does not support the selected lock/install tool, use its supported pip-based installation while retaining a single documented source of dependency truth. Verify this against the actual CodeRange build environment. |

Do not use a development server for external production traffic. Set secrets
and database configuration through environment variables. Keep debug mode off
in the deployed environment.

## Database And Feed Behavior

PostgreSQL is the selected production database, subject to CodeRange support.
All social and campaign data should use database foreign keys and transactions.
The implementation should use migrations and document how a fresh database
is initialized. Do not rely on data that exists only in a developer's local
database.

Recommended organic feed behavior:

- Include posts authored by the signed-in user and users they follow.
- Sort by creation time, newest first. Do not add engagement ranking or
  recommendations.
- Paginate the organic result, initially using a modest page size. Do not load
  the full history into memory.
- Query relationships efficiently and avoid per-item database queries. Use
  appropriate joins/prefetching and indexes.
- Define ties with a stable secondary key so pagination does not reorder
  posts with the same timestamp.
- Show a useful empty state when the user has no eligible social posts.

This read-time query is called fan-out-on-read. It is the simplest approach
for a basic network because a new post does not need to be copied into every
follower's timeline. A precomputed timeline, ranking system, or feed cache is
deferred until measured traffic justifies it.

## Post Creation, Images, And Sharing

### Text And Image Posts

Use one post form and model for text, image, or text-plus-image content if
product approval permits both. Validate that an original post has at least
text or one valid image. Enforce a configurable file-size limit. Check file
content and decode the image; do not trust the extension or browser-supplied
MIME type. Reject formats that are not explicitly supported. Avoid SVG unless
there is a clear sanitization policy because it can contain active content.

Generate storage keys rather than using untrusted original file names. Serve
media as media, not executable application files. Keep upload handling behind
the storage interface. The implementation plan must test invalid formats,
oversized files, and storage errors. Image moderation, editing, and video are
out of scope.

The application needs durable image storage if uploaded content must survive
deployments. The choice between a CodeRange persistent volume and an external
object store is a human/platform decision, not something to guess in code.

### Sharing

Recommended simplest meaning: a user re-shares an existing post into their
feed without writing commentary. Store a new share/feed item with the
resharing user, a reference to the root original, and its own creation time.
Render the original content and author inside the new item. This preserves
attribution and makes the new share visible in the feed without changing the
original. Do not copy the original body into the share. If commentary is
required, add it to the share record only after approval; do not store it by
mutating the source post.

## Advertisement Creation, Purchase, And Feed Insertion

### Advertisement Lifecycle

Separate creative state from purchase confirmation. A suggested minimal
lifecycle is:

1. A manager creates a draft campaign.
2. The manager submits a purchase request through the selected payment mode.
3. The purchase becomes confirmed only through the trusted purchase path.
4. A confirmed campaign becomes eligible only while active and within its
   approved time window.
5. A manager can pause/end the campaign if those controls are approved.

Treat these states as explicit, testable values. Never infer purchase from a
query parameter, a user-editable flag, or the client redirect from a payment
provider. Feed eligibility should require a confirmed purchase plus active
campaign state and any approved date conditions. A draft or failed/pending
campaign must never appear.

For a simulated flow, use an unmistakable simulation state and label the
checkout UI as a demonstration. Do not describe a simulation as a real charge
or persist fabricated payment-provider details. Use a database transaction
so repeated form submissions do not create conflicting purchases.

### Real Versus Simulated Purchase

Real processing is possible but not a small extension to a button. It requires
a payment provider account, a product/pricing model, secure checkout, signed
webhook validation, idempotent event processing, failure/refund behavior,
reconciliation, operational secrets, and a publicly reachable HTTPS callback.
The application must never receive or store raw card data. Provider sandbox
mode is useful for integration tests but still needs provider credentials and
webhook configuration.

For the stated basic MVP, the approved approach is a simulated purchase flow,
clearly marked as such. It verifies campaign ownership, state transitions,
and feed eligibility without creating financial or credential dependencies.
It is not suitable for collecting money or claiming advertisements were paid
for in a real-world sense.

Even with simulation, the product owner must specify what the manager is
"purchasing": a demo activation, fixed campaign period, credits, or another
unit. Do not invent a price, currency, duration, refund policy, or financial
claim. Those details are not present in the request.

### Feed Insertion

Approved MVP policy:

- Show ads to all eligible regular users. Do not implement personal or
  demographic targeting.
- Insert one sponsored item after a fixed block of organic posts. Select and
  approve the exact ratio before implementation; a small fixed interval is
  easier to explain and test than random ranking.
- Rotate eligible campaigns in a simple, non-personalized order. Do not build
  bidding, relevance scoring, impressions, clicks, or per-user profiles.
- Mark each item `Sponsored` and visually separate it from user-authored
  posts. Keep the campaign identity clear.
- Paginate the organic feed first, then compose the requested page with
  eligible ad items. This keeps page boundaries and social ordering
  understandable. Decide whether ads should also appear when the organic feed
  is empty; the approved policy is no, to avoid showing a monetized empty
  state.
- Keep eligibility and insertion in a testable server-side function, not only
  in template logic.

The approved fixed-frequency and deterministic-rotation policy is not a final
ad-sales policy; the exact frequency remains open. If real
advertising revenue is intended, targeting, pacing, fairness, impression
accounting, and pricing need a separate approved product design.

## Validation And Error Handling

- Validate all forms on the server, including role-sensitive operations.
- Reject malformed or missing fields with field-level feedback. Do not expose
  stack traces or secret configuration to users.
- Reject self-follow and invalid share targets. Handle repeated follow/share
  submissions safely according to the approved product behavior.
- Return a not-found or denied response when a user accesses a campaign that
  they do not own. Do not rely on hidden links for access control.
- Keep campaign purchase and activation state transitions atomic.
- Handle missing images, invalid images, file-size violations, and storage
  outages with a clear error and no partially-created invalid post.
- Validate advertisement destination URLs. Do not fetch a user-supplied URL
  from the server. If links open a new tab, apply safe browser link attributes.
- Use database constraints for invariants such as unique follows. Convert
  expected constraint failures into safe user-facing outcomes.

## Security And Privacy

Security requirements for implementation include:

- Use Django's password hashers, session authentication, CSRF protection, and
  output escaping. Do not build custom password cryptography or session
  tokens.
- Require authentication and server-side role/ownership checks for protected
  actions. Test direct requests as well as template visibility.
- Do not trust role, owner, purchase state, or campaign status supplied by a
  browser form.
- Keep `SECRET_KEY`, database credentials, object-storage credentials, and
  payment credentials in runtime secrets/environment settings. Never commit
  them.
- Configure allowed hosts, HTTPS behavior, secure cookies, and CSRF trusted
  origins for the actual CodeRange forwarded host. Trust forwarded headers
  only when the platform proxy is known and configured.
- Validate uploaded image content and size. Use generated storage names and
  keep media separate from executable application code.
- Limit public signup and file uploads against basic abuse. Use platform
  controls or a small, justified rate-limit mechanism if needed; do not add a
  distributed rate-limit service by default.
- Keep user data collection minimal. The request does not define private
  profiles or visibility restrictions, so feed visibility must be clarified
  before implementation if public posts are not intended.
- Do not store payment card data. A real payment flow must use a provider's
  hosted/tokenized flow and verify server-to-server events.

## Automated Testing Strategy

Use Django's test runner as the initial test system. Tests should state
observable behavior, not constrain the project to unnecessary internal
details. Use the same database engine in integration/CI tests as the selected
deployment engine where practical. Use temporary storage or a fake storage
backend so tests do not need an external bucket.

Required test areas:

- Registration creates the regular role and cannot accept a privileged role
  from submitted data. Manager-only operations reject regular users, and
  regular-only actions follow the approved role policy.
- Unauthenticated users cannot perform protected actions.
- A manager can create and manage their own campaign but cannot access
  another manager's campaign by changing an identifier.
- Text and image post validation accepts supported content and rejects empty,
  malformed, oversized, and unsupported uploads.
- Follow creates one relationship, rejects self-follow, and unfollow removes
  the relationship. The feed includes only the user and followed accounts'
  posts under the approved visibility model.
- Feed chronology and pagination are stable and bounded.
- Resharing creates a separate feed item, preserves the original author and
  content, and does not change the original record. Invalid targets are
  rejected.
- Draft, unpaid, failed, paused, or expired advertisements do not appear.
  Only approved confirmed and active advertisements appear.
- Feed advertisements obey the approved placement ratio and are marked
  sponsored. Normal social ordering remains intact.
- Simulated purchase is visibly simulated and safe under repeated submission.
  If real payments are later approved, test webhook signature failure,
  duplicate events, delayed events, and payment failure without live charges.
- CSRF-protected form behavior and safe handling of invalid requests.

A small number of browser-level checks may be added if template rendering and
mobile layout cannot be validated adequately through Django's test client.
Do not introduce a browser automation stack without a reason. The current
repository has no application tests to run; this study does not execute any.

## CodeRange And Runtime

The deployment must bind externally to `0.0.0.0` and the port provided by
CodeRange, expected to be available through the `PORT` environment variable.
Do not hard-code localhost or assume a fixed port. A production-shaped
runtime can run Django through Gunicorn's WSGI application and bind to
`0.0.0.0:${PORT}`. A local default port may be used only for development.

Before deployment, confirm:

- The supported Python version and dependency installation mechanism.
- The application start command and required health-check behavior.
- Whether PostgreSQL is available and how connection configuration is
  provided.
- Whether persistent media volumes exist and survive restarts/deployments.
- Which public/forwarded host and HTTPS headers CodeRange supplies.
- How static files are built and served.
- Whether the app is expected to accept external traffic during deployments
  and what port value is required.

The deployed app should use `DEBUG=False`, an environment-provided secret,
explicit trusted hosts/origins, database migrations, and a documented static
asset step. Test through the forwarded CodeRange host; a localhost request is
not an external deployment test.

## Dependency Management And Reproducibility

Declare all runtime and development dependencies in a standard Python project
file. Commit resolved dependency versions in the supported lock format or
equivalent pins. The final choice between `pyproject.toml` plus a lock file
and a pip-compatible pinned requirements file depends on CodeRange's build
support; verify it and document one canonical installation command.

Reproducibility requirements for the later plan:

- Document exact Python version, dependency installation, environment
  variables, database setup, migration, static-file, image-storage,
  test, and start commands.
- Provide a safe example environment file with placeholders, not secrets.
- Document how to create the first privileged operator without shipping a
  default password or committed production account.
- Ensure media and required project files are not ignored accidentally.
- Verify from a fresh clone and an empty database before submission.
- Do not depend on manually installed CodeRange packages or untracked local
  state.

## Exogenous Inputs

These inputs cannot be created by the coding harness and must not be invented:

| Input | Needed when | Notes |
| --- | --- | --- |
| CodeRange project/runtime details, supported Python version, port, forwarded host, and deployment commands | Deployment planning | Confirm against the actual CodeRange project settings. The requested external bind is `0.0.0.0:[PORT]`. |
| PostgreSQL service/credentials | If PostgreSQL is approved | Do not commit credentials. If CodeRange does not provide a database, decide whether to provide one or approve a different database with known tradeoffs. |
| Durable media volume or object-storage account, bucket, endpoint, and credentials | Required for persistent uploaded images | A local ephemeral container disk does not satisfy persistence. No storage service or credentials are present in this repository. |
| Payment provider account, API keys, webhook secret, business identity, and publicly reachable HTTPS webhook URL | Only if real payment processing is approved | Do not start live integration before ownership, pricing, currency, and refund behavior are supplied. Never request or store card numbers in this application. |
| Campaign pricing, purchase unit, currency, duration, and activation rules | Needed for a meaningful purchase flow | The request does not define these. A simulated flow still needs a clearly specified demo behavior. |
| Campaign-manager onboarding policy and any invitation/approval process | Before enabling manager accounts | Do not let public registration choose the manager role. |
| Logo, brand assets, or ad assets | Only if required by product/design | No assets are supplied. They are not technically required for a text-first MVP. |
| Email provider account and credentials | Only if email verification, password reset, or transactional email is required | No email provider is required for username/password authentication without email delivery. |
| Domain/business/legal policies for real advertising and user-generated content | Before public commercial launch | Requirements are unspecified. Do not invent policy or legal claims. |

The approved simulated purchase flow and text-first UI do not require a
payment provider, image bucket, email service, or third-party ad service.
Persistent images still require either a confirmed durable CodeRange volume or
an approved external storage service.

## Tradeoffs And Deferred Risks

- **Server rendering versus SPA:** server rendering is less interactive but
  greatly reduces deployment, authentication, API, and test complexity. The
  stated flows do not need a SPA.
- **PostgreSQL versus SQLite:** PostgreSQL needs an external database service
  or CodeRange integration but provides stronger production concurrency and
  constraint behavior. SQLite is simpler but less suitable for concurrent
  writes and a multiuser deployed service.
- **Local persistent disk versus object storage:** local persistent storage is
  cheaper and simpler if CodeRange guarantees durability and single-instance
  access. Object storage is more durable and scalable but requires external
  credentials, configuration, and network service availability.
- **Simulated versus real purchase:** simulation tests product state without
  financial risk but cannot claim actual payment. Real processing enables
  genuine purchases but adds provider operations, webhooks, and business
  decisions.
- **Chronological read-time feed versus precomputed feed:** read-time query is
  easy to reason about and sufficient for MVP traffic. A high-follow-count
  network may later require a different architecture; there is no evidence
  for that complexity now.
- **All-user ads versus targeting:** all-user delivery is explainable and
  avoids collecting sensitive profile data. It may be less useful for
  advertisers, but the request does not specify data or business rules for
  targeting.
- **Exclusive versus combined role:** exclusive roles simplify authorization
  and registration. Allowing managers to use social functions could improve
  account experience but makes capabilities additive and needs explicit
  product rules.

## Open Questions

The recommendations in questions 1-6 below were approved as the product and
technical baseline on 2026-09-28. Some operational parameters and external
inputs remain unresolved. The canonical record is
`doc/canonical/mvp-decisions.md`.

### 1. Advertisement Purchase (Approved)

**Approved decision:** Use a clearly labeled simulated purchase for this MVP.
It models campaign purchase and feed eligibility without processing money,
provider credentials, webhooks, refunds, or financial operations. It must not
claim or imply that a real charge occurred.

**Still needed:** Define what the simulated purchase represents, including any
demo price, currency display, campaign duration, activation, and expiration
rules. Real payment processing is outside the approved MVP scope.

### 2. Advertisement Feed Insertion (Approved Policy)

**Approved decision:** Use a fixed placement interval after a block of organic
posts, with simple non-personalized rotation among eligible campaigns. Mark
each ad clearly as `Sponsored`; do not add impression tracking or auction
logic. Do not show ads when there are no organic posts.

**Still needed:** Select the exact numeric frequency. The fixed-interval
policy, simple rotation, sponsorship label, and empty-feed behavior are
approved.

### 3. Ad Targeting (Approved)

**Approved decision:** Target all regular users equally for the MVP. Use no
personal, demographic, behavioral, location, or inferred-interest targeting.
This avoids a new privacy/data model. Any targeting feature requires a later
approved scope change and its own privacy and data requirements.

### 4. Campaign Managers' Social Actions (Approved)

**Approved decision:** Keep campaign-manager accounts separate from regular-user
actions for the first MVP. They can authenticate and manage campaigns, but
cannot create social posts, follow users, or use a regular social feed. This
keeps the role boundary and authorization rules simple. Staff administration
is separate from both roles.

**Still needed:** Choose how campaign-manager accounts are provisioned, such as
administrator creation or invitation. Public self-selection is prohibited.

### 5. Uploaded Image Storage (Approved Conditional Approach)

**Approved decision:** Use Django's storage abstraction. For deployment, prefer a
CodeRange-provided persistent media volume if it is durable across redeploys
and adequate for one application instance. If not, use an S3-compatible
service. Use local filesystem storage only for development or a verified
persistent deployment volume.

**Still needed:** Confirm CodeRange persistent-volume support. If object
storage is needed, an owner must provide the provider, bucket/endpoint, and a
secure credential delivery path. The repository contains none of these.

### 6. Meaning Of "Share Post" (Approved)

**Approved decision:** Treat sharing as a one-click reshare without
commentary. Store a new feed
item pointing to the original post and show the original author/content with
the sharer's attribution. Do not mutate or duplicate the original. Do not add
commentary in the MVP. A reshare of a reshare points to the root original.

**Still needed:** Decide whether users can reshare their own posts.

### Additional Open Decisions Before Planning

- Confirm the campaign lifecycle details that are not covered by the approved
  simulation: activation timing, pause behavior, and expiration.
- Confirm whether post detail/profile pages may expose posts to users outside
  the feed's author-and-followed-account set.
- Confirm CodeRange support for PostgreSQL and durable media storage. If
  PostgreSQL is unavailable, request approval before selecting a fallback.
- Confirm the target Python version and dependency installation contract for
  CodeRange.
- Confirm CodeRange's forwarded-host behavior, required port, and deployment
  command.
- Confirm the exact fixed advertisement interval.
- Confirm whether users can reshare their own posts.
- Select how campaign-manager accounts are provisioned.
- Supply object-storage credentials if CodeRange has no durable media volume.
- Define the simulated purchase unit, any demo price display, and campaign
  duration. Do not invent real currency or charges.

## Approved Decision Summary

The approved baseline is Django, server-rendered templates, PostgreSQL,
Django's test runner, a single monolithic deployment, chronological read-time
feeds, exclusive regular-user and campaign-manager roles, one-click attributed
reshares, all-user non-targeted advertisements, fixed clearly labeled
sponsored placements, and simulated purchases. Use persistent local media
storage only when CodeRange confirms durability; otherwise use an approved
object-storage service. The open operational details above must be resolved
before the plan fixes dependent behavior. No application code has been
implemented.
