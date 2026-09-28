# An Approved Plan Is Not Running Software

The study, canonical decisions, and implementation plan define an approved
target. They do not provide application behavior.

The current repository has no application source, dependency manifest,
`manage.py`, database migrations, test suite, or CodeRange startup settings.
Therefore:

- Do not tell users that accounts, posts, feeds, shares, campaigns, or
  simulated purchases work.
- Do not run or document Django startup or migration commands as available
  commands until the required files exist.
- Do not claim that tests pass. There is no project test suite to run.
- Do not claim that CodeRange can serve the application. There is no process
  that binds to `0.0.0.0:$PORT`.
- Do not infer that SQLite or local media files are durable across CodeRange
  redeployments. Image loss across redeployments is an approved MVP
  limitation; SQLite data-file persistence still needs an operational
  decision.
- Do not treat a local port default such as 8000 as CodeRange's assigned
  external port. The port and forwarded-host contract were not exposed in the
  inspected environment.

Keep this warning accurate until an implementation is present and verified.
Then replace it with specific non-obvious behavior and operational risks in
the implemented system.
