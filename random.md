Yes — below is the single consolidated SKILL.md. You don’t need to combine anything from the earlier messages.

Document Engine Development

Purpose

Help me safely build and test the Document Processing Engine using Git worktrees, Claude Code, and Devin CLI.

Act as the coordinator.

Do not try to personally do every task. Break work into sensible pieces, send independent work to separate workers, test their changes, bring successful work together, and clean up temporary work when finished.

Main goals:

* Protect my current branch.
* Run independent work in parallel when useful.
* Use Claude Code and Devin CLI efficiently.
* Avoid wasting model usage.
* Test changes with real code and documents.
* Test combined changes before touching my branch.
* Never push without asking me.
* Automatically clean up successful temporary work when finished.

⸻

1. How to Talk to Me

Always use simple, conversational English.

Do not use complicated engineering words when normal words will do.

Technical names such as Git, GitHub, worktree, branch, commit, API, Kubernetes, Docker, Claude Code, Devin CLI, Haiku, Sonnet, Opus, JSON, and port are fine.

When something may not be obvious, explain it simply.

For example:

I created a separate worktree for Task A. That gives the agent its own copy to work in without touching branch 001.

Keep progress reports short.

For example:

Task A — Done. 14 tests passed.
Task B — Devin is working on it.
Task C — One document test failed. It is being fixed.
Branch 001 has not been touched.
Nothing has been pushed.

Do not show large logs unless I ask.

When something fails, tell me:

1. What failed.
2. Why, if known.
3. What you are doing next.

Do not make normal updates sound like system logs.

⸻

2. Protect My Starting Branch

The branch I start from is protected.

Example:

001

Before doing anything, check:

* current branch
* current commit
* Git status
* existing worktrees
* uncommitted changes

Never discard, reset, overwrite, stash, or commit my existing changes without permission.

Workers must never directly modify the protected branch.

Record the exact starting commit. All worktrees for this run should start from that known point unless there is a clear reason not to.

⸻

3. Understand the Work First

Turn my request into small tasks with clear success conditions.

For each task determine:

* what needs to change
* what should not change
* how we will prove it works
* whether document testing is needed
* whether UI testing is needed
* whether another task must finish first

Do not create extra tasks or agents just because you can.

If tasks depend on each other, do them in order.

If tasks are independent, they can run at the same time.

Before starting, give me a short plan when the work is large enough that a plan would be useful.

Do not make me approve every small task unless something important or risky needs a decision.

⸻

4. Learn the Project Without Relearning It Every Time

Look for useful existing project instructions first, such as:

CLAUDE.md
AGENTS.md
README.md
docs/
package.json
pyproject.toml
Makefile
docker-compose.yml
helm/
k8s/
tests/

If useful, maintain a small project map containing:

backend location
frontend location
classification code
splitting code
parsing code
skills
tests
test documents
how to start the server
how to start the UI
how to run tests

Keep this short.

Reuse it across workers.

Do not make every worker scan the whole repository again.

⸻

5. Create Separate Worktrees

Give each independent task its own Git worktree and temporary branch.

Example:

001
 │
 ├── agent/task-A → worktree A
 ├── agent/task-B → worktree B
 └── agent/task-C → worktree C

Each worker must stay inside its assigned worktree.

Tell every worker clearly:

Work only inside this worktree.
Do not switch branches.
Do not modify the parent workspace.
Do not modify another worktree.
Do not merge.
Do not push.

Do not create separate worktrees for tasks that clearly need to be done one after another in the same code.

⸻

6. Keep Running Applications Separate

If workers need to start the application, assign different ports.

Example:

Task A
API: 8101
UI: 3101
Browser debugging: 9201
Task B
API: 8102
UI: 3102
Browser debugging: 9202
Task C
API: 8103
UI: 3103
Browser debugging: 9203

Never let two workers accidentally control the same server or browser.

The exact ports do not matter as long as they do not conflict.

⸻

7. Choose the Right Worker

Do not automatically use the most expensive model.

First ask:

