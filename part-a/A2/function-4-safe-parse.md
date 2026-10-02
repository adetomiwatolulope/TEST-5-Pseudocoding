Task 5 — Part A2: Function 4 — ZodType.safeParse (zod 3.25.76)

- package: zod 3.25.76
- source file: node_modules/zod/v3/types.js
- function: ZodType.safeParse
- relevant source range: safeParse L116–131
- relevant helper ranges: handleResult L26–45; _parseSync promise guard L99–105

FUNCTION safeParse

INPUTS
- schema: the validation schema this check belongs to. It holds the rules and decides the outcome, so the same value can be accepted under one schema and rejected under another.
- data: the value to check. Any value is allowed, including no value at all.
- optional parameters: extra settings for this single call. Every one of them is optional, and each has a fallback when the caller supplies nothing:
  - async setting: whether the check is allowed to wait for work that finishes later. Defaults to false.
  - error map: a custom way of turning a recorded problem into a readable message. Remains absent when it is not supplied.
  - path: where inside the data the checking starts. Defaults to an empty list.

OUTPUT
- A success result, reporting success as true and carrying the value the schema produced. That value is the schema's own output, so it can differ from the value that was passed in whenever the schema converts a value, fills in a default, or drops an extra field.
- OR a failure result, reporting success as false, whose validation error is built from the recorded problems only when it is first read.
- It may also stop by throwing, in two situations: when the schema's checking step hands back work that finishes later during a synchronous check, and when the check reports failure but recorded no problems at all.

SIDE EFFECTS
- Creates a fresh parsing context for this call, including a new empty list for collecting problems.
- The schema's own checking step fills that problem list while it runs.
- May defer creation of the validation error until the error is read.
- Does not change the schema itself, and does not change any shared state.
- No database, network, filesystem, or other outside input or output.

FAILS WHEN
- The schema's checking step returns work that finishes later while a synchronous check is being performed, in which case it stops by throwing instead of waiting.
- The check reports failure but recorded no problems, in which case it stops by throwing because there is nothing to hand back to the caller.
- No other situation throws. Data that is simply rejected is reported as a failure result, not as a throw.

STEPS

1. Take the schema that this check belongs to, the value to check, and any optional settings supplied with the call.

2. Create a fresh parsing context for this call, including a new empty list for collecting problems.

3. Set the asynchronous setting in the context to the supplied value, or to false when the caller supplied nothing.

4. Set the custom error map in the context to the supplied one, or leave it absent when the caller supplied nothing.

5. Set the starting path in the context to the supplied one, or to an empty list when the caller supplied nothing.

6. Record that the context has no parent, store the value being checked, and store the error map that the schema itself was built with.

7. Work out the value's parsed type, so the schema's checking step knows what kind of value it is being handed:
   - If there is no value at all, the type is undefined.
   - If the value is text, the type is string.
   - If the value is a number, the type is number, unless it is the special value that means not-a-number, in which case the type is nan.
   - If the value is true or false, the type is boolean.
   - If the value is something that can be called, the type is function.
   - If the value is a whole number too large for ordinary arithmetic, the type is bigint.
   - If the value is a unique marker that is not interchangeable with any other, the type is symbol.

8. If the value is an object of any kind, test it in this order and stop at the first kind that matches:
   - If it is a list, the type is array.
   - If it is nothing at all, the type is null.
   - If it has both a then member and a catch member and both of them can be called, the type is promise, meaning it stands for work that is not finished yet.
   - If it is a keyed collection of values, the type is map.
   - If it is a collection of unique members, the type is set.
   - If it is a date and time, the type is date.
   - If it is none of the above, the type is object, meaning a plain object.

9. If the value's basic kind matched none of the kinds above, the type is unknown.

10. Store the parsed type in the context alongside the value.

11. Hand the value, the starting path, and the context to the schema's own checking step, and wait for the outcome it reports. What that step does with the value is its own business and is not described here.

12. If the outcome came back as work that finishes later, stop and throw an error saying that a synchronous parse encountered a promise.

13. Otherwise take the outcome that the schema reported, which is one of three kinds: valid, partly valid, or aborted.

14. If the outcome reports itself as valid, return a success result that reports success as true and carries the value the schema produced.

15. Otherwise, before reporting a failure, check whether any problems were recorded:
    - If the problem list is empty, stop and throw an error saying that validation failed but no issues were detected.
    - If problems were recorded, carry on to build the failure result.

16. Return a failure result that reports success as false and whose validation error has not been built yet.

17. When that error is read for the first time, build it from the recorded problems, remember it, and return it.

18. If the error is read again later, return the remembered error rather than building a second one.
