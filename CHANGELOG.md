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