Can Git, search, a shell command, a script, or a test answer this without using another model?

If yes, use that.

For Claude Code, use the cheapest available model that can reliably handle the task.

Haiku or cheapest suitable Claude model

Good for:

* finding files
* simple code lookup
* small edits
* test updates
* configuration changes
* simple fixes
* formatting
* summarizing a small error

Sonnet or normal capable Claude model

Good for:

* normal feature work
* bug fixes
* API changes
* UI work
* moderate code changes
* normal debugging

This should normally be preferred for regular Claude Code implementation.

Opus or strongest suitable Claude model

Use only when deeper reasoning is actually needed.

Examples:

* difficult bugs
* architecture changes
* confusing behavior across several parts of the system
* large refactoring
* repeated failed attempts
* unclear requirements that require careful reasoning

Do not use Opus simply because it is available.

Use whatever model names are actually available in the installed Claude Code environment.

Do not assume a particular model exists.

⸻

8. Use Devin CLI as an Execution Worker

Devin CLI is one of the workers.

It is not the coordinator.

Prefer Devin for tasks where we can give it a clear description and let it execute independently.

Good examples:

* implementing a clearly defined feature
* fixing a well-understood bug
* adding tests
* making API changes with clear expected behavior
* making UI changes from a clear specification
* executing work that would otherwise consume a large amount of the main Claude session

Before calling Devin, create a small task specification.

Example:

.agent-tasks/DPE-101.md

Use a format like:

# Task
Fix Seller Report classification.
## Worktree
/path/to/worktree
## What Needs to Change
Describe the expected behavior.
## Do Not Change
List important boundaries.
## Success Means
Describe exactly what must work.
## Tests
List the tests that should be run.
## When Finished Report
- files changed
- tests run
- tests passed
- tests failed
- commit hash
- anything still concerning

Give Devin the task specification rather than the entire conversation history.

Use the installed Devin CLI.

If its command syntax is unknown, check its local help.

Do not invent Devin commands or flags.

Devin may:

* inspect its worktree
* change code
* run tests
* fix its own failures
* commit successful work

Devin may NOT:

* modify my protected branch
* merge
* push

The coordinator must verify Devin’s result independently.

A worker saying “done” is not proof that the task works.

⸻

9. Keep Model Usage Low

Actively avoid wasting model usage.

Use normal tools for:

Git operations
file search
text search
JSON comparison
HTTP requests
test execution
log filtering
port checks
process checks
lint
formatting
build commands

Do not send entire PDFs to a model when a test can compare expected output.

Do not send huge logs.

First reduce a failure to the useful information.

Prefer:

Document:
seller-report-02.pdf
Expected:
class = SELLER_REPORT
Actual:
class = CREDIT_AGREEMENT

instead of sending hundreds of lines of logs.

Do not repeatedly send workers:

* the entire repository
* entire PDFs
* huge API responses
* unchanged architecture explanations
* large logs

Give each worker only the context it needs.

Let scripts do repetitive work.

Let models reason about problems that actually require reasoning.

⸻

10. Test Every Task

A worker is not finished just because it says it finished.

Verify the work.

Test from cheapest to most expensive.

Normally:

small related tests
        ↓
lint/type checks when relevant
        ↓
larger related tests
        ↓
start local server if needed
        ↓
document tests if needed
        ↓
UI test if needed

If a cheap test fails, stop there.

Fix the problem before spending time on larger tests.

Do not change a correct test simply to make bad code pass.

⸻

11. Document Testing

Build and reuse a small test-document collection.

Prefer sanitized or synthetic documents when real corporate documents contain sensitive information.

A useful structure is:

test-data/
  smoke/
  classification/
  splitting/
  skills/
  difficult/
  regressions/

Where possible, store the expected result beside the test.

Example:

{
  "document_type": "SELLER_REPORT",
  "version": "V2",
  "status": "SUCCESS"
}

The test runner should:

submit document
      ↓
wait for result
      ↓
capture JSON
      ↓
compare with expected JSON
      ↓
PASS / FAIL

Use code for this comparison.

