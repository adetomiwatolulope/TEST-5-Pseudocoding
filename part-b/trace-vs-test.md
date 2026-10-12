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

## Step-by-step mapping (pseudocode step → implementation)

Every numbered step in `part-b/pseudocode.md` was checked against
`Medlink-ai/modules/patients/scheduling.rules.ts`. Line numbers are the implementation's.

| Pseudocode step | Implementation | Match |
|---|---|---|
| 1 Take proposed start, end, and the list | Function signature and parameters, lines 47–50 (`proposed`, `existingAppointments`) | Yes |
| 2 Proposed start not valid → `invalid` | Lines 51–54, using `isValidTime` (31–33) | Yes |
| 3 Proposed end not valid → `invalid` | Lines 56–59, using `isValidTime` | Yes |
| 4 End not later than start on the same date → `invalid` | Lines 61–64, via `endsAfterStartsOnSameDate` (43–45) and `isSameCalendarDate` (35–41) | Yes |
| 5 Empty list → `no-conflict` | Lines 66–69 | Yes |
| 6 Begin the walk on the first entry | Line 72, `for (const appointment of existingAppointments)` | Yes |
| 7 Past the last entry → go to step 15 | Loop termination; control reaches lines 96–97 | Yes |
| 8 Take the current entry | The loop variable `appointment` (line 72) | Yes |
| 9 Ignored status → skip, times never checked | Lines 73–76, using `IGNORED_STATUSES` (25–29); `continue` before any time check | Yes |
| 10 Active start not valid → `invalid` | Lines 79–85 (combined condition) | Yes |
| 11 Active end not valid → `invalid` | Lines 79–85 (combined condition) | Yes |
| 12 Active end not later than start same date → `invalid` | Lines 79–85, via `endsAfterStartsOnSameDate` | Yes |
| 13 Strict overlap → `conflict` carrying the entry | Lines 87–93 (strict `<` and `>`) | Yes |
| 14 Move to the next entry, go to step 7 | Loop iteration; `continue` and the `for` advance | Yes |
| 15 Return `no-conflict` | Lines 96–97 | Yes |

No step in the pseudocode is left unimplemented, and no numbered step was reordered.

## Implementation details beyond, or interpreting, the pseudocode

These are recorded rather than glossed over, as the brief requires.

- **Step 9 (ignored statuses).** The set of three ignored statuses is expressed once as
  `IGNORED_STATUSES` (lines 25–29). This is an implementation detail of step 9 and matches
  the spec's ignored list (CANCELLED, REFUNDED, COMPLETED) exactly.
- **Steps 10–12 collapsed.** The three separate invalid checks are written as one combined
  `if` (lines 79–85). The behaviour is identical because all three return the same result.
- **Type declarations.** `AppointmentStatus`, `ProposedAppointment`, `ExistingAppointment`,
  and `SchedulingResult` (lines 1–23) are type-only declarations; they add no runtime
  behaviour beyond the pseudocode.
- **Field name for the carried appointment (interpretation).** The pseudocode says the
  conflict result "carries that appointment" but never names the field; the code calls it
  `conflictingAppointment` (lines 23 and 92). This is the same unresolved naming ambiguity
  recorded in C2.
- **What counts as a valid time value (interpretation).** The pseudocode defines a valid
  time value broadly ("can be interpreted as a real calendar date together with a real time
  of day"). The code accepts only an actual `Date` instance with a usable time
  (`isValidTime`, lines 31–33), so a numeric timestamp or an ISO string would be rejected as
  invalid even though it could denote a real moment. This is a narrow but reasonable reading
  of the ambiguous wording, recorded in C2 and C3.
- **"Same calendar date" (interpretation).** `isSameCalendarDate` (lines 35–41) compares the
  machine's **local** year/month/day. The pseudocode never names a calendar or time zone, so
  results near midnight can depend on where the code runs. Recorded in C2 and C3.
- **Out-of-contract inputs (divergence from "never raises an error").** The pseudocode states
  the function "never raises an error". The code will throw if `proposed` is `null`/`undefined`
  (reading `.startTime`) or if `existingAppointments` is `null` (reading `.length`). This only
  occurs for inputs outside the stated shape and is the same point recorded in C2 and C3; the
  pseudocode decision (return a verdict vs. throw) remains open.

## What the author must still do personally (B2 no-AI requirement)

The implementation at `Medlink-ai/modules/patients/scheduling.rules.ts` is currently
**AI-generated and therefore does not satisfy the B2 no-AI requirement.** The mapping above
documents the existing code; it does not remedy that non-compliance. The author must:

1. **Write the implementation independently.** Use `part-b/pseudocode.md` as the only
   source, without AI assistance, producing the same behaviour step for step.
2. **Re-run the five B1 trace inputs** against the independently written implementation and
   record the actual results against the expected results, as in the comparison table above.
3. **Confirm the step mapping still holds** for the independent code, recording any step the
   new code does not correspond to, or any behaviour it adds beyond the pseudocode.

To test with the five B1 inputs: place the times on a single calendar date, call
`hasSchedulingConflict(proposed, existing)`, and check the returned `status` (and, for a
conflict, the carried appointment). Traces 1–5 are the required five; the document also
contains a Trace 6 (early conflict before later invalid data) that is worth keeping as an
extra check of the early exit.

## Files

- Created: `Medlink-ai/modules/patients/scheduling.rules.ts`
- Created: `part-b/trace-vs-test.md` (this file)
- Not modified: `part-b/pseudocode.md`
- The throwaway test harness was created outside the project and deleted after the run.
