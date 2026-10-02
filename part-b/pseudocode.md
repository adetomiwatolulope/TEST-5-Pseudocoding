# Part B1 Specification: hasSchedulingConflict

**Proposed file/module:** `Medlink-ai/modules/patients/scheduling.rules.ts`

**Feature purpose:** Determine whether a proposed appointment conflicts with an existing doctor's appointment.

**Inputs:**
- proposed appointment: startTime, endTime
- existing appointments: startTime, endTime, status

**Status of this document:** specification only. No implementation has been written, no module has been created, and no project file has been modified.

---

## Pseudocode

```
FUNCTION hasSchedulingConflict

INPUTS
- proposed appointment: the time the proposed appointment is due to start,
  and the time it is due to end.
- existing appointments: a list held in a fixed order. Each entry carries the
  time that appointment starts, the time that appointment ends, and its status.
  A status is always one of these six names: PENDING_PAYMENT, CONFIRMED,
  CANCELLED, REFUNDED, IN_PROGRESS, or COMPLETED.
- A valid time value: a value that can be interpreted as a real calendar date
  together with a real time of day, so that it names a moment that genuinely
  exists. A value that is absent, that is not a time at all, or that names a
  moment which cannot exist, such as a date that never occurs or a time outside
  the day, is not a valid time value.
- The two times of an appointment belong to one calendar date. An appointment is
  not allowed to cross midnight, so an end time falling on a later date than its
  start time is not permitted.

OUTPUT
- A single result. It always carries a status, and it carries the conflicting
  appointment only when there is one.
- The status is always one of these three names:
  - invalid, meaning the time data cannot be trusted and no reliable answer can
    be given.
  - no-conflict, meaning no blocking existing appointment conflicts with the
    proposed appointment.
  - conflict, meaning the blocking appointment found earliest in the list has
    been identified.
- When the status is conflict, the result also carries that appointment, so the
  caller can see which booking clashed.
- When the status is no-conflict or invalid, the result carries no appointment.
- It never raises an error, and it never returns nothing. Exactly one of those
  three statuses is produced on every call.

SIDE EFFECTS
- None. It reads its inputs and returns an outcome.
- It does not change the proposed appointment, does not change any existing
  appointment, and does not change any stored state.
- No database, network, or filesystem access, and no outside input.
- It reads the list in order and stops as soon as it has an answer, so it may
  never look at every entry in the list.

FAILS WHEN
- It never stops by raising an error. Every situation produces one of the three
  outputs above.
- It returns a result whose status is invalid in exactly these situations:
  - the proposed start time is not a valid time value;
  - the proposed end time is not a valid time value;
  - the proposed end time is not later than the proposed start time on the same
    calendar date, which also covers an appointment that would cross midnight;
  - an existing appointment whose status is not CANCELLED, REFUNDED, or
    COMPLETED has a start time or an end time that is not a valid time value;
  - an existing appointment whose status is not CANCELLED, REFUNDED, or
    COMPLETED has an end time that is not later than its start time on the same
    calendar date.
- An existing appointment whose status is CANCELLED, REFUNDED, or COMPLETED is
  skipped completely. Its start and end times are never validated, and it can
  never be reported as a conflict. Invalid time data on an ignored appointment
  does not cause an invalid result.
- COMPLETED belongs in that ignored list deliberately, not by oversight. A
  completed appointment has already taken place and no longer holds its slot, so
  it must not stop that slot being booked again.
- It returns the first conflict it finds and stops at that point, so entries
  further along the list are never examined. Invalid time data held in a later
  active appointment is therefore only discovered when no earlier appointment
  caused an exit.
- Two appointments that meet exactly at an endpoint are not a conflict and
  never cause a failure. A proposed start time equal to an existing end time is
  not earlier than it, and a proposed end time equal to an existing start time
  is not later than it, so the overlap test is not satisfied.
- An empty list of existing appointments is not a failure. It produces a
  no-conflict result.
```

1. Take the proposed start time, the proposed end time, and the list of existing appointments.

2. If the proposed start time is not a valid time value, return a result whose status is invalid and stop.

3. If the proposed end time is not a valid time value, return a result whose status is invalid and stop.

4. If the proposed end time is not later than the proposed start time on the same calendar date, return a result whose status is invalid and stop.

5. If the list of existing appointments is empty, return a result whose status is no-conflict and stop.

6. Begin a walk through the list of existing appointments, positioned on the first entry.

7. If the walk has moved past the last entry in the list, go to step 15.

8. Take the existing appointment that the walk is currently positioned on.

9. If that appointment's status is CANCELLED, or REFUNDED, or COMPLETED, move the walk on to the next entry and go back to step 7. Do not check its start or end time.

10. If that appointment's start time is not a valid time value, return a result whose status is invalid and stop.

