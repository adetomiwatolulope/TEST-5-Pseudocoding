# Task 5 — Part C2: Difference Table

**Original specification:** `part-b/pseudocode.md`
**AI behavior described by:** `part-c/C2/reverse-pseudocode.md`
**AI implementation inspected for confirmation:** `Medlink-ai/modules/patients/scheduling.rules.ai.ts`

**Method:** Every numbered step, every INPUTS/OUTPUT/SIDE EFFECTS/FAILS WHEN clause, and
every helper in the AI implementation was compared against the original pseudocode.
Items that were compared and found equivalent are listed under "Behaviours compared and
found equivalent" so the table only contains genuine differences. The AI implementation
was not modified, and `part-b/pseudocode.md` was not modified.

---

## Difference table

| # | Original pseudocode behavior | Actual AI behavior | Classification | Changes observable behavior? | Correct action |
|---|---|---|---|---|---|
| 1 | A valid time value is "a value that can be interpreted as a real calendar date together with a real time of day, so that it names a moment that genuinely exists." Absent values, non-time values, and impossible moments (a date that never occurs, a time outside the day) are invalid. The concrete representation is never named. | A valid time value is only a `Date` instance whose `getTime()` is not `NaN`. Strings, numbers, plain objects, and date-shaped or realm-foreign objects are rejected even when they denote a real moment. No interpretation of the value is performed. | 3 — AI interpreted an ambiguous step differently (borders on 1, because the spec never fixed the representation and the AI added a `Date`-instance precondition) | Yes. The string `"2026-02-28T10:00"` or the epoch value `1772263200000` denotes a real moment: under the original it is a valid time value, under the AI it produces status `invalid`. | Keep the AI as-is for this pass. Fix the original spec to name the concrete representation and the exact test (Ambiguities to Fix #1). |
| 2 | "Same calendar date" and "belong to one calendar date" are used as pass/fail tests, but no calendar and no time zone is stated anywhere. | Compares the host machine's local year, month, and day through `getFullYear` / `getMonth` / `getDate`. The result therefore depends on the host time zone. | 3 — AI interpreted an ambiguous step differently | Yes, for instants near local midnight and whenever the host time zone differs from the caller's expectation. Two moments that are on the same date in one zone can be on different dates in another. | Keep the AI as-is for this pass. Fix the original spec to name the calendar and time zone (Ambiguities to Fix #2). |
| 3 | "It never stops by raising an error. Every situation produces one of the three outputs above." and "It never raises an error, and it never returns nothing." | Validates time fields only. Throws a `TypeError` if `proposed` is null/undefined (reading `startTime`), if `existingAppointments` is null/undefined (reading `length`), or if a list element is null/undefined (reading `status`). Bad time values do not throw; they return `invalid`. | 2 — AI omitted specified behavior | Yes, for malformed or absent container inputs. No throw occurs for inputs that match the declared TypeScript parameter types (a non-null window object and a non-null array of non-null entries). | Decide the intended contract. If the no-throw guarantee covers malformed inputs, add container and element guards to the implementation. Otherwise scope the guarantee to well-formed inputs in the spec. No change in this pass. |
| 4 | On `conflict` the result "carries that appointment" / "the conflicting appointment"; on `invalid` or `no-conflict` it "carries no appointment." No property name is ever given. | The carried appointment is stored in a property named `conflictingAppointment`. The property is absent for `invalid` and `no-conflict`. | 3 — AI interpreted an ambiguous step differently (unspecified naming) | Yes, for any caller that reads a named property; no for a caller that reads only `status`. | Keep the AI as-is for this pass. Fix the original spec to name the property and state its absence otherwise (Ambiguities to Fix #3). |

---

## Behaviours compared and found equivalent

These were explicitly checked and are **not** differences:

- Strict overlap test. Both use "proposed start earlier than existing end" and "proposed end later than existing start," with strict comparison. Appointments that touch at an endpoint are not a conflict in both.
- Cross-midnight appointments. Both make a proposed or active appointment invalid when its end is not later than its start on the same calendar date, which covers a window that ends on a later date.
- Empty list. Both return `no-conflict` after the proposed-appointment checks pass.
- Ignored statuses. Both skip exactly `CANCELLED`, `REFUNDED`, and `COMPLETED` completely, before any time validation, so malformed times on those entries never cause `invalid`.
- First conflict and early exit. Both return the earliest entry in list order that conflicts and stop; later entries (including later invalid active data) are never examined.
- Status set. Both treat the three statuses above as the only ignored ones; any other status is active. The original guarantees a status is always one of six names; the AI does not re-check membership. This only differs for inputs outside the stated contract, so it is not counted as a difference.
- Result statuses. Both always produce exactly one of `invalid`, `no-conflict`, `conflict`, and never return nothing, for well-formed inputs.
- Absent carried appointment. The AI omitting the `conflictingAppointment` property satisfies the original's "carries no appointment" wording.
- Order of the proposed-appointment checks (start valid, then end valid, then same-date ordering) is the same.

---

## Ambiguities to Fix

### 1. The concrete representation of "a valid time value"

- **Ambiguous wording:** "A valid time value: a value that can be interpreted as a real calendar date together with a real time of day, so that it names a moment that genuinely exists. A value that is absent, that is not a time at all, or that names a moment which cannot exist, such as a date that never occurs or a time outside the day, is not a valid time value."
- **Why it is ambiguous:** It says a value "can be interpreted," but never states what kinds of values are acceptable or who performs the interpretation. That permits at least three readings: (a) only `Date` objects, (b) any string/number the function parses, or (c) any structured date-like value. The examples (a date that never occurs, a time outside the day) even imply non-`Date` representations, because a `Date` cannot hold those. The AI chose reading (a), the narrowest.
- **Exact replacement wording:** "A valid time value is a `Date` object whose time value is not `NaN`, meaning it is not an invalid `Date`. A value that is absent, is not a `Date` object, or is a `Date` whose time value is `NaN` is not a valid time value. No other representation is accepted."

  *(If the intent was instead to accept parseable values, the replacement must name the parser and format explicitly, e.g. "a value is valid when it is a `Date`, or a string in ISO 8601 form that parses to a real moment; otherwise it is invalid" — but the wording above is the precise version that matches the AI implementation.)*

### 2. The calendar and time zone behind "same calendar date"

- **Ambiguous wording:** "the proposed end time is not later than the proposed start time on the same calendar date" (steps 4 and 12) and "The two times of an appointment belong to one calendar date" (INPUTS), together with "same calendar date" in FAILS WHEN.
- **Why it is ambiguous:** "calendar date" is used as a decisive test but the calendar and time zone are never defined. Local time, UTC, and a fixed practice time zone each give a different answer for moments near midnight, and an implementation without this rule cannot be compared for equality with another.
- **Exact replacement wording:** "Two moments fall on the same calendar date when they have the same year, month, and day in the host machine's local time zone. The year, month, and day are read locally, not in UTC."

  *(If a fixed zone is preferred for reproducibility, replace "the host machine's local time zone" with that zone, for example "UTC" or the clinic's time zone, but it must be named.)*

### 3. The name of the carried conflicting appointment

- **Ambiguous wording:** "When the status is conflict, the result also carries that appointment, so the caller can see which booking clashed. When the status is no-conflict or invalid, the result carries no appointment."
- **Why it is ambiguous:** The result is described as a single object with a status, but the property that holds the appointment is never named. A caller cannot read the field deterministically, and two implementations could use different names (`appointment`, `conflict`, `conflictingAppointment`) while both claiming to follow the spec. It is also unstated whether the absent case omits the property or sets it to null.
- **Exact replacement wording:** "The result is an object with a `status` field whose value is exactly one of `invalid`, `no-conflict`, or `conflict`. When the status is `conflict`, the same object also has a field named `conflictingAppointment` holding that appointment. When the status is `no-conflict` or `invalid`, the `conflictingAppointment` field is absent."

---

*End of Part C2 comparison. No changes were made to `part-b/pseudocode.md` or to the AI implementation. C3 and C4 were not started.*
