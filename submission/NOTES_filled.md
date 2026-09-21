# Notes

Fill this in and send it back, together with your workflows.

Both files go in this `submission` folder:

- `NOTES.md`, this file, filled in.
- your two workflows, exported from n8n with **Download**: the repaired daily
  report, and the backfill. Any file names are fine. Send one file if you did
  not get to part 3.

Keep it short. Short and honest beats long and polished. Delete the italic
prompts as you go.

A root cause is not a symptom. "It reports 100 orders" is a symptom. "It asks
for one page and ignores the rest" is a cause. We are marking the causes.

---

## Fault 1: "the report says exactly 100 orders on busy days"

**Root cause, in one sentence:**

The workflow requested only page 1 from the paginated `/orders` endpoint, whose page size is capped at 100, and ignored the remaining pages.

**What I changed:**

I used the API's `total_pages` value to create one request per page, fetched all pages, and combined their `data` arrays before counting and aggregating the orders.

**How to prove it is gone:**

Run:

```powershell
Invoke-RestMethod -Method Post http://localhost:5678/webhook/daily-report -ContentType "application/json" -Body '{"date":"2026-03-10"}'
```

For `2026-03-10`, `order_count` should be `137`, not `100`.

---

## Fault 2: "the revenue is too low"

**Root cause, in one sentence:**

The workflow added MYR, USD and SGD amounts directly as if they were all MYR.

**What I changed:**

I added a runtime `GET /rates` request and convert each order into MYR using the rate for its currency before adding it to the daily total.

**How to prove it is gone:**

Run:

```powershell
Invoke-RestMethod -Method Post http://localhost:5678/webhook/daily-report -ContentType "application/json" -Body '{"date":"2026-03-10"}'
```

For `2026-03-10`, the result should show `total_myr` as `578886.3` (RM578,886.30).

---

## Fault 3: "late orders land on the wrong day"

**Root cause, in one sentence:**

The workflow treated the requested date as a UTC calendar day by appending `T00:00:00Z`, instead of using the Kuala Lumpur business day (UTC+8).

**What I changed:**

I convert Kuala Lumpur midnight to UTC before calling `/orders`, so `2026-03-10` uses the UTC range `2026-03-09T16:00:00.000Z` to `2026-03-10T16:00:00.000Z`.

**How to prove it is gone:**

Trigger `2026-03-10` and inspect the `Work out the day` node. It should output:

```text
from = 2026-03-09T16:00:00.000Z
to   = 2026-03-10T16:00:00.000Z
```

With those boundaries the orders API reports `137` orders for the Kuala Lumpur day.

---

## Part 2: how did you make a second run safe?

**What the workflow does now when the day already has a report:**

Before posting, the workflow calls `GET /report` and checks whether that date already exists. If it does, it skips the `POST /report` step and returns the report without creating another row.

**How to prove it:**

Reset the reports, trigger the same date twice, then check the stored reports:

```powershell
Invoke-RestMethod -Method Post http://localhost:4100/reset

Invoke-RestMethod -Method Post http://localhost:5678/webhook/daily-report -ContentType "application/json" -Body '{"date":"2026-03-10"}'

Invoke-RestMethod -Method Post http://localhost:5678/webhook/daily-report -ContentType "application/json" -Body '{"date":"2026-03-10"}'

Invoke-RestMethod -Method Get http://localhost:4100/report
```

`GET /report` should still show exactly one report for `2026-03-10`.

---

## Part 3: the backfill

**How it decides which days to do:**

It generates every date in the inclusive `from`/`to` range, reads the existing reports once, and only continues processing dates that do not already have a report.

**What it does with a day that has no orders, and why:**

It skips that date and returns a reason such as `no orders for this day`. It does not create a zero report because the brief requires a visible gap rather than treating missing/no-order data as a confirmed zero-sales report.

**What it does when one day in the range fails:**

The orders request is configured to continue on error. A failed date is added to `failed` with a reason, while the other dates continue through the workflow.

**How to prove it:**

Create only the March 10 report, force March 9 to fail, then run the backfill:

```powershell
Invoke-RestMethod -Method Post http://localhost:4100/reset

Invoke-RestMethod -Method Post http://localhost:5678/webhook/daily-report -ContentType "application/json" -Body '{"date":"2026-03-10"}'

Invoke-RestMethod -Method Post http://localhost:4100/_fail -ContentType "application/json" -Body '{"date":"2026-03-09"}'

$response = Invoke-RestMethod -Method Post http://localhost:5678/webhook/backfill -ContentType "application/json" -Body '{"from":"2026-03-08","to":"2026-03-11"}'
$response | ConvertTo-Json -Depth 10
```

The response should show March 9 in `failed`, March 8 in `skipped` for no orders, March 10 in `skipped` because it already exists, and the other valid missing day should still be processed.

---

## Which fault would have been caught earlier, and by what?

The pagination fault could have been caught by an automated integration test using a day with more than 100 orders and asserting that the report's `order_count` matches the API's `total` value rather than the length of the first page.

## What is still wrong that you did not fix?

The duplicate protection is a check-then-post workflow rather than an atomic database upsert. It is safe for repeated sequential runs, but two truly simultaneous executions could theoretically both pass the existence check before either posts. The provided mock API does not expose an atomic upsert or unique-date constraint.

---

## Anything else

I did not modify `fixtures/orders.json` or `mock-api/server.js`. Exchange rates are fetched from `/rates` at run time rather than hardcoded.
