# Seller Report — QA Defects and Engineering Review

## Objective

Review the issues identified during the Seller Report QA walkthrough. For each issue:

1. Trace the current implementation across the frontend, backend, database, and processing workflow.
2. Determine whether the issue is reproducible, already fixed, caused by outdated data, or expected behavior.
3. Identify the root cause before making changes.
4. Check whether the issue affects other deals or report types.
5. Propose the smallest safe fix and add appropriate tests.
6. Do not modify shared framework functionality unless the change is necessary and its impact is understood.

**Important:** The QA discussion involved multiple deals, including Dell and GMF. Several problems appear related to job isolation, mapping state, processing results, and Excel output. Investigate whether these issues share an underlying cause rather than treating every symptom as a separate defect.

---

# Section A — Processing and Job Management

## DEF-01: Processing delays and jobs appearing stuck

**QA observation**

During the walkthrough, a monthly Seller Report was uploaded for processing. The same operation had completed relatively quickly earlier in the morning, but during the QA session it took significantly longer.

The team waited while the processing screen continued to show an ongoing job. There were no obvious errors in the logs.

It was mentioned that the additional processing time introduced by the application's LLM calls might be around 30–50 seconds, depending on the gateway, while a larger portion of the overall processing time comes from LlamaParse.

The testers were concerned because they could not determine whether the job was still processing or had become stuck.

**Expected behavior**

- Jobs should progress through clearly defined states.
- The UI should distinguish between queued, processing, completed, and failed jobs.
- A slow job should not appear indefinitely stuck without useful feedback.
- Users should be able to leave the screen and return to see the current job status.
- Long-running jobs should be investigated without assuming that every delay is an application defect.

**Engineering investigation**

1. Trace the job from upload through queue submission, worker execution, LlamaParse, LLM processing, and result storage.
2. Measure the time spent in each stage.
3. Check for jobs waiting in the queue or being claimed by an unexpected worker.
4. Check polling behavior and whether the frontend stops receiving status updates.
5. Review timeout handling and failure reporting.
6. Compare fast and slow runs using the same document where possible.
7. Check whether retries or vendor-side caching contribute to delays.

**Acceptance criteria**

- Processing stages and their durations can be identified in logs.
- A failed or timed-out job reaches a meaningful terminal state.
- The UI continues to reflect the actual backend job status.
- Normal long-running processing is not incorrectly reported as failure.

**Classification:** Reported intermittent issue; root cause not confirmed.

## DEF-02: Opening GMF displays an existing Dell processing job

**QA observation**

During the call, a report was being processed for Dell. The tester then navigated to GMF to upload a separate monthly Seller Report.

Instead of presenting an independent GMF processing experience, the application appeared to return to the existing Dell processing job.

The tester noticed that the displayed information belonged to Dell, even though the intended operation was for GMF.

The team explicitly identified this as a major problem because a business user could leave one deal processing and begin working on another.

**Expected behavior**

Each deal should have an independent processing context.

For example:

- Dell has Job A, which is processing.
- The user opens GMF.
- GMF should display its own mapping and processing state.
- Uploading a GMF report should create Job B associated with GMF.
- Returning to Dell should show Job A.
- The two jobs should never overwrite or display each other's data.

**Engineering investigation**

1. Inspect how the frontend identifies the active deal and active processing job.
2. Check whether the application stores one global active job ID regardless of deal.
3. Review browser state, session state, route parameters, and cached API responses.
4. Verify that every job lookup is scoped to the correct deal and report.
5. Check whether the backend returns the most recent job globally instead of the most recent job for the requested deal.
6. Verify that changing deals clears or refreshes deal-specific frontend state.
7. Test switching between deals while jobs remain active.

**Acceptance criteria**

- Opening GMF never displays Dell's processing data.
- Each new upload is associated with the correct deal.
- Multiple jobs can exist simultaneously without sharing state.
- Refreshing or reopening the browser restores the correct deal-specific job.
- The UI never mixes results, status, mappings, or downloads between deals.

**Classification:** High-priority defect observed during QA.

## DEF-03: Multiple processing jobs are not independently accessible

**QA observation**

The discussion established that processing is asynchronous. When a user uploads a document, the application creates a job ID and places the work into an internal queue.

The processing continues even when the user leaves the screen.

