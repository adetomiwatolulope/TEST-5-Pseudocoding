# Task 5 Part A3 - Planted Bug

Evidence file. Every test result quoted below was produced by actually running the
command shown. Nothing here is predicted or assumed.

Workspace: `C:\Users\user\Desktop\TEST 5 Pseudocoding`
Project under test: `C:\Users\user\Desktop\Medlink-ai`

---

## Function selected

`sanitizeOriginalName` - a pure string-sanitising helper that makes an untrusted
client-supplied filename safe to store.

### Source file and function name

`Medlink-ai/modules/patients/documents.rules.ts` line 50

```ts
export function sanitizeOriginalName(name: string): string {
  const basename = name.split(/[\\/]/).pop() ?? name;
  const cleaned = basename.replace(/[^a-zA-Z0-9._ -]/g, '_').trim();
  return cleaned.slice(0, 200);
}
```

### Why this function was chosen

- **Small.** Three statements, no branching.
- **Easy to reason about.** Each line does one obvious thing.
- **Pure.** No I/O, no clock, no randomness, no shared module state. Same input always
  gives the same output, so a wrong result is unambiguous.
- **Already covered by tests.** Two existing tests in
  `Medlink-ai/tests/patients/documents.rules.test.ts`.
- **Blast radius is one caller.** `Medlink-ai/modules/patients/documents.service.ts:66`.
- **Safe to revert.** One character on one line in a git-tracked file.

---

## Original behavior

Given a filename supplied by an untrusted client, the function returns a safe name in
three steps:

1. **Reduce to a basename.** Split on `/` or `\`, keep the last segment. This is the
   directory-traversal guard: `../../etc/passwd.png` becomes `passwd.png`.
2. **Scrub characters.** Replace every character outside `a-z A-Z 0-9 . _ space -`
   with `_`, then trim surrounding whitespace.
3. **Cap the length.** Truncate to 200 characters so a long name cannot exceed storage
   key limits.

### Intended behaviour — pseudocode

```
FUNCTION sanitizeOriginalName

INPUTS
- name: string — an untrusted, client-supplied filename.

OUTPUT
- A safe filename string, at most 200 characters long.

SIDE EFFECTS
- None. The function is pure: it reads no external state and writes none.

FAILS WHEN
- Never. Any string, including an empty string, produces a string.

STEPS
1. Split the name on the slash and backslash characters and keep the last segment. If the
   name contains no such separator, keep the whole name.
2. Replace every character that is not a letter, a digit, a dot, an underscore, a space, or
   a hyphen with an underscore.
3. Trim the surrounding whitespace.
4. Return the result cut to at most 200 characters.
```

### Actual behaviour with the bug — pseudocode

Identical to the intended pseudocode above except for the constant in the final step:

```
4. Return the result cut to at most 201 characters.
```

The only difference is the boundary constant in the final cut: 200 becomes 201. Every other
step is unchanged, so the two descriptions behave identically for inputs of 200 characters or
fewer and differ by exactly one character for longer inputs.

---

## Bug introduced

A single-character off-by-one on the truncation length, line 53.

```diff
-  return cleaned.slice(0, 200);
+  return cleaned.slice(0, 201);
```

Recorded before the edit:

| Property | Value |
| --- | --- |
| SHA256 of `documents.rules.ts` | `A20DBC443A481F75496C346C3D73682C1E9789B72AA042CDCA096C0C73A86492` |
| MD5 of `documents.rules.ts` | `EEA18EB978F34B9E0C387B17ED2E6AFF` |
| Size | 2906 bytes |
| `git status --porcelain` before edit | clean, 0 entries |
| Occurrences of `slice(0, 200)` before edit | 1 |
| Occurrences of `slice(0, 201)` before edit | 0 |

`git diff` after the edit confirmed the change was isolated to one line in one file:

```
 modules/patients/documents.rules.ts | 2 +-
 1 file changed, 1 insertion(+), 1 deletion(-)