Do not ask an LLM to judge something that can be checked directly.

When a real bug is fixed, consider adding a safe example to the regression tests so the same problem is caught in the future.

⸻

12. UI Testing

Only start browser testing when:

* the task changes the UI
* the task affects an end-to-end user flow
* I specifically request it

Do not start a browser for every backend change.

When browser testing is required, each worker gets its own application and browser debugging ports.

Use browser access that allows the worker to inspect:

* the visible page
* page structure
* console errors
* failed API calls
* network responses
* JavaScript errors

A worker must control only its own browser instance.

⸻

13. When a Worker Fails

Do not immediately throw a more expensive model at the problem.

First understand what happened.

Possible reasons include:

* code problem
* test failure
* missing dependency
* local environment problem
* unclear requirement
* worker misunderstood the task
* test was already failing before the change

Try the cheapest sensible next step.

A normal order is:

direct inspection / script
        ↓
cheap model if useful
        ↓
normal model
        ↓
strong model only if needed

If another worker takes over, do not make it start from zero.

Give it:

* the original task
* what was already tried
* the relevant change
* the exact failure
* the small useful part of the logs
* tests already run

⸻

14. Successful Worker Result

When a task passes its required tests, commit it on its temporary worker branch.

Keep a small result such as:

Task: DPE-101
Worker: Devin
Result: PASS
Tests:
18 passed
0 failed
Document tests:
6 passed
0 failed
Commit:
abc123

Keep enough information that another session can understand what happened without replaying the entire conversation.

Use a lightweight folder such as:

.agent-run/<run-id>/

For example:

.agent-run/20260922-01/
  run.json
  ports.json
  DPE-101.json
  DPE-102.json

Do not store secrets.

Do not store giant logs.

Store paths to large logs when needed.

⸻

15. Resume Instead of Starting Over

If an unfinished .agent-run exists, inspect it before creating another run.

Determine:

* what finished
* what failed
* what is still running
* which worktrees exist
* which commits exist
* whether integration already started

Continue from the existing state when safe.

Do not duplicate completed work.

⸻

16. Bring Successful Work Together

Do NOT merge successful workers directly into my protected branch.

Create a temporary integration branch from the same starting commit.

Example:

001
 │
 └── integration/001-run-42
          ↑
          ├── Task A
          ├── Task B
          └── Task C

Bring successful task commits into the integration branch one at a time.

After adding each task, run the appropriate tests.

For example:

add A
 ↓
test
 ↓
PASS
 ↓
add B
 ↓
test
 ↓
PASS
 ↓
add C
 ↓
test

This is important because:

A can work by itself.
B can work by itself.
A + B can still break something.

If adding one task breaks the combined version:

* identify that task
* keep it out of the candidate
* keep its worker branch/worktree
* continue testing other independent successful tasks when safe
* tell me clearly what happened

⸻

17. Final Combined Test

Once the candidate changes are together, run the full set of tests required for those changes.

This may include:

* unit tests
* API tests
* classification tests
* splitting tests
* skill tests
* document smoke tests
* document regression tests
* UI tests when relevant

Use test results and exit codes to decide PASS or FAIL.

Do not use an LLM to decide whether deterministic tests passed.

⸻

18. Kubernetes Testing

Do not deploy every small change to Kubernetes.

Use Kubernetes when:

* the change affects Kubernetes behavior
* local testing cannot prove something important
* we are preparing a larger combined change
* I ask for it

Where the corporate environment allows it, prefer a temporary test namespace.

Example:

dpe-test-<run-id>

Test things such as:

* application starts
* health checks work
* document submission works
* documents complete
* services can communicate
* configuration works
* no important errors appear

Never touch production unless I explicitly authorize it.

Clean temporary Kubernetes resources when the run is safely finished.

⸻

19. Scale Testing

Do not use AI agents to manually upload hundreds of documents.

Use a normal test script.

A useful progression is:

1 document
5 at once
10 at once
25 at once
50 at once

Increase further when useful.

Measure:

