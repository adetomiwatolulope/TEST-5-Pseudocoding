# Task 5 — Part C4: Recording Notes

**Purpose:** a speaking outline for a recording of no more than five minutes, explaining what the
AI-generated `hasSchedulingConflict` implementation does. These notes describe behaviour in plain
language, drawn from the specification and the comparison documents. Use the pseudocode as your
notes, not the code.

**Sources used:** `part-b/pseudocode.md`, `part-c/C2/reverse-pseudocode.md`,
`part-c/C2/difference-table.md`, `part-c/C3/implementation-comparison.md`.

**Framing for the recording:** the implementation was produced by an AI from the pseudocode. Speak
about what it does and how it lines up with the specification.

**Suggested pacing (about 4–5 minutes total):**

| Segment | Topic | Target |
|---|---|---|
| 1 | What the feature does | 20 s |
| 2 | Inputs | 20 s |
| 3 | Proposed appointment validation | 30 s |
| 4 | Processing existing appointments in order | 30 s |
| 5 | Ignored statuses and why | 30 s |
| 6 | Invalid appointments | 25 s |
| 7 | Conflict detection | 25 s |
| 8 | Early exit | 20 s |
| 9 | No-conflict outcome | 15 s |
| 10 | What differs or is ambiguous | 45 s |
| 11 | The three specification ambiguities | 45 s |
| 12 | B2 and C1 agreement on all ten inputs | 20 s |

---

## Speaking outline

### 1. What the feature does (20 s)
- One sentence: it decides whether a proposed appointment clashes with an existing appointment for a doctor.
- It answers with one of three verdicts: the time data cannot be trusted, there is no clash, or there is a clash and it names the appointment that clashed.
- We think of it as a booking guard: before a slot is offered, does it collide with something already booked?

### 2. What inputs it accepts (20 s)
- A proposed appointment: a start time and an end time.
- A list of existing appointments, each with a start time, an end time, and a status.
- Both times of any one appointment are expected to sit on a single calendar date; an appointment may not run past midnight.
- Mention that the exact representation of a "time" is one of the specification ambiguities covered later.

### 3. Proposed appointment validation (30 s)
- First check the proposed start is a valid time value.
- Then check the proposed end is a valid time value.
- Then check the end is strictly later than the start and on the same calendar date.
- If any of those fail, the verdict is "invalid" immediately, and the existing list is never examined.
- Note that "strictly later" matters: a zero-length appointment is invalid.

### 4. How existing appointments are processed in order (30 s)
- The list is walked in the order given.
- Each entry is handled one at a time before moving to the next.
- For an active entry, its start and end are validated, then its ordering on a single date is checked.
- Any failure while validating an active entry stops the walk with "invalid".
- The walk can also stop early with a conflict, explained shortly.

### 5. Which statuses are ignored and why (30 s)
- Three statuses are skipped completely: cancelled, refunded, and completed.
- Skipped means their times are never even inspected, so bad time data on them cannot produce "invalid", and they can never be reported as a conflict.
- The reason: these appointments no longer hold their slot. A cancelled or refunded booking has been released, and a completed appointment has already taken place, so it should not stop the slot being booked again.
- This applies no matter how malformed the ignored entry's times are.

### 6. How invalid appointments are handled (25 s)
- "Invalid" means the time data cannot be trusted, so no reliable answer can be given.
- It is returned when the proposed window fails its checks.
- It is also returned when an active existing appointment has an unusable start or end, or an end that is not later than its start on the same date.
- Important ordering point: ignored entries are skipped before validation, so the status check comes first; invalid data on an ignored entry is passed over silently.

### 7. How conflicts are detected (25 s)
- The test is a strict overlap test, applied only to active, valid appointments.
- Both halves must hold: the proposed start is earlier than the existing end, and the proposed end is later than the existing start.
- Because the comparisons are strict, two appointments that merely touch at an endpoint are not a conflict. Back-to-back bookings are allowed.
- The first active entry in list order that overlaps is the one reported.

### 8. The early-exit behavior (20 s)
- The function stops as soon as it has an answer.
- If it finds a conflict, it returns immediately and never looks at later entries.
- A consequence: invalid data sitting in a later active appointment is only discovered if no earlier entry already caused an exit.
- So an early conflict wins over later bad data, while "invalid" is only reached if the walk actually gets there.

### 9. What happens when there is no conflict (15 s)
- An empty list produces "no-conflict" once the proposed window has passed its checks.
- If the whole list is walked without a conflict, the final verdict is "no-conflict".
- In the no-conflict and invalid cases the result carries no appointment; only the conflict case names one.

### 10. What the AI implementation does differently or ambiguously (45 s)
- Keep it to the comparison findings rather than reading code.
- The AI behaves as specified on the main path, but the comparison surfaced four points.
- First, a valid time value is treated narrowly: only an actual date object with a usable time is accepted; other representations that could denote a real moment are rejected as invalid.
- Second, the calendar date comparison uses the machine's local time zone, so results near midnight can depend on where the code runs.
- Third, the specification promises the function never raises an error, but the AI can fail hard if the proposed appointment or the list itself is missing, rather than returning a verdict. This only matters for inputs outside the stated shape.
- Fourth, the field that carries the clashing appointment is named by the AI rather than by the spec, which never fixed a name.

### 11. The three specification ambiguities found in C2 (45 s)
- Ambiguity one: what counts as a "valid time value". The spec says a value that "can be interpreted" as a real date and time but never says what concrete values are acceptable, so the AI chose a narrow reading.
- Ambiguity two: what "same calendar date" means. The spec never names a calendar or time zone, so local time and universal time would give different answers near midnight.
- Ambiguity three: the name of the field that carries the clashing appointment. The spec says the result carries it but never names the field, so callers have nothing fixed to read.
- For each, the fix is to make the wording explicit rather than to guess; the exact replacement wording is in the difference table.

### 12. B2 and C1 agreement on all ten inputs (20 s)
- The original specification implementation and the AI implementation were tested with the same ten inputs: the five original traces plus five new awkward cases.
- They agreed on all ten, with no disagreements.
- That includes invalid proposed input, an ignored entry with unusual data, an early conflict that must stop processing, both sides of the endpoint boundary, and a cross-midnight proposed window.

### Closing line
- In short: the AI implementation matches the specification on every tested behaviour, and the only open items are wording ambiguities in the specification itself.

---

## Nontechnical Listener Evidence

**Instructions to the recorder:** ask a nontechnical listener to explain the feature back in their
own words without showing them this outline. Record the outcome below. Do not include sensitive
personal information.

- **Listener (relationship / non-identifying role, no sensitive personal information):**
  _e.g. "a colleague outside engineering, referred to as Listener A"_

- **Question asked:** "Can you explain back to me what this feature does?"

- **Their explanation (paraphrase or short quote):**
  _record what they said_

- **Did they correctly understand the main behavior?**
  - Yes / Partly / No
  - Which parts they got right:
    _e.g. checks a proposed appointment against existing ones; reports clash / no clash / bad data_

- **Any misunderstanding that occurred:**
  _e.g. thought touching appointments conflict, or thought cancelled appointments are still checked_

- **Follow-up given (if any):**
  _e.g. clarified that endpoint-touching is allowed and that ignored statuses are skipped before validation_
