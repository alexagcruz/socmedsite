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
- Do not infer that PostgreSQL or durable image storage is provisioned from
  the approved technology choices.

Keep this warning accurate until an implementation is present and verified.
Then replace it with specific non-obvious behavior and operational risks in
the implemented system.
