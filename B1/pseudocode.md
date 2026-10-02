FUNCTION assertIndependentSignOff

INPUTS
- requestedBy: the id of the user who asked for the AI suggestion to be produced. This is the person who uploaded the document or requested the suggestion.
- signedOffBy: the id of the user who is trying to approve that AI suggestion. This is the person performing the sign-off.

OUTPUT
- A successful check returns nothing at all. The caller learns only that the function did not raise anything, which means the two users are different people and the sign-off may continue.

SIDE EFFECTS
- None. The function reads its two inputs and decides.
- It does not change either input, does not change the suggestion, and does not change any shared state.
- It does not perform the role check or the sign-off state check. Those remain the responsibility of the project's existing sign-off checks, and this function does not repeat or replace them.
- No database, network, filesystem, or other outside input or output.

FAILS WHEN
- The two ids are exactly the same string, meaning requestedBy and signedOffBy identify the same user. In that case the function stops by raising the project's existing AiError with code ERR_SIGN_OFF_FORBIDDEN, because a user must not approve their own AI suggestion.
- Two ids that are not exactly the same are not a failure, however they differ.
- No other situation raises anything. When the two ids are different, the check passes and the caller continues unchanged.

STEPS

1. Take the id of the user who requested the AI suggestion.

2. Take the id of the user who is signing it off.

3. Compare the two ids with each other as the supplied user ID strings.

4. If the two ids are exactly the same, stop by raising the project's existing AiError with code ERR_SIGN_OFF_FORBIDDEN, with a message explaining that a user may not sign off an AI suggestion they requested themselves.

5. Otherwise, do nothing further and finish successfully, because the two ids belong to different users, so there is nothing wrong with this sign-off.