```

No other file was modified.

---

## Why the bug is subtle

- **One character.** The line still reads as a deliberate "truncate long names" guard,
  so it does not look like a mistake when skimming a diff.
- **No signature or shape change.** The function still returns `string`, imports are
  untouched, nothing is renamed or added.
- **The type checker does not see it.** `tsc --noEmit` was run with the bug in place
  and exited 0 with no output. Both `200` and `201` are valid `number` arguments to
  `slice`, so there is nothing for the type system to complain about.
- **Normal-looking behaviour.** Names of 200 characters or fewer are still handled
  identically. Only over-long names differ, and only by one character. The function
  still appears to work.
- **Wrong in a way that is easy to argue away.** A reviewer may read `201` as an
  intentional tweak, or assume an off-by-one elsewhere, rather than a defect.

---

## Baseline test result (before the bug)

Command:

```
node --test tests/patients/documents.rules.test.ts
```

Result:

```
exit code: 0
tests 11
pass 11
fail 0
```

---

## Test result with the bug

Same command, run with the bug in place:

```
exit code: 1
tests 11
pass 10
fail 1
```

Failing test reported by the runner:

```
test at ..\Medlink-ai\tests\patients\documents.rules.test.ts:74:1
sanitizeOriginalName truncates to 200 characters
  AssertionError [ERR_ASSERTION]: Expected values to be strictly equal:
  201 !== 200
      at ..\Medlink-ai\tests\patients\documents.rules.test.ts:76:10
  actual: 201
  expected: 200
  operator: 'strictEqual'
