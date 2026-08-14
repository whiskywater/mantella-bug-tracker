# Mantella Feature Roadmap

This roadmap tracks planned Mantella and Mantella-Spell capabilities. These are ideas, not implementation commitments, unless their status says otherwise.

## Feature status legend

- `Idea` — a possible future capability
- `Planned` — intended for a future implementation, not yet started
- `Investigating` — design or feasibility work is underway
- `In Development` — implementation is underway
- `Testing` — implementation exists and is being validated
- `Implemented` — implementation is complete
- `Upstream PR` — implementation is proposed upstream
- `Merged` — implementation has landed upstream

## Feature #1 — NPC-written notes, letters, or orders that establish player authority

- **Status:** `Idea / planned`
- **Goal:** Allow an NPC to produce an in-world written note, letter, order, writ, authorization, recommendation, or similar document that the player can carry and present to another NPC.
- **Use cases:** A guard captain could write orders granting passage; a Jarl or steward could provide written authorization; a commander could give orders to show soldiers; or an NPC could provide a letter of introduction or written proof of protection.
- **Design goals:** The document must represent something the issuing NPC actually agreed to; it must not let the LLM fabricate authority without in-game action/state backing; and the receiving NPC should be able to inspect or receive it and gain the relevant conversation context. Ideally this integrates with Skyrim inventory/books/notes rather than existing only as invisible prompt state.
- **Implementation:** Not started.

## Feature #2 — Tactical strategy commands and persistent tactical states

- **Status:** `Idea / planned`
- **Goal:** Allow the player to issue richer tactical instructions to followers or allied NPCs and have those instructions persist as meaningful state rather than one-line roleplay.
- **Examples:** Hold position; defend an area; stay behind the player; take point; keep distance; prioritize ranged combat; protect a named NPC; focus a named enemy; retreat if badly injured; regroup; guard an entrance; avoid initiating combat; aggressive or defensive posture; and formation or spacing preferences.
- **Design goals:** Commands should correspond to real game behavior where possible; NPC dialogue should remain truthful about whether the command/state succeeded; tactical state should be inspectable and updateable; state should survive normal conversational turns for as long as appropriate; and tactical state should remain separate from ordinary dialogue memory.
- **Implementation:** Not started.

## Feature #3 — NPC-initiated addition of one other NPC to the conversation

- **Status:** `Idea / planned`
- **Goal:** Allow a participating NPC to dynamically add up to one additional NPC to the active Mantella conversation without requiring the player to explicitly invite that NPC.
- **Use cases:** An NPC could say “You should hear this from Lydia” and bring Lydia into the conversation; a guard could ask their captain to join; or an official could call another guard or witness into the discussion.
- **Constraints:** At most one NPC may be added by NPC initiative per triggering action or event; the added NPC must be a real valid Skyrim actor; the initiating NPC must have a contextual reason to involve them; arbitrary hallucinated NPC creation is not allowed; and stable actor identity/ref_id must be preserved.
- **Design goals:** The feature must integrate safely with participant lifecycle and same-name actor routing. Adding an NPC must not cancel or swallow an in-flight player utterance or response, and summaries and speaker routing must remain actor-identity-safe.
- **Implementation:** Not started.
