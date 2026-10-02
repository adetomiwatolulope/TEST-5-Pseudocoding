FUNCTION summarizeDocument — Hand Traces

TEST 1 — NORMAL INPUT

Expected behavior:
- The function completes successfully and returns the generated summary.
Input:
- documentId: valid document ID
- patientId: patient who owns the document
- requestedBy: valid requesting user ID
- ip: valid IP address
- AI response:
  - text contains a valid summary
  - confidence: 0.93
  - attempts: 1

Hand trace:
- Pseudocode step 1: Storage is configured → continue.
- Pseudocode step 2: Document is found for the patient → continue.
- Pseudocode step 3: Document exists, so no document access error is thrown → continue.
- Pseudocode step 4: AI prerequisites are ready → continue.
- Pseudocode step 5: Prerequisites are ready, so no blocked error is returned → continue.
- Pseudocode step 6: AI provider is selected and its configuration is loaded → continue.
- Pseudocode step 7: AI suggestion job is created with PENDING status.
- Pseudocode step 8: Document is read from storage → succeeds.
- Pseudocode step 9: Read succeeds, so no storage error is returned → continue.
- Pseudocode step 10: Document is sent to the AI provider with its file type and instructions → succeeds.
- Pseudocode step 11: Provider call succeeds, so no AI provider error is returned → continue.
- Pseudocode step 12: Attempts are recorded → attempts is 1.
- Pseudocode step 13: AI output text is valid → continue.
- Pseudocode step 14: Output is valid, so no invalid-output error is returned → continue.
- Pseudocode step 15: Output is normalized → succeeds.
- Pseudocode step 16: Normalized output passes the R11 safety check.
- Pseudocode step 17: Safety check passes, so no safety error is returned → continue.
- Pseudocode step 18: Audit log is created for the AI summarization request.
- Pseudocode step 19: Summary is encrypted → succeeds.
- Pseudocode step 20: Encryption succeeds, so no encryption error is returned → continue.
- Pseudocode step 21: Job is updated to COMPLETED, and the encrypted summary, confidence 0.93, provider and model, and attempts 1 are saved.
- Pseudocode step 22: Saved summary is decrypted.
- Pseudocode step 23: Successful result is returned with the suggestion ID, completed status, decrypted summary, confidence, provider, and model.


TEST 2 — EDGE INPUT

Expected behavior:
- The function completes successfully.
- Attempts defaults to 1.
- Confidence remains null.
Input:
- documentId: valid document ID
- patientId: patient who owns the document
- requestedBy: valid requesting user ID
- ip: valid IP address
- AI response:
  - text contains a valid summary
  - confidence: not provided
  - attempts: not provided

Hand trace:
- Pseudocode step 1: Storage is configured → continue.
- Pseudocode step 2: Document is found for the patient → continue.
- Pseudocode step 3: Document exists, so no document access error is thrown → continue.
- Pseudocode step 4: AI prerequisites are ready → continue.
- Pseudocode step 5: Prerequisites are ready, so no blocked error is returned → continue.
- Pseudocode step 6: AI provider is selected and its configuration is loaded → continue.
- Pseudocode step 7: AI suggestion job is created with PENDING status.
- Pseudocode step 8: Document is read from storage → succeeds.
- Pseudocode step 9: Read succeeds, so no storage error is returned → continue.
- Pseudocode step 10: Document is sent to the AI provider with its file type and instructions → succeeds.
- Pseudocode step 11: Provider call succeeds, so no AI provider error is returned → continue.
- Pseudocode step 12: No attempts value is reported → attempts defaults to 1.
- Pseudocode step 13: AI output text is valid → continue.
- Pseudocode step 14: Output is valid, so no invalid-output error is returned → continue.
- Pseudocode step 15: Output is normalized → succeeds.
- Pseudocode step 16: Normalized output passes the R11 safety check.
- Pseudocode step 17: Safety check passes, so no safety error is returned → continue.
- Pseudocode step 18: Audit log is created for the AI summarization request.
- Pseudocode step 19: Summary is encrypted → succeeds.
- Pseudocode step 20: Encryption succeeds, so no encryption error is returned → continue.
- Pseudocode step 21: Job is updated to COMPLETED, and the encrypted summary, provider and model, and attempts 1 are saved. No confidence was provided, so confidence is saved as null.
- Pseudocode step 22: Saved summary is decrypted.
- Pseudocode step 23: Successful result is returned with the suggestion ID, completed status, decrypted summary, null confidence, provider, and model.


TEST 3 — INVALID INPUT

Expected behavior:
- The function stops when the document is not found for the patient.
- The function throws the document access error.
- The AI provider is not called.
Input:
- documentId: document ID that does not belong to the patient
- patientId: valid patient ID
- requestedBy: valid requesting user ID
- ip: valid IP address
- AI response: not reached because the document lookup fails

Hand trace:
- Pseudocode step 1: Storage is configured → continue.
- Pseudocode step 2: Document is looked up using the document ID and patient ID → no matching document found.
- Pseudocode step 3: Document does not exist, so processing stops and a document access error is thrown.
- Pseudocode steps 4 to 23 are not reached. No AI suggestion job is created and the AI provider is not called.

REAL CODE VERIFICATION

An execution attempt was made by running scripts/evidence/01-jobs-table.mjs against the MedLink project.

Execution blocker:
- PostgreSQL was expected at localhost:5433.
- The configured database is "medlink".
- The "medlink" database does not exist on the available PostgreSQL instance.
- Therefore the script stopped before it reached AiService.summarizeDocument.

Test 1 — Normal input
- Execution status: Not completed.
- The existing evidence script could not reach summarizeDocument because the required MedLink database was unavailable.

Test 2 — Edge input
- Execution status: Not executed.
- The existing evidence script could not reach summarizeDocument because the required MedLink database was unavailable.

Test 3 — Invalid input
- Execution status: Not executed.
- The existing evidence script could not reach summarizeDocument because the required MedLink database was unavailable.


COMPARISON

| Test case | Hand-trace result | Real result | Match? | Mismatch/correction |
|---|---|---|---|---|
| Normal input | Function should complete successfully and return the generated summary | Not completed because database was unavailable | Cannot compare | Real execution was blocked |
| Edge input | Function should complete successfully; attempts defaults to 1 and confidence remains null | Not executed because database was unavailable | Cannot compare | Real execution was blocked |
| Invalid input | Function should stop at the document lookup and throw the document access error | Not executed because database was unavailable | Cannot compare | Real execution was blocked |