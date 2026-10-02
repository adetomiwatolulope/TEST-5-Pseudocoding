# Task 5 — Part A2: Function 5 — prepareValue (pg 8.23.0)

> **AI-ASSISTED / AI-GENERATED EVIDENCE.**
> This document was produced with AI assistance. It is **not** a compliant no-AI first
> pass. The Task 5 brief requires the first A2 reading/pseudocode pass to be done without
> AI; this evidence does not satisfy that requirement and is labelled accordingly.

**Library and version:** `pg` 8.23.0
**Source file:** `Medlink-ai/node_modules/pg/lib/utils.js`

## Source references
- `prepareValue` — lines 44–70
- `prepareValueWrapper` (exported public entry) — lines 199–203
- `arrayString` — lines 16–38
- `prepareObject` — lines 72–83
- `dateToString` — lines 85–117
- `dateToStringUTC` — lines 119–142
- `escapeElement` — lines 7–11
- `defaults.parseInputDatesAsUTC` — `Medlink-ai/node_modules/pg/lib/defaults.js` line 61 (default `false`)

---

## Pseudocode

FUNCTION prepareValue

INPUTS:
- value: the value to convert. Any value is allowed: nothing at all (null or undefined), a
  number, a string, a boolean, a big whole number, a symbol, a function, a Buffer, a
  typed-array or data view, a date, an array (including nested arrays), or an object.
- seen (internal only): an optional list of objects already visited while resolving an
  object that converts itself. The public entry never supplies it.

OUTPUT:
- For nothing at all (null or undefined): nothing at all.
- For a Buffer: that same Buffer, unchanged.
- For a typed-array or data view: a new Buffer built from that view.
- For a date: a date-and-time text (local or universal, chosen by the project setting).
- For an array: an array-literal text.
- For an object that converts itself: the conversion of whatever its own conversion
  produces, which may then be converted further.
- For any other object: the object's JSON text.
- For any other value: that value's own text form.

SIDE EFFECTS:
- May create a new Buffer from a typed-array or data view.
- May add the object to the seen list while resolving an object that converts itself.
- Reads the project-wide setting that says whether dates are treated as UTC.
- Reads the host's local time-zone offset when a date is converted in local mode.
- Makes no change to the input value, and no database, network, filesystem, or shared-state
  change.

FAILS WHEN:
- An object that converts itself leads back to an object already recorded in the seen list;
  preparation stops by throwing a circular-reference error that names that object.
- A value reaches JSON serialization that cannot be serialized, such as an object holding a
  big whole number or a plain object that contains itself; that serialization error is not
  caught and stops preparation.

STEPS
1. (Entry.) The public conversion entry takes exactly one value and passes it to the
   internal conversion step with no seen list; extra arguments to the public entry are
   ignored.
2. Receive the value and the seen list, which may be absent.
3. If the value is nothing at all (null or undefined), stop and return nothing at all.
4. If the value is not an object, stop and return the value's own text form. (This covers
   numbers, strings, booleans, big whole numbers, symbols, and functions.)
5. The value is an object. If the value is a Buffer, stop and return that same value
   unchanged.
6. If the value is a typed-array or data view, stop and return a new Buffer built from that
   view's underlying buffer, its byte offset, and its byte length.
7. If the value passes the host's date test, stop and return a date text:
   - If the project-wide setting says dates are treated as UTC, return the result of the
     universal date text step on the value.
   - Otherwise, return the result of the local date text step on the value.
8. If the value is an array, stop and return the result of the array text step on the value.
9. Otherwise the value is some other object. Stop and return the result of the
   self-converting object step on the value and the seen list.

(No further step is reached in the same call once a step stops and returns.)

---

FUNCTION arrayString

INPUTS:
- value: an array.

OUTPUT:
- An array-literal text: an opening brace, each element's prepared text separated by commas,
  and a closing brace.

SIDE EFFECTS:
- May create new Buffer objects; calls back into the conversion step; makes no shared-state
  change.
- Does not use a seen list.

FAILS WHEN:
- An array contains itself, directly or through nested arrays, because the conversion step
  is called here without a seen list, so the recursion is unbounded.

