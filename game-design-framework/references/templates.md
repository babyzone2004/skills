# Templates

Use these templates when proposing, critiquing, debugging, or playtesting game mechanics.

## Feature Proposal or Critique

```markdown
## Player Goal & Context
[What is the player trying to do, under what pressure, and why does it matter?]

## System Rules
[Core behavior, failure conditions, state transitions, resources, feedback, and edge cases.]

## 5-Component Evaluation
- Clarity: [Strong/Adequate/Weak/Unknown] - [reason]
- Motivation: [Strong/Adequate/Weak/Unknown] - [reason]
- Response: [Strong/Adequate/Weak/Unknown] - [reason]
- Satisfaction: [Strong/Adequate/Weak/Unknown] - [reason]
- Fit: [Strong/Adequate/Weak/Unknown] - [reason]

## Risks & Abuse Cases
[Exploits, degenerate strategies, readability failures, economy risks, accessibility risks.]

## Playtest Scenarios
[New player, stress, skill, abuse, readability scenarios.]

## Tuning Priority
[What to adjust first if the mechanic feels wrong. Do not tune numbers before Clarity and Response are verified.]
```

## State Machine Spec

```markdown
## State: [Name]

Entry conditions:
- [Allowed previous states, required resources, required inputs, external triggers.]

Exit conditions:
- [Timer, input release, landing, hit, cancel, resource zero, external event.]

Interruptibility:
- Player input: [allowed cancels]
- Damage/hitstun: [behavior]
- Other abilities: [priority rules]
- System events: [pause, cutscene, death, checkpoint, network correction]

Chained actions:
- [State A] -> [State B] when [condition]

Resource cost:
- On entry: [cost]
- On sustain: [cost over time]
- On cancel/refund: [refund or penalty]

Edge cases:
- Slopes:
- Ceilings:
- Moving platforms:
- Walls/corners:
- Hitstun:
- Resource zero:
- Camera:
- Multiplayer authority, if applicable:
```

## Edge Case Enumeration

```markdown
Movement/collision:
- Grounded, airborne, slopes, ledges, ceilings, walls, moving platforms.

Timing/state:
- Startup, active, recovery, cancel window, input buffer, hitstop, pause, slow motion.

Resources:
- Exact cost, below cost, zero during sustain, refund, duplicate spend prevention.

Camera/readability:
- Off-screen, occluded, zoomed, locked-on, split attention, screen shake.

Persistence/progression:
- Save/load, checkpoint, death, retry, unlock rollback, version migration.

Multiplayer:
- Prediction, reconciliation, disconnect, duplicate events, late packets, cheating.
```

## Debugging Flow

```markdown
Symptom: [Feels wrong / boring / clunky / weak / unfair]

1. Clarity check:
   Can the player predict the result before it resolves?
   If no, add telegraph, UI, audio, animation, or camera support.

2. Response check:
   Is input acknowledged and can intent be expressed under pressure?
   If no, inspect buffering, cancels, lockouts, latency, and camera.

3. Satisfaction check:
   Does success use at least two feedback channels and match consequence?
   If no, add or retime visual, audio, animation, hitstop, rumble, or camera feedback.

4. Fit check:
   Does timing, weight, audio, and consequence match the game identity?
   If no, adjust feel vocabulary before numeric balance.

5. Motivation check:
   Does the player care about success or failure?
   If no, connect outcome to stakes, progression, route, resources, or mastery.
```

## Playtest Script

```markdown
Goal:
[What feature or feeling is being validated?]

Setup:
[Build, level, character, input device, player profile, debug overlays.]

Scenarios:
1. New player test:
   Task: [Can they infer the rule without instruction?]
   Pass: [Observable success metric]

2. Stress test:
   Task: [Spam inputs, hit boundaries, force edge cases.]
   Pass: [No stuck states, lost inputs, unreadable feedback, crashes.]

3. Skill test:
   Task: [Repeat with mastery opportunity.]
   Pass: [Better timing/knowledge improves outcome.]

4. Abuse test:
   Task: [Try to skip content, farm value, avoid risk, break state.]
   Pass: [Exploit blocked or tradeoff preserved.]

5. Readability test:
   Task: [Observer watches without explanation.]
   Pass: [Observer can explain what happened and why.]

Numbers:
[Source-backed values or Starting value + pass/fail metric + adjustment direction.]

Notes:
[What failed first: Clarity, Response, Satisfaction, Fit, or Motivation?]
```

