# Bandits in your POIs: a guide for POI builders (NPCCore 3.4.06)

NPCCore's bandits are hostile survivors built on the game's own character system: they fight with real weapons, use cover,
throw explosives, and sleep in your POIs like zombies do. This guide covers what you need to place them and test them.

## What to install

- `0-SCore` 3.4.25.925 or newer and `0-XNPCCore` 3.4.06 (the Core zip): required.
- The voice add-on (`1-XNPCVoiceControl-Voice` or `-Complete`): optional. Without it bandits work the same, they only stay
  silent (their one spoken line needs text to speech).
- Generate a NEW random world after installing your POI pack: an existing world never gains new POIs.
- Adding or changing NPC behaviour (UAI actions or packages) in a modlet: read `UAI-MODDING.md` first (load order, weights, action
  order, and what each check costs).

## Sleeper groups for your volumes

Put one of these in a sleeper volume, with its min and max count from the Spawn Count area (shown in the prefab XML, `SleeperVolumeGroup`, for example
`GroupNPCBanditsAll,2,4`):

| Group | Who spawns |
|---|---|
| `GroupNPCBanditsAll` | melee and gun bandits mixed |
| `GroupNPCBanditsMelee` | melee bandits only |
| `GroupNPCBanditsRanged` | gun bandits only |
| `GroupNPCBanditsBoss` | always tier 3 |
| `GroupNPCBanditsSentry` | bandit sentries: awake on their spot, holding it (see "Sentries" below); tier 1 brings the one melee sentry, tiers 2 and 3 guns |
| `GroupNPCBanditsSentryRanged` | bandit sentries with guns only |
| `GroupNPCSurvivorsSentry` | FRIENDLY sentries: survivors who hold their spot until the player hires them (see "Friendly sentries" below) |

(These are NPCCore's `S_-Group_NPC_Bandits_All` / `_Melee` / `_Ranged` / `_Boss` / `_Sentry` / `_SentryRanged` and
`S_-Group_NPC_Survivors_Sentry`; the game strips the `S_-` and the underscores.) You can mix bandit volumes and zombie volumes in
one POI.

## Which bandits spawn: the party's gamestage, with your POI's difficulty as a floor

A room's bandits come from the gamestage of the party near it when it wakes (every player within 100 m inside your POI). Your POI's
difficulty sets a floor on that: difficulty 1-2 gives at least tier 1 bandits, 3-4 at least tier 2, 5 tier 3. A higher party
gamestage still raises it. (The floor is `<Bandits><PoiTierFloors>` in Settings.xml. From difficulty 3 up the game also trims the
count a little, as for zombies.)

| Gamestage (or the floor) | Bandits | Weapons | Throws |
|---|---|---|---|
| below 100 | tier 1 | melee (clubs, knives, spears, sledges); in gun rooms a pipe pistol or pipe rifle | gun bandits: 2 pipe bombs |
| 100 to 399 | tier 2 | iron melee, pipe guns, the pistol, bows | 1 molotov |
| 400 and up | tier 3 | steel melee, the better guns (rifles, shotguns, machine guns, compound bows) | 3 molotovs |