However, the testers questioned what happens when someone opens another browser window or moves to another deal while the first job continues.

The observed Dell/GMF behavior suggested that the application might not properly handle multiple active jobs.

**Expected behavior**

The application should support independent jobs across different deals, even when they are started by the same user.

The user should be able to navigate between those jobs and see the correct status and results.

**Engineering investigation**

- Verify how jobs are created and associated with users, deals, and reports.
- Check whether the frontend supports multiple active job IDs.
- Confirm whether job lookup uses the deal ID and report ID.
- Test two simultaneous uploads for different deals.
- Test two browser tabs using the same authenticated session.
- Verify that completion of one job does not change another job's status.

**Acceptance criteria**

Two independent deals can be processed concurrently without data or state being mixed.

**Classification:** Related to DEF-02; investigate together before creating a separate fix.

## DEF-04: Users cannot cancel an ongoing processing job

**QA observation**

The team explained that once a document is uploaded, the backend job continues running even if the user leaves the processing screen.

There is currently no way for the user to cancel that job midway.

The concern was raised while a processing job appeared to be taking an unusually long time.

**Expected behavior**

The team needs to confirm the intended product behavior.

If cancellation is required, it must be handled by the backend rather than simply hiding the processing screen.

A cancelled job should not later publish results as though it completed successfully.

**Engineering investigation**

1. Determine whether cancellation is part of the current product requirements.
2. Review whether queued jobs can be cancelled before execution.
3. Determine whether active vendor calls can be stopped.
4. Check how a cancellation request would affect database state and temporary files.
5. Ensure cancellation cannot accidentally affect another job.

**Acceptance criteria**

Document the supported cancellation behavior. Do not introduce cancellation without an agreed product requirement.

**Classification:** Product capability gap; not a confirmed defect.

---

# Section B — Mapping and Attribute Management

## DEF-05: Mapped attributes are missing from processing results

**QA observation**

The tester explained that a deal had approximately 45–50 mapped attributes.

After uploading and processing the monthly Seller Report, the results screen did not show the complete expected set.

During the discussion, examples included:

- Approximately 50 attributes configured in the mapping.
- Only around 10 attributes appearing in one processing result.
- Another processing result showing only 3 expected fields.

The team agreed that if 45 fields were saved in the finalized mapping, the processing results should reflect those 45 expected fields.

There was also discussion that backend changes had been deployed recently, so existing results might have been generated using an older implementation.

**Expected behavior**

If the finalized mapping contains 45 applicable attributes, the processing results should account for all 45.

This does not mean every field must have an extracted value.

A field may be successfully extracted, missing, withheld, or require review, but it should not silently disappear from the expected field count.

**Engineering investigation**

1. Retrieve the finalized mapping for the affected deal.
2. Compare the attribute IDs in the mapping against those passed to the processing workflow.
3. Check whether the skills file or extraction instructions contain the expected attributes.
4. Inspect the raw extraction response.
5. Compare the raw response with the stored processing results.
6. Check whether frontend filtering incorrectly removes fields.
7. Verify whether ignored and unused fields are excluded correctly.
8. Determine whether the affected job was processed before or after the latest deployment.

**Acceptance criteria**

- Every applicable finalized attribute is represented in the expected processing results.
- Missing values are shown with an appropriate status rather than silently omitted.
- Expected field counts are consistent across mapping, processing, and review screens.
- Results from older processing versions are distinguishable where necessary.

**Classification:** High-priority defect; possible deployment or mapping-version dependency.

## DEF-06: Mapping screen unexpectedly displays hundreds of attributes

**QA observation**

During mapping setup, the tester recalled reviewing approximately 43–50 attributes for Dell.

However, when the mapping was reopened, the UI displayed counts such as 417 or 479 attributes.

The tester was confident that hundreds of attributes had not been manually reviewed.

The team also noticed that the list included attributes marked Ignore and Not Used.

There was uncertainty about whether the issue came from an older mapping, an incorrect count, or the save process.

**Expected behavior**

The UI should distinguish between:

- All attributes available in the source template.
- Attributes applicable to the deal.
- Attributes requiring user review.
- Attributes marked Ignore or Not Used.
- Attributes included in the finalized mapping.

The displayed count should accurately describe what it represents.

**Engineering investigation**

