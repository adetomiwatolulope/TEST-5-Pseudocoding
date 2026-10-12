# Task 5 — Pseudocoding

This README explains Task 5, the pseudocode standard used throughout it, what each part
requires, the evidence expected, and the honest current status of the work. It is a
navigation and explanation document only; it does not contain the deliverables
themselves.

---

## 1. Task 5 purpose

Task 5 exists to prove that the author understands a piece of code independently of any
tool. The test is whether the behaviour of a function can be described in plain language,
precisely enough that a reader could implement it without seeing the source, and then
whether that description holds up when checked against real code and against an
independent (AI) implementation.

The value being demonstrated is understanding that does not depend on the editor, the
type checker, a test runner, or a code generator. Pseudocode is the medium: it forces the
author to state inputs, outputs, branches, loops, early exits, failures, external calls,
and state changes explicitly.

---

## 2. The pseudocode standard used

Every pseudocode document in this task is written to the following standard:

- **FUNCTION** — the function being described, named once at the top.
- **INPUTS** — every value the function receives, including optional values and defaults.
- **OUTPUT** — what the function gives back on each path, including "returns nothing"
  where that is the case.
- **SIDE EFFECTS** — every read or write that leaves a trace outside the function's own
  return value, such as storage, database, logging, encryption, or shared state.
- **FAILS WHEN** — exactly the situations that stop the function, and how each one stops
  (throw, controlled error, or returned failure).
- **Numbered steps** — the body of the function as an ordered list.
- **One action per step** — each step does a single thing.
- **Plain English, present tense** — no programming syntax; describe what happens.
- **Explicit branches** — every decision is written out, including the branch not taken
  where it matters.
- **Loops** — expressed as walking a collection element by element, with the stop
  condition stated.
- **Early exits** — every point that returns or stops is called out explicitly.
- **External calls marked** — any call to storage, the database, an AI provider, or
  another module is named as an external action.
- **State changes marked** — any value that is created, updated, or persisted is called
  out as a change of state.

---

## 3. Part A

### A1 — three functions from own work
Choose three functions from the author's own work and pseudocode each one. The set must
include:

- the most complex function chosen,
- a money or authorization function,
- an error handler.

Each chosen function must have:
- its own pseudocode document written to the standard above, and
- hand traces covering a **normal** input, an **edge** input, and an **invalid** input,
- with the real code compared against the traces where the environment allows it.

### A2 — three open-source functions, increasing difficulty
Choose three open-source functions, deliberately increasing in difficulty from the first
to the third, and pseudocode each. Each requires hand traces (normal, edge, invalid) and
a comparison against the real open-source code. The comparison notes record what matched,
what did not, and any discrepancy that was found.

### A3 — planted bug
Introduce a small, deliberate bug into an existing function, then record:
- the function's intended (original) behaviour,
- the bug that was introduced,
- the actual behaviour once the bug is present, including whether type checking or tests
  caught it and why,
- the identification of the bug and its precise location,
- confirmation that the change was reverted and the file restored.

Hand traces and code-vs-trace comparisons are required wherever the brief asks for them.

---

## 4. Part B

### The new feature
The new feature built for Part B is **`hasSchedulingConflict`** — a booking guard that
decides whether a proposed appointment clashes with an existing appointment.

### B1 — pseudocode and hand traces
B1 is the specification:
- the pseudocode for `hasSchedulingConflict`, written to the standard in section 2, and
- hand traces that walk the pseudocode step by step, including awkward and invalid
  inputs (ignored statuses with malformed times, an active entry with impossible times,
  endpoint-touching bookings, and an early exit that skips later entries).

### B2 — implementation and trace-vs-test comparison
B2 is the implementation of the `hasSchedulingConflict` feature and the trace-vs-test
evidence: the B1 hand-trace inputs are run against the implementation, and each actual
result is compared with the result the pseudocode predicts. The implementation should map
back to the pseudocode **step by step**; any step that the code does not correspond to, or
any behaviour the code adds beyond the pseudocode, must be recorded rather than glossed
over.

---

## 5. Part C

### C1 — the AI implementation
C1 is a separate implementation of the same `hasSchedulingConflict` feature, produced by
an AI from the pseudocode alone. It is kept separate from the B2 implementation and is not
allowed to modify or reuse it.

