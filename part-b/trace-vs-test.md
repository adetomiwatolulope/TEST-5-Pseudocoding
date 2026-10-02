# Task 5 — Part B2 Evidence: trace-vs-test

**Part:** Task 5 Part B2
**Specification (B1):** `part-b/pseudocode.md`
**Implementation:** `Medlink-ai/modules/patients/scheduling.rules.ts`
**Function:** `hasSchedulingConflict`

## Method

The five B1 hand-trace inputs were run against the current implementation, and the actual results were compared with the expected results recorded in the B1 hand traces.

- Node 24 executes the project's TypeScript directly, so the function was called through a throwaway harness kept outside the project. The harness was deleted after the run; no test file was added to the project.
- All times are placed on a single calendar date, as the B1 traces assume same-date appointments.
- The result is a single object carrying a `status` of `invalid`, `no-conflict`, or `conflict`. A `conflict` result also carries `conflictingAppointment`.
- Traces 1–5 are used, matching the five B1 hand traces. (The document now also contains a Trace 6 added later; it is outside the five requested here.)
- `npm run typecheck` (`tsc --noEmit`) was run and passed with exit code 0.

## Comparison table

| # | Input | Expected (B1) | Actual (implementation) | Verdict |
|---|---|---|---|---|
| 1 | Proposed 09:00–10:00; Existing: `PENDING_PAYMENT` 11:00–12:00 | status `no-conflict` | status `no-conflict` | **MATCH** |
| 2 | Proposed 10:00–11:00; Existing: `CONFIRMED` 09:30–10:30, then `CONFIRMED` 10:45–11:30 | status `conflict`, carrying entry 1 (09:30–10:30 `CONFIRMED`) | status `conflict`, carrying entry 1 (09:30–10:30 `CONFIRMED`) | **MATCH** |
| 3 | Proposed 09:00–10:00; Existing: `CONFIRMED` 10:00–11:00 | status `no-conflict` | status `no-conflict` | **MATCH** |
| 4 | Proposed 10:00–11:00; Existing: `CANCELLED` 10:30–09:00 | status `no-conflict` | status `no-conflict` | **MATCH** |
| 5 | Proposed 09:00–10:00; Existing: `COMPLETED` 14:00–15:00, `REFUNDED` 08:00–08:30, `PENDING_PAYMENT` 10:30–09:00 | status `invalid` | status `invalid` | **MATCH** |

## Captured run output

```
Trace 1 (no conflict)
  expected: no-conflict
  actual:   no-conflict
  verdict:  MATCH

Trace 2 (genuine conflict)
  expected: conflict carrying 09:30-10:30 CONFIRMED
  actual:   conflict carrying 09:30-10:30 CONFIRMED
  verdict:  MATCH

Trace 3 (touch at endpoint)
  expected: no-conflict
  actual:   no-conflict
  verdict:  MATCH

Trace 4 (ignored CANCELLED with invalid times)
  expected: no-conflict
  actual:   no-conflict
  verdict:  MATCH

Trace 5 (invalid active entry)
  expected: invalid
  actual:   invalid
  verdict:  MATCH

SUMMARY matched=5 mismatched=0
```

## Notes per case

- **Trace 1** — the proposed window sits entirely before the existing `PENDING_PAYMENT` window, so the second half of the strict overlap test fails and the walk ends with `no-conflict`.
- **Trace 2** — the first existing entry overlaps the proposal; it is returned immediately, and entry 2 is never reached. The carried appointment is entry 1 (09:30–10:30 `CONFIRMED`), confirming both the strict overlap and the first-conflict early exit.
- **Trace 3** — the proposal ends at exactly 10:00 and the existing entry starts at 10:00. Equality is not "later than", so the overlap test fails and the result is `no-conflict`, confirming that touching endpoints do not conflict.
- **Trace 4** — the `CANCELLED` entry has reversed times (10:30–09:00), but `CANCELLED` is skipped before any time validation, so the malformed times are never inspected and the result is `no-conflict`.
- **Trace 5** — the `COMPLETED` and `REFUNDED` entries are skipped without time checks; the `PENDING_PAYMENT` entry is active and its end (09:00) is not later than its start (10:30), so the result is `invalid`.

## Mismatches

None. No explanation of a difference is required because no case mismatched.

## Summary

- **Matched: 5 of 5**
- **Mismatched: 0 of 5**

All five B1 hand-trace inputs produce the expected B1 results against the current implementation.

## Files

- Created: `Medlink-ai/modules/patients/scheduling.rules.ts`
- Created: `part-b/trace-vs-test.md` (this file)
- Not modified: `part-b/pseudocode.md`
- The throwaway test harness was created outside the project and deleted after the run.
