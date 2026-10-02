# Task 5 — Part C2: Reverse-Engineered Pseudocode (AI implementation)

**Source of truth for this document:** `Medlink-ai/modules/patients/scheduling.rules.ai.ts` only.
**Provenance:** produced by reading that AI implementation and describing what it actually does. No other file was read or consulted.
**Scope:** describes observable behaviour of the code as written, including supporting helpers. It is not corrected or reconciled with any other specification.

---

## Pseudocode

```
FUNCTION hasSchedulingConflict

INPUTS
- proposed: an object with two fields, startTime and endTime.
- existingAppointments: an ordered collection of objects. Each object has
  startTime, endTime, and status. The collection is expected to support a count
  of its elements and to be walked in order.
- Each time field is expected to be a Date object.
- status is expected to be one of six named strings: PENDING_PAYMENT,
  CONFIRMED, CANCELLED, REFUNDED, IN_PROGRESS, or COMPLETED. Only three of
  those names are treated specially.
- A valid time value, as this code means it: a value that is an instance of
  Date and whose time value is not NaN. A missing value, a value that is not a
  Date, or a Date whose time is NaN is not a valid time value.

OUTPUT
- A single result object carrying a status field, which is exactly one of these
  three strings:
  - "invalid"
  - "no-conflict"
  - "conflict"
- When the status is "conflict", the result object also carries a
  conflictingAppointment field holding the existing appointment that was
  matched.
- When the status is "invalid" or "no-conflict", the result object does not
  carry a conflictingAppointment field at all.
- The result object is a fresh object on every return.

SIDE EFFECTS
- None. The function reads its inputs and returns a new result object.
- It does not modify proposed, does not modify existingAppointments, and does
  not modify any element of existingAppointments.
- No database, network, filesystem, or other external access.
- It processes existingAppointments in order and stops at the first result, so
  it may not examine every element.

SUPPORTING HELPERS (these decide the observable behaviour above)

- isRealTimeValue(value)
  - Returns true only when value is an instance of Date AND the numeric time
    value of that Date is not NaN.
  - Returns false for: a missing value, null, undefined, a number, a string, a
    plain object, and a Date whose time is NaN.
  - Note that a plain object shaped like a date, or a value that could be
    parsed into a date but is not itself a Date instance, returns false.

- isSameCalendarDate(a, b)
  - Returns true when a and b have the same local calendar year, the same local
    calendar month, and the same local calendar day.
  - It reads the year, month, and day through the host machine's local time
    zone.
  - It does not compare the time of day, only the calendar day parts.

- endsAfterStartOnOneDate(start, end)
  - Returns true only when isSameCalendarDate(start, end) is true AND the
    numeric time value of end is strictly greater than the numeric time value
    of start.
  - Returns false when the two values fall on different local calendar days,
    even if end is a later moment in absolute terms.
  - Returns false when end equals start, and when end is earlier than start.

FAILS WHEN
- It does not return an error object for bad time data. Bad time data is
  reported as a result whose status is "invalid".
- It does not defend against a missing container. If proposed is null or
  undefined, reading its startTime causes an exception. If existingAppointments
  is null or undefined, reading its element count causes an exception. If the
  collection contains a missing element, reading that element's status causes
  an exception. These cases throw rather than returning a result.
- A Date whose time is NaN does not throw; it is treated as invalid data and
  produces a result whose status is "invalid".
- A proposed appointment whose end is not later than its start, or whose times
  are not on the same local calendar date, produces "invalid".
- A proposed appointment crossing midnight (end on a later local calendar day)
  produces "invalid".
- An empty existingAppointments produces "no-conflict", but only after the
  proposed appointment checks have passed.
- An existing appointment whose status is exactly "CANCELLED", "REFUNDED", or
  "COMPLETED" is skipped before any time validation. Such an entry can never be
  returned as a conflict and can never make the result "invalid", no matter how
  its times are formed.
- Any status that is not exactly one of the three ignored names is treated as
  an active appointment, and its times are then validated. There is no check
  that the status is one of the six recognised names.
- The overlap test uses strict comparisons, so appointments that touch exactly
  at an endpoint are not a conflict.

STEPS

1. Take proposed and existingAppointments.

2. If proposed.startTime is not a valid time value, return a result whose status is "invalid" and stop.

3. If proposed.endTime is not a valid time value, return a result whose status is "invalid" and stop.

4. If endsAfterStartOnOneDate(proposed.startTime, proposed.endTime) is false, return a result whose status is "invalid" and stop.

5. If existingAppointments has no elements, return a result whose status is "no-conflict" and stop.

6. Begin a walk through existingAppointments, positioned on the first element.

7. If the walk has moved past the last element, go to step 15.

8. Take the element the walk is currently positioned on, calling it the current appointment.

9. If the current appointment's status is exactly "CANCELLED", or exactly "REFUNDED", or exactly "COMPLETED", move the walk on to the next element and go back to step 7. Do not check its start or end time.

10. If the current appointment's startTime is not a valid time value, return a result whose status is "invalid" and stop.

11. If the current appointment's endTime is not a valid time value, return a result whose status is "invalid" and stop.

12. If endsAfterStartOnOneDate(the current appointment's startTime, the current appointment's endTime) is false, return a result whose status is "invalid" and stop.

13. If the proposed startTime is strictly earlier than the current appointment's endTime, and the proposed endTime is strictly later than the current appointment's startTime, return a result whose status is "conflict" carrying the current appointment as its conflictingAppointment field, and stop.

14. Move the walk on to the next element and go back to step 7.

15. Return a result whose status is "no-conflict".
```

---

## Notes on what the code actually does

- The function has exactly one observable entry point, `hasSchedulingConflict`, plus three internal helpers that together determine the validity, same-day, and ordering behaviour.
- Every return is a plain object literal. The "invalid" and "no-conflict" returns contain only a status field; the "conflict" return contains a status field and a conflictingAppointment field.
- Validation is performed with strict equality and strict numeric comparison only. There is no parsing, no coercion, no rounding, and no normalisation of any time value.
- The local time zone of the running machine participates in the same-day decision, because the calendar-day comparison uses local getters.
