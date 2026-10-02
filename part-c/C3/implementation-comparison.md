# Task 5 — Part C3: Implementation Comparison

**B2 implementation:** `Medlink-ai/modules/patients/scheduling.rules.ts`
**C1 AI implementation:** `Medlink-ai/modules/patients/scheduling.rules.ai.ts`
**Specification (authority):** `part-b/pseudocode.md`
**Function under test:** `hasSchedulingConflict`

**Constraints honoured:** Neither implementation was modified. The original pseudocode was
not modified. C4 was not started. No project files were added or changed. The runner was a
throwaway `.mjs` file kept in the pre-approved temp directory and deleted after the run.

---

## Method

- Both implementations were imported into a single Node 24 runner and called with the
  **same** input objects for each case, so the comparison is apples-to-apples.
- Cases 1–5 are exactly the five original B2 hand-trace inputs recorded in
  `part-b/trace-vs-test.md`.
- Cases 6–10 are five new inputs written for this comparison, covering: an invalid
  proposed input, an ignored appointment with unusual data, an early result that must
  stop processing, the opposite boundary/touching side, and a cross-midnight proposed
  window.
- All ordinary times use the local date 15 March 2026; the cross-midnight case ends on
  16 March 2026. An invalid time value is produced with `new Date(NaN)` (an Invalid Date).
- Results are normalised to `status` plus, for a conflict, the carried appointment's
  `start-end` and `status`.
- Node 24 ran the project TypeScript directly through syntax detection. It emitted a
  benign `MODULE_TYPELESS_PACKAGE_JSON` warning (the project has no `"type"` field); the
  run still completed with exit code 0.

---

## Side-by-side results

| # | Kind | Input (proposed; existing in list order) | Expected (pseudocode) | B2 result | C1 result | Verdict |
|---|---|---|---|---|---|---|
| T1 | B2 trace 1 | Proposed 09:00–10:00; `PENDING_PAYMENT` 11:00–12:00 | `no-conflict` | `no-conflict` | `no-conflict` | AGREE |
| T2 | B2 trace 2 | Proposed 10:00–11:00; `CONFIRMED` 09:30–10:30, `CONFIRMED` 10:45–11:30 | `conflict`, carry 09:30–10:30 `CONFIRMED` | `conflict`, carry 09:30–10:30 `CONFIRMED` | `conflict`, carry 09:30–10:30 `CONFIRMED` | AGREE |
| T3 | B2 trace 3 | Proposed 09:00–10:00; `CONFIRMED` 10:00–11:00 | `no-conflict` | `no-conflict` | `no-conflict` | AGREE |
| T4 | B2 trace 4 | Proposed 10:00–11:00; `CANCELLED` 10:30–09:00 | `no-conflict` | `no-conflict` | `no-conflict` | AGREE |
| T5 | B2 trace 5 | Proposed 09:00–10:00; `COMPLETED` 14:00–15:00, `REFUNDED` 08:00–08:30, `PENDING_PAYMENT` 10:30–09:00 | `invalid` | `invalid` | `invalid` | AGREE |
| N6 | new — invalid proposed input | Proposed start = Invalid Date, end 10:00; existing `[]` | `invalid` | `invalid` | `invalid` | AGREE |
| N7 | new — ignored appointment, unusual data | Proposed 10:00–11:00; `CANCELLED` InvalidDate–InvalidDate, `CONFIRMED` 14:00–15:00 | `no-conflict` | `no-conflict` | `no-conflict` | AGREE |
| N8 | new — early result stops processing | Proposed 10:00–11:00; `CONFIRMED` 10:30–11:30, `CONFIRMED` 14:00–13:00 | `conflict`, carry 10:30–11:30 `CONFIRMED` | `conflict`, carry 10:30–11:30 `CONFIRMED` | `conflict`, carry 10:30–11:30 `CONFIRMED` | AGREE |
| N9 | new — opposite boundary/touching | Proposed 09:00–10:00; `CONFIRMED` 08:00–09:00 | `no-conflict` | `no-conflict` | `no-conflict` | AGREE |
| N10 | new — cross-midnight proposed | Proposed 22:00–02:00 next day; `CONFIRMED` 23:00–23:30 | `invalid` | `invalid` | `invalid` | AGREE |

**Summary: 10 of 10 AGREE. 0 disagreements.**

---

## Captured run output