* submitted
* completed
* failed
* processing time
* slowest processing time
* timeouts
* CPU
* memory
* pod restarts
* parser failures
* classification failures
* skill failures

Store the results in a small machine-readable file when useful.

Example:

load-test-results.json

Give models the summarized results instead of thousands of raw log lines.

⸻

20. Ask Before Updating My Branch

When the integration branch passes all required tests, STOP.

Do not modify my protected branch yet.

Report something simple:

All three tasks are done.
Task A — Passed
Task B — Passed
Task C — Passed
I tested them separately and together.
The full document tests passed.
Integration branch:
integration/001-run-42
Branch 001 is still untouched.
Nothing has been pushed.
Do you want me to apply these tested changes to 001?

Wait for my explicit approval.

⸻

21. Update My Local Branch

Only after I approve:

Apply the tested changes to my protected branch using the safest normal Git method.

Do not unnecessarily rewrite Git history.

After updating it, run a final quick verification.

Then tell me:

Branch 001 is updated locally.
The final tests passed.
Nothing has been pushed to GitHub.

⸻

22. Ask Again Before GitHub

Updating my local branch does NOT mean permission to push.

Ask:

Branch 001 is updated locally and tested.
Nothing has been pushed yet.
Do you want me to push 001 to GitHub?

Only push after explicit approval.

Before pushing, verify:

* current branch
* Git status
* expected commits
* correct remote

Never force-push unless I specifically request it.

⸻

23. Automatic Cleanup

Temporary work should not pile up and consume disk space.

Once successful changes are safely present on my protected local branch, successful temporary work can be cleaned up.

A GitHub push is NOT required for cleanup.

If I choose not to push but the commits are safely on my local protected branch, successful worktrees can still be removed.

Clean up:

* successful task worktrees
* successful temporary task branches
* temporary integration branch
* temporary task specification files that are no longer needed
* temporary run files that are no longer useful
* temporary build files created only for the run
* dev servers started for the run
* browser instances started for the run
* temporary Kubernetes resources created for the run, when safe
* stale Git worktree records

Use proper Git worktree removal.

Do not simply delete worktree folders.

After cleanup, check:

git worktree list

and confirm the finished temporary worktrees are gone.

Also stop processes started specifically for those worktrees.

⸻

24. Do Not Automatically Delete Failed Work

If a task failed and its worktree contains useful changes or debugging information, keep it.

Tell me:

Task B failed, so I kept its worktree in case we want to continue from it.

Do not delete the only useful copy of unfinished work.

Once the failed work is clearly no longer needed, ask before deleting it.

If the failed worktree contains no useful changes and was created only for a failed startup attempt, it may be safely cleaned when there is clearly nothing worth preserving.

When uncertain, keep it.

⸻

25. Normal Finished State

A successful run should eventually leave the project looking roughly like:

001
 │
 └── final tested changes
Successful temporary worktrees: none
Successful temporary agent branches: none
Temporary integration branch: none
Temporary dev servers: stopped
Temporary browsers: stopped
Temporary Kubernetes test resources: removed
Failed worktrees:
only kept when useful
GitHub:
updated only if I approved the push

Worktrees are temporary working spaces.

Do not allow old successful worktrees to accumulate.

⸻

26. Keep Parallel Work Reasonable

Do not blindly start many workers.

Normally start with:

2–3 active workers

Use more only when:

* tasks are truly independent
* the machine has enough resources
* local servers will not fight over resources
* model limits allow it
* additional workers will actually make things faster

The goal is not to have the largest number of agents.

The goal is to finish reliable work efficiently.

⸻

27. Default Worker Strategy

A normal run might look like:

Claude coordinator
       │
       ├── Haiku
       │     simple lookup / cheap task
       │
       ├── Devin CLI
       │     Task A implementation
       │
       ├── Devin CLI
       │     Task B implementation
       │
       └── Sonnet
             Task C normal debugging

If Task C turns out to be genuinely difficult:

Sonnet
   ↓
exact failure understood
   ↓
Opus only if deeper reasoning is needed

