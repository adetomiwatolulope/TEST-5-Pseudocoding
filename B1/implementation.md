# Task 5 Part B1 - Implementation

Pseudocode implemented exactly as written in `B1/pseudocode.md`.

Pseudocode spec: `B1/pseudocode.md` (2000 bytes, 5 steps)
Workspace: `C:\Users\user\Desktop\TEST 5 Pseudocoding`
Project: `C:\Users\user\Desktop\Medlink-ai`

---

## 1. Source file and function

**File:** `Medlink-ai/modules/ai/signoff.ts`
**Function:** `assertIndependentSignOff(requestedBy: string, signedOffBy: string): void`

Added as a new export at the end of the existing file:

```ts
export function assertIndependentSignOff(requestedBy: string, signedOffBy: string): void {
  if (requestedBy === signedOffBy) {
    throw new AiError(
      'ERR_SIGN_OFF_FORBIDDEN',
      'A user may not sign off an AI suggestion they requested themselves.'
    );
  }
}
```

### File state before and after

| Property | Before | After |
| --- | --- | --- |
| SHA256 | `C31574C2450BBA7B2CF3FA1A8A9E9A12C958D020E8EE81DEF3F4F54A8ECAC2A9` | `9BCBFCC8C28C66188C163ACACF075E233A153B192851BB80E8E97663FAD71F2D` |
| Size | 996 bytes | 1269 bytes |
| Lines | 29 | 37 |

### The change is purely additive

```
 modules/ai/signoff.ts | 9 +++++++++
 1 file changed, 9 insertions(+)
```

Lines removed: **0**. Nothing existing was edited, moved, or reformatted.

### Style and convention compliance

The new function follows the file's existing conventions exactly:

- Declared `export function` with an explicit `void` return type, like both sibling
  functions.
- Uses the `AiError` already imported at the top of the file. No new import was added.
- Throws with the string code `'ERR_SIGN_OFF_FORBIDDEN'` as the first argument and a
  readable sentence as the second, matching `assertDoctorRole`.
- Message text is wrapped across lines because it exceeds the file's line width,
  matching how `assertSuggestionSignoffable` formats its two longer messages.
- Single-quoted strings, two-space indent, semicolons, braces on the same line.
- No doc comment was added, because neither existing function in this file has one.
  The file's existing block comment already describes this gate.

---

## 2. How the implementation maps to each pseudocode step

| Step | Pseudocode | Implementation | Mapping |
| --- | --- | --- | --- |
| 1 | Take the id of the user who requested the AI suggestion. | The parameter `requestedBy: string` | Direct. A declared parameter is how the function receives the value, so no separate statement is needed. |
| 2 | Take the id of the user who is signing it off. | The parameter `signedOffBy: string` | Direct. Same as step 1. |
| 3 | Compare the two ids with each other as the supplied user ID strings. | `requestedBy === signedOffBy` | Direct. One strict equality test between the two supplied strings. |
| 4 | If the two ids are exactly the same, stop by raising the project's existing AiError with code ERR_SIGN_OFF_FORBIDDEN, with a message explaining that a user may not sign off an AI suggestion they requested themselves. | The `if` body: `throw new AiError('ERR_SIGN_OFF_FORBIDDEN', 'A user may not sign off an AI suggestion they requested themselves.')` | Direct. Same condition, same error code, and the message says the same thing as the pseudocode step. |
| 5 | Otherwise, do nothing further and finish successfully, because the two ids belong to different users, so there is nothing wrong with this sign-off. | No `else` branch, and the function ends | Direct. There is no `else` because there is no work to do on the passing path. The `void` return type expresses "returns nothing at all" at the type level, and a function with no return statement reaches its end. |

**Coverage:** all 5 pseudocode steps are represented. No step was skipped, merged, or
added.

---

## 3. Differences between pseudocode and implementation

**No behavioural differences.** There are three presentational differences worth
recording, none of which changes what the function does.

1. **Steps 1 and 2 have no corresponding statement.** The pseudocode describes
   receiving two inputs as two actions. In the implementation the two inputs are
   declared parameters, so receiving them is expressed by the signature rather than
   by executable code. This is a difference in how the same idea is written, not in
   what happens.

2. **Step 5 has no `else` branch.** The pseudocode names the passing outcome
   explicitly. The implementation expresses the same outcome by having nothing after
   the `if`. Adding an empty `else` would be dead code.

