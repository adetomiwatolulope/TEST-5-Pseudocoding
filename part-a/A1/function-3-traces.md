FUNCTION mapDocumentsErrorToResponse — Hand Traces

TEST 1 — NORMAL INPUT

Expected behavior:
- HTTP status 403.
- The success field is false.
- The original error message is returned to the client.
- Nothing is written to the server error log.

Input:
- error: a documents error
  - code: ERR_DOCUMENT_ACCESS_DENIED
  - message: "You are not allowed to access this document."

Hand trace:
- Pseudocode step 1: The thrown value is taken as the error to be classified. It is a documents error carrying the code ERR_DOCUMENT_ACCESS_DENIED and the message "You are not allowed to access this document." The value is not changed.
- Pseudocode step 2: The value is a documents error, so the branch is entered. The status is chosen from the code. The code is ERR_DOCUMENT_ACCESS_DENIED, so the status is 403. The option that would give 404 for ERR_PATIENT_NOT_FOUND is not taken, and the fallback option that would give 400 for any other code is not taken. A response is returned whose body sets the success field to false and sets the error text to the documents error's own message, "You are not allowed to access this document.", sent with status 403. Processing stops. The trace terminates here at step 2.
- Pseudocode step 3: Not reached, because processing stopped at the return inside step 2. The storage error check is not performed, and nothing is written to the server error log.
- Pseudocode step 4: Not reached, because processing stopped at step 2. The Prisma check is not performed.
- Pseudocode step 5: Not reached, because processing stopped at step 2. The value never reaches the unexpected error branch, so it is not logged as unexpected.
- Pseudocode step 6: This is a summary of the whole function rather than a runtime action, so nothing new happens here. It agrees with this trace: the one branch that ran was step 2, and it set the success field to false, so there is no success response.


TEST 2 — EDGE INPUT

Expected behavior:
- HTTP status 503.
- The success field is false.
- The original error message is returned to the client.
- The storage error is written to the server error log before the response is returned.

Input:
- error: a storage error
  - code: ERR_STORAGE_NOT_CONFIGURED
  - message: "Object storage is not configured."

Hand trace:
- Pseudocode step 1: The thrown value is taken as the error to be classified. It is a storage error carrying the code ERR_STORAGE_NOT_CONFIGURED and the message "Object storage is not configured." The value is not changed.
- Pseudocode step 2: The value is checked to see whether it is a documents error. It is not a documents error, so this branch is not entered. No status is chosen here, no response is returned, and nothing is logged.
- Pseudocode step 3: The value is checked to see whether it is a storage error. It is a storage error, so the branch is entered. The status is chosen from the code. The code is ERR_STORAGE_NOT_CONFIGURED, so the status is 503. The fallback option that would give 500 for any other storage code is not taken. One line is written to the server error log, reporting it as a storage error for the patients documents feature and including the code ERR_STORAGE_NOT_CONFIGURED and the message "Object storage is not configured." This logging happens before the response is returned. A response is then returned whose body sets the success field to false and sets the error text to the storage error's own message, "Object storage is not configured.", sent with status 503. Processing stops. The trace terminates here at step 3.
- Pseudocode step 4: Not reached, because processing stopped at the return inside step 3. The Prisma check is not performed.
- Pseudocode step 5: Not reached, because processing stopped at step 3. The value is not logged a second time as an unexpected error.
- Pseudocode step 6: This is a summary of the whole function rather than a runtime action, so nothing new happens here. It agrees with this trace: the one branch that ran was step 3, it logged once, and it set the success field to false, so there is no success response.


TEST 3 — INVALID INPUT

Expected behavior:
- The fallback branch is used, because a plain string is not a documents error, a storage error, or a Prisma known request error.
- The value is written to the server error log as an unexpected error.
- HTTP status 500.
- The success field is false.
- The client receives the fixed message "Internal error.", and the string that was passed in is not sent to the client.

Input:
- error: a plain string value, not an error object
  - value: "something went wrong"

Hand trace:
- Pseudocode step 1: The thrown value is taken as the error to be classified. It is the plain string "something went wrong". It is not an error object, has no code, and has no message, and it is not changed.
- Pseudocode step 2: The value is checked to see whether it is a documents error. A plain string is not one, so this branch is not entered. No status is chosen here, no response is returned, and nothing is logged.
- Pseudocode step 3: The value is checked to see whether it is a storage error. A plain string is not one, so this branch is not entered. No status is chosen, nothing is logged, and no response is returned.
- Pseudocode step 4: The value is checked to see whether it is a known Prisma request error. A plain string is not one, so this branch is not entered. Nothing is logged and no response is returned.
- Pseudocode step 5: The value is not a known Prisma request error either, so it is treated as an unexpected error. It falls in this branch because none of the earlier checks matched it, and it is a value that is not an error at all. One line is written to the server error log, reporting it as an unexpected error for the patients documents feature and including the value itself, which is the string "something went wrong". A response is then returned whose body sets the success field to false and sets the error text to the fixed message "Internal error.", sent with status 500. The string that was passed in is not sent to the client. Processing stops. The trace terminates here at step 5.
- Pseudocode step 6: This is a summary of the whole function rather than a runtime action, so nothing new happens here. It agrees with this trace: the one branch that ran was step 5, it logged once, and it set the success field to false, so there is no success response.


REAL CODE VERIFICATION

- The real function was executed using the exact three traced inputs, and all three real results matched the hand traces above (3/3).
- The hand-trace expectations above were derived from the pseudocode, and the real execution results are recorded in function-3-comparison.md.