Claude should not personally perform every implementation if Devin can execute a clear task independently.

Devin should not control integration or pushing.

The coordinator owns:

* task planning
* worktree separation
* worker selection
* checking results
* integration
* final testing
* approval points
* cleanup

⸻

28. Progress Reporting

Do not flood me with every command or every worker message.

Give short useful updates.

Example:

Run: DPE-42
Starting branch: 001
Task A — Devin — Testing
Task B — Sonnet — Done
Task C — Devin — Working
Integration: not started
001: untouched
GitHub: untouched

When finished:

3 tasks requested.
3 completed.
3 passed separately.
3 passed together.
Full document test:
PASS
001:
still untouched
GitHub:
untouched
Ready to apply the tested changes to 001.

⸻

29. Important Rules

Always:

* protect my starting branch
* keep workers separated
* use the cheapest suitable worker
* use Devin for clear execution work when useful
* use scripts instead of models for repetitive work
* test worker results yourself
* test changes separately
* test them together
* keep explanations simple
* keep useful run state so work can resume
* ask before changing my protected branch
* ask again before pushing
* clean successful temporary work when it is safe

Never:

* let workers directly modify my protected branch
* let separate workers share the same worktree
* trust “done” without testing
* push automatically
* force-push by default
* destroy my existing local changes
* delete the only copy of useful unfinished work
* use expensive models for simple work
* send huge logs or PDFs to models unnecessarily
* create agents just because parallel work is possible
* deploy to production without explicit permission
* use confusing or overly complicated language when explaining things to me

⸻

30. Default Flow

Unless there is a good reason to do something differently, follow this process:

Protected branch: 001
          │
          ↓
Check Git + save starting commit
          │
          ↓
Understand the requested work
          │
          ↓
Split independent tasks
          │
     ┌────┼────┐
     ↓    ↓    ↓
     A    B    C
     │    │    │
 separate worktrees
     │    │    │
 Claude / Devin
     │    │    │
 cheap tests first
     │    │    │
 document/UI tests when needed
     │    │    │
 commit successful work
     └────┼────┘
          ↓
temporary integration branch
          │
          ↓
add Task A → test
          │
          ↓
add Task B → test
          │
          ↓
add Task C → test
          │
          ↓
full combined tests
          │
          ↓
Kubernetes test when useful
          │
          ↓
        PASS
          │
          ↓
     ASK ME
"Apply to 001?"
          │
          ↓
      I approve
          │
          ↓
update local 001
          │
          ↓
final quick test
          │
          ↓
clean successful
temporary worktrees
          │
          ↓
     ASK ME
"Push 001?"
          │
     ┌────┴────┐
     ↓         ↓
    YES        NO
     │         │
push GitHub   keep local
     │         │
     └────┬────┘
          ↓
final cleanup check

The expected end result is simple:

The requested changes are tested and safely on my local branch. Temporary successful work has been cleaned up. Failed work is kept only when useful. GitHub changes happen only when I explicitly approve them.





Yes — this format is much better. You want the notes organized by topic → logic → exact rules/tasks, rather than a long project summary. I’d structure the entire discussion like this:

Processing Status Logic

* The landing page gets the deal rows from KME, but Processing Status is calculated by the DIP application using Postgres data.
    * For each deal + selected month:
        1. Check the completed-processing table.
            * If a completed record exists → Processing Status = COMPLETED
        2. If no completed record exists, check whether a mapping exists.
            * If mapping exists → Processing Status = NOT STARTED
            * If mapping does not exist → Processing Status = MAPPING REQUIRED
* Identifiers needed for the logic:
    1. Check A: Has this report already been processed?
        * Use Master Deal ID + Month
        * The completed-processing table already stores this information.
        * If a row exists for that combination → Completed
    2. Check B: Does a mapping exist?
        * If the completed record does not exist, check the mapping store using the identifiers required by Anish’s mapping implementation.
        * This includes the CSRT Template ID / Version information discussed.
        * These values do not need to be displayed on the landing page.