1. Inspect the raw template attributes and their status.
2. Compare the stored mapping against the attributes displayed in the UI.
3. Check whether ignored and unused attributes are included in the visible count.
4. Review the mapping save and retrieval logic.
5. Determine whether old mappings are being merged with new ones.
6. Check whether duplicate attributes are being introduced.
7. Reproduce the scenario by reviewing approximately 45 attributes, saving, and reopening the mapping.

**Acceptance criteria**

- Mapping counts accurately reflect the intended set of attributes.
- Previously ignored or unused fields do not unexpectedly reappear as active fields.
- Reopening a saved mapping does not introduce additional attributes.
- The UI clearly distinguishes total template attributes from mapped attributes.

**Classification:** Reported defect; exact reproduction still needed.

## DEF-07: Ignore and Not Used attributes appear in user-facing views

**QA observation**

While reviewing the unusually large mapping, the team identified fields that were marked as unused or ignored.

These fields were present in the source template or stored data but were not expected to appear as active fields requiring business review.

The tester specifically questioned why these attributes were visible.

**Expected behavior**

Attributes marked Ignore or Not Used should be handled according to their configured status.

They should not appear as active review items or inflate counts intended to represent applicable mapped attributes.

**Engineering investigation**

- Trace the status of each attribute from the source template through the mapping API.
- Check whether frontend filtering is missing or incorrect.
- Confirm whether ignored fields should remain stored for traceability.
- Verify that ignored attributes are excluded from extraction instructions and expected results where applicable.
- Ensure that the filtering behavior is consistent across screens.

**Acceptance criteria**

Ignored and unused attributes are excluded from the appropriate business-facing views and counts without being incorrectly deleted from the underlying template data.

**Classification:** Defect or filtering inconsistency; product display rules should be confirmed.

## DEF-08: Save Draft accepts mappings without any review

**QA observation**

The tester demonstrated that Save Draft could be clicked even when no attributes had been reviewed or selected.

The application displayed a successful save message.

The tester questioned how the system could determine what had been saved when none of the checkboxes appeared selected.

This led to a broader discussion about whether Save Draft should exist in Phase 1.

**Expected behavior**

The final mapping save action must remain disabled until all required attributes have been reviewed or explicitly ignored.

If draft functionality remains available, its rules must be defined separately. A draft does not necessarily require complete review, but the application must clearly preserve its incomplete state.

**Engineering investigation**

1. Identify the conditions controlling the Save Draft and Save Mapping buttons.
2. Verify whether the frontend and backend enforce the same rules.
3. Check what happens when an empty draft is submitted.
4. Confirm whether an incomplete draft can accidentally replace a finalized mapping.
5. Review whether the Phase 1 decision to remove Save Draft has been implemented.

**Acceptance criteria**

- Final mapping cannot be saved before required review is complete.
- Invalid requests are rejected by the backend as well as the frontend.
- The UI reflects the final product decision about Save Draft.

**Classification:** Confirmed behavior issue; final direction was to remove Save Draft for Phase 1.

## DEF-09: Draft selections and finalized mapping are inconsistent

**QA observation**

The team discussed a scenario where a user selects four attributes, saves a draft, leaves the screen, and returns later.

The expected behavior was that the four selected attributes would still be present.

However, the mapping summary could continue displaying the previously finalized mapping while Edit Mapping opened a partial draft.

The user had no clear indication that an unfinished draft existed.

The team also questioned which mapping version should be used to generate the skills file.

**Expected behavior**

A draft must never silently replace or corrupt a finalized mapping.

If draft functionality exists:

- Draft selections should persist.
- The UI should identify the draft as incomplete.
- The finalized mapping should remain unchanged until the user completes the required review.
- Final processing instructions should be generated only from the appropriate finalized mapping.

**Engineering investigation**

1. Review how draft and finalized mappings are stored.
2. Check whether they have separate version identifiers.
3. Inspect how the summary screen chooses which mapping to display.
4. Check how Edit Mapping selects the version to load.
5. Verify when the skills file is generated.
6. Ensure processing does not accidentally use an incomplete draft.
7. Check whether partial saves overwrite existing attribute decisions.

**Acceptance criteria**

Draft and finalized states are never confused, and processing always uses the intended finalized mapping.

