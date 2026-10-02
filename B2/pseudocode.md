FUNCTION assertNoUnsupportedConfidence

INPUTS
- confidence: the confidence score the provider reported. It is either a number, or no value at all when the provider gave no score.
- summary: the summary text the provider reported.

OUTPUT
- Nothing. When the check completes successfully it returns no value at all, and the caller learns only that nothing was raised.

SIDE EFFECTS
- None. The function reads its inputs and decides.
- It reads one outside value, which is the configured minimum length that a summary must reach. This is the only value it takes from the project's configuration, it is read only when a confidence score was supplied, and it is never changed.
- It does not change either input, does not change the configuration, and does not change any shared state.
- No database, network, or filesystem access, and no outside input beyond that one configured minimum.

FAILS WHEN
- A confidence score was supplied and the trimmed summary is strictly less than the configured minimum. In that case it stops by raising the project's existing AiError with code ERR_AI_FABRICATION_REJECTED, carrying the message: Provider output carries a confidence score without a substantive summary; rejecting as unsupported.
- A trimmed summary whose length is exactly equal to the configured minimum is not a failure. Only a length strictly below the minimum fails.
- When no confidence score was supplied, nothing is raised no matter how short the summary is, because a missing confidence score is not a confidence score.
- No other situation raises anything.

STEPS

1. Take the confidence score and the summary text supplied with the call.

2. If no confidence score was supplied, finish successfully there and stop, because a missing confidence score is not a confidence score.

3. Read the configured minimum length that a summary must reach.

4. Trim the summary text, removing whitespace from its start and removing whitespace from its end.

5. Count the characters remaining in the trimmed summary.

6. If that count is strictly less than the configured minimum, stop by raising the project's existing AiError with code ERR_AI_FABRICATION_REJECTED, carrying the message: Provider output carries a confidence score without a substantive summary; rejecting as unsupported.

7. Otherwise finish successfully without raising anything.