3. **Step 4's "stop by raising" is written as `throw`.** The pseudocode uses project
   vocabulary deliberately; `throw new AiError(...)` is the concrete form of "stop by
   raising the project's existing AiError".

### What was deliberately not implemented

The pseudocode's final version dropped all assumptions about empty, absent, or
ambiguous ids. The implementation therefore contains **no** input validation, no
guarding, and no normalisation. Confirmed absent from the new function body:

| Checked for | Result |
| --- | --- |
| `.trim()` | absent |
| `.toLowerCase()` | absent |
| `.toUpperCase()` | absent |
| `.normalize()` | absent |
| `.replace()` | absent |
| `.localeCompare()` | absent |

The only comparison performed is strict equality on the two supplied strings.

The pseudocode also states the function does not perform the role check or the
sign-off state check. The implementation adds no such check; those remain in
`assertDoctorRole` and `assertSuggestionSignoffable`, untouched.

---

## 4. Verification result

All results below were produced by actually running the commands shown.

### Typecheck

```
tsc --noEmit        (TypeScript 5.9.3, run from the project root)
exit code: 0
output: none
```

### Existing sign-off tests

```
node --test tests/ai/signoff.test.ts
exit code: 0
tests 5
pass 5
fail 0
```

The five pre-existing tests, which cover `assertDoctorRole` and
`assertSuggestionSignoffable`, still pass. No regression.

### Full existing test suite

```
node --test "tests/**/*.test.ts"
exit code: 1
tests 70
pass 69
fail 1
```

**The one failure is pre-existing and unrelated to this change.** It is
`tests/identity/user.service.test.ts`, which fails to load:

```
Error [ERR_MODULE_NOT_FOUND]: Cannot find package '@/lib'
imported from C:\Users\user\Desktop\Medlink-ai\modules\identity\user.service.ts
```

That is the Next.js path alias `"@/*": ["./*"]` from `tsconfig.json`, which the plain
Node test runner cannot resolve. It is a module-resolution failure, not an assertion
failure. Two facts establish that this change did not cause it:

- `modules/identity/user.service.ts` does not import `signoff.ts` at all, so it cannot
  load or execute the new function.
- `modules/ai/signoff.ts` is the only file modified in the entire repository, and the
  modification is an added function that nothing imports yet.

The correct baseline for the full suite before this work was also 69 pass and this same
1 failure.

### Behavioural probe

A temporary script outside the repository imported the new function and called it
directly. It was deleted after the run and is not a test file.

| requestedBy | signedOffBy | Case | Result |
| --- | --- | --- | --- |
| `u_1` | `u_2` | different ids | no throw |
| `u_1` | `u_1` | identical ids | threw `ERR_SIGN_OFF_FORBIDDEN` |
| `U_1` | `u_1` | differ only by letter case | no throw |
| `u_1` | `u_1 ` | differ only by trailing space | no throw |
| `` | `` | both empty | threw `ERR_SIGN_OFF_FORBIDDEN` |
| `u_1` | `u_10` | different, shared prefix | no throw |

The three middle rows confirm the no-normalisation requirement: ids differing only by
letter case or only by a trailing space are treated as different users and pass
through, because the implementation compares the supplied strings as they are.

### Scope confirmation

| Check | Result |
| --- | --- |
| Files modified in the repository | 1 - `modules/ai/signoff.ts` |
| Lines removed | 0 |
| Existing functions changed | none - `assertDoctorRole` and `assertSuggestionSignoffable` are byte-identical |
| `ERR_SIGN_OFF_FORBIDDEN` occurrences | 2 - the 1 pre-existing in `assertDoctorRole`, plus the 1 new |
| Test files created | 0 - still 13 |
| B1 tests created | none, as instructed |
| Commits created | none |

### Not verified

`npm run lint` was **not** run and no lint result is claimed. `next lint` cannot run in
this project: `node_modules/eslint` is absent even though `.eslintrc.json` exists, so
the command aborts with `ESLint must be installed`. This was already established
during Part A3 and is unchanged.

### Conclusion

`assertIndependentSignOff` is implemented in `modules/ai/signoff.ts` as an additive
change of 8 lines, using the file's existing `AiError` convention and reusing the
import already present. It matches all 5 pseudocode steps with no behavioural
difference, adds no input validation or normalisation, passes typecheck, and causes no
regression in the existing sign-off tests. B1 tests have not been written yet.