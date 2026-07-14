# Domain Guide

Use this file to identify active design domains before proposing or critiquing a feature. Most features touch multiple domains; name the primary domain and any secondary domains before evaluating.

## Movement

Key questions:

- What can the player start, stop, redirect, buffer, cancel, or chain?
- What terrain and collision cases change the move?
- Is mastery expressed through timing, routing, positioning, resource use, or style?

Common edge cases:

- Slopes, ceilings, ledges, moving platforms, one-way platforms.
- Landing during action, leaving ground during startup, wall contact, water or low gravity.
- Input during hitstun, cutscenes, menus, stamina zero, animation lockout.

5-component emphasis: Response and Clarity first. Satisfaction matters for jumps, dashes, impacts, landings, and near misses.

## Combat

Key questions:

- What are startup, active, recovery, hit confirm, block, dodge, parry, and punish rules?
- What can interrupt what?
- How are risk, range, resource, and reward communicated?

Common edge cases:

- Trading hits, simultaneous death, invulnerability overlap, hitstop stacking.
- Target switching, walls, corners, elevation, friendly fire, latency.
- Resource reaching zero during startup or sustain.

5-component emphasis: Response, Clarity, and Satisfaction. Fit determines weight, brutality, precision, or spectacle.

## Camera

Key questions:

- What must be visible before, during, and after the action?
- Does the camera support prediction, aiming, navigation, or spectacle?
- What owns the camera when player action, lock-on, cutscene, and collision compete?

Common edge cases:

- Tight spaces, vertical transitions, occluders, screen edges, fast reversals.
- Multiplayer framing, split objectives, boss scale, teleporting, knockback.

5-component emphasis: Clarity first. Response suffers if the camera hides input consequences or aiming context.

## Audio

Key questions:

- What information is conveyed before the player sees it?
- Which sounds are warnings, confirmations, impacts, rewards, or state loops?
- How does mix priority change under pressure?

Common edge cases:

- Repeated spam, overlapping impacts, off-screen events, muted UI, accessibility needs.
- Music masking key cues, multiplayer voice chat, low-health filters.

5-component emphasis: Satisfaction and Clarity. Fit depends heavily on material, pitch, attack, decay, and mix weight.

## UI/UX

Key questions:

- What must be known now, soon, later, and never?
- Does UI explain rules without stealing attention from play?
- Are resource, cooldown, range, target, and state changes readable at gameplay speed?

Common edge cases:

- Colorblind readability, small screens, localization, remapping, HUD clutter.
- Controller/mouse parity, pause/menu transitions, tutorial prompt timing.

5-component emphasis: Clarity and Motivation. Response matters for menus, wheels, radial selection, and input prompts.

## Progression

Key questions:

- What capabilities, knowledge, mastery, or identity changes over time?
- Does the mechanic create new decisions or only bigger numbers?
- Can players see the next meaningful goal?

Common edge cases:

- Sequence breaks, overleveling, underleveling, dead builds, irreversible choices.
- Reward farming, grind loops, difficulty spikes, tutorial debt.

5-component emphasis: Motivation and Fit. Clarity matters for build choices and upgrade consequences.

## Persistence

Key questions:

- What survives death, reload, level transition, session end, or account migration?
- What can be lost, refunded, replayed, or undone?
- How are saves, checkpoints, and rollback communicated?

Common edge cases:

- Crash during reward, quit during combat, save scumming, cloud conflict.
- Version migration, DLC removal, offline play, corrupted state.

5-component emphasis: Motivation and Clarity. Persistence defines stakes, trust, and consequence.

## Multiplayer Authority

Key questions:

- Which machine or service decides truth for movement, combat, economy, inventory, and matchmaking?
- What is predicted locally, reconciled later, or server-authoritative immediately?
- What happens under lag, packet loss, disconnect, host migration, or cheating attempts?

Common edge cases:

- Double hits, rollbacks, late inputs, desync, duplicate rewards.
- Race conditions on pickups, trades, matchmaking, respawns, and ranked outcomes.

5-component emphasis: Response and Clarity. Fit determines how much correction, delay, or uncertainty the game can tolerate.

