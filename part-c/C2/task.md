# Task 5 — Part C2: Task

## Requirement
Part C2 requires the AI implementation to be reverse-engineered into pseudocode using only
the AI code as the source, and then compared against the original B1 pseudocode. Every
meaningful difference must be classified as exactly one of:
- AI added unasked behaviour,
- AI omitted specified behaviour,
- AI interpreted an ambiguous step (where the original pseudocode is genuinely ambiguous
  and the AI made a reasonable interpretation).

Genuine ambiguities in the original specification must be documented, with proposed precise
wording that removes each ambiguity.

## Evidence required
- The reverse-engineered pseudocode of the AI implementation.
- A difference table listing every meaningful difference, each classified, with whether it
  changes observable behaviour and the correct action.
- The documented genuine ambiguities and the proposed wording fixes.

## Note on the files in this folder
- `reverse-pseudocode.md` is the C2 reverse-engineered pseudocode of the AI implementation.
- `difference-table.md` is the C2 comparison of the reverse pseudocode against the B1
  pseudocode, including the classifications and the ambiguities to fix.