**Classification:** Design and implementation issue; may become unnecessary if draft functionality is removed.

## DEF-10: Remove Save Draft and Edit Mapping from Phase 1

**QA observation**

The team initially considered retaining Save Draft because much of the functionality had already been implemented.

However, further testing exposed problems with incomplete selections, draft versioning, finalized mapping display, and the inability to distinguish an unfinished draft.

The final direction during the call was to remove Save Draft and Edit Mapping from the Phase 1 experience.

**Expected behavior**

- Remove or disable the two options as agreed.
- Keep the initial mapping setup workflow.
- Require complete review before saving the mapping.
- Preserve existing finalized mappings.
- Avoid deleting stored draft data unless explicitly approved.

**Engineering investigation**

Check whether these actions are still visible or accessible through routes or APIs. Confirm that removing them does not break initial mapping setup or existing finalized mappings.

**Acceptance criteria**

Phase 1 users follow one clearly defined mapping completion flow without incomplete draft behavior.

**Classification:** Agreed product change.

---

# Section C — Excel Output and Review Experience

## DEF-11: Excel does not automatically download when entering review

**QA observation**

The tester expected the Excel output to download automatically when entering the TriPN review screen.

During testing, the automatic download did not occur.

The tester was able to download the file manually.

The team suggested refreshing the browser to ensure the latest frontend build was loaded, but the underlying cause was not confirmed.

**Expected behavior**

If automatic download is part of the agreed workflow, opening the completed review screen should trigger the correct Excel download.

The manual download option should continue to work.

**Engineering investigation**

1. Locate the automatic download trigger.
2. Verify whether the download is triggered after processing completes or when the review screen opens.
3. Check whether the browser blocks the download.
4. Inspect frontend errors and download API responses.
5. Verify that the downloaded file belongs to the correct deal and job.
6. Check whether the current deployed frontend contains the functionality.
7. Test both direct navigation and navigation after processing completes.

**Acceptance criteria**

Automatic download works consistently according to the agreed trigger, without downloading an incorrect or incomplete file.

**Classification:** Reported defect; frontend version needs verification.

## DEF-12: Excel cell highlighting does not work consistently

**QA observation**

The tester hovered over extracted attributes in the review interface and expected the corresponding Excel cells to be highlighted.

In one example, a value associated with approximately cell 51 was discussed.

The expected highlight was not immediately visible.

Some other attributes appeared to highlight correctly.

**Expected behavior**

Selecting or hovering over an attribute should identify its corresponding Excel cell.

The relationship between the extracted attribute and the Excel destination must be consistent.

**Engineering investigation**

- Verify that each attribute contains the correct worksheet and cell reference.
- Check whether the frontend uses the correct cell reference.
- Confirm that highlighting works for cells outside the current viewport.
- Test attributes with empty, withheld, and populated values.
- Check whether different worksheets are handled correctly.

**Acceptance criteria**

Every mapped attribute with a valid Excel destination highlights the correct cell.

**Classification:** QA defect.

## DEF-13: Excel does not scroll to the selected cell

**QA observation**

In some cases, the Excel cell appeared to be highlighted, but it was outside the visible area.

The tester had to scroll manually to locate it.

The team acknowledged that the application was not bringing the selected cell into view.

**Expected behavior**

When the user selects an attribute, the Excel preview should automatically navigate to the corresponding cell.

The cell should be visible and highlighted without requiring manual scrolling.

**Engineering investigation**

1. Review how the Excel preview handles selected cell references.
2. Check whether the scroll operation occurs before the cell is rendered.
3. Verify handling of rows and columns outside the visible area.
4. Test worksheet switching.
5. Confirm that scrolling and highlighting remain synchronized.

**Acceptance criteria**

Selecting an attribute automatically scrolls the Excel preview to the correct cell and highlights it.

**Classification:** Confirmed UI defect.

## DEF-14: Withheld values are still written into Excel

**QA observation**

During the review, the tester noticed that some fields were marked Withheld or Needs Review in the UI.

However, the downloaded Excel file appeared to contain values for some of those fields.

The team explained that the current implementation may hide a value from the UI when it fails a validation rule without necessarily preventing the value from being written into Excel.

The tester challenged this behavior because a business user would reasonably expect a withheld value not to be treated as accepted output.

**Expected behavior**