### C2 — reverse-engineering the AI implementation
C2 reverse-engineers the C1 AI implementation back into pseudocode, using only the AI code
as the source, and then compares that reverse pseudocode against the original B1
pseudocode. Every meaningful difference is classified as exactly one of:

- **AI added unasked behaviour**, or
- **AI omitted specified behaviour**, or
- **AI interpreted an ambiguous step** (where the original pseudocode is genuinely
  ambiguous and the AI made a reasonable interpretation).

Genuine ambiguities in the original specification are documented, with proposed precise
wording that would remove each ambiguity.

### C3 — implementation comparison
C3 runs the **own (B2) implementation** and the **AI (C1) implementation** on the **same
ten inputs** — the original trace inputs plus additional awkward cases — and records the
results side by side. Any disagreement is resolved by tracing the original pseudocode
step by step, using the pseudocode as the authority rather than deciding by preference.

### C4 — recording and listener check
C4 requires:
- a recording in which the author explains the AI implementation using **only the
  pseudocode as notes** (not the code), and
- a nontechnical listener check: a listener who does not read pseudocode is asked to
  explain the feature back in their own words, and their understanding (and any
  misunderstanding) is recorded as evidence.

---

## 6. Evidence and submission requirements

The submission is expected to contain:

- hand-trace tables for every pseudocode function,
- real-code comparisons against the traces,
- A2 AI comparison notes,
- planted-bug evidence for A3,
- the C2 difference table,
- the ten-input implementation comparison for C3,
- the C4 recording and the nontechnical listener evidence,
- `.env` must **not** be committed,
- `.env.example` should exist,
- work should be recorded in **incremental commits**,
- nothing outside the assessment brief should be added.

---

## 7. Current Task 5 status

This is an honest snapshot of the existing files. It is not a claim of completion.

- **A1:** pseudocode and hand traces exist for three functions. A real-code comparison
  exists for **Function 3** (the error handler). The real-code runs attempted for
  **Function 1** and **Function 2** were **blocked by the environment** (Function 1 by a
  missing `medlink` database, Function 2 by a database that requires a SCRAM password that
  was not available), so those two remain hand-traced only.
- **A2:** three open-source functions are documented in increasing difficulty —
  **Function 4** (`safeParse`, Zod 3.25.76), **Function 5** (`prepareValue`, pg 8.23.0),
  and **Function 6** (`parse`, path-to-regexp 6.3.0) — each with pseudocode, hand traces,
  and a comparison against the installed source. **Functions 5 and 6 are AI-assisted** and
  are explicitly labelled as **not** a compliant no-AI first pass; only Function 4's first
  pass is claimed as a genuine first pass. The first-pass inaccuracy previously recorded in
  Function 4's hand traces has been corrected and the comparison notes updated to match.
- **A3:** intended behaviour, a pseudocode description of the intended **and** the buggy
  behaviour, the planted bug, its precise location, the test that caught it, the actual
  results before and with the bug, and the verified revert are all documented.
- **B1:** **complete** — the `hasSchedulingConflict` pseudocode and hand traces exist.
- **B2:** the implementation and the trace-vs-test evidence exist, and
  `part-b/trace-vs-test.md` now contains a verified **step-by-step mapping** of the B1
  pseudocode to the current implementation, plus the interpretations and out-of-contract
  behaviour it adds. **However, the implementation was AI-generated and therefore does NOT
  satisfy the assessment's requirement that B2 be implemented without AI.** This part is
  non-compliant on that point and must not be described as meeting the no-AI requirement.
  The author must still write the implementation independently and re-run the traces.
- **C1:** exists at
  `C:\Users\user\Desktop\Medlink-ai\modules\patients\scheduling.rules.ai.ts`.
- **C2:** the reverse pseudocode and the difference table exist at
  `part-c/C2/reverse-pseudocode.md` and `part-c/C2/difference-table.md`.
- **C3:** exists and compares the own implementation and the AI implementation across ten
  inputs; all ten agree.
- **C4:** the recording notes and a suggested listener question set exist, but the
  **actual recording and the nontechnical listener evidence are still missing**.

Task 5 is **not complete and not submission-ready**. The main outstanding items are the
blocked A1 real-code comparisons for Functions 1 and 2, the AI-assisted (non-no-AI) A2
Functions 5 and 6, the B2 no-AI non-compliance (the author must still write the
implementation independently), and the missing C4 recording and listener evidence.
