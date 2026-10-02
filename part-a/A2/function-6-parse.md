# Task 5 — Part A2: Function 6 — parse (path-to-regexp 6.3.0)

**AI-ASSISTED / AI-GENERATED — NOT A COMPLIANT NO-AI FIRST PASS**

> This document was produced with AI assistance. The Task 5 brief requires the A2 first
> reading/pseudocode pass to be done without AI. This evidence does not satisfy that
> requirement and must not be presented as a compliant no-AI first pass.

- **package/version:** path-to-regexp 6.3.0
- **source file:** `node_modules/path-to-regexp/dist/index.js`
- **function name:** `parse`
- **source line range:** 96–192

## External tokenizer contract (called by parse, not pseudocoded here)
`parse` first calls the file's tokenizer, `lexer`, passing the raw path text. The tokenizer
returns a list of tokens, and each token has three fields: a type, an index, and a value.
The token types it can produce are:
- CHAR — one literal character (value is that character).
- ESCAPED_CHAR — one character that was written after a backslash (value is that character).
- OPEN — an opening curly brace.
- CLOSE — a closing curly brace.
- NAME — a parameter name (value is the name text; the token index is the position of the
  leading colon).
- PATTERN — a parameter pattern (value is the pattern text between parentheses).
- MODIFIER — one of star, plus, or question mark.
- END — a sentinel appended after the last character (value is empty text; its index is the
  length of the input).

Also used by the default-pattern helper: the external `escapeString` helper returns a piece
of text with a backslash inserted before each regular-expression special character. Those
special characters are: dot, plus, asterisk, question mark, equals sign, caret, exclamation
mark, colon, dollar sign, opening curly brace, closing curly brace, opening parenthesis,
closing parenthesis, opening square bracket, closing square bracket, vertical bar, slash,
and backslash. A hash is not among them.

---

## Pseudocode

FUNCTION parse

INPUTS:
- pathText: the raw route path text to parse.
- options (optional): an object that may carry a "prefixes" entry (the set of characters
  allowed to sit immediately before a parameter) and a "delimiter" entry (the characters
  that separate path segments).

OUTPUT:
- A result list. Each entry is either a plain text string (a literal run of the path) or a
  parameter object. Each parameter object has:
  - name: the parameter name, or a generated number when the parameter was written with a
    pattern but no name, or empty text when neither name nor pattern was present.
  - prefix: text allowed to appear before the parameter.
  - suffix: text that appears after the parameter, taken from the open-brace form.
  - pattern: the regular-expression source text for the parameter.
  - modifier: the repeat/optional modifier text (star, plus, question mark), or empty text.

SIDE EFFECTS:
- Calls the external tokenizer on the path text.
- Reads the optional settings but does not change them.
- Builds and returns a new list; changes no input and no shared or external state.

FAILS WHEN:
- A required token is not the next token. This happens while requiring the closing brace of
  an open-brace form, or while requiring the end sentinel; a type error is thrown whose
  message names the unexpected token's type and index and the expected token type.
- Two parameters are adjacent with no text or prefix between them. The default-pattern
  helper throws a type error whose message names the earlier parameter.

STEPS
1. Start with an empty result list, a generated-name counter of zero, a token position of
   zero, and an empty pending text run.
2. Call the external tokenizer on the path text and store the token list it returns.
3. Set the allowed prefixes to the options' prefixes entry when present, otherwise to the
   dot character and the slash character.
4. Set the delimiters to the options' delimiter entry when present, otherwise to the slash
   character, the hash character, and the question-mark character.
5. While the token position is less than the number of tokens, perform the loop body in
   steps 6 through 30.
6. Take the pending character: if the token at the current position has type CHAR, remember
   its value and advance the position by one; otherwise the pending character is nothing.
7. Take the parameter name: if the token at the current position has type NAME, remember its
   value and advance the position by one; otherwise the parameter name is nothing.
8. Take the parameter pattern: if the token at the current position has type PATTERN,
   remember its value and advance the position by one; otherwise the parameter pattern is
   nothing.
9. Branch: if the parameter name is non-empty or the parameter pattern is non-empty, go to
   step 10; otherwise go to step 16.
10. Set the prefix candidate to the pending character when it is non-empty, otherwise to
    empty text.
11. Branch: if the prefix candidate is not one of the allowed prefixes, append the prefix
    candidate to the pending text run and set the prefix candidate to empty text.
12. Branch: if the pending text run is non-empty, append it to the result list as a plain
    text entry and set the pending text run to empty text.
13. Choose the first-branch parameter name: use the parameter name when it is non-empty;
    otherwise use the generated-name counter, and increase the counter by one after using it
    this way.
14. Choose the first-branch parameter pattern: use the parameter pattern when it is
    non-empty; otherwise use the result of the default-pattern helper applied to the prefix
    candidate.
15. Take the modifier: if the token at the current position has type MODIFIER, remember its
    value and advance the position by one; otherwise the modifier is empty text. Then append
    a parameter object to the result list built from the chosen name, the prefix candidate,
    an empty suffix, the chosen pattern, and the modifier, and continue to the next loop
    iteration (go back to step 5).
16. Take the literal value: use the pending character when it is non-empty; otherwise, if
    the token at the current position has type ESCAPED_CHAR, remember its value and advance
    the position by one; otherwise the literal value is nothing.
17. Branch: if the literal value is non-empty, append it to the pending text run and
    continue to the next loop iteration (go back to step 5).