The product must define whether a failed validation rule should prevent a value from being written into the final Excel file.

At a minimum, the UI, validation result, and Excel output must be consistent.

A value that fails validation should not silently appear as an approved final value.

**Engineering investigation**

1. Trace extracted values through validation, status assignment, and Excel generation.
2. Check whether Excel generation uses raw extracted values or validated values.
3. Identify the rules that produce Withheld and Needs Review.
4. Determine whether a failed rule should block Excel population.
5. Verify that rejected values cannot be mistaken for accepted values.
6. Check whether the behavior affects all fields or only certain rule types.

**Acceptance criteria**

Validation status and Excel output follow one agreed policy. Failed values are not silently treated as accepted values.

**Classification:** High-priority behavior inconsistency; final business rule requires confirmation.

## DEF-15: Withheld fields do not display the extracted value

**QA observation**

The team explained that when a field fails a validation rule, the UI may hide the extracted value and display a Withheld status.

For example, a rule might require a value to be above a certain threshold, but the extracted value falls outside the allowed range.

The tester cannot see the original value or understand why it was withheld.

The team acknowledged that the UI should do a better job of showing this information.

**Expected behavior**

For a field that was successfully extracted but failed validation, display:

- The extracted value.
- The validation status.
- The reason the validation failed.
- The expected condition, when appropriate.

A field that could not be extracted should be distinguished from a field that was extracted but failed a rule.

**Engineering investigation**

- Verify whether the backend retains the original extracted value.
- Check whether the API returns the validation failure reason.
- Review whether the frontend intentionally hides failed values.
- Ensure that displaying the raw value does not incorrectly mark it as accepted.
- Check how missing values and failed validation values are represented.

**Acceptance criteria**

Users can understand what value was extracted and why it failed validation.

**Classification:** Confirmed usability issue and requested improvement.

## DEF-16: Needs Review status does not clearly explain the issue

**QA observation**

The tester asked what users were expected to do when fields displayed Needs Review.

The team explained that this status could indicate different situations, including an unsuccessful extraction or a value that failed a configured rule.

However, the UI did not clearly distinguish those situations.

There was also concern that the particular job being reviewed was already returning an incorrect number of expected fields.

**Expected behavior**

The UI should explain why a field needs review.

Examples include:

- Value not found in the source document.
- Value extracted but failed validation.
- Value could not be confidently associated with the correct source.
- Processing encountered an error.

The precise statuses and permitted user actions should follow the agreed business rules.

**Engineering investigation**

Review how the backend assigns review statuses and whether the frontend receives sufficient information to explain them.

**Acceptance criteria**

Users can distinguish extraction failures from validation failures and understand what action, if any, is required.

**Classification:** Reported usability issue; also verify underlying processing results.

## DEF-17: Unexpected values appear in the Excel output

**QA observation**

The tester observed values in the Excel template that did not appear to have been extracted from the PDF.

Some cells were highlighted in yellow.

The team explained that certain values might be fixed values already present in the Excel template, rather than values extracted from the document.

The tester could not easily distinguish these cases.

**Expected behavior**

The application should preserve legitimate fixed template values while clearly distinguishing them from AI-extracted values.

Fields intentionally excluded from extraction should not be reported as extracted values.

**Engineering investigation**

1. Compare the original Excel template against the generated Excel output.
2. Identify cells containing pre-existing formulas or fixed values.
3. Check whether those cells are incorrectly included in extraction results.
4. Review the Ignore and Not Used mapping rules.
5. Confirm that processing does not overwrite fixed values unexpectedly.
6. Verify whether output statistics incorrectly count fixed values as extracted.

**Acceptance criteria**

Fixed template values remain intact, extracted values are correctly identified, and the UI does not misrepresent the source of a value.

**Classification:** Requires investigation; some observed behavior may be expected.

---

# Section D — PDF Source Highlighting

## DEF-18: Extracted values do not always highlight correctly in the PDF

**QA observation**

The team demonstrated source highlighting, where selecting an extracted attribute should identify the corresponding value in the original PDF.

The implementation uses coordinates returned by LlamaParse and maps those coordinates back onto the displayed PDF.

The team explained that the coordinates may sometimes be inaccurate, especially when:

- The document contains very small text.
- OCR does not recognize text accurately.
- The returned coordinates do not align perfectly with the displayed document.