11. If that appointment's end time is not a valid time value, return a result whose status is invalid and stop.

12. If that appointment's end time is not later than its start time on the same calendar date, return a result whose status is invalid and stop.

13. If the proposed start time is earlier than that appointment's end time, and the proposed end time is later than that appointment's start time, return a result whose status is conflict, carrying that appointment, and stop.

14. Move the walk on to the next entry in the list and go back to step 7.

15. Return a result whose status is no-conflict.

---

## Hand traces

### Trace 1: Normal case, no conflict

**Input**
- Proposed: starts at 09:00, ends at 10:00
- Existing: one entry, starts at 11:00, ends at 12:00, status PENDING_PAYMENT

**Branches taken**

| Step | What happens | Branch |
|---|---|---|
| 1 | Inputs taken | continue |
| 2 | 09:00 is a valid time | continue |
| 3 | 10:00 is a valid time | continue |
| 4 | 10:00 is later than 09:00 on the same calendar date | continue |
| 5 | List holds one entry, not empty | continue |
| 6 | Walk positioned on the first entry | continue |
| 7 | Not past the last entry | continue |
| 8 | Take 11:00 to 12:00, PENDING_PAYMENT | continue |
| 9 | Status is not CANCELLED, REFUNDED, or COMPLETED | continue, so validate times |
| 10 | Start time 11:00 is valid | continue |
| 11 | End time 12:00 is valid | continue |
| 12 | 12:00 is later than 11:00 on the same calendar date | continue |
| 13 | 09:00 earlier than 12:00 is yes. 10:00 later than 11:00 is no | not a conflict |
| 14 | Move to next entry, and there is no next entry | go to step 7 |
| 7 | Walk has moved past the last entry | go to step 15 |
| 15 | Return | status is no-conflict |

**Final output:** a result whose status is no-conflict, carrying no appointment

**Why that is correct:** the proposed 09:00 to 10:00 sits entirely before the existing 11:00 to 12:00. The second half of the overlap test fails, so the two share no time and cannot clash.

---

### Trace 2: Normal case, genuine conflict

**Input**
- Proposed: starts at 10:00, ends at 11:00
- Existing, in list order:
  1. starts 09:30, ends 10:30, status CONFIRMED
  2. starts 10:45, ends 11:30, status CONFIRMED

**Branches taken**

| Step | What happens | Branch |
|---|---|---|
| 1 | Inputs taken | continue |
| 2 | 10:00 is a valid time | continue |
| 3 | 11:00 is a valid time | continue |
| 4 | 11:00 is later than 10:00 on the same calendar date | continue |
| 5 | List holds two entries, not empty | continue |
| 6 | Walk positioned on the first entry | continue |
| 7 | Not past the last entry | continue |
| 8 | Take 09:30 to 10:30, CONFIRMED | continue |
| 9 | Status is not CANCELLED, REFUNDED, or COMPLETED | continue, so validate times |
| 10 | Start time 09:30 is valid | continue |
| 11 | End time 10:30 is valid | continue |
| 12 | 10:30 is later than 09:30 on the same calendar date | continue |
| 13 | 10:00 earlier than 10:30 is yes. 11:00 later than 09:30 is yes | status is conflict, return result carrying entry 1 |

**Final output:** a result whose status is conflict, carrying the first entry, 09:30 to 10:30, CONFIRMED

**Why that is correct:** the two appointments share 10:00 to 10:30, so both halves of the overlap test hold. Entry 1 is examined first and matches, so it is returned immediately. Entry 2 also overlaps, but it is never examined. Returning the earliest match is the required behaviour, so this is correct rather than merely acceptable.

---

### Trace 3: Edge case, appointments touch at an endpoint

**Input**
- Proposed: starts at 09:00, ends at 10:00
- Existing: one entry, starts at 10:00, ends at 11:00, status CONFIRMED

**Branches taken**

| Step | What happens | Branch |
|---|---|---|
| 1 | Inputs taken | continue |
| 2 | 09:00 is a valid time | continue |
| 3 | 10:00 is a valid time | continue |
| 4 | 10:00 is later than 09:00 on the same calendar date | continue |
| 5 | List holds one entry, not empty | continue |
| 6 | Walk positioned on the first entry | continue |
| 7 | Not past the last entry | continue |
| 8 | Take 10:00 to 11:00, CONFIRMED | continue |
| 9 | Status is not CANCELLED, REFUNDED, or COMPLETED | continue, so validate times |
| 10 | Start time 10:00 is valid | continue |
| 11 | End time 11:00 is valid | continue |
| 12 | 11:00 is later than 10:00 on the same calendar date | continue |
| 13 | 09:00 earlier than 11:00 is yes. 10:00 later than 10:00 is no, because equal is not later | not a conflict |
| 14 | Move to next entry, and there is no next entry | go to step 7 |
| 7 | Walk has moved past the last entry | go to step 15 |
| 15 | Return | status is no-conflict |

