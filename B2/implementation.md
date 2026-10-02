# Task 5 Part B2 - Implementation

Evidence for the Part B2 pseudocode-to-code mismatch exercise on an existing function.

Function under exercise: `assertNoUnsupportedConfidence`
Source: `Medlink-ai/modules/ai/r11.ts`
Pseudocode: `B2/pseudocode.md`
Workspace: `C:\Users\user\Desktop\TEST 5 Pseudocoding`
Project: `C:\Users\user\Desktop\Medlink-ai`

---

## 1. Pseudocode

`B2/pseudocode.md` contains the 7-step pseudocode for `assertNoUnsupportedConfidence`, written to the Task 5 pseudocode standard with the sections `FUNCTION`, `INPUTS`, `OUTPUT`, `SIDE EFFECTS`, `FAILS WHEN`, and `STEPS`.

It was presented in a message for review and explicitly approved before any implementation was generated.

It has not been modified at any point during the exercise. Verified by hash at three separate moments:

| Checkpoint | Size | SHA256 |
| --- | --- | --- |
| After saving | 2340 bytes | `FD2D256A25134B6533D6ADEE90543721EB438D1EF219AB26B6A38E87603E7DC2` |
| After implementation | 2340 bytes | `FD2D256A…` |
| After mismatch and restoration | 2340 bytes | `FD2D256A…` — full match |

Still 7 numbered steps, contiguous 1 to 7. The pseudocode remained the source of truth throughout.

---

## 2. AI-Generated Implementation

The generated implementation is `assertNoUnsupportedConfidence(confidence: number | null, summary: string): void` in `Medlink-ai/modules/ai/r11.ts`, replacing the body of the existing function at the same location while preserving its signature and exported name.

The original function was inspected first, only to establish where the implementation belongs and which imports and helpers were already available: `AiError` is imported at line 1, and a private helper `minSubstantiveSummaryLength()` exists at line 22. The generated version reuses both and adds no import.

The generated implementation follows the pseudocode step by step: an early return when no confidence score was supplied, then reading the configured minimum, trimming the summary, counting the remaining characters, and raising the project's existing `AiError` with code `ERR_AI_FABRICATION_REJECTED` and the exact message from the pseudocode when that count is strictly less than the minimum.

The generated form differs structurally from the original, which used a single compound condition. The original was:

```
if (confidence !== null && summary.trim().length < minSubstantiveSummaryLength()) {
```

The generated form uses an early return followed by a separate comparison, because pseudocode step 2 states that the check stops when no confidence score was supplied.

**Before the deliberate mismatch, the generated implementation was behaviourally equivalent to the original.** Both produce the same result for every input, including the case where a missing confidence score means the configured minimum is never read. This equivalence was assessed by reading both versions against each other; it was not established by a test run.

Typecheck passed with the generated implementation in place:

```
tsc --noEmit   (TypeScript 5.9.3)
exit code: 0
elapsed: 2.9s
output: none
```

---

## 3. Deliberate Mismatch

The comparison was deliberately changed from strict less-than to less-than-or-equal in one operator, on line 56 of `modules/ai/r11.ts`:

```diff
-  if (trimmedSummaryLength < minimumLength) {
+  if (trimmedSummaryLength <= minimumLength) {
```

That was the only edit made to introduce the mismatch. The pseudocode, all test files, and every other line of the source file were left untouched.

This violated the pseudocode's strict-less-than rule. `B2/pseudocode.md` states in `FAILS WHEN` that only a length strictly below the minimum fails, and that a length exactly equal to the minimum is not a failure. Step 6 says the check applies when the count is "strictly less than the configured minimum". The `<=` version fails on equality, which the pseudocode explicitly excludes.

**The existing R11 test suite still passed 6 of 6.**

```
node --test tests/ai/r11.test.ts
exit code: 0
tests 6   pass 6   fail 0
```

The reason is that the suite contains no test sitting on the boundary. The configured minimum is `clamp(parseIntOr(AI_MIN_SUMMARY_LENGTH, 20), 0, 100000)` at `config.ts:112`, which is 20 by default.

Walking the six tests:

- **accepts a legitimate extraction summary** and **rejects a summary asserting a diagnosis** both pass `confidence: null`, so they return early and never reach the comparison.
- **rejects "suffers from" phrasing**, **accepts extractive text that merely mentions a diagnosis label**, and **rejects fabricated citations** all call a local helper that invokes only the diagnosis and citation assertions. None reaches this function.
- **rejects confidence without a substantive summary** is the only test that reaches the comparison. It passes `summary: 'Note.'`, a trimmed length of 5, against a minimum of 20. Under `<`, 5 is less than 20 and the function throws. Under `<=`, 5 is still less than or equal to 20 and the function throws. The assertion expects a throw, so it passes either way.