```
ID  | EXPECTED (pseudocode)                  | B2                                      | C1                                      | VERDICT
--------------------------------------------------------------------------------------------------------------------
T1  | no-conflict                            | no-conflict                             | no-conflict                             | AGREE
T2  | conflict; carry 09:30-10:30 CONFIRMED  | conflict; carry 09:30-10:30 CONFIRMED   | conflict; carry 09:30-10:30 CONFIRMED   | AGREE
T3  | no-conflict                            | no-conflict                             | no-conflict                             | AGREE
T4  | no-conflict                            | no-conflict                             | no-conflict                             | AGREE
T5  | invalid                                | invalid                                 | invalid                                 | AGREE
N6  | invalid                                | invalid                                 | invalid                                 | AGREE
N7  | no-conflict                            | no-conflict                             | no-conflict                             | AGREE
N8  | conflict; carry 10:30-11:30 CONFIRMED  | conflict; carry 10:30-11:30 CONFIRMED   | conflict; carry 10:30-11:30 CONFIRMED   | AGREE
N9  | no-conflict                            | no-conflict                             | no-conflict                             | AGREE
N10 | invalid                                | invalid                                 | invalid                             | AGREE

SUMMARY agree=10 disagree=0
```

---

## Pseudocode basis for each expected result

- **T1** — steps 2–4 accept the proposed window; steps 8–12 validate the active `PENDING_PAYMENT` entry; step 13 fails because the proposed end (10:00) is not later than the existing start (11:00); step 15 returns `no-conflict`.
- **T2** — step 13 holds for the first entry, so step 13 returns `conflict` carrying entry 1; the early exit means entry 2 is never examined.
- **T3** — step 13 requires the proposed end to be strictly later than the existing start; 10:00 is not later than 10:00, so the overlap fails and step 15 returns `no-conflict`.
- **T4** — step 9 skips `CANCELLED` completely before steps 10–12, so its reversed times are never validated; step 15 returns `no-conflict`.
- **T5** — step 9 skips `COMPLETED` and `REFUNDED`; the `PENDING_PAYMENT` entry is active and step 12 finds its end (09:00) is not later than its start (10:30) on the same date, returning `invalid`.
- **N6** — step 2 finds the proposed start is not a valid time value (Invalid Date), returning `invalid`.
- **N7** — step 9 skips the `CANCELLED` entry, so its Invalid Date times are never validated; the `CONFIRMED` entry is active and valid but step 13 fails against the proposal; step 15 returns `no-conflict`.
- **N8** — step 13 holds for entry 1, returning `conflict` and stopping the walk; entry 2's reversed times are never reached at step 12.
- **N9** — the existing entry ends exactly at the proposed start; step 13 fails because the proposed start (09:00) is not strictly earlier than the existing end (09:00); step 15 returns `no-conflict`.
- **N10** — step 4 requires the proposed end to be later than the proposed start on the same calendar date; the end is on 16 March while the start is on 15 March, so step 4 returns `invalid`.

---

## Disagreements

**None.** All ten inputs produce identical results from B2 and C1.

Because there were no disagreements, no step-by-step pseudocode adjudication of a
disagreement was required. The "Pseudocode basis" section above records the governing
steps for each case as evidence that the shared result matches the authority, not merely
that the two implementations match each other.

### Why agreement is expected, not coincidental

The two implementations are structurally equivalent:

- Both validate the proposed start, then the proposed end, then the same-date ordering, in that order (steps 2–4).
- Both return `no-conflict` on an empty list (step 5).
- Both walk the list in order and skip exactly `CANCELLED`, `REFUNDED`, and `COMPLETED` before any time check (step 9). B2 uses a `Set` of ignored statuses; C1 uses an array with `includes`. These have identical membership semantics for these string values.
- Both validate active entries with the same start/end/same-date checks (steps 10–12). B2 combines them in one short-circuited `if`; C1 uses three separate `if`s. The observable result is identical.
- Both apply the same strict overlap test and return the first matching entry (step 13).
- Both fall through to `no-conflict` (step 15).
- B2 models the result as a discriminated union and C1 as an interface with an optional field, but at runtime both omit the appointment key for `invalid` and `no-conflict` and include it for `conflict`. The carried objects are normalised identically for comparison.

No input in this set exposes a behavioural difference between the two implementations.

---

## Files

- Created: `part-c/C3/implementation-comparison.md` (this file)
- Created and deleted: throwaway Node runner `cmp-harness.mjs` in the temp directory
- Not modified: `Medlink-ai/modules/patients/scheduling.rules.ts`
- Not modified: `Medlink-ai/modules/patients/scheduling.rules.ai.ts`
- Not modified: `part-b/pseudocode.md`
