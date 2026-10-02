FUNCTION safeParse — Hand Traces

TEST 1 — NORMAL INPUT

Expected behavior:
- A success result is returned, reporting success as true.
- The value the schema produced is carried in the result.
- No problem is thrown, and no validation error is built.

Input:
- data: an object holding one field:
  - email: "alice.smith@example.com"

Schema and parameters:
- schema: the forgot-password schema, which expects an object with one email field that must be valid.
- optional parameters: none supplied.

Hand trace:
- Pseudocode step 1: The schema, the value to check, and the optional settings are taken. The caller supplied no optional settings, so there are none to take.
- Pseudocode step 2: A fresh parsing context is created for this call, including a new empty list for collecting problems. The list starts empty.
- Pseudocode step 3: The asynchronous setting is set to false, because the caller supplied nothing.
- Pseudocode step 4: The custom error map is left absent, because the caller supplied nothing.
- Pseudocode step 5: The starting path is set to an empty list, because the caller supplied nothing.
- Pseudocode step 6: The context is recorded as having no parent. The value being checked is stored, and the error map the schema itself was built with is stored. That schema-bound error map is absent here, because this schema was declared without a custom message map.
- Pseudocode step 7: The value's parsed type is worked out. The value is an object, so its basic kind is object.
- Pseudocode step 8: Because the value is an object, it is tested in order and the search stops at the first match. It is not a list. It is not nothing at all. It has no then and catch members that can be called, so it is not a promise. It is not a keyed collection, not a collection of unique members, and not a date and time. It is none of those, so its type is object, meaning a plain object.
- Pseudocode step 9: Skipped, because the value's basic kind did match one of the kinds above, so there is no need for the fallback.
- Pseudocode step 10: The parsed type, plain object, is stored in the context alongside the value.
- Pseudocode step 11: The value, the starting path, and the context are handed to the schema's own checking step, and its outcome is waited for. The delegated step reports an outcome of valid. The trace takes that outcome as given, because step 11 does not describe what the delegated step does internally.
- Pseudocode step 12: The outcome did not come back as work that finishes later, so no error is thrown and processing carries on.
- Pseudocode step 13: The outcome taken is valid, which is one of the three kinds named at this step.
- Pseudocode step 14: The outcome reports itself as valid, so a success result is returned that reports success as true and carries the value the schema produced. That value is an object holding the email, unchanged from the input, because the email was already trimmed and already lowercase and therefore needed no converting. The trace ends here.
- Pseudocode step 15: Not reached, because the function already returned at step 14. The problem list is not checked.
- Pseudocode step 16: Not reached. No failure result is built.
- Pseudocode step 17: Not reached. No validation error is built, because no failure result was returned.
- Pseudocode step 18: Not reached. There is no remembered error to return.

Expected output:
- A result reporting success as true.
- The value carried is an object holding the email "alice.smith@example.com".
- Nothing is thrown, and the problem list is never turned into a validation error.

Why this output occurs:
- The data satisfies the schema's rules, so the delegated step reported a valid outcome.
- Step 14 is the first step that returns a success result, and it is reached as soon as the outcome reports itself as valid.
- Steps 15 to 18 all sit after that return, so a successful call never inspects the problem list and never builds a validation error. A success result contains no error to read, so the lazy creation described in steps 17 and 18 has nothing to act on.


TEST 2 — EDGE INPUT

Expected behavior:
- A success result is still returned, reporting success as true.
- The value the schema produced is different from the value that was passed in.
- The supplied starting path makes no difference to the outcome.
- No problem is thrown, and no validation error is built.

Input:
- data: an object holding one field:
  - email: "  Test@Example.COM  "

Schema and parameters:
- schema: the forgot-password schema, which expects an object with one email field that must be valid. Its email field is converted before it is checked, so surrounding spaces are removed and capital letters are lowered.
- optional parameters: supplied, with a starting path of two marks meaning the field name "request":
  - path: supplied, naming one level called "request"

