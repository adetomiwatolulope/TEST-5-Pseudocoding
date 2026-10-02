FUNCTION summarizeDocument

INPUTS
- documentId: string — identifies the patient's document
- patientId: string — identifies the patient
- requestedBy: string — identifies who requested the summary
- ip: string — IP address used for audit logging

OUTPUT
- AiSuggestionResult — returned on successful execution, containing the suggestion ID, completed status, decrypted summary, confidence, provider, and model.
- Some failure cases throw an error instead of returning, for example when the document does not exist for the patient.
- Other failure cases are controlled returned failures, for example when AI summarization is not configured.

SIDE EFFECTS
- Reads the patient's document from storage
- Creates and updates an AI suggestion job
- Sends the document to an AI provider
- Creates audit log records
- Encrypts and decrypts the generated summary

FAILS WHEN
- Storage is not configured
- Document does not exist for the patient
- AI is not configured
- Document cannot be read
- AI provider fails
- AI returns invalid output
- Safety validation fails
- Encryption fails
- Database operation fails
STEPS

1. Check that object storage is configured.

2. Find the document using the document ID and patient ID.

3. If the document does not exist, stop and throw a document access error.

4. Check whether the AI summarization prerequisites are ready.

5. If the AI prerequisites are not ready:
   - Create an audit log showing that summarization was blocked.
   - Return a controlled error saying AI summarization is not configured.

6. Select the AI provider and load its configuration.

7. Create an AI suggestion job with PENDING status.

8. Try to read the document from storage.

9. If reading the document fails:
   - Mark the job as FAILED.
   - Create an audit log showing the summarization failed.
   - Stop and return a storage error.

10. Send the document to the AI provider together with its file type and summarization instructions.

11. If the AI provider fails:
   - Mark the job as FAILED.
   - Create an audit log showing the summarization failed.
   - Stop and return an AI provider error.

12. Record the number of attempts reported by the AI provider.

13. Validate the AI provider's returned text.

14. If the AI output is invalid:
   - Mark the job as FAILED.
   - Create an audit log showing the summarization failed.
   - Stop and return an invalid-output error.

15. Normalize the validated AI output.

16. Run the normalized output through the R11 safety check.

17. If the safety check fails:
   - Mark the job as FAILED.
   - Create an audit log showing the summarization failed.
   - Stop and return the safety error.

18. Create an audit log showing that an AI summarization request was made.

19. Encrypt the generated summary.

20. If encryption fails:
   - Mark the job as FAILED.
   - Create an audit log showing the summarization failed.
   - Stop and return an encryption error.

21. Update the AI suggestion job:
   - Set its status to COMPLETED.
   - Save the encrypted summary.
   - Save the confidence score.
   - Save the provider and model.
   - Save the number of attempts.

22. Decrypt the saved summary.

23. Return a successful result containing the suggestion ID, completed status, decrypted summary, confidence, provider, and model.