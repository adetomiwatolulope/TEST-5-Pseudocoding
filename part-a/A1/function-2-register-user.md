FUNCTION registerUser

INPUTS
- input: the registration details
  - email: string — the email address, required
  - phoneNumber: string — the phone number, required
  - password: string — the plaintext password, required
  - firstName: string — required
  - middleName: optional string — may be absent
  - lastName: string — required
  - role: optional role — may be absent, in which case the patient role is used
  - dateOfBirth: optional string — may be absent
  - gender: optional string — may be absent
  - bloodGroup: optional string — may be absent
  - allergies: optional list of strings — may be absent
  - emergencyContact: optional string — may be absent
- opts: optional settings — may be absent entirely
  - idempotencyKey: optional string — may be absent, in which case the function does no replay checking and makes no reservation

OUTPUT
- A result object containing the user and a replayed flag.
  - The user is the newly created user, including its patient profile when one was created.
  - The replayed flag is true when the user was fetched from an earlier completed registration that used the same idempotency key.
  - The replayed flag is false when the user was created by this call.
- Failure cases throw an error rather than returning, for example when the same idempotency key is already in progress, or when the email or phone number is already registered.

SIDE EFFECTS
- Reads from the database: an existing registration request for the idempotency key, and existing user records
- Writes to the database: a registration request row reserved with PENDING status, later changed to COMPLETED or FAILED
- Writes to the database: a new user row
- Writes to the database: a new patient profile row, only when the effective role is PATIENT
- Hashes the supplied password with a cost factor of 12
- Opens and commits or abandons a database transaction at the SERIALIZABLE isolation level

FAILS WHEN
- An idempotency key is supplied and a registration request for that key is already PENDING, in which case a duplicate-request error is thrown
- The database rejects the new user because the email or phone number already exists, in which case a duplicate-user error is thrown
- The database rejects the new user because of some other unique constraint, in which case an ERR_DB_CONSTRAINT error is thrown
- The password cannot be hashed
- The database is unavailable, or a database call fails, in which case the failure surfaces to the caller. A failure while looking up the idempotency key in step 5, or while looking up the previously linked user in step 7, happens outside the error-handling steps and propagates directly and unchanged. A failure inside the transaction, or any other failure once the transaction has started, is handled by the error-handling steps and results in the original error being rethrown unchanged unless the error-handling steps replace it with the duplicate-user error or the ERR_DB_CONSTRAINT error.
- The transaction cannot be started, or is aborted by the database
- The reservation cannot be marked COMPLETED after the user is created
STEPS

1. Normalize the email address by converting it to lower case and trimming whitespace from both ends.

2. Normalize the phone number by trimming whitespace from both ends. The phone number is not case-converted.

3. Build the request fingerprint from the registration details, using the normalized email and normalized phone in place of the originals.

4. Take the fingerprint to be a SHA-256 hash of a fixed, ordered summary of the registration details, containing the normalized email, the normalized phone, the trimmed first name, the trimmed last name, the requested role or PATIENT when no role was requested, the date of birth or an empty value when none was supplied, and the gender or an empty value when none was supplied.

5. If an idempotency key was supplied, look up the registration request recorded against that key.

6. If no idempotency key was supplied, skip the replay check and reservation entirely and continue with the normal flow at step 10.

7. If the lookup found a registration request, decide what to do based on its status:
   - If the status is COMPLETED and it records a user:
     - Look up the recorded user by its stored identifier, including that user's patient profile.
     - If the user is found, stop and return the replayed result, containing that user and a replayed flag set to true.
   - If the status is PENDING, stop and throw the duplicate-request error, reporting that a registration with this idempotency key is already in progress.
   - If the status is FAILED, continue as a fresh registration request.
   - If the status is COMPLETED but it does not record a user, continue as a fresh registration request.

8. If the lookup found no registration request, continue as a fresh registration request.

9. If an idempotency key was supplied, reserve that key by creating a registration request that stores the key, stores the request fingerprint, and has status PENDING.

10. Note whether a reservation was made. If no idempotency key was supplied, there is no reservation to complete or cancel later.

11. Start a database transaction at the SERIALIZABLE isolation level.

12. Hash the supplied password using a cost factor of 12.

13. If hashing the password fails, let the error be handled by the error-handling steps below.

14. Determine the role to assign: use the requested role when one was supplied, otherwise use PATIENT.

15. Create the user record with the normalized email, the normalized phone, the password hash, the trimmed first name, the trimmed last name, the chosen role, and an active flag set to true.

16. Set the user's middle name to the trimmed middle name when one was supplied, and to no value when it was absent or blank.

17. If the chosen role is PATIENT, also create a patient profile linked to the new user:
   - Set the date of birth from the supplied value, or to the first of January 1990 when no date of birth was supplied.
   - Set the gender from the supplied value, or to Not Specified when no gender was supplied.
   - Set the blood group from the supplied value, or to no value when none was supplied.
   - Set the list of allergies from the supplied value, or to an empty list when none was supplied.
   - Set the emergency contact from the supplied value, or fall back to the normalized phone number when none was supplied.

18. If the chosen role is anything other than PATIENT, do not create a patient profile for the new user.

19. Finish the transaction, committing the new user and any patient profile. Returning the created user as part of the step.

20. If a reservation was made, complete it by setting its status to COMPLETED and recording the new user's identifier against it.

21. Return the result containing the newly created user, including its patient profile when one was created, and a replayed flag set to false.

22. If any step from starting the transaction up to returning the result fails, enter error handling:
   - If a reservation was made, mark that reservation as FAILED. If marking it as FAILED itself fails, ignore that failure and continue with the original error.
   - If the original error is a known database request error carrying the code P2002, meaning a unique constraint was violated:
     - Read the list of fields named by the violated constraint, treating the list as empty when the error names no fields.
     - If that list includes email or phoneNumber, stop and throw the duplicate-user error, reporting that a user with this email or phone number already exists.
     - If that list includes neither email nor phoneNumber, stop and throw ERR_DB_CONSTRAINT, reporting a unique constraint violation during registration.
   - Otherwise stop and rethrow the original error unchanged.