# Task 5 — Part C1: AI Implementation

**Part:** Task 5 Part C1
**Source of truth:** `part-b/pseudocode.md` (the B1 specification)
**Status:** documentation of the C1 deliverable. This file describes the deliverable; it
does not reproduce it.

---

## 1. Part C1 purpose

C1 exists to produce a **separate** implementation of the same feature described by the B1
pseudocode, written by an AI. The purpose is to have an independent implementation that
can later be reverse-engineered (C2) and compared against the author's own implementation
(C3).

Two constraints define C1:

- The C1 implementation must be built from the **pseudocode only**. It must not reuse or
  inspect the author's own B2 implementation for intended behaviour.
- The C1 implementation must remain a **separate file**, so that it does not overwrite or
  interfere with the B2 implementation.

---

## 2. Exact path to the AI implementation

```
C:\Users\user\Desktop\Medlink-ai\modules\patients\scheduling.rules.ai.ts
```

The implementation is a TypeScript source file. It lives in the **Medlink-ai repository**,
not in the `TEST 5 Pseudocoding` workspace. It was deliberately **not** moved or copied
into the workspace; the C1 deliverable stays in the repository so that the workspace holds
only the documentation and evidence around it.

---

## 3. Function implemented

One function is implemented:

- `hasSchedulingConflict` — the appointment booking guard that decides whether a proposed
  appointment conflicts with an existing appointment, returning one of three statuses:
  `invalid`, `no-conflict`, or `conflict`.

The file also declares the supporting types it uses (the appointment window shape, the
existing booking shape, the status union, and the result shape). No other exported
function is part of C1.

---

## 4. Generated from the B1 pseudocode

The C1 implementation was generated from the B1 pseudocode for `hasSchedulingConflict`
(`part-b/pseudocode.md`). The pseudocode was the only behavioural input:

- The AI was **not** shown the B2 implementation for intended behaviour.
- The generation used the pseudocode's INPUTS, OUTPUT, SIDE EFFECTS, FAILS WHEN clauses
  and its numbered steps as the specification.
- The generated code corresponds block by block to the pseudocode: proposed-appointment
  validation, the empty-list case, the in-order walk of existing appointments, the
  ignored-status skip, the active-appointment validation, the strict overlap test, and the
  final fall-through.

---

## 5. Instruction given to the AI

The AI was instructed to:

- implement **one function per pseudocode function block**, and
- add **no additional behaviour** beyond what the pseudocode specifies,
- keeping the implementation limited to the Part C scope.

Accordingly, the file contains the single `hasSchedulingConflict` function and the types it
requires, and no extra exports or unrequested behaviour.

---

## 6. Typecheck status

The project typecheck was run against the C1 file:

- Command: `npm run typecheck` (which runs `tsc --noEmit`).
- Result: **exit code 0** — passed, no output.

The C1 implementation compiles cleanly within the Medlink-ai project.

---

## 7. Ambiguities noted during C1

The B1 pseudocode left several points open, and the C1 implementation had to choose a
reading for each. These were recorded during C1 rather than silently resolved:

1. **Runtime shape of a "valid time value."** The pseudocode says a value that "can be
   interpreted" as a real date and time. C1 treats a valid time value as an actual `Date`
   instance whose time is not `NaN`. A parseable string or epoch number would be treated as
   invalid, because the pseudocode does not fix a concrete representation.
2. **What "same calendar date" is measured against.** The pseudocode names no calendar or
   time zone. C1 compares the **local** year, month, and day, so the result can depend on
   where the code runs.
3. **The name of the field carrying the conflicting appointment.** The pseudocode says the
   result "carries" the conflicting appointment but never names the field. C1 uses the
   field name `conflictingAppointment`.
4. **Statuses outside the six named values.** The pseudocode states a status is always one
   of six names. C1 special-cases only the three ignored names and treats any other status
   as active.
5. **Result container shape.** C1 models the result as one object with a `status` and an
   optional carried-appointment field. The pseudocode fixes the three status values but not
   whether the absent case omits the field or sets it to a null-like value.

These ambiguities were carried forward into the C2 reverse-engineering and comparison;
the proposed wording fixes are recorded in `part-c/C2/difference-table.md`.

---

## 8. Location of the actual implementation

The actual C1 implementation **remains in the Medlink-ai repository** at the path given in
section 2. It is not duplicated in the `TEST 5 Pseudocoding` workspace, and this document
does not reproduce it. The surrounding C1/C2/C3 evidence lives in the workspace under
`part-c/`.

---

## 9. Related Part C files (documentation only)

- `part-c/C2/reverse-pseudocode.md` — C2, the reverse-engineered behaviour of the C1
  implementation.
- `part-c/C2/difference-table.md` — C2, the classified differences between the B1
  pseudocode and the reverse-engineered behaviour.
- `part-c/C3/implementation-comparison.md` — C3, the ten-input comparison of the author's
  implementation and the C1 AI implementation.
