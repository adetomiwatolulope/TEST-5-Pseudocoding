# Task 5 — Part A1: Function 3 — Trace vs Real Execution (mapDocumentsErrorToResponse)

**Function:** `mapDocumentsErrorToResponse`
**Source file:** `Medlink-ai/app/api/v1/patients/guard.ts` (lines 46–66)
**Pseudocode:** `part-a/A1/function-3-error-handler.md`
**Hand traces:** `part-a/A1/function-3-traces.md`
**Runtime:** Node v24.16.0; Next.js 14.2.18 (`NextResponse.json`).

**Execution method:** The real exported function was imported from the real
`guard.ts` and called with real `DocumentsError` and `StorageError` instances from the
project's own modules. Because Node's built-in type stripping cannot elide the
type-only `import { NextAuthOptions }` in `modules/identity/auth.ts`, the real
TypeScript sources were transpiled with the project's installed TypeScript
(`ts.transpileModule`, module = CommonJS) and resolved with a require hook for the `@/`
alias — the same transpile step Next/tsc performs. No mocks, stubs, fake results, or
source changes were used. The function was invoked with `error` only and returns a
`NextResponse`; the actual result was read back with `res.status` and `await res.json()`.

**Safety:** The function is a pure classifier/formatter. This run performed no database
writes, no authentication, and no external service calls. It writes only the same
`console.error` lines the function writes in production.

---

## TEST 1 — Normal input

**Input (exactly as documented):**
- `error`: a `DocumentsError`
  - code: `ERR_DOCUMENT_ACCESS_DENIED`
  - message: `"You are not allowed to access this document."`

**Expected behavior (from `function-3-traces.md`):**
- HTTP status 403.
- `success` is `false`.
- The original error message is returned to the client.
- Nothing is written to the server error log.

**Actual execution result:**
- status = `403`
- body = `{"success":false,"error":"You are not allowed to access this document."}`
- No log line was emitted for this case.

**Result: MATCH**

**Correction required:** None.

---

## TEST 2 — Edge input

**Input (exactly as documented):**
- `error`: a `StorageError`
  - code: `ERR_STORAGE_NOT_CONFIGURED`
  - message: `"Object storage is not configured."`

**Expected behavior (from `function-3-traces.md`):**
- HTTP status 503.
- `success` is `false`.
- The original error message is returned to the client.
- The storage error is written to the server error log before the response is returned.

**Actual execution result:**
- status = `503`
- body = `{"success":false,"error":"Object storage is not configured."}`
- Log line emitted (real, captured on stderr):
  `[patients:documents] storage error ERR_STORAGE_NOT_CONFIGURED Object storage is not configured.`

**Result: MATCH**

**Correction required:** None.

---

## TEST 3 — Invalid input

**Input (exactly as documented):**
- `error`: a plain string value, not an error object
  - value: `"something went wrong"`

**Expected behavior (from `function-3-traces.md`):**
- The fallback branch is used (not a documents error, storage error, or Prisma known request error).
- The value is written to the server error log as an unexpected error.
- HTTP status 500.
- `success` is `false`.
- The client receives the fixed message `"Internal error."`, and the passed-in string is not sent to the client.

**Actual execution result:**
- status = `500`
- body = `{"success":false,"error":"Internal error."}`
- Log line emitted (real, captured on stderr):
  `[patients:documents] unexpected error something went wrong`
- The passed-in string `"something went wrong"` does not appear in the response body.

**Result: MATCH**

**Correction required:** None.

---

## Summary

| Test case | Expected (hand trace) | Actual (real run) | Result |
| --- | --- | --- | --- |
| Normal | 403, `success:false`, original message, no log | 403, `success:false`, original message, no log | MATCH |
| Edge | 503, `success:false`, original message, storage log line | 503, `success:false`, original message, storage log line | MATCH |
| Invalid | 500, `success:false`, `"Internal error."`, unexpected log line | 500, `success:false`, `"Internal error."`, unexpected log line | MATCH |

## Correction required

None for the pseudocode or the hand traces. All three documented expectations are
factually correct against the real function.

## Conclusion

The real `mapDocumentsErrorToResponse` was executed against the exact three documented
inputs. All three actual results match the existing hand traces (3/3 MATCH). No
pseudocode or trace correction is needed.

Note (documentation only, not a trace error): `function-3-traces.md` still contains a
`REAL CODE VERIFICATION` section stating the real function was not executed and that
"no real execution result is being claimed." That statement is now outdated given this
run; it was not modified here per the instruction not to alter the traces unless a
genuine factual error required it. The real results are recorded in this comparison.

No source code was modified. `function-3-error-handler.md` and `function-3-traces.md`
were not modified. Functions 1 and 2 were not touched.
