# Changelog

## 2026-08-13

- Created the personal Mantella bug tracker.
- Initialized the canonical registry with Bugs #1–#8.

- Updated Bug #7 to record the live-validated stable actor identity and in-flight generation fix.
- Recorded the remaining same-name physical speaker-routing sub-issue as Bug #7a.
- Recorded the open cold participant-transition latency and confirmed that Piper stderr diagnostics did not resolve Piper reliability.
- Confirmed the same-name Stormcloak Soldier physical speaker-routing fix in live Skyrim validation.
- Recorded the new generic-NPC voice playback regression as Bug #7b after Wood Elf synthesis succeeded but audible playback was absent.
- Updated Bugs #7, #7a, and #7b with source commits, PR references, and successful live validation of same-name identity, physical speaker routing, and generic-NPC fallback playback.
- Added confirmed Bug #9 for normal dialogue colons being misparsed as multi-NPC speaker labels.
- Added `FEATURES.md` with the initial roadmap and status legend for Features #1–#3.
- Expanded `README.md` to cover bugs, validation, upstream PRs, planned features, and feature implementation status.

## 2026-08-14

- Marked Bug #9 fixed and live-validated after boundary metadata preserved ordinary grammatical colons in NPC dialogue.
- Added Bug #9 PR #747 (`b76999b751c664ec31075832f86ced26f999e562`) and recorded the pytest dependency/network limitation alongside successful compileall and diff-check validation.
- Generalized Bug #8 to action-authority / action-prefix compliance, recording live missing-`Follow:` and unauthorized repeated-`Inventory:` examples. Bug #8 remains open and unimplemented.
- Reopened Bug #9 as partially fixed after live and strict pytest evidence showed ordinary grammatical colons at response/line boundaries are still discarded as unknown speakers.

## 2026-08-15

- Recorded the approved Bug #8 action-authority implementation work: immutable per-turn authorization, stable actor/ref-ID scoping, persistent authoritative transfer references, event-before-snapshot ordering, bounded corrective generation, verification-dependent dialogue fencing, and generation-stream cleanup. Validation covered production-flow pytest, headless scenarios, and packaged smoke tests; Bug #8 remains open pending broader action/live validation.
- Recorded the live-validated streamed Equip parser fix on Mantella PR #743 (`feature/equip-action`, commit `76b09f07f099e4ed3281b73dfe16bb4e5c0256b2`). Golden Saint Shield, Stormcloak Cuirass, and Ancient Nord Sword all equipped successfully in Skyrim with authoritative post-equip refresh; redundant post-result Equip remained blocked by current-turn authorization.
- Added Bug #10: missing-action retry can fabricate an impossible Equip target when authoritative inventory makes the requested item unavailable. Bug #10 remains open and separate from the parser fix.
- Implemented the remaining tested Bug #8 dialogue/action boundary for explicit state-changing requests: prose such as “Lead on” is held and discarded unless the current turn accepts the corresponding Follow/action invocation. Focused action-authority and OutputManager tests pass; live Skyrim validation remains pending, so Bug #8 is not marked fixed.
