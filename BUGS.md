# Canonical Bug Registry

## Bug #1 — Travel / fast-travel awareness

- **Status:** `FIXED / PR OPEN`
- **First observed behavior:** NPCs did not reliably understand location changes as ordinary in-world travel.
- **Root cause:** Current location and meaningful travel transitions were not consistently promoted into active conversation context.
- **Fix / current approach:** Authoritative travel/location context with a stable prompt prefix was implemented; dynamic current state is supplied without rewriting the large static prompt.
- **Validation:** Live Whiterun → Solitude → Morthal testing passed; prompt-prefix reuse and latency behavior also passed.
- **Relevant work:** Mantella PR #746; branch `feature/travel-location-awareness`; commit `4f9debc1c7a6c254c079e3dd161ceaf835fe91a1`.
- **Notes:** Unsupported surrounding lore remains tracked separately as Bug #6.

## Bug #2 — Piper / TTS reliability and latency

- **Status:** `VALIDATING / LOCAL CHANGES ONLY`
- **First observed behavior:** Piper timeouts, process restarts, skipped voice lines, startup greeting problems, combat-related failures, and perceived startup latency.
- **Root cause:** Not yet established; multiple local diagnostics and experiments exist.
- **Fix / current approach:** Continuous stderr draining and bounded diagnostics were explored locally. The underlying reliability issue is not declared fixed.
- **Validation:** Requires further gameplay validation.
- **Relevant work:** Local-only Mantella changes; intentionally not pushed as a completed fix.
- **Notes:** Do not treat diagnostics experiments as a resolution.

## Bug #3 — Character bio leakage into long-term memory

- **Status:** `OPEN`
- **First observed behavior:** Character biography information can leak into generated summaries or long-term memories before it was revealed or learned during gameplay.
- **Root cause:** NPC knowledge and epistemic boundaries are not reliably enforced during summary/memory generation.
- **Fix / current approach:** Not yet implemented.
- **Validation:** None.
- **Relevant work:** None.
- **Notes:** This is an NPC knowledge-boundary problem.

## Bug #4 — Stale conversation lifecycle state swallowing the next conversation

- **Status:** `FIXED`
- **First observed behavior:** Ending one conversation could leave stale lifecycle/session-end state that terminated or swallowed the first utterance or generation of the next conversation.
- **Root cause:** Stale terminal lifecycle traffic could be applied to a newer active conversation.
- **Fix / current approach:** Session ownership, stale-request rejection, idempotent teardown, and coordinated participant-transition handling were implemented.
- **Validation:** Fixed and verified in live Skyrim gameplay.
- **Relevant work:** Mantella PR #745 and Mantella-Spell PR #151, with earlier lifecycle commits preserved in the development history.
- **Notes:** Newer participant-transition and in-flight-generation behavior is tracked separately as Bug #7.

## Bug #5 — Wrong speaker / multi-NPC speaker authority

- **Status:** `OPEN`
- **First observed behavior:** Multi-NPC conversations can mishandle which NPC is authorized or expected to speak, especially during participant changes and first-speaker selection.
- **Root cause:** Not yet established.
- **Fix / current approach:** Not yet implemented.
- **Validation:** None.
- **Relevant work:** None.
- **Notes:** Track separately from Bug #7 even where participant infrastructure overlaps.

## Bug #6 — Unsupported location/lore embellishment

- **Status:** `OPEN`
- **First observed behavior:** The model can receive the correct authoritative current location while inventing unsupported surrounding lore, political facts, motivations, or local context.
- **Root cause:** Grounding is incomplete around otherwise-correct authoritative context.
- **Fix / current approach:** Not yet implemented.
- **Validation:** None.
- **Relevant work:** None.
- **Notes:** Examples include incorrectly calling Solitude the capital of the Reach or inventing that the Jarl of Hjaalmarch requested the party's presence.

## Bug #7 — Participant transition / generation cancellation and actor identity collision

- **Status:** `FIX IMPLEMENTED / NEEDS GAMEPLAY VALIDATION`
- **First observed behavior:** Participant transitions could delay dialogue by approximately 6–7 seconds, cancel in-flight generation, produce zero-token responses, and collide when duplicate NPC display names were present.
- **Root cause:** Participant state was keyed by display name instead of stable actor identity. For example, `Stormcloak Soldier - 0799F9` and `Stormcloak Soldier - 0799F8` shared the same display name but were different actors.
- **Fix / current approach:**
  - `src/characters_manager.py`: active/all participant maps use `ref_id` with deterministic fallback; stable-ID lookup and ID participation logs were added.
  - `src/conversation/context.py`: transient NPC updates resolve by `ref_id`.
  - `src/remember/summaries.py`: summary interval tracking, filtering, and snapshot selection use stable identity; duplicate names receive distinct internal keys.
  - `src/conversation/conversation.py`: in-flight generations receive immutable participant snapshots; participant refreshes defer until generation completion; generation diagnostics were added.
- **Validation:** Focused automated identity/lifecycle tests pass; live gameplay validation remains required.
- **Relevant work:** Current local Mantella development branch; no new commit or PR yet.
- **Notes:** Preserve Bug #4 session protections and asynchronous Bug #7 summary behavior. Do not conflate this with Bug #5 speaker authority.

## Bug #8 — NPC verbally claims an action occurred when no action command executed

- **Status:** `OPEN`
- **First observed behavior:** An NPC can claim or imply that a state-changing action occurred even when no Mantella action command was emitted or successfully executed.
- **Root cause:** Dialogue is not yet strictly gated on authoritative action execution state.
- **Fix / current approach:** Not yet implemented. An NPC should not claim a state change until Mantella/Skyrim confirms it.
- **Validation:** None.
- **Relevant work:** None.
- **Notes:** Example: an NPC says “I am with you” after being asked to follow, but no `Follow:` action is emitted.
