# Adding UAI actions and packages for NPCCore characters

NPCs in this mod choose what to do with SCore's Utility AI (UAI): every few tenths of a second each package scores its actions, and
the best one runs. The rules below decide whether your action ever gets a chance, what it costs every NPC, and whether it can break
the behaviour around it. Read them before you write a modlet that adds to `utilityai.xml`.

## 1. Load after NPCCore, and patch, never replace

- The game loads mod folders in name order (sorted folder names). NPCCore's `Mods/0-XNPCCore/Config/utilityai.xml` starts with
  `<remove xpath="/utility_ai/ai_packages"/>` and then adds its own complete package set (it includes its own copies of SCore's
  packages). **Anything a mod added to `utility_ai` before NPCCore loaded is gone.**
- So your modlet's folder must sort AFTER `0-XNPCCore` (for example `1-MyNpcActions`), and your file must PATCH ours with XPath
  (`append`, `insertBefore`, `insertAfter`, `set`, `setattribute`, `remove`), never remove `ai_packages` itself.
- The first line of every XML file is exactly `<?xml version="1.0"?>` (an `encoding` attribute stops the game's XML loader), and an
  XML comment must not contain `--` or a long dash.

## 2. Which packages our characters run

| who | packages (`AIPackages` property) |
|---|---|
| SDCS survivors and hires (`npcSdcsBase`) | `SdcsNpcCore, SdcsNpcRangedAdvanced, SdcsNpcHired, SdcsNpcMedic, SdcsNpcLoot, SdcsNpcLeave, SdcsCover` |
| bandits (`npcSdcsBandit*`) | `SdcsNpcCore, SdcsBanditCombat, SdcsCover` |

- To add an action to an existing package, insert it into that package (section 4). To add a whole new package, define it under
  `/utility_ai/ai_packages` and add its name to the character's `AIPackages` property (`setattribute`, or the `csv` operation to
  add one value to the list). A package that no character lists never runs.
- A package's own `weight` (default 1) multiplies its best action's score when packages are compared. Leave it at 1 unless you mean
  your whole package to win or lose against ours.

## 3. Pick the weight from the bands

Every action's best possible score is its `weight` (an action never scores above it). Our packages group weights into bands, each
marked in the file with a `<!-- band: NAME -->` comment above the action:

| band | weights | what lives there |
|---|---|---|
| SAFETY | 11-12 | playbacks that must not be interrupted (a forced roll, a bandage) |
| DEFENCE | 8-10 | backing off, fleeing, jump orders |
| AID | 7 | the medic |
| ENGAGE | 5-6.5 | shooting, throwing, melee, approaching, seeking |
| JOBS | 4 | looting, harvesting, crafting, posts |
| ORDERS | 3-3.5 | stay, guard, follow, the never-idle hold |
| AMBIENT | 0.5-2 | wander, chat, idle |

Put your action in the band it belongs to and give it the band marker comment. An action weighted above a band can never lose to
anything inside it, so a weight is a promise: a 9 beats every fight, an 0.5 only runs when nothing else wants to.

## 4. List actions highest weight first

SCore skips an action whose weight is already below the best score found earlier in the same package: it cannot win, so its
considerations are never run. That skip only helps when the package lists its actions **highest weight first**, and all of ours do
(an automated check keeps our shipped file that way).

- `append` puts your action at the END of the package, after the lowest weight there (the 0.5 idle, or the never-idle hold): it
  breaks the order, and every action above it now runs its considerations for nothing. Use `insertBefore` / `insertAfter` on the action of the next higher or lower weight instead:

```xml
<insertBefore xpath="/utility_ai/ai_packages/ai_package[@name='SdcsBanditCombat']/action[@name='BackupRanged']">
	<!-- band: ENGAGE -->
	<action name="MyBanditTaunt" weight="5.5" entity_filter="IsEnemy">
		...
	</action>
</insertBefore>
```

- Actions of equal weight run in file order, and on an exact score tie the one listed first wins.

## 5. Order the considerations: cheap first, expensive last, and know which ones keep state

SCore multiplies an action's considerations in order and **stops at the first one that scores 0**. The result does not depend on the
order, the cost does: put the checks that fail most often and cost least first.

- **Cheap** (a cvar, buff, order, flag or table read): `SelfHasCVar`, `SelfNotHasCVar`, `SelfNotHasBuff`, `HasOrder`, `NotHasOrder`,
  `TargetDistance`, `NoActiveChat`, `HasActiveChat`, and our `SdcsMayAct`, `SdcsReachable`, `SdcsKeepTarget`, `SdcsOffPost`.
- **Expensive** (a sight ray, a world scan, a path or a store walk): `CanSeeTarget`, `CanNotSeeTarget`, `CanHearTarget`,
  `InWeaponRange`, `NotInWeaponRange`, `SdcsEnemyNotNear`, `SdcsLineOfFire`, `SdcsSeesShot`, `SdcsMeleeReach`, `SdcsThrowWanted`,
  `SdcsCoverWanted`, `MayEngage`. Keep these last.
- **Checks that keep state while they score**: `MayEngage`, `SdcsLostSight`, `SdcsPatientNeedsAid`, `CanHearTarget`,
  `SdcsHasLootTarget`, `SdcsLeaderStill`, `SdcsLeaderNear`, `CombatLinger`. Each remembers something every time it is scored (a
  give-up clock, a last-seen spot, a patient note). Do not move them inside our actions, and do not put an action that holds one of
  them where a higher weight will skip it more often than before: the feature behind it stops updating.
- `SdcsMayAct` goes FIRST in any combat, loot or medic action: it scores 0 while the NPC is knocked down, so nothing starts mid-fall.
- `SdcsReachable` scores 0 for a target she has proved she cannot reach with her melee weapon, so she picks another one; it scores
  1 while she holds a gun (she can shoot through the gap). Put it AFTER `MayEngage` and `SdcsLostSight` in the same action, the one
  place where a cheap check goes after expensive ones: a 0 ahead of them stops their bookkeeping for as long as the mark lasts.

## 6. Mind the cost: it is paid by every NPC, many times a second

An action with `entity_filter="IsEnemy"` is scored once per enemy in sight, for every NPC, every rescore. A 30 microsecond check on a
bandit facing five enemies is 150 microseconds per rescore, times every bandit. Keep expensive checks behind cheap ones, and measure:

- `sdcsperf on 120` then `sdcsperf off` (server console): the cost of every action and consideration, per band and per NPC.
- `sdcsbench` (with two hires, logging off): a repeatable 25 bandit, 25 zombie stress run that prints one report and cleans up. Run
  it before and after your change at the same spot.
- `vcdebug on`: our verbose log lines (`[SDCS-UAI]` shows which action each NPC chose).

Do not change `max_entities` or `action_delay` on `ai_packages`: every NPC's cost scales with them.

## 7. A survivor's sight at a stealthy target

When a check asks whether one of our characters can see a crouched or shaded target, the game's
stealth test divides the distance by the character's stealth range and lerps the light the target
needs between its two ends over that distance. A survivor's stealth range is her class's
`SightRange` property (the mod sets it on the entity once the class values are read, so the test
runs the class's designed range rather than a pinned one). The light the target needs runs from
`SightLightThreshold` x (close) to y (at her `SightRange`); beyond the range the far end stands.
The `npcSdcs*` classes set no `SightLightThreshold` of their own, so the game default (30, 100)
applies: a target needs just over 30 of light at close range and the full 100 at `SightRange`
and beyond.
