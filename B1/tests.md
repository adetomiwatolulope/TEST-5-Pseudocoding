# Task 5 Part B1 - Tests

Test evidence for `assertIndependentSignOff`, run against the real implementation.

Function under test: `assertIndependentSignOff(requestedBy, signedOffBy)`
Source: `Medlink-ai/modules/ai/signoff.ts`
Pseudocode: `B1/pseudocode.md` (2000 bytes, unmodified by this stage)
Implementation record: `B1/implementation.md`

Every result below was produced by actually running the suite shown in this file
against the real function in `Medlink-ai`. Nothing is predicted.

---

## How the tests were run

The tests use the project's own runner, `node:test`, and import the real function by
its file URL. The suite was kept outside the repository in the system temp directory
and deleted afterwards, so that no test file was added to `Medlink-ai` and nothing was
committed. The code is reproduced below, so the run is repeatable.

```ts
import { test } from 'node:test';
import assert from 'node:assert/strict';
import { assertIndependentSignOff } from 'file:///.../modules/ai/signoff.ts';
import { AiError } from 'file:///.../modules/ai/errors.ts';

const FORBIDDEN = 'ERR_SIGN_OFF_FORBIDDEN';

function throwsForbidden(requestedBy: string, signedOffBy: string): boolean {
  try {
    assertIndependentSignOff(requestedBy, signedOffBy);
    return false;
  } catch (error: unknown) {
    return error instanceof AiError && error.code === FORBIDDEN;
  }
}

function passes(requestedBy: string, signedOffBy: string): boolean {
  try {
    assertIndependentSignOff(requestedBy, signedOffBy);
    return true;
  } catch {
    return false;
  }
}
```

Two helpers are used so that each case can assert both directions: that the function
did not raise, and that it did not raise the wrong thing. A case that simply checked
"did it throw" would pass even if the wrong error code were raised.

---

## Test cases

### T1 - Different normal user IDs pass

| | |
| --- | --- |
| **Input** | `requestedBy = 'u_1'`, `signedOffBy = 'u_2'` |
| **Expected behavior** | No error raised. Two different users, so the sign-off is allowed. |
| **Actual behavior** | Function returned normally. Nothing was raised. |
| **Result** | **PASS** |

Also asserted that `throwsForbidden('u_1', 'u_2')` is false, confirming it did not
raise a differently-coded error.

### T2 - Same user ID throws ERR_SIGN_OFF_FORBIDDEN

| | |
| --- | --- |
| **Input** | `requestedBy = 'u_1'`, `signedOffBy = 'u_1'` |
| **Expected behavior** | Raises `AiError` with code `ERR_SIGN_OFF_FORBIDDEN`. |
| **Actual behavior** | Raised an `AiError` whose code was `ERR_SIGN_OFF_FORBIDDEN`. |
| **Result** | **PASS** |

This is the one case that must fail, and it is the reason the function exists.

### T3 - Case-sensitive difference passes

| | |
| --- | --- |
| **Input** | `requestedBy = 'U_1'`, `signedOffBy = 'u_1'`, and the reverse `'u_1'` / `'U_1'` |
| **Expected behavior** | No error raised. The two strings are different, and the implementation does not lowercase either one. |
| **Actual behavior** | Both calls returned normally. Nothing was raised in either direction. |
| **Result** | **PASS** |

Confirms no lowercasing or case folding anywhere in the comparison.

### T4 - Trailing whitespace passes because IDs are compared exactly

| | |
| --- | --- |
| **Input** | `requestedBy = 'u_1'`, `signedOffBy = 'u_1 '`, and the reverse `'u_1 '` / `'u_1'` |
| **Expected behavior** | No error raised. The two strings differ by a trailing space, and no trimming is performed. |
| **Actual behavior** | Both calls returned normally. Nothing was raised in either direction. |
| **Result** | **PASS** |

Confirms no trimming. This case is the direct consequence of the correction made to
the pseudocode earlier in this stage, which removed the assumption that ids should be
normalised before comparison.

### T5 - Similar but different IDs pass

| | |
| --- | --- |
| **Input** | `'u_1'` / `'u_10'`, the reverse, and `'patient_1'` / `'patient_12'` |
| **Expected behavior** | No error raised. Each pair is a different user id, including where one is a prefix of the other. |
| **Actual behavior** | All three calls returned normally. |
| **Result** | **PASS** |

Guards against a comparison that would wrongly treat a shared prefix as the same user,
for example a prefix match instead of equality.

### T6 - Two identical UUIDs throw

| | |
| --- | --- |
| **Input** | `requestedBy = signedOffBy = '3f2504e0-4f89-41d3-9a0c-0305e82c3301'` |
| **Expected behavior** | Raises `AiError` with code `ERR_SIGN_OFF_FORBIDDEN`. |
| **Actual behavior** | Raised an `AiError` whose code was `ERR_SIGN_OFF_FORBIDDEN`. |
| **Result** | **PASS** |

