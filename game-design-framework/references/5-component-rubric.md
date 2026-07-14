# 5-Component Rubric

Use this file when the quick filter is not enough to evaluate a mechanic. Score components qualitatively as Strong, Adequate, Weak, or Unknown. Unknown is not a pass; either validate or label the assumption.

## Clarity

Core question: Can the player predict what will happen?

Strong signals:

- Telegraph appears before resolution, not simultaneously with it.
- The cause of success or failure is visible or audible.
- The same input and situation produce the same apparent result.
- UI, animation, camera, and audio do not contradict each other.

Failure modes:

- Attacks, hazards, or state changes resolve before the player can read them.
- Rules are hidden in implementation details.
- Similar-looking entities behave differently without a cue.
- Feedback explains that something happened but not why.

Design knobs:

- Anticipation frames, windups, silhouettes, VFX shape, UI indicators.
- Audio warning, camera framing, screen direction, animation pose readability.
- Rule simplification, tutorial prompt timing, preview ghosts, range markers.

Acceptance checks:

- New player can predict the next consequence after seeing the telegraph once or twice.
- Observer can describe why the player succeeded or failed.
- Player can distinguish harmless, warning, active, and recovery phases.

## Motivation

Core question: Does the player care about the outcome?

Strong signals:

- Outcome affects persistent state, route, resources, ranking, story, social standing, build expression, or future opportunity.
- Failure changes decision pressure instead of merely replaying the same moment.
- Rewards reinforce the fantasy or mastery loop.

Failure modes:

- Success is decorative and does not change options, stakes, or identity.
- Reward is numerically present but emotionally flat.
- Failure is pure friction with no learning, tension, or tradeoff.
- The mechanic is required but not connected to why the player is playing.

Design knobs:

- Resource stakes, progression gates, optional mastery rewards, risk/reward tuning.
- Unlock cadence, visible goals, character reactions, route access, score multipliers.

Acceptance checks:

- Player can explain why they want to engage with the mechanic.
- Ignoring the mechanic has a meaningful cost or tradeoff.
- Mastery creates visible improvement in outcomes.

## Response

Core question: Do player inputs matter?

Strong signals:

- Inputs are acknowledged immediately, even when resolution is delayed.
- Actions can be buffered, cancelled, queued, aimed, redirected, or intentionally committed depending on the game's identity.
- Recovery, lockout, and cooldown rules are legible.
- Failure feels like the player's decision or timing, not the interface.

Failure modes:

- Inputs vanish silently during animation, hitstun, landing, menu transitions, or state changes.
- Long lockouts prevent correction without selling commitment as fantasy.
- Animation priority overrides player intent in ways the player cannot learn.
- Latency, camera, or collision creates apparent input failure.

Design knobs:

- Input buffering, coyote time, cancel windows, aim assist, snap rules.
- Interrupt priority, recovery duration, hitstop, queue depth, input visualization.

Acceptance checks:

- Player can intentionally repeat the same action under pressure.
- Spam tests do not produce stuck, ignored, or contradictory states.
- Skilled timing improves outcome without requiring hidden knowledge.

## Satisfaction

Core question: Does success feel earned?

Strong signals:

- At least two feedback channels fire for significant actions: visual + audio minimum.
- Feedback escalates with skill, risk, rarity, or impact.
- The moment has texture: anticipation, impact, aftermath.
- Rewards feel proportional to effort and consequence.

Failure modes:

- Success is mechanically correct but emotionally quiet.
- Effects are loud but disconnected from timing or consequence.
- Feedback fires before the player action resolves, weakening causality.
- Repeated actions become noisy or fatiguing.

Design knobs:

- Animation pose, hitstop, camera shake, particles, sound layers, controller rumble.
- Score callouts, slow motion, freeze frames, enemy reactions, environment response.

Acceptance checks:

- Observer can identify the successful moment without reading UI text.
- Player reports the action as strong, clean, sharp, heavy, elegant, or otherwise aligned with intent.
- Feedback remains readable after repetition.

## Fit

Core question: Does it match the game's identity?

Strong signals:

- Timing, weight, audio, animation, and consequence match the entity type and game fantasy.
- The mechanic reinforces genre promises instead of importing unrelated conventions.
- UI vocabulary, camera behavior, and feedback style belong to the same world.

Failure modes:

- Heavy entities move or sound weightless.
- Precise competitive actions use vague feedback.
- Cozy or reflective play is interrupted by harsh urgency without intent.
- A mechanic solves a design problem while diluting the identity.

Design knobs:

- Motion curves, animation weight, sound material, camera distance, UI language.
- Failure severity, forgiveness, reward tone, pacing, affordance style.

Acceptance checks:

- If shown without explanation, the mechanic appears to belong to this game.
- Adjustments preserve the core fantasy, not just mechanical correctness.
- The same rule would feel wrong if transplanted unchanged into a different game identity.

