# Approved MVP Decisions

- Status: Approved baseline
- Approval recorded: 2026-09-28
- Source study: `doc/study/1790599824_social_media_mvp_feasibility.md`

This document records the human-approved product scope, architecture, and
technical direction for the social-media MVP. Treat these decisions as
authoritative. Do not infer approval for the remaining items in
[Open Decisions](#open-decisions).

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
| Application | Python with a currently supported Django LTS release | Django provides authentication, sessions, ORM, migrations, forms, CSRF protection, and request testing in one established framework. Verify CodeRange's supported Python version before selecting an exact Django release. |
| Architecture | One Django monolith and one relational database | The required flows do not justify microservices, a separate API, a task queue, or a distributed system. Keep important state transitions in small, testable server-side services. |
| Frontend | Django templates, semantic HTML, responsive CSS, and minimal progressive JavaScript | Server-rendered pages fit the feed and form flows and avoid a separate frontend build and API authentication design. A React/Vue SPA is not part of the MVP. |
| Database | PostgreSQL | Relational constraints and transactions fit account relationships, posts, campaigns, and purchase state. CodeRange must provide or be configured with PostgreSQL. If PostgreSQL is unavailable, stop and request a decision; do not silently switch the deployed database to SQLite. |
| Image handling | Django's upload and storage interfaces; validate image contents and use generated storage names | This supports secure image uploads while keeping the application independent of one storage vendor. |
| Image persistence | Use a CodeRange persistent media volume if it is confirmed durable across restarts and deployments. Otherwise use an approved S3-compatible object store. | Uploaded images must not be stored on ephemeral deployment disk. External object storage requires an account, bucket/endpoint, credentials, and network access. |
| Tests | Django's built-in test runner and test client; temporary or fake media storage | These tools cover permissions, forms, feed behavior, and business rules without adding a separate test framework. Add browser automation only if a concrete browser behavior requires it. |
| Runtime | Gunicorn with Django WSGI for the synchronous application | This is a simple production runtime for request-and-response pages. Bind to `0.0.0.0` and the port supplied by CodeRange, normally through `PORT`. ASGI is deferred unless a real asynchronous or long-lived connection requirement appears. |
| Dependencies | Declare Python dependencies in `pyproject.toml` and commit a lock file if CodeRange supports that install workflow | Repeatable dependency versions support fresh-clone builds. If the CodeRange build system does not support the lock workflow, use a documented pinned pip-compatible requirements file as the canonical install mechanism. |

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
- A share is a one-click reshare without commentary. It creates a new feed
  item that references the root original and retains the sharer's attribution.
  It must not copy or modify the original content.
- Validate post text and uploaded image content on the server. A post must
  contain text, an image, or both. Reject unsupported and oversized uploads.

## Approved Advertisement Behavior

- Use a clearly labeled simulated purchase flow. It must not collect payment
  card data or represent a simulation as a real charge.
- Only a confirmed simulated purchase and an active campaign can make an ad
  eligible for the feed. Draft, pending, failed, paused, or expired campaigns
  must not appear.
- Show advertisements to all eligible regular users. Do not implement personal,
  demographic, behavioral, or location targeting.
- Insert a sponsored item at a fixed interval among organic feed items. Use a
  simple non-personalized rotation. Clearly label and visually distinguish
  each item as `Sponsored`.
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
- Test against PostgreSQL when it is available. Do not treat localhost testing
  as proof that CodeRange external access works.
- Configure `DEBUG=False`, trusted hosts/origins, HTTPS behavior, and the
  CodeRange forwarded host for deployment. Do not trust arbitrary forwarded
  headers.

## Open Decisions

These items remain unresolved and must be answered or verified before the
implementation plan fixes dependent behavior:

- Confirm CodeRange's supported Python version, deployment command, forwarded
  host, and required port. The application must bind to `0.0.0.0:$PORT` (or
  CodeRange's documented equivalent).
- Confirm CodeRange PostgreSQL availability and how its connection settings
  are supplied. If unavailable, request approval for an alternative before
  implementation.
- Confirm that CodeRange provides durable media storage. If not, an owner must
  provide an S3-compatible storage service and credentials before image
  deployment.
- Select the exact fixed ad interval. The approved policy is a fixed interval,
  but no numeric frequency was supplied or approved.
- Define what the simulated campaign purchase represents, including any demo
  price, currency display, duration, activation timing, expiration, and pause
  behavior. Do not invent a real price or financial claim.
- Decide how campaign-manager accounts are provisioned (for example,
  administrator-created or invitation-based). Public self-selection remains
  prohibited.
- Decide whether a user can reshare their own post. Shares of shares should
  point to the root original, as approved.
- Confirm post visibility outside the feed. The approved feed includes the
  author's and followed accounts' posts, but broader profile/detail-page
  privacy behavior was not specified.
- Confirm the dependency install mechanism supported by CodeRange.

Username/password authentication without email verification or password-reset
email is the approved MVP scope. Adding email delivery later requires an
approved provider and credentials.

## Change Control

If a later requirement conflicts with this record, update this document with
the new human-approved decision and its reasoning before implementation. Do
not treat unlisted features or external services as implicitly approved.