Confirms the rule holds for the real id format used by the schema, not only for short
hand-written ids. The `User.id` column is a UUID.

### T7 - Two empty IDs are exactly the same and throw

| | |
| --- | --- |
| **Input** | `requestedBy = ''`, `signedOffBy = ''` |
| **Expected behavior** | Raises `ERR_SIGN_OFF_FORBIDDEN`, because the two empty strings are exactly the same. |
| **Actual behavior** | Raised an `AiError` whose code was `ERR_SIGN_OFF_FORBIDDEN`. |
| **Result** | **PASS** |

This is the limit of the specification, and it is worth being explicit about. The
pseudocode states only one failing condition: the two ids are exactly the same string.
Two empty strings are exactly the same string, so they fail. Nothing in the
specification says an absent id should be tolerated, so the function does not tolerate
it. This behaviour follows from the specification rather than being an extra decision.

In production the `AiSuggestion.requestedBy` and `AiSuggestion.signedOffBy` columns are
non-nullable and `signedOffBy` is only set during sign-off, so this case should not
arise from real data.

---

## Test run output

```
> node --test b1-signoff.test.ts

✔ T1 different normal user IDs pass (1.0454ms)
✔ T2 same user ID throws ERR_SIGN_OFF_FORBIDDEN (0.1854ms)
✔ T3 case-sensitive difference passes (0.1413ms)
✔ T4 trailing whitespace passes because IDs are compared exactly (0.1351ms)
✔ T5 similar but different IDs pass (0.1448ms)
✔ T6 two identical UUIDs throw (0.1474ms)
✔ T7 two empty IDs are exactly the same and throw (0.1399ms)
ℹ tests 7
ℹ pass 7
ℹ fail 0
exit code: 0
```

---

## Additional verification

### Existing sign-off tests still pass

```
node --test tests/ai/signoff.test.ts
exit code: 0
tests 5
pass 5
fail 0
```

The five pre-existing tests, which cover `assertDoctorRole` and
`assertSuggestionSignoffable`, are unaffected by the addition.

### Typecheck still passes

```
tsc --noEmit          (TypeScript 5.9.3)
exit code: 0
output: none
```

### Pseudocode not modified

`B1/pseudocode.md` is 2000 bytes, the same size as at the end of the pseudocode
stage. It was not touched during the testing stage.

### Nothing committed

The repository has exactly one commit, the pre-existing
`7d8c6e9 Initial commit: MedLink MVP source...`. No commit was created during Task 5,
and the working tree still shows only the single uncommitted modification described
below.

---

## Summary

| Metric | Value |
| --- | --- |
| Tests written | 7 |
| Tests passed | 7 |
| Tests failed | 0 |
| Exit code | 0 |
| Coverage of the 5 required cases | 5 of 5, as T1 through T5 |
| Additional cases | 2 - T6 UUIDs, T7 empty ids |

### Discrepancies

**None between the pseudocode and the implementation.** All 7 tests behaved exactly as
the pseudocode specifies. Specifically:

- T1, T3, T4, T5 all passed, confirming the implementation performs no lowercasing, no
  trimming, and no normalisation of either id.
- T2 and T6 both raised the required `ERR_SIGN_OFF_FORBIDDEN`, confirming the error
  code is the one the pseudocode names.
- No test produced a result that the pseudocode did not anticipate.

**One pre-existing failure elsewhere in the project, unrelated to this work.** The full
suite reports 69 pass and 1 fail. The failure is
`tests/identity/user.service.test.ts`, which cannot load because it imports the
Next.js alias `@/lib` (`"@/*": ["./*"]` in `tsconfig.json`), which the plain Node
runner does not resolve. It is a module-resolution error, not an assertion failure.
`modules/identity/user.service.ts` does not import `signoff.ts`, so it cannot reach the
new function, and this failure was present before any B1 work began.

**Not verified: lint.** `npm run lint` maps to `next lint`, which cannot run because
`node_modules/eslint` is not installed even though `.eslintrc.json` is present. No lint
result is claimed in either direction.

---

## Final git status

```
$ git -C Medlink-ai status --porcelain
 M modules/ai/signoff.ts

$ git -C Medlink-ai rev-list --count HEAD
1

$ git -C Medlink-ai log -1 --format='%h %s'
7d8c6e9 Initial commit: MedLink MVP source (identity, scheduling, clinical, AI document summarization slice)
```

| Item | State |
| --- | --- |
| Files modified | 1 - `modules/ai/signoff.ts` only |
| Lines removed | 0 |
| Test files added to the repository | 0 - still 13 |
| Unrelated production files touched | 0 |
| Commits created | 0 |
| Working tree | dirty by exactly the one intended uncommitted change |

The function remains unimplemented in any committed sense: the change sits uncommitted
in the working tree, so it can be reverted with a single command at any point.