Hand trace:
- Pseudocode step 1: The schema, the value to check, and the optional settings are taken. This time the caller did supply an optional setting, so the supplied path is taken as well.
- Pseudocode step 2: A fresh parsing context is created for this call, including a new empty list for collecting problems. The list starts empty.
- Pseudocode step 3: The asynchronous setting is set to false, because this setting was not supplied and falls back to false.
- Pseudocode step 4: The custom error map is left absent, because this setting was not supplied.
- Pseudocode step 5: The starting path is set to the supplied one, the single level called "request". The empty-list fallback is not used, because a path was supplied.
- Pseudocode step 6: The context is recorded as having no parent. The value being checked is stored, and the schema's own error map is stored, which is absent here.
- Pseudocode step 7: The value's parsed type is worked out. The value is an object, so its basic kind is object.
- Pseudocode step 8: Because the value is an object, it is tested in order and the search stops at the first match. It is not a list. It is not nothing at all. It has no then and catch members that can be called. It is not a keyed collection, not a collection of unique members, and not a date and time. So its type is object, a plain object. The padded email inside it does not change the type of the value being classified, because only the whole value is classified.
- Pseudocode step 9: Skipped, because the basic kind matched a listed kind.
- Pseudocode step 10: The parsed type, plain object, is stored in the context alongside the value.
- Pseudocode step 11: The value, the supplied starting path, and the context are handed to the schema's own checking step. That step converts the email by removing the surrounding spaces and lowering the capital letters, and the remaining checks then pass, so it reports an outcome of valid. The trace takes that outcome as given, because step 11 does not describe the delegated step internally.
- Pseudocode step 12: The outcome did not come back as work that finishes later, so no error is thrown.
- Pseudocode step 13: The outcome taken is valid.
- Pseudocode step 14: The outcome reports itself as valid, so a success result is returned reporting success as true and carrying the value the schema produced. That value is an object holding the email "test@example.com". The trace ends here.
- Pseudocode step 15: Not reached, because the function already returned at step 14.
- Pseudocode step 16: Not reached. No failure result is built.
- Pseudocode step 17: Not reached. No validation error is built.
- Pseudocode step 18: Not reached. There is no remembered error to return.

Expected output:
- A result reporting success as true.
- The value carried is an object holding the email "test@example.com".
- The input held "  Test@Example.COM  ", so the value that comes back is not the value that went in.
- Nothing is thrown.

Why this output occurs:
- Converting the email does not spoil validity in this library. Removing spaces and lowering letters rewrites the value without recording a problem, so the delegated step still reports valid and step 14 returns a success result.
- This is the important lesson of this case: a success result does not mean the value came back untouched. The value carried in the result is the schema's own output, so it can differ from the input whenever the schema converts a value, fills in a default, or drops a field.
- The supplied starting path changes nothing here. The path only decides where a recorded problem is reported from, and this call recorded no problems, so the supplied path has nothing to act on. A path would only become visible in the contents of a validation error, which is never built for a successful call.


TEST 3 — INVALID INPUT

Expected behavior:
- A failure result is returned, reporting success as false.
- The recorded problems are turned into a validation error, and the error is built only when it is read.
- Reading the error twice gives the same error.
- Nothing is thrown.

Input:
- data: an object holding these fields:
  - email: "not-an-email"
  - phoneNumber: "07700 900123"
  - password: "weak"
  - firstName: "Alice"
  - lastName: "Smith"
- No other fields are supplied. Every remaining field of this schema is optional, so leaving them out is allowed. The real run recorded no problem for any of them.

Schema and parameters:
- schema: the registration schema, which expects an object whose email field must be a valid email address and whose password field must be at least eight characters long and contain a lowercase letter, an uppercase letter, a number, and a special character.
- optional parameters: none supplied.