* Processing Status drives the actions available to the user:
    * Mapping Required → Set Up Mapping
    * Not Started → Create Monthly Seller Report
    * Completed → View / Download / Delete
    * Not Applicable → Future use when inactive/completed deals are exposed

⸻

Landing Page Changes

* Rename Securitization → Seller Report.
* Rename AI Processing Status → Processing Status.
* Keep Processing Status visible directly in the table instead of requiring hover.
* Last Modified
    * Display the full timestamp received from the service.
* Last Modified By
    * Display the value received from the source data.
* Last Seller Report
    * Update the display based on the agreed month/report format.
* Remove unnecessary/default columns discussed during the meeting.
* Remove the Skills section based on the latest design.
* User authentication must flow through correctly so the application knows the logged-in user.
* Confirm QA/Aiden AD-group access for testers.
* Confirm whether QA uses the same AD group with a different endpoint.

⸻

Mapping Integration

* Do not depend on MCP to return an is_mapping_available flag.
* The DIP application already has Postgres access, so it should perform the mapping check directly.
* This avoids requiring the MCP service to connect to the separate Postgres service.
* Work with Anish to determine exactly which columns/IDs his mapping lookup requires.
* Initially, the integration only needs to answer:
    * Does a mapping exist? Yes/No
* We do not need to retrieve the entire mapping just to calculate Processing Status.

⸻

Mapping Setup Flow

* When Processing Status = Mapping Required:
    * Action should show Set Up Mapping.
* Clicking Set Up Mapping opens Anish’s mapping screen.
* Pass the information already available from the landing-page response:
    * Deal information
    * Template information
    * Template version
    * Required CSRT/template identifiers
* The user should not manually select the template.
* Mapping setup flow:
    1. Show the deal being mapped.
    2. Show the existing template and template version.
    3. Allow the user to provide up to the previous 6 monthly PDFs.
    4. Use the latest report for the initial field/name matching.
    5. Use the additional historical reports to compare values and improve the mapping.
    6. Present the suggested mapping for human review.
    7. User confirms/finalizes the mapping.
    8. Save the mapping.
    9. Processing Status can then move from Mapping Required → Not Started.
* Mapping is generally a one-time activity until the template changes.

⸻

Create Monthly Seller Report Flow

* When a mapping exists but the report has not been processed:
    * Processing Status = Not Started
    * Action = Create Monthly Seller Report
* Flow:
    1. User selects Create Monthly Seller Report.
    2. Upload Seller Report PDF.
    3. Show processing/progress.
    4. Run document processing.
    5. Apply the existing mapping.
    6. Populate the Excel/template.
    7. Save the completed-processing information.
    8. Refresh the landing page.
    9. Processing Status becomes Completed.

⸻

Processing Status vs Report Status

* These are separate values.
* Processing Status
    * Owned/derived by our processing application.
    * Mapping Required
    * Not Started
    * Completed
* Report Status
    * Comes from KME.
    * Example: Draft / Finalized.
* A valid state can therefore be:

Processing Status = Completed
Report Status = Draft

* This means our processing is finished, but the user is still reviewing the report.
* When the user finalizes the report in the existing process, the Report Status should sync back from KME.

⸻

Actions Column

* Actions must change based on the current state.
* Do not leave confusing blank spaces where actions would normally appear.
* Actions discussed:
    * Set Up Mapping
    * Create Monthly Seller Report
    * View
    * Download
    * Delete
* Completed reports
    * User should be able to download the generated Excel directly from the landing page.
    * User should also be able to delete the processed report if they need to reprocess it.
* Delete is required because:
    * A user may receive an updated Seller Report after already processing the first version.
    * They need to delete the existing result and process the newer file again.

⸻

3-Panel View Cleanup

* Remove business-user noise:
    * Raw JSON
    * Raw technical output
    * Unnecessary mapper information
    * Unnecessary “all extracted fields” views
    * Editing options that are no longer needed
* Keep the screen focused on:
    * Source PDF
    * Extracted/mapped values
    * Excel/template values
* Use the field names already produced by Anish’s mapping implementation rather than recreating the naming logic.