This was described as a known limitation rather than a completely unexpected defect.

**Expected behavior**

When valid source coordinates are available, the application should highlight the correct location in the PDF.

If coordinates are unavailable or unreliable, the application should avoid displaying a misleading highlight.

Excel destination highlighting should continue to work independently.

**Engineering investigation**

1. Inspect the source coordinates returned by LlamaParse.
2. Verify page numbering and coordinate conversion.
3. Check whether the PDF viewer applies scaling correctly.
4. Test small text, tables, and scanned documents.
5. Compare raw vendor coordinates with rendered highlights.
6. Check whether errors are caused by the vendor response or frontend rendering.
7. Avoid assuming that every incorrect highlight requires a change to the extraction logic.

**Acceptance criteria**

Valid coordinates produce accurate highlights. Unreliable or missing coordinates are handled gracefully.

**Classification:** Known limitation; investigate reproducible application-side errors.

---

# Section E — Report Listing and Status

## DEF-19: AI Processing Status does not consistently reflect mapping state

**QA observation**

The team discussed a previously reported issue with the AI Processing Status column.

There was uncertainty about whether the fix had already been deployed.

The backend changes had reportedly been deployed the previous night, while frontend changes were still being completed.

The tester observed Not Started but could not confirm whether Mapping Required was functioning correctly.

The team suggested changing or removing mapping data in the database to verify the behavior because the UI did not provide a way to delete the mapping.

**Expected behavior**

The report listing should show the correct status based on the actual mapping and processing state.

For example:

- Mapping Required: No valid mapping exists.
- Not Started: Mapping exists, but the monthly report has not been processed.
- Completed: Processing has completed successfully.

**Engineering investigation**

1. Inspect how the backend derives AI Processing Status.
2. Verify behavior when mapping data is absent.
3. Verify behavior when mapping exists but no monthly report has been uploaded.
4. Check completed and failed processing states.
5. Confirm the frontend displays the latest backend status.
6. Check whether stale responses cause incorrect status.
7. Verify deployment versions before retesting.

**Acceptance criteria**

The status column accurately reflects the report's current state across all supported scenarios.

**Classification:** Existing defect under verification.

## DEF-20: Column sorting does not work correctly

**QA observation**

The report listing contained sorting arrows, but the tester reported that some columns were not sorting as expected.

The team mentioned that a fix was expected in the frontend build being deployed.

**Expected behavior**

Sorting controls should work consistently for supported columns.

Columns that should not support sorting should not display misleading sorting controls.

**Engineering investigation**

- Inspect the sorting configuration for each column.
- Verify whether sorting occurs on the frontend or backend.
- Check whether status values are sorted by display label or internal value.
- Confirm that the Actions column is handled appropriately.
- Test ascending and descending sorting.
- Verify the latest deployed build.

**Acceptance criteria**

All supported sorting controls work as expected, and unsupported columns do not present nonfunctional controls.

**Classification:** Reported defect; fix may already be in progress.

## DEF-21: Last Modified does not display time

**QA observation**

The tester was trying to determine when a mapping had been modified.

The UI displayed only the date.

This made it difficult to distinguish multiple updates made on the same day.

The team agreed that showing the timestamp would be useful and suggested adding it to the backlog.

**Expected behavior**

Last Modified should display both date and time in a readable format.

**Engineering investigation**

1. Check whether the backend stores the full modification timestamp.
2. Verify whether the API returns the time.
3. Check whether the frontend truncates the timestamp.
4. Confirm consistent timezone handling.
5. Determine whether historical records contain time information.

**Acceptance criteria**

The UI displays the available date and time accurately without inventing time values for older records.

**Classification:** Agreed enhancement; backlog priority.

## DEF-22: Deal name is missing from processing and review screens

**QA observation**

While navigating between Dell and GMF, the tester pointed out that the screen did not clearly identify which deal was being processed.

The deal ID appeared in the URL, but the business-friendly deal name was not visible.

This made the cross-deal processing problem harder to recognize.

The team agreed that a recognizable deal name should be displayed.

**Expected behavior**

The processing and review screens should clearly display the deal name associated with the current report.

**Engineering investigation**

