# Task 22 — Deterministic CI + schema guard (outcome brief)

You are executing exactly ONE task. Read AGENT_DIRECTIONS.md first — hard rules and
the platform boundary always win. Read the code, choose the implementation yourself.
This brief defines OUTCOMES and PROOF. Techniques mentioned are hints, not orders.

## The goal in one line
A red test must always mean something is broken, and a worker must never run against
a database whose shape it doesn't recognize.

## Outcome 1 — CI never talks to the live gateway
Today the test suite calls the real LLM, so results vary run to run
(test_12_kyc_tax_forms flakes for exactly this reason).

**Done means:** the merge-blocking test suite runs fully offline against recorded or
fixture responses — same input, same output, every single time. Live-gateway runs
still exist, but only as the on-demand benchmark mode (Task 2's third mode); they
never block a merge. The flaky test becomes deterministic.
(Hint, optional: a record-once/replay layer at the gateway client boundary.)

**Fence:** fixture documents and recorded responses must be SYNTHETIC — no real
client names, TINs, or values may be committed anywhere. If existing test docs are
real client data, stop and ask.

**Proof:** the full CI suite passes twice in a row with network access disabled,
producing identical results both times; the previously flaky test passes 10/10 runs.

## Outcome 2 — Versioned migrations + a schema guard at startup
The corporate Postgres jobs table is missing case_label — schema changes are not
traveling with the code.

**Done means:** schema changes live as ordered, versioned migrations that bring ANY
environment (fresh or existing, SQLite or Postgres) to the current version without
losing data. And the startup checks from Task 20 gain one more gate: on boot, the
worker compares the database schema version to what the code expects — on mismatch
it refuses to accept jobs and says exactly which migration is missing.

**Fence:** never auto-migrate a production database silently at startup — the guard
REPORTS and refuses; applying migrations is an explicit, human-invoked command.

**Proof:** a fresh SQLite DB and a fresh Postgres DB migrate to the same version and
pass the suite; a deliberately stale DB makes the worker refuse jobs with a clear,
named error; migrating an existing DB with data preserves every row.

## Out of scope (human decisions, not yours)
- The repo policy that gitignores tests/ — flag it in NOTES.md, do not change it.
- Running anything against shared/corporate infrastructure — stop and ask first.

## Working loop
Read brief → read code → implement one outcome at a time → write and run its proof
as an automated test → fix until green → run the full regression suite → commit on
branch task-22 → summarize what you chose and why.
