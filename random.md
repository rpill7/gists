Build the Seller Reports landing-page calendar default. Do not wait for more spec. Find the month picker in THIS tree (the office tree has drifted; do not assume laptop paths) and wire this rule.

## What this is

The month picker is "which month of data are we looking at," not "when did someone last touch the file."

Today it likely defaults to the current calendar month. That is wrong. Seller-report data for a month is only done on the 15th of the month two months later. Example: July data is done on 15 September. Until that day, open on July. From that day, open on August.

A person can still pick any other month. This rule is only the default when the screen first opens.

## The rule (exact)

Use the user's local calendar date (year, month, day of month). Do not use UTC. Do not use the server clock if the UI already has a local "today."

Let today = (Y, M, D) where M is 1–12.

if D < 15:
    default = (Y, M) minus 2 months
else:   # D is 15 or later, including the last day of the month
    default = (Y, M) minus 1 month

When subtracting months, wrap the year:

- month 1 minus 1 → December of previous year
- month 1 minus 2 → November of previous year
- month 2 minus 2 → December of previous year

Return a year-month only (YYYY-MM or whatever this app already uses for the picker). Never a day.

Equivalent one-liner: offset = 2 if D < 15 else 1; default = current month minus offset.

## Why (so you do not "simplify" it)

July's report is finished on 15 September. So:

- 1 Sep through 14 Sep → July   (current minus 2)
- 15 Sep through 30 Sep → August (current minus 1)
- 1 Oct through 14 Oct → August  (current minus 2)
- 15 Oct through 31 Oct → September (current minus 1)

It automatically stays on the same reporting month across the 1st of the next month, then flips again on the 15th.

## Must-pass examples (freeze "today" in tests; do not depend on the real clock)

| Frozen today     | Default month |
| ---------------- | ------------- |
| 2026-09-14       | 2026-07       |
| 2026-09-15       | 2026-08       |
| 2026-09-16       | 2026-08       |
| 2026-09-30       | 2026-08       |
| 2026-10-01       | 2026-08       |
| 2026-10-14       | 2026-08       |
| 2026-10-15       | 2026-09       |
| 2026-01-14       | 2025-11       |
| 2026-01-15       | 2025-12       |
| 2026-03-14       | 2026-01       |
| 2026-03-15       | 2026-02       |
| 2026-02-14       | 2025-12       |
| 2026-02-15       | 2026-01       |

15th is inclusive: on the 15th, use minus 1, not minus 2.

## How to implement in this repo

1. Find the Seller Reports landing page month picker / calendar control. Search for month filter, reporting period, DatePicker with month view, YYYY-MM, current month default.
2. Put the rule in one small pure function, e.g. defaultReportingMonth(today) → year-month. No React, no fetch, no store inside it. Easy to unit test.
3. On first load, if no month is already chosen, call that function and set the picker + the deal grid to that month.
4. If the URL (or existing routing) already has a month, keep that month. Deep links from KME win over the default.
5. After load, changing the picker still works as today. Do not lock the picker. Do not hide other months.
6. Match existing date format and state (YYYY-MM vs Date vs {year, month}). Do not invent a second month field.

## Do not do

- Do not change last-modified / last-updated. That is "when the report was last touched in our app," which can be 14 Sep while the picker shows July.
- Do not change report status, hover actions, search, or columns.
- Do not fetch KME to decide the default month. This is a date rule only.
- Do not default to "today" or "current month."
- Do not use day-of-month 15 as a reporting month. The result has no day.
- Do not refactor unrelated landing-page code.

## Tests

Add a focused unit test of the pure function with every row in the table above. Freeze the date; do not read the real clock.

If this repo uses `tests/_harness.py` + `make test-one TEST=...`, follow that. If the month picker lives in the frontend, add the matching frontend unit test next to the function.

## Done looks like

- Opening Seller Reports with no month in the URL shows the default month from the rule, not the current calendar month.
- On a frozen 14 Sep 2026 it shows July 2026. On a frozen 15 Sep 2026 it shows August 2026.
- Picking another month still filters the grid.
- A URL that already names a month still opens on that month.

Reply with:
1. Files you changed (real paths on this machine)
2. The function you added (name + signature)
3. Test command and pass/fail
4. Where the landing page reads the default (file + a short note)
