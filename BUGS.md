# Canonical Bug Registry

## Current validated baseline

- Dedicated Summary LLM routing remains working, including strict dedicated routing with no fallback to the dialogue LLM.
- Non-blocking background summary generation remains working.
- Travel/location awareness remains working.
- Stale conversation/session-end lifecycle protections remain working.
- Piper/TTS synthesis timeouts and restarts remain open and separate from the participant fixes.
- Unsupported local-lore embellishment remains open as Bug #6.

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
- **Fix / current approach:** Continuous stderr draining and bounded diagnostics were explored locally. The latest stderr-draining experiment did not fix the underlying reliability issue.
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

- **Status:** `FIX IMPLEMENTED / PARTIAL LIVE VALIDATION`
- **First observed behavior:** Participant transitions could delay dialogue by approximately 6–7 seconds, cancel in-flight generation, produce zero-token responses, and collide when duplicate NPC display names were present.
- **Root cause:** Participant state was keyed by display name instead of stable actor identity. For example, `Stormcloak Soldier - 0799F9` and `Stormcloak Soldier - 0799F8` shared the same display name but were different actors.
- **Fix / current approach:**
  - `src/characters_manager.py`: active/all participant maps use `ref_id` with deterministic fallback; stable-ID lookup and ID participation logs were added.
  - `src/conversation/context.py`: transient NPC updates resolve by `ref_id`.
  - `src/remember/summaries.py`: summary interval tracking, filtering, and snapshot selection use stable identity; duplicate names receive distinct internal keys.
  - `src/conversation/conversation.py`: in-flight generations receive immutable participant snapshots; participant refreshes defer until generation completion; generation diagnostics were added.
- **Validation:** Focused lifecycle/identity tests, broader lifecycle/character tests, same-name summary regression, compileall, and diff checks passed. Live Skyrim validation confirmed same-name actors are distinguished, separate summary paths are used, first responses are no longer swallowed, and asynchronous summaries do not cancel dialogue.
- **Relevant work:** Mantella PR #745, commits `e3a8be37b8819727b35596f3ae92278e8e1a4802` and `560a9d494dd03795fe2ae47724dad83db18b69be`; Mantella-Spell PR #151, commit `15be8a66abfe7f0f335fcd0722378b76107965cb`.
- **Notes:** Same-name participant separation, summary targeting, and first-response preservation are live validated. Cold participant-transition latency remains open: participant changes can produce `static_prefix_changed=True` and roughly 4–8 second first responses, while warm unchanged-prefix turns are typically 1–2 seconds.

### Bug #7a — Same-name physical speaker routing

- **Status:** `FIXED`
- **First observed behavior:** After female `Stormcloak Soldier` A (`0799F9`) leaves and male `Stormcloak Soldier` B (`0799F8`) joins, Mantella generates B's dialogue and selects B's male voice, but the audio physically originates from A's body.
- **Root cause:** The generated speaker identity was reduced to a display name before the Skyrim handoff, so Papyrus selected the first matching participant.
- **Fix / current approach:** `mantella_actor_refid` now crosses the protocol boundary and Papyrus resolves the exact actor by FormID first, retaining name fallback only for legacy/unique cases.
- **Validation:** Live Skyrim validation passed with female and male same-name Stormcloak Soldiers. The correct male voice, physical actor, lip-sync, and first response were preserved; the departing actor did not speak.
- **Relevant work:** Mantella PR #745 commit `560a9d494dd03795fe2ae47724dad83db18b69be`; Mantella-Spell PR #151 commit `15be8a66abfe7f0f335fcd0722378b76107965cb`.
- **Notes:** This is distinct from Piper voice-model selection and from Bug #5's general speaker authority. Generic-NPC playback fallback is tracked below and is now validated.

### Bug #7b — Generic NPC voice playback after stable speaker routing

