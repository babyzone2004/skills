---
name: game-design-framework
description: Use when designing, critiquing, implementing, tuning, or debugging game mechanics, player actions, combat, movement, camera, feedback, progression, balance, or "feels wrong/clunky/boring" gameplay issues.
---

# Game Design Framework

## Purpose

Use this constraint system to evaluate and implement game mechanics with focus on player experience over feature completion.

Core principle: Mechanics are code. Gameplay is the player's experience of that code. The goal is not to implement features, but to implement Relevance.

## Quick Reference: The 5-Component Filter

Before implementing or critiquing any game feature, evaluate against:

| Component | Core Question | Quick Check |
| --- | --- | --- |
| Clarity | Can the player predict what will happen? | Telegraph exists before resolution |
| Motivation | Does the player care about the outcome? | Outcome affects persistent state |
| Response | Do player inputs matter? | Actions can be buffered/cancelled meaningfully |
| Satisfaction | Does success feel earned? | Multiple feedback channels fire: visual + audio minimum |
| Fit | Does it match the game's identity? | Weight, timing, audio match entity type |

Conflict priority: Response > Clarity > Satisfaction > Fit > Motivation.

For detailed evaluation rubrics, consult `references/5-component-rubric.md`.

## Operating Protocol

1. Identify active domain(s) from `references/domain-guide.md`.
2. Evaluate against the 5-Component Filter.
3. Complete the State Machine Checklist if the feature involves player state changes.
4. Check the Numbers Policy before proposing any values.
5. Use the templates in `references/templates.md` for critiques, state specs, debugging, and playtests.

## Numbers Policy

When proposing any numeric value such as timing windows, costs, speeds, damage, cooldowns, range, spawn rates, or reward values, choose one:

**Option A - Source-backed:** Cite a verifiable reference such as a GDC talk, postmortem, or published analysis.

Example: "Coyote time of 80-150ms. Source: Maddy Thorson's Celeste postmortem."

**Option B - Starting value with test plan:** Label explicitly as "Starting value" and include a micro test plan, pass/fail metric, and adjustment direction if it fails.

Example: "Starting value: 120ms. Test: players make intended jumps 9/10 times. If fail rate is above 20%, increase by 30ms increments."

Never claim "industry standard" or "common practice" without a source.

## Assumption Labeling

When critical information is missing, state:

```text
ASSUMPTION: [what you're assuming]
IMPACT: [why it matters to the design]
IF WRONG: [failure mode]
VALIDATE: [how to check quickly]
```

## Research Triggers

Search before proposing when:

- About to claim "best practice" or "standard approach".
- Balance or economy values need benchmarks.
- Accessibility requirements apply.
- Comparative references are needed from similar games.

If search is unavailable, convert the claim to Assumption + Test Plan format.

## State Machine Checklist

For any feature that changes player state, including movement abilities, combat actions, and status effects, define:

| Property | Must Define |
| --- | --- |
| Entry conditions | What states can transition into this? |
| Exit conditions | What ends this state: timer, input, external event? |
| Interruptibility | What can cancel this: damage, player input, other abilities? |
| Chained actions | What states can this transition to? |
| Resource cost | What is consumed on entry? On sustain? |
| Edge cases | Behavior on slopes, ceilings, moving platforms, during hitstun, at resource zero |

## Debugging Protocol

When told "it feels wrong/boring/clunky," diagnose in order:

| Symptom | Check First | Before Tuning Numbers |
| --- | --- | --- |
| "I didn't know that would happen" | Clarity | Add telegraph, audio cue, UI indicator |
| "I don't care" | Motivation | Connect to progression, increase stakes |
| "It feels laggy" | Response | Add buffering, allow cancels, reduce lockouts |
| "It feels weak" | Satisfaction | Add feedback channels: minimum 2 |
| "It doesn't fit" | Fit | Adjust timing, weight, audio texture |

Rule: Do not tune damage or timing numbers until Clarity and Response are verified as not the root cause.

## Playtest Requirements

Every significant feature must include scenarios for:

- New player test: Can they infer the rules without being told?
- Stress test: Spam inputs, boundary conditions, edge cases.
- Skill test: Can mastery improve outcomes meaningfully?
- Abuse test: Can this be exploited to skip content or trivialize risk?
- Readability test: Can an observer understand what happened and why?

## Red Flags

Stop and clarify when:

- State machine transitions are undefined, such as "works from any state".
- Multiplayer authority is unspecified.
- Economy or currency feature has no balance targets.
- Camera behavior during action is undefined.
- Feature scope is actually 3+ features in disguise.

## Definition of Done

- [ ] 5-Component Filter evaluated and documented.
- [ ] State Machine Checklist completed, if applicable.
- [ ] Edge cases enumerated and handled.
- [ ] Minimum 2 feedback channels for significant actions.
- [ ] Playtest script written and smoke-tested.
- [ ] Numbers justified per Numbers Policy.

## Output Structure

When proposing or critiquing a feature, structure the answer as:

1. Player Goal & Context - What is the player trying to do and why?
2. System Rules - Core behavior, failure conditions, edge cases.
3. 5-Component Evaluation - Which components are strong or weak?
4. Risks & Abuse Cases - What could break or be exploited?
5. Playtest Scenarios - How to validate quickly.
6. Tuning Priority - What to adjust first if it does not feel right.