**Final output:** a result whose status is no-conflict, carrying no appointment

**Why that is correct:** the proposed appointment ends exactly when the existing one starts. A doctor booked 09:00 to 10:00 is free from 10:00 onwards, so back-to-back bookings are allowed. The rule holds because "later than" is strict: 10:00 is not later than 10:00, so the overlap test never succeeds. This is the most likely point of disagreement between two implementations, because phrasing the comparison as "not earlier than", or using an inclusive boundary, would wrongly report a conflict.

---

### Trace 4: Ignored status with deliberately invalid times

**Input**
- Proposed: starts at 10:00, ends at 11:00
- Existing: one entry, starts at 10:30, ends at 09:00, status CANCELLED

The entry is deliberately malformed. It claims to end at 09:00 having started at 10:30, so its end time is not later than its start time.

**Branches taken**

| Step | What happens | Branch |
|---|---|---|
| 1 | Inputs taken | continue |
| 2 | 10:00 is a valid time | continue |
| 3 | 11:00 is a valid time | continue |
| 4 | 11:00 is later than 10:00 on the same calendar date | continue |
| 5 | List holds one entry, not empty | continue |
| 6 | Walk positioned on the first entry | continue |
| 7 | Not past the last entry | continue |
| 8 | Take 10:30 to 09:00, CANCELLED | continue |
| 9 | Status is CANCELLED, so the entry is skipped completely. Its start and end times are never examined, so no validation takes place | move to the next entry, go to step 7 |
| 7 | Walk has moved past the last entry | go to step 15 |
| 15 | Return | status is no-conflict |

**Final output:** a result whose status is no-conflict, carrying no appointment

**Why that is correct:** the entry is identified as CANCELLED at step 9 and skipped immediately, before steps 10, 11 and 12 are ever reached. Its reversed times are therefore never validated, and they do not cause an invalid result. Had the entry been reached at step 12 instead, the test would have found that 09:00 is not later than 10:30 and returned an invalid result. Because the status check comes first, that validation never happens and the malformed data is simply passed over.

Step 13 is never evaluated for this entry either, so the proposed 10:00 to 11:00 is never compared against the reversed times, and the entry can never be returned as a conflict. The walk then continues at step 7 to look for further appointments. The cancelled entry was the only entry in the list, so the walk moves past the last entry and falls through to step 15, returning no conflict. That is correct because there are no active appointments left that could block the proposed appointment.

The guarantee demonstrated here is stronger than "a cancelled appointment does not block". A cancelled appointment is never inspected at all, so it cannot produce a conflict, and it cannot produce an invalid result either, no matter how badly its times are formed.

---

### Trace 5: Awkward and invalid case

**Input**
- Proposed: starts at 09:00, ends at 10:00, which is valid
- Existing, in list order:
  1. starts 14:00, ends 15:00, status COMPLETED
  2. starts 08:00, ends 08:30, status REFUNDED
  3. starts 10:30, ends 09:00, status PENDING_PAYMENT, which ends before it starts

**Branches taken**

| Step | What happens | Branch |
|---|---|---|
| 1 | Inputs taken | continue |
| 2 | 09:00 is a valid time | continue |
| 3 | 10:00 is a valid time | continue |
| 4 | 10:00 is later than 09:00 on the same calendar date | continue |
| 5 | List holds three entries, not empty | continue |
| 6 | Walk positioned on the first entry | continue |
| 7 | Not past the last entry | continue |
| 8 | Take 14:00 to 15:00, COMPLETED | continue |
| 9 | Status is COMPLETED | skipped completely, times never checked, move to entry 2 |
| 7 | Not past the last entry | continue |
| 8 | Take 08:00 to 08:30, REFUNDED | continue |
| 9 | Status is REFUNDED | skipped completely, times never checked, move to entry 3 |
| 7 | Not past the last entry | continue |
| 8 | Take 10:30 to 09:00, PENDING_PAYMENT | continue |
| 9 | Status is not CANCELLED, REFUNDED, or COMPLETED | continue, so validate times |
| 10 | Start time 10:30 is a valid time value | continue |
| 11 | End time 09:00 is a valid time value | continue |
| 12 | 09:00 is not later than 10:30 on the same calendar date | status is invalid, stop |

**Final output:** a result whose status is invalid, carrying no appointment

**Why that is correct:** the third entry claims to end at 09:00 having started at 10:30, which is impossible. Such an entry cannot be compared meaningfully, and silently ignoring it could hide a double booking or wrongly permit one. Returning an invalid result surfaces the corrupt data instead of guessing. The walk stops here and never reports a conflict.