Hand trace:
- Pseudocode step 1: The schema, the value to check, and the optional settings are taken. The caller supplied no optional settings.
- Pseudocode step 2: A fresh parsing context is created for this call, including a new empty list for collecting problems. The list starts empty.
- Pseudocode step 3: The asynchronous setting is set to false, because the caller supplied nothing.
- Pseudocode step 4: The custom error map is left absent, because the caller supplied nothing.
- Pseudocode step 5: The starting path is set to an empty list, because the caller supplied nothing.
- Pseudocode step 6: The context is recorded as having no parent. The value being checked is stored, and the schema's own error map is stored.
- Pseudocode step 7: The value's parsed type is worked out. The value is an object, so its basic kind is object.
- Pseudocode step 8: Because the value is an object, it is tested in order and the search stops at the first match. It is not a list, not nothing at all, has no then and catch members that can be called, is not a keyed collection, is not a collection of unique members, and is not a date and time. So its type is object, a plain object.
- Pseudocode step 9: Skipped, because the basic kind matched a listed kind.
- Pseudocode step 10: The parsed type, plain object, is stored in the context alongside the value.
- Pseudocode step 11: The value, the starting path, and the context are handed to the schema's own checking step. That step finds that the email is not a valid email address and that the password is too short. It records problems into the problem list and reports an outcome that is not valid. It records no problem at all for the fields that were left out, because every one of them is optional. The trace takes that outcome as given, because step 11 does not describe the delegated step internally.
- Pseudocode step 12: The outcome did not come back as work that finishes later, so no error is thrown. Note that this step only guards against a later-finishing result, so it does not fire for a merely invalid result.
- Pseudocode step 13: The outcome taken is one of the three kinds named here, and it is not the valid kind, so the outcome is recorded here as not valid.
- Pseudocode step 14: The outcome does not report itself as valid, so no success result is returned. This is the first place the two kinds of outcome part ways.
- Pseudocode step 15: Because the outcome is not valid, the problem list is checked before anything is reported. The list is not empty, because the delegated step recorded problems. The empty-list branch that would throw is therefore not taken, and processing carries on to build the failure result. The trace does not count the problems individually, because counting them is the delegated step's work, which step 11 does not describe. Only one fact matters here, which is that the list is not empty.
- Pseudocode step 16: A failure result is returned that reports success as false and whose validation error has not been built yet. At this moment nothing has been built from the problem list.
- Pseudocode step 17: When the error is read for the first time, it is built from the recorded problems, remembered on the result, and returned. The problems come from the schema's own rules, so the messages are the ones the schema declares, namely that a valid email address is required and that the password must be at least eight characters long. Every one of the recorded problems comes from the email field or the password field, and none comes from a field that was left out.
- Pseudocode step 18: If the error is read again later, the remembered error is returned rather than a second one being built.

Expected output:
- A result reporting success as false.
- Nothing is thrown.
- Reading the error gives the recorded problems, including the messages the schema declares for the email and the password.
- Reading the error a second time gives the same error.

Why this output occurs:
- Step 15 is the pivot. It only throws when the outcome is not valid and the problem list is empty, which would mean there is nothing to hand back. Here problems were recorded, so the function reports a failure result instead of stopping.
- Because the failure result is returned before the error is built, the call itself is cheap. The problems are only turned into an error if the caller actually reads it, which is what steps 17 and 18 describe.
- Had the delegated step reported an outcome that was not valid while recording no problems at all, step 15 would have stopped and thrown instead. That is the one case where bad data produces a thrown error rather than a failure result, and it is not what happens in this trace.


REAL CODE VERIFICATION

- These hand traces were checked against real execution. The safeParse function was executed, and no real execution result is being assumed.
- Every expectation above comes from reading the pseudocode in function-4-safe-parse.md together with the real Zod source. Where a trace depends on what the delegated checking step reported, the trace says so and treats it as given.
- The real execution results are recorded in function-4-comparison.md, and all three cases match:
  - TEST 1 normal: success true; the returned data held the email "alice.smith@example.com" unchanged; no error property was present; 0 problems were recorded by the schema's checking step; MATCH.
  - TEST 2 edge: success true; the returned data held the email "test@example.com", which differs from the input "  Test@Example.COM  "; no error property was present; 0 problems were recorded; the supplied starting path had no visible effect; MATCH.
  - TEST 3 invalid: success false; the error was a ZodError with 5 problems (1 for the email field and 4 for the password field); reading the error a second time returned the same object; MATCH on the function's behaviour.