The melee groups (`GroupNPCBanditsMelee`, NPCCore's `npcBanditsMeleeGroupGS*`) are melee only at every tier: no guns, bows or
throwables, also in high-tier POIs.
| Boss volume | tier 3 | as tier 3 | 3 molotovs |

They throw only at a group (three or more targets close together).

## How each room type behaves with bandits

- **Active:** walking into the room wakes every bandit in it at once. Before that, each sleeping bandit hears and sees like a
  sleeping zombie: a loud noise near her (gunfire) wakes her, and so does a player within about 6 m in front of her in enough
  light. A player who crouches in the dark and stays quiet can sneak past.
- **Passive:** the same senses. Unlike passive zombies, which sleep until they are hurt, bandits in a passive room still wake to
  gunfire or to being seen.
- **Attack:** the bandits are awake and waiting when the player arrives: an ambush.
- **Trigger:** as the game does it.

A bandit never triggers a room herself. The player's hired survivors do: a hire walking into a room wakes it.

## Layout

- **Bandits cannot break blocks.** They open closed, unlocked doors in their way, climb single-block steps and use stairs; a
  locked door, bars or a wall stops them. Give a room a door or an opening toward the player's route if you want its bandits to
  come out.
- **Headroom:** a tight spiral stair with low headroom blocks them.
- **Room size:** each bandit is a full NPC (her own AI, weapons and pathing). We have not measured big rooms yet: until we have,
  prefer several rooms of up to about 8 over one room of 20 or more, and tell us your frame rate if you build big.
- **Cover:** gun bandits take cover and peek; melee bandits move from cover to cover, or charge when there is none. Furniture
  and half walls give them something to work with.

## Sentries and guards: bandits who hold a spot

A bandit from the ordinary groups has no post: once she is awake and no enemy is near, she wanders off. Two ways to make one
hold a spot:

**Sentries (sleeper volumes).** Name `GroupNPCBanditsSentry` or `GroupNPCBanditsSentryRanged` in a volume. Each sentry spawns
AWAKE on her sleeper spot, facing the way it faces, so she watches for the player with normal sight. The tier follows the
player's gamestage, as for the other groups.

**Guards (spawn blocks).** Place one of these blocks (the creative menu, search "Bandit Guard Spawn"). Each spawns one bandit,
awake, on the block when its chunk loads, facing the way the block faces. The tier is the block's, not the player's gamestage.

| Block | Bandit |
|---|---|
| `SpawnCubeBanditT1Stay` / `SpawnCubeBanditT1Guard` | tier 1 |
| `SpawnCubeBanditT2Stay` / `SpawnCubeBanditT2Guard` | tier 2 |
| `SpawnCubeBanditT3Stay` / `SpawnCubeBanditT3Guard` | tier 3 |

**Weapons.** Every sentry rolls from her tier's guns and spawns with 120 rounds, loaded and ready; the tier 1 bandit melee sentry
(`npcSdcsBanditSentryMelee`) is the one melee sentry.

Both kinds keep their spot in a fight, never leaving her spot's level and never jumping down:
- **With a gun** she steps at most about one block, for a clear shot or cover (`<Bandits><SentryLeashMetres>`), and fights from there.
- **With a melee weapon** (or a gun run dry, her club out) she is a guard at a gate: she steps out to fight an enemy within
  `<Bandits><SentryMeleeLeashMetres>` (6 m) of her spot and walks back when the fight is over. She never takes a route she cannot walk
  back up: if the way to the enemy drops off a ledge, she leaves that enemy for a while and returns to her spot.
- Knocked off, she walks back.

Out of a fight a Guard block's bandit walks a short patrol around her spot (`<Guard><PatrolMetres>`), on her own floor only, never
stepping off it; a Stay block's bandit and a sentry stand still.

**Placing a sentry.** Put her sleeper spot at least 1.5 blocks back from any drop, or give her perch a low wall or lip. Her limits
only cover her OWN moves: an enemy that climbs up and presses against her in a melee fight can push her body off the edge (the
game's own body push between two characters). She steps back to where she stood when she is shoved and walks back if she falls, but a
perch she cannot be pushed off is the reliable answer.

**Friendly sentries.** Name `GroupNPCSurvivorsSentry` in a volume for friendly survivors who hold their spot the same way (one class
per tier and sex, `npcSdcsSurvivorSentryMaleT1` .. `npcSdcsSurvivorSentryFemaleT3`, by gamestage like the resident survivors). A
friendly sentry fights any enemy within `<Bandits><SentryEngageMetres>` (20 m) of her spot, not only one that comes for her, and
never the player. She keeps her gun out while the player talks to her. When the player hires her, her post ends for good: she
follows like any hire, and dismissing her later does not post her again. A sentry is never made a POI resident.

**Your own sentries.** What makes a sentry is one class property: `<property name="SdcsSentry" value="true"/>`. Give it to your own
survivor or bandit class (it no longer needs the `bandit` tag) and any sleeper volume spawns her as a sentry.

**Your own guard blocks.** To make your own block, copy one of ours and set its `Config`: `eg=<entity group>` (or `ec=<class name>`), then `task=stay` or
`task=guard`, with `;` between the settings. Please test whether a guard keeps her post over a save and reload, and tell us.

## Loot

A dead bandit drops a loot bag for her tier (plain, blue at tier 2, red at tier 3). Half the time it also holds her weapon: at
tier 1 the weapon itself (quality 1 or 2, empty); a gun the game crafts from parts as 1 or 2 of those parts, now and then the
whole gun too. Your POI's own loot containers are not touched.

## Quests: please test

Your POI's quest tags (clear, fetch, restore_power, infested) work as the game defines them, but we have not yet tested a trader quest with
bandits as the sleepers. Please run a clear or infested quest at your POI and tell us whether killing the bandits completes it.

## Your own bandits

A bandit of your own is an entity class that extends one of ours and overrides what differs (see the MANUAL, "Your own NPC or
bandit boss"). Add it to our groups from your pack's `entitygroups.xml`, by group name:

```xml
<configs>
    <append xpath="/entitygroups/entitygroup[@name='npcBanditsAllGroupGS100']">
        <e n="myBanditRaider" p="0.004"/>
    </append>
</configs>
```

The groups are `npcBandits{All,Melee,Ranged,Boss}GroupGS{01,50,100,200,400,800}`, one per gamestage step. The weights are
relative inside a group: read ours in NPCCore's `Config/entitygroups.xml` and pick yours against them.

## Timid survivors (villagers who run)

A survivor class can be timid: she runs from an enemy she can see within 15 m (`Settings.xml <Combat><TimidFleeDistanceMetres>`) instead
of fighting, hired or not, then comes back. `npcSdcsSurvivorFemaleTimid` is a ready example you can place in a POI. To make your own
class timid, extend any SDCS survivor class and set the cvar on these three triggers (all three, or she forgets after a reload):

```xml
<effect_group name="Timid">
    <triggered_effect trigger="onSelfFirstSpawn" action="ModifyCVar" cvar="$Timid" operation="set" value="1" />
    <triggered_effect trigger="onSelfEnteredGame" action="ModifyCVar" cvar="$Timid" operation="set" value="1" />
    <triggered_effect trigger="onSelfRespawn" action="ModifyCVar" cvar="$Timid" operation="set" value="1" />
</effect_group>
```

## Characters who hide in plants

Our survivors and bandits judge sight with the game's stealth value, and each plant between them and a target lowers it
(`Settings.xml <Combat><SightPlantStealthFactor>`). A character with the cvar `$HidesInPlants` set to 1 is much harder to spot in
plants while he crouches, standing still or moving (`SightPlantStealthFactorHidden`). Standing up exposes him. Set it the same way
as `$Timid`, on all three triggers:

```xml
<effect_group name="HidesInPlants">
    <triggered_effect trigger="onSelfFirstSpawn" action="ModifyCVar" cvar="$HidesInPlants" operation="set" value="1" />
    <triggered_effect trigger="onSelfEnteredGame" action="ModifyCVar" cvar="$HidesInPlants" operation="set" value="1" />
    <triggered_effect trigger="onSelfRespawn" action="ModifyCVar" cvar="$HidesInPlants" operation="set" value="1" />
</effect_group>
```

A buff's `ModifyCVar` can set it too, so a player can carry it.

## Resident survivors (survivors who live in your POI)

Put `GroupNPCSurvivors` (NPCCore's `S_-Group_NPC_Survivors`) in a sleeper volume to place friendly survivors who live in your
POI. The tier follows the party's gamestage like the bandits (tier 1 below 100, tier 2 below 400, tier 3 from 400). Each survivor
it spawns is a resident of the POI that holds the volume: she claims the nearest free **bed** inside the POI's box as her home and
the nearest free **workstation** as her work place, one survivor per bed and per station. Any friendly survivor a sleeper volume
spawns (the Friendly groups too) counts the same; bandits never do, and survivors from the biome never move into a POI.

- Use vanilla blocks: any bedroll, bed, bunk bed, frameless bed, the flat mattress or a hospital bed for beds; a workbench,
  campfire, forge, chemistry station or cement mixer for work. The lists are `Settings.xml <Routine><BedBlocks>` and `<WorkBlocks>`.
- Give each resident her own bed and station: the second survivor takes the next free one, or goes without.
- Like every sleeper, residents spawn when a player comes near and despawn after he leaves; their beds free up for the next ones.
- Residents keep a day: by day each stands at her workstation, she heads home an hour before dusk, sleeps on her bed through the
  night and idles at home after dawn until work starts. On a blood moon night she stays home from an hour
  before dusk until dawn. The hours follow the world's dusk and dawn; the offsets are `Settings.xml <Routine>`. With no workstation
  she idles near her bed by day, with no bed near her workstation at night.
- At her workstation she faces it and works it with a hire's craft moves (a campfire kneeling, a workbench or forge hammering,
  a chemistry station or cement mixer working it) for a spell, then takes a break nearby, then works again
  (`Settings.xml <Routine><WorkSeconds>`, `<BreakSeconds>`). She crafts nothing and uses no fuel; a player can use the station
  while she works. Leave the space in front of the station free to stand in.
- She walks between her places (no teleport) and never leaves the POI's box, so put the bed and the station where a survivor can
  walk between them. A fight, an order or a chat comes first; she goes back after.
- At night she walks to the spot beside her bed and lies on it: onto a raised bed (a bed, bunk bed, gurney or hospital bed) with
  a lie-down, straight onto a low one (a bedroll, frameless bed or flat mattress). She gets up at dawn. Leave the space beside
  the bed free to stand in. With no free bed she sleeps on a bedroll from her pack where she stands (nothing a player can take).
  A hostile she sees or hears close by, an attack or an order gets her up at once; talking to her wakes her. The bed heights are
  `Settings.xml <Routine><BedTopMetres>` (a modded bed needs its own entry; one that lies her the wrong way round takes a yaw:
  `family:top:180`).
- Check the claims and her day with `sdcsresident status`; `sdcsresident phase work|home|sleep` skips the wait for dusk (below).

## Testing tools (the F1 console)

| Command | What it does |
|---|---|
| `tppoi` | the POI teleporter: pick your POI |
| `gamestage` | your gamestage (it picks the bandits' tier) |
| `giveselfxp 100000` | levels you up (repeat until `gamestage` reads 100+ for tier 2, 400+ for tier 3) |
| `settime <day> 8 0` | jumps the clock; days survived count toward gamestage too |
| `sdcsbandit` | lists the bandits in the world, with their class |
| `sdcsbandit group 2` | spawns a tier 2 group beside you (or `3`) |
| `sdcsresident` | makes the survivor you look at a resident of the POI she stands in; `sdcsresident status` shows her POI, bed and workstation claims and her day, `sdcsresident clear` undoes it |
| `sdcsresident phase work` | forces her day's phase (`work`, `home`, `sleep` or `wake`): she walks to that phase's place (`sleep`: she lies down on her bed once there); `sdcsresident phase auto` hands her back to the clock |

## Reporting a problem

1. Turn on the detailed log: `Mods/0-XNPCCore/Config/Settings.xml`, set `<VerboseLogging>true</VerboseLogging>`, start the game.
2. Play the test, quit, and copy `Player.log` straight away from
   `%USERPROFILE%\AppData\LocalLow\The Fun Pimps\7 Days To Die\` (the game keeps only the last two sessions).
3. Send it with your POI's name, the room, your gamestage and what the bandits did.
