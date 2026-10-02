FUNCTION mapDocumentsErrorToResponse

INPUTS
- error: unknown — one thrown value passed in to be classified. Its type is not assumed in advance. It may be a documents error, a storage error, a known Prisma request error, or any other value.

OUTPUT
- A single NextResponse holding a JSON body and an HTTP status. The body always reports failure, with the success field set to false and an error message alongside it. The status and message depend on the kind of value that was passed in:
  - A documents error whose code is ERR_DOCUMENT_ACCESS_DENIED: status 403, with the error's own message as the error text.
  - A documents error whose code is ERR_PATIENT_NOT_FOUND: status 404, with the error's own message as the error text.
  - Any other documents error: status 400, with the error's own message as the error text.
  - A storage error whose code is ERR_STORAGE_NOT_CONFIGURED: status 503, with the error's own message as the error text.
  - Any other storage error: status 500, with the error's own message as the error text.
  - A known Prisma request error: status 500, with the fixed error text "Internal error."
  - Any other or unexpected value: status 500, with the fixed error text "Internal error."
- There is no success response. No input produces a body with the success field set to true.

SIDE EFFECTS
- Writes one line to the server error log when the value is a storage error, recording it as a storage error for the patients documents feature and including the storage error's code and message.
- Writes one line to the server error log when the value is a known Prisma request error, recording it as a database error for the patients documents feature and including that error's code and message.
- Writes one line to the server error log when the value is neither of those, recording it as an unexpected error for the patients documents feature and including the value itself.
- Writes nothing to the server error log when the value is a documents error.

FAILS WHEN
- The function has no failure path of its own. Nothing in the implementation raises an error, and there is no internal call that can raise one, so the function always returns a response rather than throwing.
- The function also has no controlled failure indicator, no error code returned to the caller, and no path that reports the outcome as anything other than failure. The HTTP status and the returned message are the only things that describe the outcome, and the body always reports failure.
- The function is a pure classifier plus formatter with respect to the value it is given. It does not retry, rethrow, wrap, or replace the value it is given, and it does not recover the original call.

STEPS

1. Take the single thrown value that was passed in as the error to be classified. Do not assume its type, and do not change it.

2. Check whether the error is a documents error. If it is:
   - Choose the response status from the error's code:
     - If the code is ERR_DOCUMENT_ACCESS_DENIED, choose 403.
     - Otherwise, if the code is ERR_PATIENT_NOT_FOUND, choose 404.
     - Otherwise, choose 400.
   - Return a response whose body sets the success field to false and sets the error text to the documents error's own message, sent with the status chosen in this step.
   - Stop. Do not log anything, and do not check for storage or Prisma errors.

3. If the error is not a documents error, check whether it is a storage error. If it is:
   - Choose the response status from the error's code:
     - If the code is ERR_STORAGE_NOT_CONFIGURED, choose 503.
     - Otherwise, choose 500.
   - Write one line to the server error log, reporting it as a storage error for the patients documents feature, including the storage error's code and its message.
   - Return a response whose body sets the success field to false and sets the error text to the storage error's own message, sent with the status chosen in this step.
   - Stop. Do not check for Prisma errors.

4. If the error is neither a documents error nor a storage error, check whether it is a known Prisma request error, meaning a database request failure that Prisma recognizes. If it is:
   - Write one line to the server error log, reporting it as a database error for the patients documents feature, including the Prisma error's code and its message.
   - Return a response whose body sets the success field to false and sets the error text to the fixed message "Internal error.", sent with status 500.
   - Stop. The Prisma error's own message is not sent to the client.

5. If the error is not a known Prisma request error either, treat it as an unexpected error. This covers every value that the earlier checks did not match, including values that are not errors at all.
   - Write one line to the server error log, reporting it as an unexpected error for the patients documents feature, including the value itself.
   - Return a response whose body sets the success field to false and sets the error text to the fixed message "Internal error.", sent with status 500.
   - Stop.

6. Confirm the outcome across the whole function: the four branches above are the only branches, each one returns, and every one of them sets the success field to false. No branch sets it to true, so there is no success response.
