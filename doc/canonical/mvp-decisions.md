# Approved MVP Decisions

- Status: Approved baseline
- Approval recorded: 2026-09-28
- Source study: `doc/study/1790599824_social_media_mvp_feasibility.md`

This document records the human-approved product scope, architecture, and
technical direction for the social-media MVP. Treat these decisions as
authoritative. Do not infer approval for the remaining items in
[Open Decisions](#open-decisions).

## Subsequent Approved Decisions

On 2026-09-28, the product owner approved decisions that supersede parts of
the original study recommendation:

- Use SQLite instead of the study's PostgreSQL recommendation. The current
  CodeRange environment has no usable PostgreSQL server or connection
  configuration. An external database would add accounts, credentials, and
  networking that this MVP does not require. Use Django's normal database
  abstraction and do not add SQLite-specific application logic.
- Store uploaded images in local application media storage. Loss of image files
  after a CodeRange redeploy is an accepted MVP limitation. Do not add an
  external object store or its credentials.
- Use a fully pinned `requirements.txt`, installed with
  `python3 -m pip install -r requirements.txt`. The environment has Python 3.12.3
  and pip 24.0, and has `venv`/`ensurepip`; it does not have `uv`, Poetry,
  PDM, or `pip-compile`.
- Do not require `PORT` to exist during implementation. The application must
  bind to `0.0.0.0` and accept a specified runtime port. Do not guess the
  externally assigned CodeRange port or forwarded host.
- Insert one eligible advertisement after every 10 organic feed posts.
- A simulated purchase activates a campaign immediately. It remains active
  for 7 days unless its manager ends it earlier. Only active, non-expired,
  purchased campaigns are eligible for feed insertion.
- Administrators provision campaign-manager accounts. Public registration
  cannot assign this role.
- Users cannot reshare their own posts. A share links to the root original and
  preserves its original author.
- Any authenticated regular user can view an individual post. A regular user's
  feed remains limited to their own posts and posts by accounts they follow,
  plus eligible advertisements.

The original PostgreSQL recommendation remains in the study as historical
analysis. The SQLite decision above is the approved implementation choice.

## Product Scope

The MVP supports two distinct user types:

- Regular users can create accounts, authenticate, create text and image
  posts, follow and unfollow users, view a chronological feed, and reshare
  posts.
- Campaign managers can authenticate, create advertisement campaigns, and
  complete the approved simulated purchase flow.
- Purchased, active advertisements can appear in regular-user feeds. Ads must
  be clearly labeled as sponsored and visually distinct from social posts.

Keep the MVP limited to these flows. Defer likes, comments, messaging,
notifications, search, recommendation or ranking systems, targeting,
analytics, video, native clients, a public API, and real payment processing.
Do not add functionality without an approved requirement.

## Approved Architecture And Technology

| Area | Approved approach | Reason and boundary |
| --- | --- | --- |
| Application | Python 3.12.3 in the inspected environment with a compatible, supported Django LTS release | Django provides authentication, sessions, ORM, migrations, forms, CSRF protection, and request testing in one established framework. Pin the Django version after compatibility and runtime tests. |
| Architecture | One Django monolith and one SQLite database | The required flows do not justify microservices, a separate API, a task queue, or a distributed system. Keep important state transitions in small, testable server-side services. SQLite avoids an external database service for this basic MVP. |
| Frontend | Django templates, semantic HTML, responsive CSS, and minimal progressive JavaScript | Server-rendered pages fit the feed and form flows and avoid a separate frontend build and API authentication design. A React/Vue SPA is not part of the MVP. |
| Database | SQLite through Django's standard database backend | The current environment has no usable PostgreSQL service or connection settings. SQLite is included with Python and removes an external service from the MVP. It has lower write concurrency and local-file durability limits; do not add SQLite-specific behavior to application services. |
| Image handling and storage | Django's storage interface backed by local application media files | This meets image-post requirements without an external account or credential. Image-file loss across CodeRange redeployments is an accepted MVP limitation. |
| Tests | Django's built-in test runner and test client; temporary media storage | These tools cover permissions, forms, feed behavior, and business rules without adding a separate test framework. Tests use SQLite for the approved MVP. |
| Runtime | Gunicorn with Django WSGI for the synchronous application | Bind to `0.0.0.0` and accept an explicit port. Read `PORT` when supplied, but allow a documented local development default so its absence does not block implementation. Do not claim the default is CodeRange's assigned port. |
| Dependencies | Fully pinned `requirements.txt` installed with `python3 -m pip install -r requirements.txt` | pip 24.0 is available, while lockfile tools such as `uv`, Poetry, PDM, and `pip-compile` are not. Pinned requirements are the simplest supported reproducible workflow. |

Use Django session authentication. Store an explicit `regular` or
`campaign_manager` role on a custom user model created before the first
migration. Keep staff/superuser administration separate from those product
roles. Public registration must create regular users only; users must not be
able to grant themselves campaign-manager permissions. Enforce authorization
on the server and scope campaign operations to their owner.

## Approved Social Behavior

- Build the feed at read time from the signed-in user's posts and posts by
  accounts they follow.
- Order social posts newest first, paginate them, and use a stable ordering
  for equal timestamps. Do not rank posts or build a precomputed timeline for
  the MVP.
- Allow any authenticated regular user to view an individual post. Keep the
  normal feed limited to the user's own posts and posts by accounts they
  follow.
- A share is a one-click reshare without commentary of another user's post. It
  creates a new feed item that references the root original and retains the
  sharer's attribution. Reject a share if the root original belongs to the
  current user. Never copy or modify the original content.
- Validate post text and uploaded image content on the server. A post must
  contain text, an image, or both. Reject unsupported and oversized uploads.

## Approved Advertisement Behavior

- Use a clearly labeled simulated purchase flow. It must not collect payment
  card data or represent a simulation as a real charge.
- Only a confirmed simulated purchase and an active campaign can make an ad
  eligible for the feed. Draft, pending, failed, paused, or expired campaigns
  must not appear.
- A successful simulated purchase immediately activates the campaign for
  seven days. The manager can end it earlier. Expiration is determined from
  timestamps; no background scheduler is required.
- Show advertisements to all eligible regular users. Do not implement personal,
  demographic, behavioral, or location targeting.
- Insert one sponsored item after every 10 organic feed posts. Count the
  organic timeline across page boundaries; sponsored items do not increment
  the count. Use deterministic, non-personalized campaign ordering. Clearly
  label and visually distinguish each item as `Sponsored`.
- Do not show ads in an empty organic feed.
- Do not add auctions, budgets, impression/click analytics, or ad ranking.

## Security And Quality Direction

- Use Django password hashing, sessions, CSRF protection, and template output
  escaping. Do not implement custom password or session cryptography.
- Treat roles, campaign ownership, and purchase state as server-controlled.
- Keep secrets in runtime configuration, not in the repository.
- Validate uploaded image bytes and size. Use opaque generated storage keys.
- Validate ad destination URLs. Do not make server-side requests to
  user-supplied destinations.
- Test role boundaries, ownership, follow/unfollow, feed membership and order,
  sharing without source mutation, upload validation, simulated purchase
  states, and sponsored-ad placement.
- Test against SQLite for this MVP. Do not treat localhost testing as proof
  that CodeRange external access works.
- Configure `DEBUG=False`, trusted hosts/origins, HTTPS behavior, and the
  CodeRange forwarded host for deployment. Do not trust arbitrary forwarded
  headers.

## Open Decisions

Product decisions for the MVP are resolved. The following runtime items remain:

- The externally assigned CodeRange port and forwarded-host/proxy contract
  are not exposed in the current environment. This does not block
  implementation. Supply/verify them before external CodeRange access testing.
- The repository filesystem is an overlay and no durable data volume was
  found. Image persistence loss across redeployments is accepted. Confirm
  whether SQLite account/post/campaign data must survive redeployments; if it
  must, provide a durable local volume path. Otherwise document the data-loss
  limitation as well.

The first Django superuser must be created interactively by the operator.
Never commit its password. No payment, PostgreSQL, or object-storage account is
required for the approved MVP.

Username/password authentication without email verification or password-reset
email is the approved MVP scope. Adding email delivery later requires an
approved provider and credentials. SQLite's standard Python module requires
no database package or external database credentials.

## Change Control

If a later requirement conflicts with this record, update this document with
the new human-approved decision and its reasoning before implementation. Do
not treat unlisted features or external services as implicitly approved.