- **Status:** `FIXED`
- **First observed behavior:** A generic `Wood Elf` (`000EFD`) can generate dialogue and successfully synthesize the correct voice, yet produce no audible in-game playback after stable actor-ref routing was deployed.
- **Root cause:** Stable FormID lookup was exclusive. When the normalized generic-NPC reference could not resolve in Papyrus, `NpcSpeak()` was skipped and successful synthesis produced no audible playback.
- **Fix / current approach:** Stable FormID remains authoritative; unresolved IDs use a unique-name fallback, while ambiguous duplicate-name fallback is refused.
- **Validation:** Live Skyrim validation passed for Wood Elf `000EFD`; the actor spoke again. Same-name Stormcloak routing continued to pass.
- **Relevant work:** Mantella-Spell PR #151 commit `15be8a66abfe7f0f335fcd0722378b76107965cb`.
- **Notes:** Keep separate from general Piper timeouts/restarts and Bug #5 speaker authority.

## Bug #8 — Action-authority / action-prefix compliance failure

- **Status:** `IMPLEMENTED / CORE LIVE VALIDATED / FINAL REGRESSION CHECKS PENDING`
- **First observed behavior:** An NPC can verbally comply with an explicitly requested action without emitting the required executable action prefix, or can emit/re-trigger an action based on prior conversation history when the current player turn does not authorize it.
- **Root cause:** Non-verification-dependent explicit actions such as Follow were allowed to queue ordinary dialogue immediately, even when the current-turn action obligation had not produced an accepted action. This let prose such as “Lead on” imply that Follow had succeeded while no Follow invocation was emitted. Historical/stale action state was also required to remain isolated from the immutable current-turn authorization context.
- **Fix / current approach:** The action-authority path now holds dialogue for explicit current-turn state-changing obligations until the corresponding current-turn action is accepted. Existing immutable turn/actor authorization, stale-generation rejection, action-only dispatch, and verification-dependent fencing remain in force.
- **Validation:** Focused Follow regression tests pass for both missing-action prose suppression and valid current-turn Follow dispatch. The action-authorization suite passes with its documented expected failure. Core live Skyrim validation now passes for authoritative transfer/equipment invalidation, natural `Put your clothes back on` resolution to owned Roughspun Tunic, category armor Equip to the returned Stormcloak Cuirass, exact shield Equip, and a successful Follow action that put the NPC into actual follower state. The negative Follow/no-action case, unrelated-turn Follow retrigger check, final-candidate participant transition, and teardown/immediate-next-conversation checks remain pending.
- **Relevant work:** Mantella [PR #743](https://github.com/art-from-the-machine/Mantella/pull/743), branch `feature/equip-action`, current head `2f6b26035941227de32e6937e2cb9b8d5cc1b8aa`. The coherent implementation includes immutable current-turn and actor-scoped authorization, authoritative inventory/equipment tracking with stale-transfer invalidation, natural/category Equip handling with concrete owned-target validation, fail-closed stale/unowned/wrong-category targets, rejected-action dialogue fencing, Follow truthfulness protection, and the compatible participant/lifecycle/summary foundations required by the integration. Reproducible candidate packaging also hardens Tk/Gradio, tiktoken/tiktoken_ext, Silero, and Moonshine assets while preserving installation-owned `piper/` and `moonshine/` runtimes.
- **Notes:** Live examples include a Wood Elf saying “If you offer a path that leads away from this... stagnation, then I am ready to walk it. Lead on, Emilia.” without emitting `Follow:`, and a later unrelated turn (`Either you confess and get better, or I leave you.`) emitting `Inventory:` again after Inventory had only been requested on the prior turn. Do not implement as part of Bug #9.

- **Additional validated sub-fix:** The streamed Equip parser regression that dropped the item after an incomplete `Equip:` prefix was fixed on Mantella PR #743 (`feature/equip-action`, commit `76b09f07f099e4ed3281b73dfe16bb4e5c0256b2`). Live Skyrim validation succeeded for Golden Saint Shield, Stormcloak Cuirass, and Ancient Nord Sword, including authoritative post-equip equipment refresh. This parser sub-fix does not close the broader action-authority bug.
- **Separate issue:** Missing-action retry can fabricate an impossible Equip target; see Bug #10.

## Bug #9 — Normal dialogue colons can be misparsed as speaker labels

- **Status:** `REOPENED / PARTIALLY FIXED`
- **Category:** Multi-NPC response parsing / speaker attribution
- **First observed behavior:** The LLM generated: `Stormcloak Soldier: I fare well, though the night is cold. The wind howls like a wolf hungry for freedom. You ask how I am, but I ask you:`. Mantella correctly parsed the initial `Stormcloak Soldier:` but later treated `You ask how I am, but I ask you:` as another speaker label. The affected dialogue was discarded.
- **Root cause:** Response/newline boundary information was discarded before `change_character_parser`, so every colon-delimited unknown prefix was treated as an unauthorized speaker regardless of whether it began a real speaker line.
- **Fix / current approach:** Response and newline boundary metadata now survives cleaning and sentence accumulation. Unknown speaker prefixes are rejected only at a real response or line boundary; unmarked inline colons remain ordinary dialogue, and recognized action prefixes continue through action parsing.
- **Validation:** The earlier live validation confirmed `Wood Elf:` was recognized and `But know this:` remained spoken dialogue, but newer live and strict pytest evidence shows the fix is incomplete. At a response/line boundary, ordinary prose such as `I have a few things:`, `There is one problem:`, `My plan is simple:`, `The truth is:`, and `Here's what I know:` is still treated as an unknown speaker prefix and discarded. Legitimate labels such as `Lydia: We should move.` must continue to work. `python3 -m compileall -q src tests` and `git diff --check` pass; the five grammatical-colon regressions remain visible strict expected failures.
- **Relevant work:** Mantella PR [#747](https://github.com/art-from-the-machine/Mantella/pull/747); branch `feature/bug9-colon-boundaries`; commit `b76999b751c664ec31075832f86ced26f999e562`.
- **Notes:** Previously fixed boundary metadata behavior remains useful for real speaker labels and inline colons, but ordinary unknown prefixes at response/line boundaries remain unresolved. Do not treat Bug #9 as fully validated until those five grammatical-colon cases are corrected. This tracker update does not change Bug #9 production code.

## Bug #10 — Missing-action retry can fabricate impossible Equip target

- **Status:** `OPEN / CONFIRMED`
- **Category:** Action retry / Equip authorization / impossible-action handling
- **First observed behavior:** The player asked `Equip the helmet.` while the Wood Elf did not possess a helmet. The model correctly refused: `I do not have a helmet, Emilia. I cannot equip what I do not possess.` Mantella nevertheless treated the missing `Equip:` prefix as a missing requested action and logged `Retrying missing requested action once ... actions=['mantella_npc_equip']`. In a subsequent reproduction, the corrective response fabricated `Equip: Ancient Nord Helmet | I have no helmet to equip, Emilia. However, I am ready to follow your lead.`
- **Root cause:** Not yet established. The missing-action retry currently treats a syntactic action request as an obligation even when authoritative inventory state makes the requested action impossible, allowing a corrective generation to pressure the model toward an invented target.
- **Expected behavior:** If an explicitly requested action cannot validly execute because its target/item is absent or not possessed, the NPC may refuse or explain. Mantella must not force a corrective action merely because no prefix was emitted, and nonexistent item names must never be invented to satisfy an action obligation.
- **Safety observation:** The fabricated helmet did not result in a successful Skyrim equip.
- **Investigation direction:** Compare accidental omission of a required action prefix with a correct refusal grounded in authoritative inventory. Inspect current-turn obligation detection, inventory/equip authorization, missing-action retry prompts, and Equip target validation. This is separate from the now-live-validated streamed Equip parser fix.