- Check whether the deal name is available from the existing API.
- Verify the relationship between deal ID and canonical deal name.
- Ensure the displayed name changes correctly when navigating between deals.
- Avoid using stale deal information from a previous screen.
- Check whether the report name and reporting month should also appear.

**Acceptance criteria**

Users can identify the active deal without inspecting the URL.

**Classification:** Agreed UI improvement.

---

# Section F — Internal Messages and Developer Features

## DEF-23: Developer option is exposed to business users

**QA observation**

During the demonstration, a developer-oriented option was visible.

The team explained that the functionality had originally been exposed to demonstrate the framework's rule-based validation capabilities.

The rules are used to verify and normalize extracted values before they are accepted or stored.

Examples include date normalization, minimum and maximum values, and expected ranges.

The team discussed either renaming the option in simple business language or changing its visibility.

**Expected behavior**

The product team should decide whether this functionality belongs in the business-user experience.

If retained, the label should clearly describe its purpose, such as validating AI-extracted values using configured rules.

**Engineering investigation**

- Identify the feature and its intended audience.
- Check whether it is controlled by user permissions or a feature flag.
- Verify whether hiding the UI affects backend validation.
- Confirm that rules continue to execute regardless of whether the developer option is visible.

**Acceptance criteria**

Business users see only the agreed functionality, while rule-based validation continues to work.

**Classification:** UI/product improvement.

## DEF-24: Internal AI messages and unnecessary banners appear

**QA observation**

The testers noticed messages and banners containing internal or technical information.

Some appeared to reference internal processing details that were not meaningful to business users.

The team agreed that these messages were unnecessary and should be removed.

**Expected behavior**

Only messages that help users understand the processing status, required action, or result should appear.

Internal debugging information should remain available through appropriate logs rather than being shown in the business interface.

**Engineering investigation**

1. Identify the components generating the messages.
2. Check whether messages come from the frontend, backend, or AI responses.
3. Separate diagnostic messages from user-facing notifications.
4. Remove unnecessary banners without hiding real errors.
5. Confirm that useful processing and validation messages remain visible.

**Acceptance criteria**

Business users no longer see irrelevant internal messages or banners.

**Classification:** Agreed UI cleanup.

---

# Section G — LlamaParse and Processing Consistency

## DEF-25: Vendor-side caching may reuse failed processing results

**QA observation**

The team reported discovering that LlamaParse performs caching behind the scenes.

They believed that results from a previously failed or problematic processing attempt could be reused for up to 24 hours.

A parameter was reportedly identified that could force fresh processing or reset the relevant cache behavior.

The team intended to incorporate this into the framework.

This was raised as a possible explanation for intermittent issues that were difficult to reproduce.

**Expected behavior**

The framework should use vendor caching intentionally.

A retry should not unknowingly reuse an unsuitable result when fresh processing is required.

However, caching should not be disabled globally without understanding its impact on performance and cost.

**Engineering investigation**

1. Verify the exact LlamaParse caching behavior using the installed SDK and current API.
2. Identify the parameter discussed by the team.
3. Determine whether failed results are actually cached and under what conditions.
4. Check whether retries use the same document and processing configuration.
5. Verify whether the application already passes the parameter.
6. Determine when a forced fresh parse is appropriate.
7. Add logging to distinguish cached results from newly processed results where supported.

**Acceptance criteria**

Caching behavior is documented, retries behave predictably, and the application does not unknowingly reuse unsuitable results.

**Classification:** Technical investigation; the reported 24-hour behavior must be verified.

## DEF-26: Mapping, skills, processing, and results may use inconsistent versions

**QA observation**

Several problems raised during the meeting suggested a possible mismatch between mapping configuration and processing results.

Examples included:

- Approximately 45 mapped attributes but only 3 expected fields.
- Hundreds of attributes appearing unexpectedly.
- Draft and finalized mapping confusion.
- Backend and frontend changes being deployed at different times.

The team also explained that rules may exist in skills files or supporting configuration scripts.

The exact cause of these inconsistencies was not established.

**Expected behavior**

Each processing job should use a clearly identified mapping configuration.

The mapping, generated skills, extraction instructions, validation rules, and final results should remain consistent for that job.

**Engineering investigation**