Worth noting: the two ignored entries are skipped without their times being examined at all, because step 9 runs before any validation. Only the third entry is validated. Had either ignored entry carried impossible times, this walk would still have continued past it and stopped on the third entry.

---

### Trace 6: First valid conflict returned before later invalid active data

**Input**
- Proposed: starts at 10:00, ends at 11:00
- Existing, in list order:
  1. starts 10:30, ends 11:30, status CONFIRMED. This entry conflicts with the proposed appointment.
  2. starts 14:00, ends 13:00, status CONFIRMED. This entry has invalid times, because it claims to end at 13:00 having started at 14:00.

**Branches taken**

| Step | What happens | Branch |
|---|---|---|
| 1 | Inputs taken | continue |
| 2 | 10:00 is a valid time | continue |
| 3 | 11:00 is a valid time | continue |
| 4 | 11:00 is later than 10:00 on the same calendar date | continue |
| 5 | List holds two entries, not empty | continue |
| 6 | Walk positioned on the first entry | continue |
| 7 | Not past the last entry | continue |
| 8 | Take 10:30 to 11:30, CONFIRMED | continue |
| 9 | Status is not CANCELLED, REFUNDED, or COMPLETED, so entry 1 is active | continue, so validate times |
| 10 | Start time 10:30 is a valid time value | continue |
| 11 | End time 11:30 is a valid time value | continue |
| 12 | 11:30 is later than 10:30 on the same calendar date, so entry 1 is valid | continue |
| 13 | 10:00 earlier than 11:30 is yes. 11:00 later than 10:30 is yes | status is conflict, return result carrying entry 1 and stop |

Entry 2 is never taken up. The walk does not return to step 7, so steps 8 to 13 are never performed for it.

**Final output:** a result whose status is conflict, carrying the first entry, 10:30 to 11:30, CONFIRMED

**Why that is correct:** entry 1 is active, because CONFIRMED is not one of the three ignored statuses, so its times are validated rather than skipped. Its times are valid, since 11:30 is later than 10:30. It then overlaps the proposed 10:00 to 11:00, because 10:00 is earlier than 11:30 and 11:00 is later than 10:30, so both halves of the overlap test hold and step 13 returns it immediately.

That return ends the walk. Entry 2 is never inspected, so its reversed times are never validated and the fact that 13:00 is not later than 14:00 is never discovered. Entry 2 therefore produces no invalid result. Had the walk continued to entry 2, step 12 would have returned an invalid result instead, so the two outcomes depend entirely on the early exit.

This confirms that the first valid conflict takes precedence over any invalid active data held later in the list. The function answers with the conflict it has already found rather than scanning the rest of the list for bad data, which is what the side effects rules describe, stopping as soon as it has an answer.

This trace locks down the early exit behaviour. Together with trace 5, which returns invalid because no earlier entry caused an exit, it fixes both halves of the ordering: an early conflict wins, and invalid active data is only discovered when the walk actually reaches it.

---

## Ambiguities found in the requirements

All four ambiguities raised in earlier revisions have now been resolved. The decisions are recorded here rather than left implicit, so that anyone implementing this can see what was chosen and why.

**Resolved: what counts as a valid time value.** Previously the phrase "is not a valid time value" appeared at steps 2, 3, 10 and 11 without ever being defined. The rule is now stated in the inputs: a value is valid when it can be interpreted as a real calendar date together with a real time of day, so that it names a moment that genuinely exists. A value that is absent, that is not a time at all, or that names a moment which cannot exist, is not valid. This gives the four validation steps something definite to test against.

**Resolved: the shape of the conflict result.** Previously the three outcomes were described in prose, and one of them returned a bare appointment while the other two returned a bare status, so a caller could not read them uniformly. The rule is now a single result that always carries a status of invalid, no-conflict, or conflict, and additionally carries the conflicting appointment only when the status is conflict. Every numbered step now states which status it returns.

**Resolved: whether COMPLETED appointments are ignored.** COMPLETED remains in the ignored list, and the rules section now says this is deliberate rather than an oversight: a completed appointment has already taken place and no longer holds its slot, so it must not stop that slot being booked again. No behaviour changed as a result of this, because COMPLETED was already skipped at step 9.

**Resolved: whether appointments may cross midnight.** Previously nothing forbade it, so an intended 22:00 to 02:00 booking would have been rejected at step 4 or step 12 as though its end merely appeared earlier, by accident rather than by rule. The inputs now state that both times of an appointment belong to one calendar date, and steps 4 and 12 test that the end is later than the start on that same date. An appointment crossing midnight is therefore explicitly invalid.

**No ambiguities remain open.** None of the four decisions changed the behaviour already fixed by traces 1 to 6, so no trace outcome was altered. The trace tables were reworded only where a branch label had to name the returned status or the same-date test.