```

### Typecheck with the bug

Command: `tsc --noEmit`, TypeScript 5.9.3, run from the project root.

```
exit code: 0
no output
```

The bug is invisible to the type checker.

### Lint with the bug

**Not executed - lint could not run in this project.**

`npm run lint` maps to `next lint`, and `next lint` aborted with:

```
ESLint must be installed: npm install --save-dev eslint
```

Confirmed: `node_modules/eslint` does not exist, although `.eslintrc.json` is present.
No lint result is claimed here, in either direction.

---

## Which test caught it

Exactly one test:

`tests/patients/documents.rules.test.ts:74`
`sanitizeOriginalName truncates to 200 characters`

```ts
test('sanitizeOriginalName truncates to 200 characters', () => {
  const long = 'a'.repeat(500) + '.png';
  assert.equal(sanitizeOriginalName(long).length, 200);
});
```

---

## Why the test caught it

Walk the planted bug through the function by hand.

1. `long = 'a'.repeat(500) + '.png'` is 503 characters of plain ASCII.
2. `name.split(/[\\/]/)` finds no `/` or `\`, so the single segment is the whole
   string, and `.pop()` returns it unchanged. `basename` is 503 characters.
3. `.replace(/[^a-zA-Z0-9._ -]/g, '_')` finds nothing to replace, because every
   character in the input is an ASCII letter or a dot. `cleaned` is still 503
   characters and identical to `basename`.
4. `.trim()` removes nothing, since the string starts with `a` and ends with `g`.
   Still 503 characters.
5. `cleaned.slice(0, 201)` returns the first 201 characters. Correct code returns 200.

So `assert.equal(result.length, 200)` receives `201` and fails, exactly as the runner
reported: `201 !== 200`.

The test pins the exact truncation length rather than a loose bound such as
"less than or equal to 500". That precision is the whole reason the bug was caught:
an off-by-one is invisible to any looser assertion.

---

## Which tests did not catch it

The other ten tests all passed with the bug in place:

| Test | Line | Why it did not catch it |
| --- | --- | --- |
| `detectFileType identifies PNG by magic bytes` | 20 | Calls `detectFileType` only. Never reaches `sanitizeOriginalName`. |
| `detectFileType identifies JPEG by magic bytes` | 25 | Calls `detectFileType` only. |
| `detectFileType identifies PDF by magic bytes` | 30 | Calls `detectFileType` only. |
| `detectFileType rejects bytes that fake an executable under a png name` | 35 | Calls `detectFileType` only. |
| `validateUploadFile rejects empty files` | 40 | `validateUploadFile` does not call `sanitizeOriginalName`; returns on size check. |
| `validateUploadFile rejects oversize files` | 46 | Same - returns on size check before any name handling. |
| `validateUploadFile rejects unsupported content regardless of extension` | 53 | Rejects on magic-byte sniffing. Name is never sanitised. |
| `validateUploadFile accepts a valid PNG and returns sha256` | 59 | Returns on the detection path; name is never sanitised. |
| `sanitizeOriginalName strips directory traversal` | 70 | Calls `sanitizeOriginalName`, but `'../../etc/passwd.png'` reduces to `passwd.png`, which is 10 characters. Both a 200 and a 201 cap return the whole string unchanged, so the bug is invisible here. |
| `buildStorageKey is ownership-encoded and unique per call` | 79 | Calls `buildStorageKey` only, which uses `crypto.randomUUID()` and never touches `sanitizeOriginalName`. |

Two categories worth noting:

- **Eight tests do not call the function at all.** They are unaffected by definition.
- **One test does call the function and still misses the bug**
  (`strips directory traversal`). It exercises the basename branch, not the
  truncation branch, because its input is far shorter than either cap. This is the
  clearest demonstration that having a passing test for a function is not the same as
  having a test that pins its contract.

Coverage gap this exercise exposed: no test checks `sanitizeOriginalName` with an
input between 200 and 201 characters other than the long one, and no test asserts
that the returned name is capped when the basename contains characters requiring
replacement. A function can be well tested overall and still hold an unpinned
boundary.

---

## Revert performed

The revert was done by editing the single character back, not by `git checkout`.

Reason, verified during the run: the repository has `core.autocrlf = true` while the
working copy of this file uses LF line endings (0 CRLF, 88 bare LF). A
`git checkout -- <file>` would therefore have written the file back with CRLF endings,
changing every line in the file and producing a different byte hash, even though git
would have reported the tree as clean. Editing the character back guarantees a
byte-identical restoration.

Steps:

1. Changed `slice(0, 201)` back to `slice(0, 200)`.
2. Verified `git status --porcelain` reports 0 entries.
3. Verified the SHA256 and MD5 match the values recorded before the edit.
4. Re-ran the test suite.

Nothing was committed. No commit was created at any point.

---

## Final verification

| Check | Result |
| --- | --- |
| Original line restored | `  return cleaned.slice(0, 200);` at line 53 |
| `slice(0, 200)` occurrences | 1 |
| `slice(0, 201)` occurrences | 0 |
| SHA256 after revert | `A20DBC443A481F75496C346C3D73682C1E9789B72AA042CDCA096C0C73A86492` |
| SHA256 before the bug | `A20DBC443A481F75496C346C3D73682C1E9789B72AA042CDCA096C0C73A86492` - **match** |
| MD5 after revert | `EEA18EB978F34B9E0C387B17ED2E6AFF` |
| MD5 before the bug | `EEA18EB978F34B9E0C387B17ED2E6AFF` - **match** |
| Size after revert | 2906 bytes - matches original |
| Line endings after revert | 0 CRLF, 88 bare LF - matches original |
| `git status --porcelain` | 0 entries - clean |
| `git diff HEAD --stat` | empty - every tracked file byte-identical to HEAD |
| Tests after revert | `exit code: 0`, tests 11, pass 11, fail 0 |
| Typecheck after revert | `tsc --noEmit` exit code 0, no output |
| Commits created | none |

Both `sanitizeOriginalName` tests pass again after the revert:

```
sanitizeOriginalName strips directory traversal
sanitizeOriginalName truncates to 200 characters
```

### One file touched that is not production source

`tsconfig.tsbuildinfo` has a recent modification time because `tsc --noEmit` ran with
`"incremental": true` in `tsconfig.json`, so the compiler refreshed its build cache.
This file is not source code and is not tracked by git - `.gitignore` line 25 excludes
`*.tsbuildinfo`. No production file, test file, or configuration file was left
modified.

### Conclusion

A one-character off-by-one in a single line of a small, pure, well-tested helper
passed the type checker silently and broke exactly one of eleven tests. The bug was
found by a test that pins an exact boundary value, not by review, type checking, or
any static tool. The defect was reverted and the file restored byte-for-byte, with
the repository left clean and uncommitted.