# Card0 authoring: a tabletop with movable components

## Deliverable and division of responsibility

Build the physical game's component kit: decks, cards with appropriate fronts/backs, boards or reference cards, tokens, rules, artwork, and an initial setup recipe. Card0 is a shared tabletop, not a custom rules engine. Players may sit together, communicate outside the app, move cards between table and hands, consult a reference together, reveal or cover components, and judge turns, legal moves, scores and victory themselves. Preserve these normal human actions in the instructions. They are not deviations from the original rules and need no special approval.

Separate three layers before assessing feasibility:

1. **Authored components:** what must be printed on each component, quantities, backs, orientation and reference markings.
2. **Initial setup:** select the playable component pool and automate supported initial dealing/placement. State any ordinary player preparation that remains. A recipe describes Start/Restart, not the rest of the game.
3. **Human play:** communication, private consultation, moving/revealing/covering pieces, applying rules and deciding victory. Do not require server enforcement, native voice/chat, event handlers, or a per-game state machine unless explicitly requested.

Validate recipe structure and component supply. Inspect actual tabletop controls before asserting an operation is absent; absence from setup.md or the authoring CLI is not evidence about gameplay controls. Where automation cannot do a preparation step, first express it as ordinary tabletop instructions with the required components. Still provide a valid recipe and automate supported preparation; do not omit all setup or claim untested automation works.

Only raise a playability blocker when a necessary component or human action cannot be represented even using the tabletop interaction model. Explain the concrete missing operation and evidence. A missing automatic referee or strict server-side secrecy alone is not a blocker under Card0's normal trust-based play. If a user explicitly requests such enforcement, report that automation limitation separately from whether the physical game can be played.

## Example: Codenames / 行动代号

The word deck and the preprinted key-card deck are independent. A key card is a 5×5 map of **positions**, not a list of identities attached to particular words. Its top-left cell describes whichever word currently occupies the top-left table position. Any valid key works with any selection of 25 words; the key is not generated after dealing, and no enumeration of word combinations is needed.

Prepare a finite collection of varied key cards as in a physical box, with the appropriate team/civilian/assassin counts and a clear orientation/starting-team indicator. Independently choose a key and 25 words. The two spymasters consult the same key together or pass that same reference using ordinary player actions; never independently draw mismatching keys for them. Shared consultation need not be simultaneous server-enforced access.

Include agent, civilian and assassin cover markers/cards. Spymasters indicate the result by moving the appropriate cover onto the guessed word, just as at a physical table. Players give clues, enforce guessing rules and determine the winner. Missing automatic identity revelation or victory detection does not make this an approximate rules variant.

The recipe restriction on conditional correlated choices concerns actual data-dependent selection. Independent words plus a positional key do not require it. Validate the key's positional counts and readability, component supply, layout orientation, and the documented human procedure; keep unverified runtime claims explicit.
