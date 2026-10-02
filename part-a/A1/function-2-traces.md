FUNCTION registerUser — Hand Traces

TEST 1 — NORMAL INPUT

Expected behavior:
- A PENDING reservation is created.
- A new user is created.
- A patient profile is created.
- The reservation becomes COMPLETED.
- The function returns the new user with replayed = false.
Input:
- email: "Alice.Smith@Example.COM"
- phoneNumber: "  +44 7700 900123  "
- password: "Str0ngPassphrase!"
- firstName: "Alice"
- middleName: not supplied
- lastName: "Smith"
- role: PATIENT
- dateOfBirth: "1992-04-17"
- gender: "Female"
- bloodGroup: "O+"
- allergies: ["Penicillin"]
- emergencyContact: "07700 900456"
- opts:
  - idempotencyKey: "reg-key-0001"
- existing registration request for "reg-key-0001": none

Hand trace:
- Pseudocode step 1: The email is converted to lower case and trimmed, giving "alice.smith@example.com".
- Pseudocode step 2: The phone number is trimmed only, giving "+44 7700 900123". It is not case-converted.
- Pseudocode step 3: The request fingerprint is built using "alice.smith@example.com" and "+44 7700 900123" in place of the original email and phone number.
- Pseudocode step 4: The fingerprint is a SHA-256 hash over the ordered summary: email "alice.smith@example.com", phone "+44 7700 900123", first name "Alice", last name "Smith", role PATIENT, date of birth "1992-04-17", gender "Female".
- Pseudocode step 5: An idempotency key "reg-key-0001" was supplied, so the registration request recorded against that key is looked up.
- Pseudocode step 6: Skipped, because an idempotency key was supplied.
- Pseudocode step 7: Skipped as a decision point, because the lookup found no registration request.
- Pseudocode step 8: The lookup found no registration request, so the function continues as a fresh registration request.
- Pseudocode step 9: An idempotency key was supplied, so the key is reserved by creating a registration request that stores the key "reg-key-0001", stores the request fingerprint, and has status PENDING.
- Pseudocode step 10: A reservation was made, so it is remembered for later completion or cancellation.
- Pseudocode step 11: A database transaction is started at the SERIALIZABLE isolation level.
- Pseudocode step 12: The supplied password is hashed using a cost factor of 12.
- Pseudocode step 13: Skipped, because hashing the password did not fail.
- Pseudocode step 14: A role was requested and it was PATIENT, so PATIENT is used as the role to assign.
- Pseudocode step 15: The user record is created with email "alice.smith@example.com", phone number "+44 7700 900123", the password hash, trimmed first name "Alice", trimmed last name "Smith", role PATIENT, and an active flag set to true.
- Pseudocode step 16: No middle name was supplied, so the middle name is set to no value.
- Pseudocode step 17: The chosen role is PATIENT, so a patient profile is created with date of birth "1992-04-17", gender "Female", blood group "O+", allergies containing "Penicillin", and emergency contact "07700 900456".
- Pseudocode step 18: Skipped, because the chosen role is PATIENT, so this branch is not taken.
- Pseudocode step 19: The transaction is finished, committing the new user and the new patient profile, and yielding the created user.
- Pseudocode step 20: A reservation was made, so it is completed by setting its status to COMPLETED and recording the new user's identifier against it.
- Pseudocode step 21: The function returns the newly created user, including its patient profile, and a replayed flag set to false.
- Pseudocode step 22: Not entered, because no error occurred. Every step from starting the transaction at step 11 up to returning the result at step 21 succeeded, so the condition guarding the error-handling block is not met and the error-handling steps are skipped entirely. In particular the reservation is not marked FAILED, no duplicate-user error is raised, no ERR_DB_CONSTRAINT error is raised, and the original error is not rethrown. The function therefore proceeds to the successful return at step 21.


TEST 2 — EDGE INPUT

Expected behavior:
- No idempotency reservation is created.
- The SERIALIZABLE transaction still starts.
- The password is hashed.
- PATIENT is used as the role.
- A patient profile is created using the documented default values.
- The function returns the new user with replayed = false.
Input:
- email: "  Bob.Jones@Example.com  "
- phoneNumber: "02079460001"
- password: "An0therPass!"
- firstName: "Bob"
- middleName: not supplied
- lastName: "Jones"
- role: not supplied
- dateOfBirth: not supplied
- gender: not supplied
- bloodGroup: not supplied
- allergies: not supplied
- emergencyContact: not supplied
- opts: supplied, with no idempotency key inside it
- existing registration request: not applicable, because no key is used