The suite therefore could not observe the difference between the two operators.

---

## 4. Catching the Mismatch

A temporary test was written specifically for the boundary rule. It was created outside the repository, in the system temp directory, and was deleted after use. No file in the project's permanent test suite was created or modified.

The test set `AI_MIN_SUMMARY_LENGTH` to 20, read the minimum back out of `loadAiConfig()` so the input was built to match the real configured value rather than an assumed one, and asserted the trimmed length before calling so the case could not silently stop testing the boundary.

| Case | Input length | Expected | Actual with the mismatch | Result |
| --- | --- | --- | --- | --- |
| A | 20, exactly equal to the minimum | must not throw | **threw `AiError`** | **FAIL** |
| B | 19, one below | must throw | threw | PASS |
| C | 21, one above | must not throw | did not throw | PASS |

Boundary test run result:

```
exit code: 1
tests 3   pass 2   fail 1
```

Case A failed with `Got unwanted exception`, message *"Provider output carries a confidence score without a substantive summary; rejecting as unsupported."* The throw originated at `r11.ts:57:11`. The input was `confidence` of 0.91 and a `summary` of twenty repeated characters, whose trimmed length is exactly 20, against a configured minimum of 20.

Cases B and C pass with either operator. A length of 19 is below the minimum under both `<` and `<=`, and a length of 21 is above the minimum under both. They are included to show the threshold is genuinely live and that the failure is specific to equality, rather than the function failing for some unrelated reason.

A test using 19 or 21 in place of 20 would have passed against the buggy implementation and demonstrated nothing.

I wrote this test myself, as part of the exercise. It is not an independent check by any other party, and it was not produced by running the implementation against a separate specification.

---

## 5. Restoration and Verification

The comparison was restored from `<=` to `<` in `modules/ai/r11.ts`, one operator on line 56:

```diff
-  if (trimmedSummaryLength <= minimumLength) {
+  if (trimmedSummaryLength < minimumLength) {
```

Zero occurrences of `<=` remain in the file.

| Verification | Result |
| --- | --- |
| Boundary test after restoration | **3 of 3 pass**, exit code 0 — run before deletion, while the file still existed |
| Temporary test cleanup | `b2-boundary.test.ts` deleted, and the `b2tests` temp directory removed entirely |
| `tests/ai/r11.test.ts` after restoration | **6 of 6 pass**, exit code 0 |
| Typecheck after restoration | `tsc --noEmit`, exit code 0, elapsed 2.7s, no output |
| Permanent tests added | none — repository test file count remains 13 |
| `B2/pseudocode.md` | unchanged, 2340 bytes, hash `FD2D256A…` confirmed matching |

Boundary test result after restoration, in full:

```
✔ A. trimmed summary EXACTLY equal to the minimum must NOT throw
✔ B. trimmed summary one character BELOW the minimum must throw
✔ C. trimmed summary one character ABOVE the minimum must NOT throw
tests 3   pass 3   fail 0
```

Case A passing after the revert confirms the defect was removed rather than masked.

### Final repository state

```
$ git -C Medlink-ai status --porcelain
 M modules/ai/r11.ts
 M modules/ai/signoff.ts
```

| File | State |
| --- | --- |
| `modules/ai/r11.ts` | contains the AI-generated implementation with the comparison restored to `<` |
| `modules/ai/signoff.ts` | the earlier Part B1 modification, hash `9BCBFCC8…`, not touched during B2 |
| Test files in repository | 13, none added or modified |
| Commits | 1, at `7d8c6e9` — nothing committed at any point |

Neither source change is committed, so both can be reverted with a single command.

**Not verified: lint.** `npm run lint` maps to `next lint`, which cannot run in this project because `node_modules/eslint` is not installed even though `.eslintrc.json` exists. No lint result is claimed in either direction.

---

## 6. B2 Conclusion

This exercise demonstrated that a passing test suite can miss a behavioural mismatch at an untested boundary.

The deliberate change from a strict less-than comparison to a less-than-or-equal comparison violated an explicit rule in the pseudocode, and `tests/ai/r11.test.ts` still reported 6 of 6 passing. The suite's only test that reaches the comparison uses a summary of 5 characters against a minimum of 20, a region where both operators behave identically. A green result in that suite therefore did not establish that the implementation matched the pseudocode.

A targeted test supplying a summary of exactly the configured minimum did detect the mismatch, reporting one failure against two passes. Its two surrounding cases, at one below and one above the minimum, passed under both operators, isolating the failure to equality.

Two facts are worth separating from the result. First, the mismatch was introduced deliberately by me as the exercise required; the reimplementation step on its own did not diverge from the original. Second, the coverage gap remains open. The boundary test was temporary and has been deleted, so `r11.ts` is once again protected only by a suite that cannot detect this class of defect.