1. Trace the finalized mapping used when a job starts.
2. Check whether the job stores or references a mapping version.
3. Verify how skills files are generated and selected.
4. Determine whether mapping updates can affect jobs already running.
5. Check whether the frontend displays the same mapping version used by the job.
6. Compare the expected attributes against the extraction instructions and output.
7. Verify that deal-specific configuration is not shared incorrectly.

**Acceptance criteria**

A job's mapping and processing configuration can be identified and traced consistently from submission through final output.

**Classification:** Cross-cutting investigation; not a separately confirmed defect.

---

# Section H — QA Delivery and Tracking

## TASK-27: Defects are not linked to their corresponding stories

**QA observation**

During the meeting, the team initially could not find the reported defects on the expected board.

The tester explained that the defects had been created under a fix version.

However, the defects were not linked to the relevant user stories, making it difficult to understand which feature or development work they belonged to.

**Expected behavior**

Each defect should be linked to the appropriate story or feature, assigned to the relevant release, and visible in the team's QA tracking process.

**Action**

Review the existing defects, link them to their parent stories, and ensure that the board shows the correct testing scope.

**Classification:** QA process task, not an application defect.

## TASK-28: Tickets are not clearly marked ready for QA

**QA observation**

The tester asked that tickets be moved to Ready for QA when their fixes were deployed.

The developer explained that backend changes had been deployed first and frontend changes were still being completed.

The intention was to move tickets to QA only after the complete change was available.

**Expected behavior**

QA should not have to guess whether a fix is available.

A ticket should move to Ready for QA only when the necessary frontend and backend changes are deployed to the testing environment.

**Action**

Establish a clear handoff process and record the deployed version or change reference on each ticket.

**Classification:** Delivery process improvement.

---

# Final Instructions to the Coding Agent

Review all 28 items, but do not assume all 28 require separate code changes.

Some are duplicate symptoms, some are expected vendor limitations, and some are product decisions rather than defects.

Prioritize the investigation in this order:

**Priority 1 — Data correctness and isolation**

- DEF-02 and DEF-03: Cross-deal processing and job isolation.
- DEF-05: Missing expected attributes.
- DEF-14: Withheld values being written into Excel.
- DEF-26: Mapping and processing version consistency.

**Priority 2 — Mapping correctness**

- DEF-06 and DEF-07: Unexpected and ignored attributes.
- DEF-08 through DEF-10: Draft and finalized mapping behavior.
- DEF-19: Incorrect processing status.

**Priority 3 — Review experience**

- DEF-11: Automatic Excel download.
- DEF-12 and DEF-13: Excel highlighting and scrolling.
- DEF-15 through DEF-18: Validation explanations and source highlighting.

**Priority 4 — Performance and remaining improvements**

- DEF-01 and DEF-25: Processing delays and caching.
- DEF-20 through DEF-24: Listing, navigation, and UI improvements.
- DEF-04: Cancellation requirements.
- TASK-27 and TASK-28: QA tracking and deployment handoff.

For each item, provide the following results:

| Field | Required information |
|---|---|
| Finding | Confirmed bug, suspected bug, expected behavior, already fixed, or product decision needed |
| Root cause | Why the issue occurs, supported by code or logs |
| Affected components | Frontend, backend, database, worker, mapping, vendor, or Excel generation |
| Reproduction | Exact steps needed to reproduce the issue |
| Proposed fix | Smallest safe code change |
| Shared impact | Whether the fix could affect other deals or use cases |
| Tests | Unit, integration, or end-to-end tests needed |
| Status | Not started, investigating, fix proposed, fixed, or needs clarification |

**Engineering constraints**

- Follow the existing project structure and coding conventions.
- Keep deal-specific behavior separate from shared framework code.
- Do not introduce deal-specific conditions into foundational components.
- Do not modify production data to reproduce QA issues.
- Use isolated test data and test jobs.
- Preserve existing API contracts unless a change is explicitly required.
- Add regression tests for confirmed defects.
- Do not remove functionality merely because it appears unused; verify dependencies first.
- Avoid broad refactoring while fixing targeted defects.
- Clearly identify which findings require product or business confirmation.

**Expected deliverable**

Produce a consolidated engineering review showing confirmed defects, root causes, recommended fixes, affected files, tests, and unresolved questions.

Group issues that share the same root cause so the team can fix them together.

Do not begin making changes until the initial investigation is complete and the findings are reviewed.
