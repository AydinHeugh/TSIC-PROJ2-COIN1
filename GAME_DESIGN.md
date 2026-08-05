# Game Design — Mario Mode

## Concept
Extends the base coin-catcher into a survival game: instead of a countdown timer alone,
the player has a heart-based health bar that drains over time and on mob contact.
Powerups temporarily counter the drain or buff the player.

## Health System
- 6 hearts total, full/half/empty visual states.
- Decay: lose half a heart every 5 seconds -> 1 heart every 10 seconds -> 6 hearts = 60
  seconds of life (matches "a whole life = one minute").
- Touching a mob: lose 1 heart instantly, on top of the passive decay.
- Game over when hearts reach 0 (replaces/joins the existing timer-based game over).

## Hazards — mob reskin
- Reuse the existing `-3`/`-5` obj_type slots and spawn/collision pipeline unchanged,
  but: (1) swap their sprite art for real mob PNGs (teammate provides, 16x16, drops into
  the existing atlas slot), (2) change their collision effect from a score penalty to a
  heart penalty (`hit_player_*` hooks in `game_ctrl` unchanged, just point the effect at
  `heart_count` instead of `score`).
- No new obj_type, no new spawn-probability logic — art + effect swap only.

## Fireball (pickup-triggered projectile, NOT button-triggered)
- Falls from the sky as a new obj_type, same as other falling objects.
- On player contact: instead of a score/heart effect, spawns a projectile at the
  player's position, moving in whatever direction `player_dir` currently faces.
- Projectile travels across the screen each frame; checks overlap against active mob
  objects and clears (`obj_valid <= 0`) any it touches — destroys mobs on contact.
- Despawns automatically off-screen (x out of bounds).
- No button needed — resolves the earlier `btn_skill`/jump conflict, since firing is
  automatic on pickup, not held-button-triggered. `skill_slot` stays unused/disabled for
  now.

## NOTE: teammate already added jump + gravity
`btn_skill` is now the jump button (`game_ctrl.v`/`game_core.v`, `player_y`/`player_vy`
added, `obj_layer.v` updated to use dynamic `player_y`). Keeping this — it's already
mostly working. `skill_slot`'s `btn_skill` input is currently hardwired to `1'b0`
(disabled), which is fine since the Fireball no longer needs a button.

## Gravity / Jump
- Already implemented by teammate (see note above). Not cut — keeping it.

## Scope tiers (updated — today is the last work day)
- **MVP (today's actual deliverable)**: health bar (time decay + mob-hit decay), mob
  reskin of -3/-5, jump/gravity (already done), Fireball pickup+projectile+mob-destroy,
  full understanding of every changed module.
- **Stretch (only if MVP is done with real time to spare)**: a second distinct mob type,
  a duration-based powerup via `skill_slot` on a 5th physical button (not today).
- **Cut**: hardware-in-the-loop RL, anything needing new physical hardware today.

## Data/RL side-track (separate from the FPGA build, does not touch chip resources)
Python simulation of the game rules (not the real Verilog), used to:
- Generate data justifying design constants (decay rate, spawn odds, powerup timing).
- Optional: train a small RL agent (stable-baselines3 default MLP) as a demo/communication
  talking point. Not hardware-connected, not required for grading, pursued only if MVP is
  done early.

## Task breakdown
- Health bar mechanics — game_ctrl.v (register + decay logic), ui_layer.v (heart rendering)
- Mob spawn + collision — spawn_queue.v / spawn_postprocess.v, game_ctrl.v
- First Aid Kit powerup — skill_slot.v hook, new effect logic
- Assets — heart icons (full/half/empty), one mob sprite, first aid icon (all 16x16,
  existing png2mem pipeline)
- Slides / presentation — non-coding teammates, in parallel with above