18. Branch: if the pending text run is non-empty, append it to the result list as a plain
    text entry and set the pending text run to empty text.
19. Take the open value: if the token at the current position has type OPEN, remember its
    value and advance the position by one; otherwise the open value is nothing.
20. Branch: if the open value is non-empty, go to step 21; otherwise go to step 30.
21. Set a new prefix to the text gathered by the text-gathering helper. That helper
    repeatedly consumes CHAR or ESCAPED_CHAR tokens at the current position and concatenates
    their values, stopping when neither type is next.
22. Set a new name to the value of the next token when that token has type NAME and advance
    the position past it; otherwise the new name is empty text.
23. Set a new pattern to the value of the next token when that token has type PATTERN and
    advance the position past it; otherwise the new pattern is empty text.
24. Set a new suffix to the text gathered by the text-gathering helper.
25. Require and consume a CLOSE token using the required-token helper, which advances the
    position past it and throws when the next token is not CLOSE.
26. Choose the open-brace parameter name: use the new name when it is non-empty; otherwise,
    when the new pattern is non-empty, use the generated-name counter and increase the
    counter by one after using it this way; otherwise use empty text.
27. Choose the open-brace parameter pattern: when the new name is non-empty and the new
    pattern is empty, use the result of the default-pattern helper applied to the new
    prefix; otherwise use the new pattern.
28. Take the modifier: if the token at the current position has type MODIFIER, remember its
    value and advance the position by one; otherwise the modifier is empty text.
29. Append a parameter object to the result list built from the chosen open-brace name, the
    new prefix, the new suffix, the chosen open-brace pattern, and the modifier, and
    continue to the next loop iteration (go back to step 5).
30. (Reached when the open value is nothing.) Require and consume an END token using the
    required-token helper.
31. After the loop ends, return the result list.

---

FUNCTION tryConsume (local helper)

INPUTS:
- a wanted token type.

OUTPUT:
- The value of the token at the current position when that token has the wanted type;
  otherwise nothing.

SIDE EFFECTS:
- Advances the token position by one when it takes a token.

FAILS WHEN:
- Never.

STEPS
1. Branch: if the token position is less than the number of tokens and the token at the
   current position has the wanted type, remember that token's value and advance the
   position by one, then return the remembered value.
2. Otherwise return nothing.

---

FUNCTION mustConsume (local helper)

INPUTS:
- a wanted token type.

OUTPUT:
- The value of the consumed token when the next token has the wanted type; otherwise it does
  not return.

SIDE EFFECTS:
- Advances the token position by one when it takes a token.

FAILS WHEN:
- The next token does not have the wanted type; a type error is thrown.

STEPS
1. Ask the take-if-present helper for the wanted type and remember what it returns.
2. Branch: if what it returned is not nothing, return it.
3. Otherwise, read the token at the current position to learn its type and its index.
4. Throw a type error whose message states that an unexpected token of that type occurred at
   that index and that the wanted type was expected.

---

FUNCTION consumeText (local helper)

INPUTS:
- none (it reads the token stream and the current position).

OUTPUT:
- The concatenated values of the consecutive CHAR and ESCAPED_CHAR tokens that start at the
  current position.

SIDE EFFECTS:
- Advances the token position past every token it consumes.

FAILS WHEN:
- Never.

STEPS
1. Start the gathered text as empty text.
2. Set the next value to nothing.
3. Repeat the following: take the next value as the value of the next CHAR token when one is
   present; if that is nothing, take it as the value of the next ESCAPED_CHAR token when one
   is present.
4. Branch: if the next value is empty or nothing, stop repeating.
5. Append the next value to the gathered text and repeat from step 3.
6. When the repetition stops, return the gathered text.

---

FUNCTION isSafe (local helper)

INPUTS:
- a piece of text.

OUTPUT:
- True when the text contains at least one delimiter character; otherwise false.

SIDE EFFECTS:
- None.

FAILS WHEN:
- Never.

STEPS
1. For each delimiter character, do the following: if the text contains that character,
   return true.
2. If no delimiter character was found, return false.

---

FUNCTION safePattern (local helper)

INPUTS:
- a prefix text.

OUTPUT:
- A regular-expression source text to use as a parameter's pattern.

SIDE EFFECTS:
- Reads the last entry of the result list.

FAILS WHEN:
- There is a last entry in the result list, but the resolved preceding text is empty; a type
  error is thrown whose message states that there must be text between two parameters and
  that the text after the earlier parameter (named by that last entry) is missing.

STEPS
1. Take the previous entry as the last entry of the result list; it may be nothing.
2. Set the preceding text to the prefix when the prefix is non-empty; otherwise to the
   previous entry when the previous entry is a plain text string; otherwise to empty text.
3. Branch: if the previous entry is present and the preceding text is empty, throw the type
   error described above.
4. Branch: if the preceding text is empty, or the preceding text contains at least one
   delimiter character, return the default pattern: an opening square bracket, a caret, the
   delimiter text with each regular-expression special character escaped by a backslash, a
   closing square bracket, and the two characters plus and question mark.
5. Otherwise return the bounded pattern: an opening parenthesis, a non-capturing group
   opener, a negative lookahead, the preceding text with each regular-expression special
   character escaped by a backslash, a closing parenthesis for the lookahead, an opening
   square bracket, a caret, the delimiter text with each regular-expression special
   character escaped by a backslash, a closing square bracket, a closing parenthesis, and
   the two characters plus and question mark.
