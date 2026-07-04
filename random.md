The goal in one line

Make one worker fast and the queue safe for many workers — without changing what
gets booked or withheld, on either backend (SQLite locally, Postgres on Kubernetes).

Outcome 1 — Parallel LLM and embedding calls on the live path

Today extraction and embedding calls leave one at a time, and the concurrency and
rate-limit settings in config exist but control nothing on the real path.

Done means: the live pipeline makes multiple LLM calls in flight at once, bounded
by the existing config settings; embedding requests are batched rather than sent one
string at a time; gateway errors and rate responses back off gracefully.
(Hint, optional: bounded async dispatch. Your call how.)

Proof: a 500-doc mixed batch shows at least 4x throughput vs the current baseline, (but we can actually write a test file and hit the LLM gateway and figure out what could be the max. )
AND the eval set produces byte-identical booked/withheld outcomes before and after.
Concurrency is transport, never semantics — if outcomes differ at all, the change is wrong.

Outcome 2 — A queue that two workers can share safely

Today claiming a job is two separate steps, so two workers can grab the same job.

Done means: claiming a job is atomic on BOTH backends through one interface, and
the database itself refuses to let one job have two active processing runs — safety
must not depend on worker politeness.
(Hint, optional: a single-statement claim-and-return works on both engines; Postgres
has a stronger variant. Pick what fits the storage layer you find.)

Proof: two workers against one queue chew through a 1,000-job batch with zero
double-processed jobs, verified from the audit chain, on both backends.

Outcome 3 — Backfills that survive death with visible progress

A checkpoint function exists but is wired to a path nothing uses.

Done means: a large backfill records its progress as it goes on the real path;
kill it mid-run and it resumes where it left off, and a human can see how far along
it is at any moment.

Proof: kill a 10,000-doc test backfill partway; on restart it resumes from the
cursor, reprocesses nothing already completed, and progress reporting is correct.

Outcome 4 — A cost meter that can answer the December question

Today we can't split cost by stage, embeddings aren't metered, and parse has no
dollar figure.

Done means: per-job cost records distinguish parse vs extraction vs re-extraction
vs embedding, embeddings are metered, and a per-doc-type report can project cost and
duration for the 2.2M backfill from real numbers.

Proof: the cost report for a test batch shows the per-stage breakdown, including
embeddings, and the projection query runs from recorded data alone.

Fences (do not cross)


No new infrastructure, no extra workers, no new services in this task.
Verification gates, their order, and withhold behavior are untouchable.
Every change must work on SQLite and Postgres through the existing backend switch.
If you need something you don't have (gateway rate limits, real sample volume),
STOP and ask — never invent numbers.


Working loop

Read brief → read code → implement one outcome at a time in the order above →
write and run its proof as an automated test → fix until green → run the full
regression suite → commit on branch task-21 → summarize what you chose and why.