Hand trace:
- Pseudocode step 1: The email is converted to lower case and trimmed, giving "bob.jones@example.com".
- Pseudocode step 2: The phone number is trimmed only, giving "02079460001".
- Pseudocode step 3: The request fingerprint is built using "bob.jones@example.com" and "02079460001" in place of the original email and phone number.
- Pseudocode step 4: The fingerprint is a SHA-256 hash over the ordered summary: email "bob.jones@example.com", phone "02079460001", first name "Bob", last name "Jones", role PATIENT because no role was requested, an empty value for date of birth because none was supplied, and an empty value for gender because none was supplied.
- Pseudocode step 5: Skipped, because no idempotency key was supplied.
- Pseudocode step 6: No idempotency key was supplied, so the replay check and the reservation are skipped entirely and the function continues with the normal flow at step 10.
- Pseudocode step 7: Skipped, because no lookup was performed.
- Pseudocode step 8: Skipped, because no lookup was performed.
- Pseudocode step 9: Skipped, because no idempotency key was supplied, so no reservation is created.
- Pseudocode step 10: No reservation was made, so there is nothing to complete or cancel later.
- Pseudocode step 11: A database transaction is still started at the SERIALIZABLE isolation level.
- Pseudocode step 12: The supplied password is still hashed using a cost factor of 12.
- Pseudocode step 13: Skipped, because hashing the password did not fail.
- Pseudocode step 14: No role was requested, so PATIENT is used as the role to assign.
- Pseudocode step 15: The user record is created with email "bob.jones@example.com", phone number "02079460001", the password hash, trimmed first name "Bob", trimmed last name "Jones", role PATIENT, and an active flag set to true.
- Pseudocode step 16: No middle name was supplied, so the middle name is set to no value.
- Pseudocode step 17: The chosen role is PATIENT, so a patient profile is created using the documented defaults: date of birth set to the first of January 1990 because none was supplied, gender set to Not Specified because none was supplied, blood group set to no value because none was supplied, allergies set to an empty list because none was supplied, and emergency contact falling back to the normalized phone number "02079460001" because none was supplied.
- Pseudocode step 18: Skipped, because the chosen role is PATIENT, so this branch is not taken.
- Pseudocode step 19: The transaction is finished, committing the new user and the new patient profile, and yielding the created user.
- Pseudocode step 20: Skipped, because no reservation was made, so there is no reservation to complete.
- Pseudocode step 21: The function returns the newly created user, including its patient profile, and a replayed flag set to false.
- Pseudocode step 22: Not reached, because no step failed.


TEST 3 — INVALID INPUT

Expected behavior:
- The existing request is detected.
- The function stops before creating a new reservation.
- No user is created.
- The function throws the duplicate-request error.
Input:
- email: "carol.dana@example.com"
- phoneNumber: "01179460002"
- password: "ValidPass123!"
- firstName: "Carol"
- middleName: not supplied
- lastName: "Dana"
- role: PATIENT
- dateOfBirth: "1988-12-02"
- gender: "Female"
- bloodGroup: not supplied
- allergies: not supplied
- emergencyContact: not supplied
- opts:
  - idempotencyKey: "reg-key-0002"
- existing registration request for "reg-key-0002":
  - status: PENDING

Hand trace:
- Pseudocode step 1: The email is converted to lower case and trimmed, giving "carol.dana@example.com".
- Pseudocode step 2: The phone number is trimmed only, giving "01179460002".
- Pseudocode step 3: The request fingerprint is built using "carol.dana@example.com" and "01179460002" in place of the original email and phone number.
- Pseudocode step 4: The fingerprint is a SHA-256 hash over the ordered summary: email "carol.dana@example.com", phone "01179460002", first name "Carol", last name "Dana", role PATIENT, date of birth "1988-12-02", gender "Female".
- Pseudocode step 5: An idempotency key "reg-key-0002" was supplied, so the registration request recorded against that key is looked up, and the lookup finds an existing registration request whose status is PENDING.
- Pseudocode step 6: Skipped, because an idempotency key was supplied.
- Pseudocode step 7: The lookup found a registration request, so the status decides the outcome. The status is PENDING, so processing stops and the duplicate-request error is thrown, reporting that a registration with this idempotency key is already in progress. The COMPLETED branch that looks up a previously linked user, the FAILED branch that continues as a fresh request, and the COMPLETED branch that continues as a fresh request are all not taken.
- Pseudocode step 8: Not reached, because processing already stopped at step 7.
- Pseudocode step 9: Not reached. No new reservation is created, because processing stopped before the reservation step.
- Pseudocode step 10: Not reached.
- Pseudocode step 11: Not reached. No SERIALIZABLE transaction is started.
- Pseudocode step 12: Not reached. The password is not hashed.
- Pseudocode step 13: Not reached.
- Pseudocode step 14: Not reached, so no role is chosen.
- Pseudocode step 15: Not reached. No user is created.
- Pseudocode step 16: Not reached.
- Pseudocode step 17: Not reached. No patient profile is created.
- Pseudocode step 18: Not reached.
- Pseudocode step 19: Not reached, so nothing is committed.
- Pseudocode step 20: Not reached.
- Pseudocode step 21: Not reached, so no result is returned.
- Pseudocode step 22: Not reached. The duplicate-request error thrown at step 7 is raised before the transaction starts, so the error-handling steps never apply, and the existing PENDING registration request is left unchanged.

REAL CODE VERIFICATION

- Real execution has not been attempted. This file contains hand traces only.

## Real-code execution status

- Real function: UserService.registerUser
- Real execution attempted: NO — execution was blocked before registerUser could run.
- PostgreSQL server at localhost:5433: reachable.
- DATABASE_URL targets the medlink_authorization database.
- PostgreSQL authentication requires SCRAM password authentication.
- The application's DATABASE_URL contains no password.
- PGPASSWORD is not available.
- No usable PostgreSQL password credential was found.
- Prisma therefore cannot authenticate to the database.
- Because the first database operation in registerUser is prisma.registrationRequest.findUnique, the function cannot be reached until database authentication is fixed.
- No real normal, edge, or invalid execution result is being claimed.
- The three hand-traced expected behaviors remain the evidence for the pseudocode comparison until the environment blocker is resolved.
