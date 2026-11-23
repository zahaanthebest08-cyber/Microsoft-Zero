Read Assets/.../MissionStateMachine.cs (the MissionSlice mission state machine).
Then add a section called "## Mission save list (MissionSlice)" to CLAUDE.md.
Only edit CLAUDE.md. Do not change any code.

The section should say what to persist when the Switch sleeps or the player
quits mid-mission, and what to rebuild on relaunch. Use these decisions, with
a one-line reason for each:

SAVE (real progress):
- currentHealth: the player returns with the same health.
- lastSafePosition: a NEW value, not in the file today. Update it only while
  isGrounded is true and isNearPit is false. Save this instead of the live
  playerPosition, so a player who quit mid-fall comes back on solid ground.
- lootPickedUp: stops collected loot from reappearing.
- chestsOpened: stops chests being reopened for repeat loot.
- enemiesDefeated: defeated enemies stay defeated and it drives the Complete
  condition.
- phase: only when it is Dead or Complete, so a failed or finished mission
  doesn't restart. For Exploring, Combat or Falling, save Exploring.

DO NOT SAVE (rebuild or discard):
- Score: derived from enemiesDefeated.Count * PointsPerEnemy. Saving it would
  create a second source of truth.
- isGrounded, isNearPit, hitPitFloor: live physics values, recomputed each frame.
- fallVelocity, isFalling: momentary physics. Resuming a fall risks instant
  death on hitPitFloor.
- engagedEnemies: rebuilt when enemies notice the player. Restoring Combat
  with it empty would flip back to Exploring on frame one anyway.
- damageFlashTimer, cameraShakeAmount, combatMusicFadeTimer, inputBuffer,
  inputBufferAge: effects lasting under two seconds, already over by relaunch.

RESTORE RULES:
- Restore after Awake(), which initializes every chest to false, so the saved
  chestsOpened values overwrite those defaults.
- Validate saved IDs against LootIds, ChestIds and EnemyIds and ignore unknown
  ones.
- Always resume in Exploring (unless Dead or Complete) at lastSafePosition.
- There is no separate objective-step value. Progress is implied by
  enemiesDefeated, chestsOpened and lootPickedUp.

OPTIONAL (mark as a future idea): a short load-in damage immunity timer
(about 1.3s). ApplyDamage has no immunity check today, so this needs a new
timer. It is a cushion, not a replacement for the safe-position respawn.

NOTE: inventory items and materials are not tracked in this file. Flag that
they need to be saved by whichever system owns them.

Keep the section short and scannable, with two bullet lists (Save / Don't
save) and a short restore-rules list.
After editing CLAUDE.md, publish it to the mission repo:

1. Run `git status` and confirm CLAUDE.md is the only changed file. If
   anything else is modified, stop and tell me instead of committing it.
1. Show me the section you added (a diff or the text) so I can check it
   before anything is committed.
1. Once I approve, create a branch named docs/mission-save-list and run:
   git add CLAUDE.md
   git commit -m "Document MissionSlice save list in CLAUDE.md"
   git push -u origin docs/mission-save-list
1. Tell me the branch name and the commit hash. Do not push to main and
   do not force-push