STEPS
1. Receive the array.
2. Start the result text as an opening brace.
3. Walk the array's indexes from the first index up to one less than its length:
   a. If this is not the first index, add a comma to the result text.
   b. Take the element at the current index.
   c. If the element is null or undefined, add the word NULL to the result text.
   d. Otherwise, if the element is an array, add the result of the array text step on that
      element. (Recursion.)
   e. Otherwise, if the element is a typed-array or data view:
      - If the element is not a Buffer, replace it with a new Buffer built from its
        underlying buffer, its byte offset, and its byte length.
      - Add the array-binary marker (two backslashes followed by the letter x), then add the
        element's hexadecimal text.
   f. Otherwise, add the result of the element-escape step applied to the result of the
      conversion step on the element. (The conversion step here is called with no seen
      list.)
4. After the walk, add a closing brace to the result text.
5. Return the result text.

---

FUNCTION prepareObject

INPUTS:
- value: an object.
- seen: an optional list of objects already visited.

OUTPUT:
- Either the conversion of whatever the object's own conversion produces, or the object's
  JSON text.

SIDE EFFECTS:
- Creates the seen list when none was supplied, and records the object in the seen list.

FAILS WHEN:
- The object is already recorded in the seen list; throws a circular-reference error that
  names the object.
- The object does not convert itself and cannot be serialized as JSON; the serialization
  error stops preparation.

STEPS
1. Receive the object and the seen list.
2. If the object is present and has a callable self-conversion member:
   a. If no seen list was supplied, start a new empty seen list.
   b. If the object is already recorded in the seen list, stop and throw a
      circular-reference error that includes the object.
   c. Record the object in the seen list.
   d. Call the object's self-conversion member, passing the conversion step itself as its
      argument, and remember what it produces.
   e. Return the result of the conversion step on that produced value and the seen list.
      (Recursion.)
3. Otherwise, return the object serialized as JSON text.

---

FUNCTION escapeElement

INPUTS:
- text: the text to escape.

OUTPUT:
- The text with every backslash doubled and every double quote preceded by a backslash,
  wrapped in a leading and trailing double quote.

SIDE EFFECTS:
- None.

FAILS WHEN:
- Never on its own.

STEPS
1. Receive the text.
2. Replace every backslash with two backslashes.
3. Replace every double quote with a backslash followed by a double quote.
4. Put a double quote before and after the result.
5. Return the result.

---

FUNCTION dateToString

INPUTS:
- date: a date.

OUTPUT:
- The date-and-time text in local time, ending with the local time-zone offset text and an
  optional era suffix.

SIDE EFFECTS:
- Reads the host's local time-zone offset for the given date.

FAILS WHEN:
- Never on its own.

STEPS
1. Receive the date.
2. Set the offset to the negative of the host's time-zone offset, in minutes, for this date.
3. Read the date's local full year.
4. Decide whether the year is before year one.
5. If it is before year one, set the year to the absolute value of the year plus one.
6. Build the text as: four-digit padded year, a dash, two-digit padded local month plus one,
   a dash, two-digit padded local day, the letter T, two-digit padded local hours, a colon,
   two-digit padded local minutes, a colon, two-digit padded local seconds, a dot,
   three-digit padded local milliseconds.
7. If the offset is negative, add a minus sign and make the offset positive; otherwise add a
   plus sign.
8. Add two-digit padded whole hours of the offset, a colon, and two-digit padded remaining
   minutes.
9. If the year was before year one, add a space and the letters BC.
10. Return the text.

---

FUNCTION dateToStringUTC

INPUTS:
- date: a date.

OUTPUT:
- The date-and-time text in universal time, ending with the universal offset text and an
  optional era suffix.

SIDE EFFECTS:
- None.

FAILS WHEN:
- Never on its own.

STEPS
1. Receive the date.
2. Read the date's universal full year.
3. Decide whether the year is before year one.
4. If it is before year one, set the year to the absolute value of the year plus one.
5. Build the text as: four-digit padded year, a dash, two-digit padded universal month plus
   one, a dash, two-digit padded universal day, the letter T, two-digit padded universal
   hours, a colon, two-digit padded universal minutes, a colon, two-digit padded universal
   seconds, a dot, three-digit padded universal milliseconds.
6. Add the universal offset text of plus zero zero colon zero zero.
7. If the year was before year one, add a space and the letters BC.
8. Return the text.

---

FUNCTION prepareValueWrapper

INPUTS:
- value: one value.

OUTPUT:
- The result of the conversion step on that value.

SIDE EFFECTS:
- The same as the conversion step.

FAILS WHEN:
- The same as the conversion step.

STEPS
1. Receive exactly one value; ignore any additional arguments.
2. Return the result of the conversion step on that value, with no seen list.