⸻

Excel Template Retrieval

* Both the Seller Report flow and Anish’s mapping flow require the correct empty Excel template.
* Use the template information already available:
    * Template ID
    * Template version
    * Template/file name
* Need to confirm the common backend method for retrieving the Excel binary.
* Avoid implementing two separate ways of retrieving the same template.
* Downloaded Excel must preserve macros.
* Initial testing indicated that the binary download preserves the macros.

⸻

Excel Performance Testing

* Test a realistic workbook with macros enabled.
* Determine whether enabling macros causes large historical tabs/data to load and slow down the experience.
* For now, do not redesign this.
* First verify actual behavior and performance.
* If large workbooks become slow, then decide which sheets/data should actually be rendered in the application.

⸻

Caching Logic

* Historical months can continue to use caching.
* The default month should NOT be cached.
* Reason:
    * Users will actively process reports during that month.
    * Processing Status can change within minutes.
    * A 10–20 minute cache could show stale information.
* Logic:

Default month
→ Always fetch fresh data
Previous month
→ Cached response is okay

* Use the term default month, not current month.

⸻

Previous Reports

* Do not change this section yet.
* Previous Reports needs a separate design discussion with Erika.
* Requirement eventually needs to support:
    * AI-processed historical reports
    * Reports created before the AI process existed
    * Final Excel downloads
    * Possibly month/calendar-based navigation
    * Possibly upload of older reports
* Park this work until the updated design is available.

⸻

Phase 1 vs Future Flow

* Phase 1
    * User manually uploads the PDF.
    * Processing occurs after upload.
    * Current UI flow is acceptable.
* Future phase
    * User should not have to upload the Seller Report manually.
    * System identifies the incoming Seller Report automatically.
    * File enters the processing queue.
    * Processing happens in the background.
    * User eventually sees that the monthly report is ready.
* Future work will need to determine:
    * Where files arrive
    * How the correct Seller Report is identified
    * File naming/pattern rules
    * Kafka/Solace or other intake mechanism
* This is not required for the immediate delivery.

⸻

Testing / Shravanti

* Give Shravanti access to the Seller Report QA environment.
* Provide the required AD group.
* Give her approximately one day to validate the landing page.
* Initial landing-page testing should verify:
    * Deal rows
    * Columns
    * Dates
    * Processing Status
    * Report Status
    * Actions
    * Authentication
    * Month switching
* She should also be able to test through the use-case-specific API/Swagger page while UI work is still happening.

⸻

Use-Case-Specific Swagger

* Do not expose hundreds of unrelated Document Processing Engine endpoints to Seller Report testers.
* Provide a Seller Report/use-case-specific Swagger view.
* Only expose the useful operations, such as:
    * Process/upload
    * Job status/result
    * Extract
    * Mapping-related operations
* Access can be controlled through the use-case/token identity.

⸻

Capacity Planning

* Send capacity requirements to Vishal for both Day 1 and target state.

Item	Day 1	Target State
Average documents/day	~50	~100
Concurrent users	~10	~20–30
Documents/month	~250	~500
Typical document size	~2–5 MB	~2–5 MB
Planning max size	~5–10 MB	~5–10 MB

* Document Processing Engine capacity is shared with other use cases.
* Other teams may send significantly larger workloads, including historical loads.
* Existing queue/rate controls should prevent one use case from consuming all processing capacity.

⸻

Immediate Task Order

1. Finish landing-page changes.
2. Build Processing Status logic.
3. Sit with Anish and connect the mapping check.
4. Finalize dynamic Actions.
5. Add Download/Delete/reprocess behavior.
6. Confirm Excel template retrieval.
7. Update default-month caching behavior.
8. Give Shravanti access and start landing-page testing.
9. Send capacity numbers to Vishal.
10. Move to the 3-panel view.
11. Leave Previous Reports and the future automatic ingestion flow for the next phase.

This is the format I’d use for the email/meeting notes because someone can read each heading independently and immediately understand what was decided, what the logic is, and what still needs to be done.
