# NPCCore — XML Modder's Guide

This guide is for people who want to customize NPC personalities, voice commands, and system settings without touching C# code. All customization happens in the `Config/` folder.

---

## How the Pieces Connect

```
entityclasses.xml (NPCCore)         personalities.xml
  <property name="personality"   →   <Personality id="trader_gruff">
            value="trader_gruff"/>        defines traits, voice, mode, language
                                          
phrasetriggers.xml                  modconfig.xml
  instant voice commands          →   LLM/STT/TTS/RAG system settings

SupertonicConfig.xml
  voice map (read by BOTH TTS engines) + Supertonic model paths + ORT settings
  (the paths/ORT part is read by supertonic-server.exe only)
```

When a player speaks to an NPC, the mod:
1. Checks `phrasetriggers.xml` for an instant match (no AI needed)
2. If no match, sends the text to the LLM with the NPC's personality from `personalities.xml` injected into the system prompt
3. TTS speaks the response using the voice assigned in `personalities.xml`

---

## TTS Engine

Two TTS server binaries ship, and both run the **same Supertonic 3 voice model** - the `<TtsEngine>` value names the binary, not the voices. **audiocpp is the default** (the shipped `modconfig.xml` sets it, since 3.2.00):

| `<TtsEngine>` | Server | Port | Notes |
|---------------|--------|------|-------|
| `audiocpp` (default) | `audiocpp_server` (audio.cpp, Vulkan with CPU fallback) | 5057 | The default engine since 3.2.00. Synthesises a command reply in 85-177 ms (measured 2026-08-28). |
| `supertonic` | `supertonic-server` (Supertonic 3, ONNX) | 5054 | The earlier engine, still fully supported. Switch by setting this one line and restarting. Both engines read the same `SupertonicConfig.xml` VoiceMap. |

Any other value (e.g. the retired `sherpa`) loads with a warning and is normalised to `audiocpp`, so old configs never crash. Deleting the line gives the same result (the code default is `audiocpp`); the shipped `modconfig.xml` ships `audiocpp`.

### Engine Config File

The server binaries read their own config (not the mod):

- **`SupertonicConfig.xml`** — the voice alias-to-file map (read by BOTH engines as their shared source of truth), plus Supertonic model paths, ORT threads and denoising steps (supertonic-server only)
- **`audiocpp-server.json`** — audio.cpp: written by the mod at boot. Nothing to edit by hand.

(`ServerConfig.xml`, the legacy Kokoro/sherpa engine file, is no longer shipped: it was removed in the 2026-08-29 content audit because nothing the mod starts reads it.)

---

## Personality Assignment — How NPCs Get Their Personality

Personalities are assigned in priority order at NPC spawn:

**Tier 1 — Explicit (highest priority)**
Add a `personality` property to the entity class in `entityclasses.xml`:
```xml
<entity_class name="npcTraderHugh">
    <property name="personality" value="trader_gruff"/>
</entity_class>
```
The value can be a personality `id` (`trader_gruff`) or display `Name` (`Gruff Trader`) — both work.
You can also list multiple IDs separated by commas — one is chosen randomly on spawn:
```xml
<property name="personality" value="trader_gruff,trader_friendly,trader_mysterious"/>
```

**Tier 2 — Entity class name match (automatic)**
If no `personality` property exists, the entity class name is checked:
- Contains `trader` -> picks a trader personality
- Contains `bandit` or `hostile` -> picks a bandit personality
- Contains `companion`, `hire`, or `follow` -> picks a companion personality

**Tier 3 — Random fallback**
If no match, any personality not marked `excludeFromRandom="true"` is picked randomly.
Use `excludeFromRandom="true"` on personalities that should only be assigned explicitly (e.g., non-American personalities, very specific characters).

---

## Silencing a Character (the `no_voice` tag)

To keep one of your characters completely out of Voice Control — no TTS, no pre-recorded clips,
no subtitles, no greetings, no voice commands, no wake-word targeting — add the `no_voice` tag
to the character's `Tags` property in `entityclasses.xml`:

```xml
<entity_class name="myQuietSurvivor" extends="npcMeleeTemplate">
    <property name="Tags" value="entity,male,npc,walker,melee,no_voice"/>
</entity_class>
```

Everything else about the character keeps working exactly as before: hiring, following,
guarding and combat through the NPC Mod's own dialog (hire / follow / guard / dismiss)
are not Voice Control surfaces and are unaffected. Voice Control simply never sees the
character — the same result you get today when an entity class carries no `npc` hint in
its type or class name, but deliberate.

**One exception, visible in that same dialog:** the three *patrol* rows (Show me a route /
Patrol your route / Cancel patrol) are **Voice Control** rows, not NPC Mod rows — this mod
added them and removed the NPC Mod's legacy patrol rows. In the SDCS survivors' dialog they
sit under the Command Menu's single **Patrol** row (a one-row deep menu); the generic NPC
dialog keeps them flat in the Command Menu. They are hidden for a `no_voice`
character because patrol cannot work without the Voice Control component the tag withholds.
A tagged character can be hired and ordered normally, just not put on a recorded patrol.
Ordered to patrol with no recorded route, or with a route recorded somewhere far away, she says so instead of
acknowledging, and keeps her current order: show her a route where you are first.

**Notes:**
- The `Tags` property **re-declares rather than appends**: copy the full tag list your template
class uses and add `no_voice` to it. A child class that omits `Tags` inherits the parent's
list — so a template with `no_voice` silences every child that does not re-declare.
- Matching is exact and case-insensitive (`no_voice`, `No_Voice`); longer tags such as
  `no_voice_extended` do NOT trigger it.
- The tag only affects Voice Control; other mods that read `Tags` see the same list.

---

## Quick Recipe: Add a New NPC Personality

**Step 1 — Add to `personalities.xml`**
```xml
<Personality id="scavenger_cynical">
    <Name>Cynical Scavenger</Name>
    <Traits>You're a world-weary scavenger who's seen everything. You speak in clipped sentences,
    distrust strangers on principle, and have a dark sense of humor about the apocalypse.
    You've survived this long by trusting no one.</Traits>
    <MaleVoice>M1</MaleVoice>
    <FemaleVoice>F4</FemaleVoice>
    <TTSLanguage>en</TTSLanguage>
    <Greetings>
        <Greeting lang="en">What do you want.</Greeting>
        <Greeting lang="en">You're not dead yet. Good enough.</Greeting>
    </Greetings>
</Personality>
```

**Step 2 — Wire it to an NPC in `entityclasses.xml` (NPCCore repo)**
```xml
<entity_class name="npcScavengerCynical">
    <property name="personality" value="scavenger_cynical"/>
</entity_class>
```

Or skip Step 2 to let it auto-assign via entity class name matching or random fallback.

**Notes:**
- `<Traits>` is injected directly into the LLM system prompt — write it as a character description in second person ("You are...")
- `<Greetings>` are pre-generated at 15m radius before the player arrives. Multiple greetings rotate randomly.
- Voices must match aliases defined in `SupertonicConfig.xml`. The current convention is to name a voice directly - one of the ten originals (`M1`-`M5`, `F1`-`F5`), one of the forty VoiceLab pack voices (`P00`-`P39`), or one of the friendly names - that is what the shipped personalities do. Kokoro-era aliases (af_*, am_*, ...) were retired on 2026-09-05 and no longer resolve: a personality that still names one logs a warning and the NPC speaks with the default voice (or, on the default `audiocpp` engine, does not speak that line). See [Voice Reference](#voice-reference).
- `<MaleVoice>` and `<FemaleVoice>` are chosen based on the NPC's gender tag in entityclasses.xml
- **A random voice:** `<MaleVoice>random</MaleVoice>` (or `<FemaleVoice>random</FemaleVoice>`) gives each NPC one of the voices of her own sex that this install can speak (the VoiceMap rows with a `sex` attribute, see [Custom voices](#custom-voices)), picked from her entity id: she keeps it every time she speaks, and every machine with the same voices picks the same. The bandits use it. Name a voice instead to pin one.
- **Optional: CombatLines** — `<CombatLines><Line>Time to die.</Line> ...</CombatLines>`: fixed lines a bandit says, unchanged, the moment she locks onto a person to fight (see [Bandits](#bandits)). Only a bandit says them; text to speech only, English. The shipped bandit personalities are `bandit_cold_male` and `bandit_cold_female`.
- **Optional: VoiceMode** — Set `<VoiceMode>clips</VoiceMode>` to use pre-recorded OGG clips instead of TTS. See [Voice Clips Mode](#voice-clips-mode) below.
- **Optional: ForceLanguage** — Set `<ForceLanguage>ja</ForceLanguage>` (or any ISO code) to force this NPC's subtitle language regardless of the player's game language setting. Every caption she shows follows it: her replies, order acknowledgements and canned lines ("I'm done crafting."). An item named in a line ("I need Wood for that.") keeps the name the player's game shows.
- **Optional: Subtitles** — What text this character shows on screen: `all` (every line: replies, chat, order acknowledgements, clip captions), `clip` (only a voice clip's own caption, never the generic reply text) or `none` (no text at all). Omit it for the default: `clip` for a clips character, `all` for everyone else. `modconfig.xml` `<Subtitles><NPCText>` caps every character at once (the stricter wins); see the Subtitles section.

---

## Quick Recipe: Add a New Voice Command

Add a `<Trigger>` block inside `<Triggers>` in `phrasetriggers.xml`:

```xml
<Trigger enabled="true">
    <Phrase>i need a medkit;do you have a medkit;got any medical supplies;</Phrase>
    <Aliases>medkit;bandage;medical;</Aliases>
    <Response lang="en">Here, take this. Don't make a habit of it.;Last one I've got.;Catch.</Response>
    <ActionType>GiveItem</ActionType>
    <ActionParam>medicalFirstAidKit</ActionParam>
    <ActionParamCount>1</ActionParamCount>
</Trigger>
```

**Phrase matching — how "follow me" matches "hey can you follow me please":**
- Each `<Phrase>` entry is checked as a case-insensitive substring
- `<Aliases>` are checked first — shorter keywords for STT variations ("follow" matches "uh can you follow me")
- STT transcription errors get fuzzy matching (1-character Levenshtein tolerance)

**Multiple responses** — separate with `;`. One is chosen randomly each time the trigger fires.

**Leader requirement** — most movement/inventory actions require the player to have hired the NPC. See the Action Types table below.

---

## Localized Responses and Greetings (Dubbed Architecture)

The mod uses a **dubbed architecture**: TTS audio is always English regardless of the NPC's language setting. The `lang` attribute on `<Response>` and `<Greeting>` controls **subtitle text only**, driven by the player's game language.

```xml
<Response lang="en">Right behind you.</Response>
<Response lang="de">Verstanden, ich folge dir.</Response>
<Response lang="ja">了解、ついていくよ。</Response>
```

**Supported subtitle languages (auto-detected from game settings):**

| Game Language | ISO Code | Example |
|--------------|----------|---------|
| English | `en` | Default — always present |
| German | `de` | Verstanden, ich folge dir. |
| Spanish | `es` | Entendido, te sigo. |
| French | `fr` | Compris, je te suis. |
| Italian | `it` | Chiaro, ti seguo. |
| Portuguese | `pt` | Entendido, vou com você. |
| Japanese | `ja` | 了解、ついていくよ。 |
| Korean | `ko` | 알겠습니다, 따라갈게요. |
| Chinese (Simplified) | `zh` | 明白，我跟着你。 |
| Russian | `ru` | Понял, я пойду за тобой. |
| Turkish | `tr` | Anladım, seninle geliyorum. |
| Polish | `pl` | Zrozumiałem, idę za tobą. |

If no matching `lang` entry exists for the player's language, the system falls back to `lang="en"`.

**For greetings**, same rule applies:
```xml
<Greeting lang="en">I am ready. What is your purpose here?</Greeting>
<Greeting lang="de">Ich bin bereit. Was ist dein Zweck hier?</Greeting>
```

**Stories have no `lang` support** — all `<Story>` entries are treated as English-only. Adding `lang` attributes to `<Story>` elements has no effect currently.

---

## Action Types Reference

| ActionType | Requires Leader | Description |
|-----------|----------------|-------------|
| `None` | No | Dialog only, no game action |
| `GiveItem` | Yes | Give item to player. Set `<ActionParam>` to ItemClass name, `<ActionParamCount>` to quantity |
| `GiveBuff` | Yes | Apply buff to player. Set `<ActionParam>` to buff name |
| `CustomDialog` | No | Show dialog. Leave `<Response>` empty to pick a random story from `<Stories>` |
| `Hire` | No | Opens the hire dialog window |
| `OpenTraderInventory` | No | Opens trader's shop (simulates pressing E) |
| `JumpDown` | Yes | Walk off the nearest ledge (UAI task UAITaskJumpDownSDX) |
| `FollowPlayer` | Yes | NPC follows the player |
| `StopFollow` | Yes | NPC holds position |
| `GuardHere` | Yes | NPC guards player's current position |
| `GuardReturn` | Yes | NPC guards and returns to player position |
| `Wander` | Yes | NPC patrols the area |
| `Loot` | Yes | NPC scavenges the area |
| `BaseRepairHere` | Yes | The base keeper's standing duty: keeps the leader's base repaired, most damaged block first (SdcsLootMode 8). Voice: "base repair", "repair the worst first"; wheel: Jobs > Base Repair |
| `CombatRepairHere` | Yes | The same standing duty, a block damaged in the last few seconds first (SdcsLootMode 9). Voice: "combat repair", "repair under fire"; wheel: Jobs > Combat Repair |
| `RepairHere` | Yes | Plain "repair the base" / "fix the base": the base mod's `Settings.xml` `<BaseKeeper><DefaultMode>` (`base` or `combat`) picks the duty (SdcsLootMode 8 or 9) |
| `SetPatrolPoint` | Yes | Start recording patrol route (SCore ExecuteCMD "SetPatrol") |
| `Patrol` | Yes | Start looping recorded patrol route (SCore ExecuteCMD "Patrol") |
| `CancelPatrolRecord` | Yes | Cancel recording / stop active patrol, clear points, order→Stay |
| `Dismiss` | Yes | Dismisses the NPC |
| `Pickup` | Yes | Picks up the NPC (SCore PickUpNPC). Refused while her pack holds anything, the way a vehicle with items in its storage is |
| `OpenInventory` | Yes* | Opens NPC's trade/inventory window. *Traders are the one exception: opening a trader's inventory needs no leader (it redirects to the trade dialog). |
| `SwapWeapon` | Yes | Swaps NPC weapon. See SwapWeapon section below. |
| `ReadBooks` | Yes | She reads every schematic and crafting skill magazine she carries that teaches her something, then says one line ("read your books"). No `<ActionParam>`. See *Hired survivors: reading books* |
| `Emote` | Yes | She plays a clip of the base mod's clip list (`Config/SdcsAnimClips.xml`) where she stands. `<ActionParam>` = the clip id (`wave`, `salute`, `fistbump`...), or `dance` for a random dance. A squad order makes each member play it. Keep an emote phrase to one content word plus the matcher's stop words ("do a dance", "salute me"), so it fires only when the whole phrase is said. See *Hired survivors: emotes* |
| `SetEngage` | Yes | Engagement. `<ActionParam>` = 0=cover me (defensive, default), 1=weapons free, 2=hold fire, 3=disengage action. |
| `SetTacticalMode` | Yes | Player-wide tactical mode toggle, but it still fires off a hired companion, so a leaderless NPC refuses it. `<ActionParam>` = 0=off (chat+commands), 1=on (command-only). |
| `SetCrouch` | Yes | Crouch override cvar. `<ActionParam>` = 0=mirror leader, 1=crouch, 2=stand. The order wheel's Orders page carries the 0/1 toggle (Crouch/Resume) plus a `Stand up` socket - see the Order Wheel section. |
| `SetFollowDistance` | Yes | Mode-aware follow distance. `<ActionParam>` = 1 (close), 2.5 (normal), 5 (far). Adjusts plain-follow min or formation socket radius depending on mode. |
| `SetFormation` | Yes | Set formation position. `<ActionParam>` = `abs:<bearing>` (e.g. `abs:0` north, `abs:90` east) or `0,0` to cancel. |
| `SetCombatMode` | Yes | Set combat behavior. `<ActionParam>` = 0, 1, or 2. **DEFERRED** — triggers disabled, code dormant for future richer-mode work. |
| `MoveToPosition` | Yes | Send NPC to a world position resolved from the player's focus raycast ("go there"). `<ActionParam>` is unused — destination is computed from where the player is looking (30 m range - the engine's own focus-ray limit, not a tunable). Switches NPC to Follow order with a positional target.

### SwapWeapon — ActionParam and ActionParam2

Weapon swapping is fully driven by the XML trigger — **no C# change is needed to add a weapon** (the old hardcoded `WeaponTagToItem` tag-to-item dictionary is gone):

- `ActionParam` — the **player item id** the NPC carries (e.g., `meleeWpnBladeT1HuntingKnife`).
- `ActionParam2` — the capability tag `sdcsSwap_<item>` on the character's `Tags` (e.g., `sdcsSwap_meleeWpnBladeT1HuntingKnife`). Present on every gated swap; absent for bare hands.

Before swapping, the mod verifies two things (see `SwapWeaponRouted` in `PhraseTriggerHandler.Actions.cs`):
1. The NPC's entity class has the `sdcsSwap_<item>` tag (SCore `HasAnyTags`, same source the SdcsNPC dialog's HasTag requirement reads).
2. The NPC actually holds the `ActionParam` item in their inventory.

If either check fails, the NPC responds "I can't use that kind of weapon" / "I don't have that weapon on me." rather than silently failing.

```xml
<ActionType>SwapWeapon</ActionType>
<ActionParam>meleeWpnClubT0WoodenClub</ActionParam>          <!-- player item id (required) -->
<ActionParam2>sdcsSwap_meleeWpnClubT0WoodenClub</ActionParam2>  <!-- tag on the character's Tags -->
```

To add a new weapon: add the `sdcsSwap_<item>` tag to the character's `Tags` (and the SdcsNPC dialog line) plus a new trigger block in phrasetriggers.xml. The 44 SwapWeapon triggers today cover the full SDCS weapon roster including the 20 player weapons (bone knife, steel club, batons, knuckles, steel axe, chainsaw, sledgehammers, spears, magnum, lever action rifle, double barrel, and bows).

### Engagement Model (SetEngage)

Three persistent stances plus one action. Controls hired NPC autonomous combat via the `vcEngage` cvar and `MayEngage` UAI gating.

| ActionParam | Mode | Behavior |
|------------|------|----------|
| `0` | Cover Me (defensive, default) | Fights what threatens the leader or a squad mate, and anything within the contact radius (`EngageContactRadius`, default 2.5 m) that is literally on top of you. One holding a gun also fights any enemy she can see within `EngageNearRangedRadius` (20 m) of herself, whatever it targets, from where she is. May walk toward an already-authorised enemy within the 8 m advance leash, but holds position instead of chasing. Following you, she takes cover or moves for a shot only at a spot you can see from where you stand, so she never slips into a building ahead of you; with none she crouches where she is. (Hold fire too; Aggressive is not limited.) |
| `1` | Weapons Free (aggressive) | Hunts and attacks any enemy on sight; closes freely. |
| `2` | Hold Fire | Self-defense only. No leader-assist. Clears attack targets when set. (Renamed from "full control" on 2026-08-26; "full control" remains a legacy spoken alias for this same stance.) |
| `3` | Disengage (ACTION, not a stance) | Breaks off now: clears targets, cancels formation, clears any commanded position, returns to Follow, and sets the stance to `2` (hold fire). For the break-off window (`DisengageWindow`, default 20 s) the NPC still defends himself against a live enemy, but walks away the moment that enemy is stunned, ragdolled or dead. |

**Self-defense outranks every stance** — the gate matches `revengeTarget` against the enemy being scored, so an NPC always fights back against whatever damages him, but Hold Fire does not degenerate into free-fight when it happens.

**Engagement is latched.** Once authorized against a specific enemy, an NPC stays authorized against that enemy until it dies, leaves `ChaseLeash` (20 m from the leader), or the stance changes. Without this an assist cancels itself: damaging an enemy makes it retarget the NPC, which invalidates the leader-assist condition that permitted the attack.

Only ActionParam `3` touches formation, orders or commanded positions. Stance changes are combat-only.

### Tactical Mode (SetTacticalMode)

Player-wide toggle that switches between chat+commands and command-only mode.

| ActionParam | Mode | Behavior |
|------------|------|----------|
| `0` | Normal | Full chat + commands. Unrecognized phrases route to LLM conversation. |
| `1` | Tactical | Command-only. Unrecognized phrases get a rotating clarification ("Say again?") instead of LLM chat. Suppresses proactive greetings and RAG flush. |

### Crouch Override (SetCrouch)

Sets the `CrouchOverride` cvar on hired NPCs. Active — SCore ships the hook at both follow and idle chokepoints.

The **voice path accepts all three values**:

| ActionParam | Mode | Behavior |
|------------|------|----------|
| `0` | Mirror Leader | Resume mirroring the player's stance (default) |
| `1` | Crouch | Force NPC to crouch |
| `2` | Stand | Force NPC to stand |

The **order wheel shows only a 0/1 toggle**: the crouch socket reads `Crouch` (arg 1) unless the target is currently crouch-locked, in which case it reads `Resume` (arg 0). There is deliberately no Stand socket - resuming leader mirroring while you are standing already stands the NPC up, so 0/1 are the only two states worth flipping between. Stand (arg 2) is voice-only (`stand up`).

---

## Order Wheel (Formation Radial)

Press the radial key (default **Semicolon**) to open the order wheel; it stays open without holding the key. Click an order for the survivor it addresses, **Shift+click** for the whole squad (a squad-capable order), click a page entry to drill down, click the centre to go back a page; the key again, a right click or Esc closes it. Hovering never commits. It is a multi-page radial, not the old single 8-socket formation picker. It is a menu feature: it works with STT/TTS/LLM all off.

**Targeting.** The wheel commands the companion you are looking at, otherwise the last one you gave an order to, otherwise the nearest. Their name sits in the middle of the ring and the mouse wheel cycles through your hires.

**From the E ring.** Hold **E** on one of your hires and the ring shows her own menu after Talk: **Orders**, **Jobs**, **Roles**, **Weapons**, **Inventory** and **Craft** (Jobs, Roles and Craft only on a survivor who can do that work). Pick an entry by resting the cursor on it, or by letting go of E over it; a quick tap of E still talks. **Orders** opens the squad wheel on its first page, on her, and E works like the radial key until it closes: click an order for her, Shift+click for the whole squad, E again closes it. **Jobs**, **Roles** and **Weapons** open the wheel straight on that page, locked on her: every pick is hers, the mouse wheel does not switch to another hire, and resting on the centre steps back up her pages, then closes the wheel. **Craft** opens her craft window (see *Hired survivors: crafting*). **Inventory** opens her pack. A wheel opened from the ring ignores the spot your cursor was resting on when it appeared: move onto a socket to pick it. Only her leader sees these entries. (The game's own **remove** entry still shows only with the debug menu on.)

**Pages.** The wheel always opens on page 0:

| Page | Sockets |
|------|---------|
| **0 (root)** | `Follow`, `Stay`, `Guard`, `Come here` commit directly; `Orders`, `Stance`, `Position`, `Formation` open a drill-down page; the `Stance` page opens a third level (Weapons). Jobs, Roles and Craft are one survivor's menus: they open from her hold-E ring (above), not from this wheel |
| **Orders** | `Emotes` (drill-down; moved here from the root when the root filled up), `Crouch` (the 0/1 toggle - Crouch / Resume resolved from her state, exactly as before), `Stand up` (the voice `stand up`, SetCrouch arg 2), `Scout` (the voice wander order), `Guard and return` (patrol the spot: walk out within `<Guard><PatrolMetres>` (6 m), look around, come back, face your way, repeat; fight freely, then back to the spot), `Record patrol` / `Run patrol` / `Cancel patrol` (the patrol trio - one survivor's route; a Shift+click goes to the target, not the squad) |
| **Roles** (from her ring) | One survivor's roles, each a toggle that reads **Stop ...** while she has it (target only, never the squad): `Medic` (the designated medic role), `Cook` (the campfire), `Blacksmith` (the forge), `Chemist` (the chemistry station), `Crafter` (the workbench and cement mixer). A survivor uses a workstation only with its role - see *Hired survivors: crafting* |
| **Craft** (from her ring) | Not a wheel page: it opens her craft window, the same window as **E > Craft...** (see *Hired survivors: crafting*) |
| **Jobs** (from her ring) | `Follow + loot` (starts **follow and loot**, exactly the voice "loot" command - see *Hired survivors: follow and loot* below), `Loot here`, the `Harvest` and `Mine` drill-downs (the wood / plants / crops / butcher orders and the mine / clay orders), the two base-keeper duties `Base Repair` (most damaged block first, the wheel arg `baserepair`) and `Combat Repair` (a block damaged in the last few seconds first, the wheel arg `combatrepair`), and `Remember base` |
| **Stance** | `Aggressive` (1), `Defensive` (0), `Hold fire` (2), `Disengage` (3) - the four SetEngage args verbatim - plus the `Weapons` drill-down |
| **Weapons** | `Guns`, `Rifles`, `Shotguns`, `Machine guns`, `Melee`, `Blades`, `Empty hands`, `Ammo` (her ammo types for the gun in her hand with their counts, the loaded one marked; pick one and she loads it, the same as the talk dialog's Ammo page) - the voice weapon-category swaps, the same resolved swap as speaking them (she equips the first weapon of that category she actually carries, and says so; a Shift+click goes to the target, not the squad) |
| **Position** | `Close` (1 m), `Normal` (2.5 m), `Far` (5 m) - the real metres are drawn in the icon slot - `Point` (the voice "take point": the companion leads 2 m ahead of you), `Go there`, `Jump down`, `Get in` / `Get out` (your vehicle). **Look first, then open the wheel:** `Go there` goes to the spot under your crosshair *when the wheel opened*, not where you aim when you pick it. |
| **Formation** | The 8 compass bearings, N at top and E to the right. Direction only: each NPC keeps its own distance |
| **Emotes** | `Wave`, `Salute`, `Fist bump`, `Stretch`, `Warm up`, `Whatever`, then the `Dances` drill-down (seven dances a page plus More) and the `More` drill-down (`Flip off`, `Insult`, `Loser`, `Threaten`). Click for the target, Shift+click for the whole squad. Only the clips the base mod's clip list holds show - see *Hired survivors: emotes* |

**Commit vs navigate.**
- **Release the key** while hovering a commit socket - the order goes to the **targeted companion**.
- **Click** while still holding the key - the order goes to the **whole squad** (every formation socket is squad-capable; a non-squad socket falls back to the target, so a click always does something).
- **Navigation sockets** (the drill-downs) never commit. They are marked with a small chevron and the navigation border tint. Open one by clicking it, or simply by **resting the cursor on it for the dwell** (0.4 s default) - a faster pass over it does nothing, so a transit can never flip the page by accident.
- **Back out:** rest the cursor on the wheel's **centre** and it returns to page 0. Clicking or releasing over the centre does nothing.

**Stay, Guard, Guard and return.** `Stay` holds her where she stands; `Guard` holds her on the spot where you stand. On
both she shoots from her spot and never backs off or runs in to melee (the AI shows the `Stay` action for both). `Guard and
return` patrols your spot: out to a random spot within 6 m, a look outward, back to the spot facing your way, again; in a fight
she moves freely, then goes back to the spot first (the AI shows the `Guard` action).

**Guarding together.** Survivors told to guard the same spot (a squad click on `Guard`, or Guard given to one after another
from where you stand) spread out instead of crowding onto it: the later ones step aside into a line beside the first, facing your
way, then a rank behind, `<Guard><SpacingBlocks>` (2) blocks apart, on spots they can stand on and walk to. A job's post (loot
here, a station, the base) is never moved.

**A recorded patrol across a save.** A save taken while she is recording a patrol brings her back following; the points recorded before the save are kept, so say `set patrol` again to continue recording (or `start patrol` to walk the points already recorded).

Every socket carries the exact router arg of the equivalent voice command, so the wheel and the voice path cannot disagree - the one deliberate exception is the crouch toggle on the Orders page, which offers Crouch **or** Resume depending on her current state (arg 1 or 0) instead of one fixed arg.

### Order Wheel settings (modconfig.xml, top-level `<FormationRadial>` section)

| Setting | Default | Description |
|---------|---------|-------------|
| `<Key>` | `Semicolon` | Unity KeyCode name that opens the wheel. Every letter A-Z has a vanilla default binding, so letters collide. Free keys: Semicolon, Quote, Comma, Period, Slash, RightBracket, Backslash. Set the key to `None` to disable the wheel (an empty value falls back to the default, Semicolon). An old `<STT><FormationRadialKey>` entry is still read as a fallback. |
| `<Dwell>` | `0.4` | Seconds the cursor must stay on a navigation socket (or the centre) before it navigates. 0 makes hover navigate on the first frame; use 0.3-1.0. |
| `<NavBorderColor>` | `80,180,255,255` | Border ring tint (r,g,b,a) for the navigation sockets (the ones carrying the chevron). |
| `<CommandBorderColor>` | `0,0,0,128` | Border ring tint for the commit sockets. Matches the window XML default, so sockets look unchanged unless you set one. |

---

## Voice Clips Mode

Instead of TTS, an NPC can play pre-recorded OGG audio clips. Set `<VoiceMode>clips</VoiceMode>` on the personality:

```xml
<Personality id="my_clip_npc">
    <Name>Clip Character</Name>
    <Traits>You speak in pre-recorded lines...</Traits>
    <VoiceMode>clips</VoiceMode>
</Personality>
```

**Clip library folder structure** (in `Resources/NPC_Voices/<FolderName>/`):

```
NPC_Voices/
  └── MyCharacter/
      ├── Greetings/
      │   └── Hello.ogg
      ├── Default/
      │   └── Laugh01.ogg
      └── Triggers/
          ├── followplayer/
          │   └── Onmyway.ogg
          ├── dismiss/
          │   └── Byebye.ogg
          └── ... (one folder per trigger action name)
```

**How clips are matched:**
- **Greetings**: Files in `Greetings/` play when the NPC first sees the player. Multiple files rotate randomly.
- **Triggers/<action>/**: When a phrase trigger fires, the mod looks for OGG files in the matching action folder (e.g., `followplayer/`, `dismiss/`). One file is picked randomly.
- **Default/**: Fallback clips when no specific clip exists for the action.

**Notes:**
- Clips bypass the LLM entirely — no AI conversation, only pre-recorded lines
- Each OGG can have an optional `.txt` sidecar for subtitles (e.g., `Hello.ogg` + `Hello.txt`)
- For subtitles in every language, add a `Localization.csv` row keyed `<voice folder>_<file name>` (e.g. `MyNpc_Hello` for
  `NPC_Voices/MyNpc/.../Hello.ogg`). The caption order is that row, then the `.txt` sidecar, then a row keyed by the bare file
  name (the old scheme, still read), so another pack's `Hello` can never caption your clip over your own sidecar
- A clips character shows only her clips' own captions by default (`<Subtitles>clip</Subtitles>`): the generic text for an order ("On my way") is not shown, because it would not match the recorded line. A clip without a caption plays with no text. Set `<Subtitles>none</Subtitles>` for no text at all, or `<Subtitles>all</Subtitles>` to show the generic text as well
- Clip libraries are discovered from all loaded mods' `Resources/NPC_Voices/` folders

---

## TTSLanguage Codes (ISO)

Set in `<TTSLanguage>` inside each `<Personality>`.

**Spoken audio is always English (dubbed architecture).** The LLM is instructed to keep `spoken_text` in English regardless of the personality or this code; the player's UI language is carried in a separate `subtitle_localized` field. This setting therefore no longer changes the language the NPC *speaks*, and it does not pick the voice - the voice comes from `<MaleVoice>`/`<FemaleVoice>` (see [Voice Reference](#voice-reference)).

What it still does:
- On the `supertonic` engine it is forwarded as the `lang` parameter (normalised, e.g. `en-GB` to `en`). The default `audiocpp` engine sends no language parameter.
- It picks the placeholder-fallback line the mod substitutes if the LLM returns garbage: `ja` gets a Japanese line, everything else English.

The ISO codes historically recognised are `en`, `en-GB`, `zh`, `ja`, `es`, `hi`, `en-IE`, `fi`, `pl`. Keep `en` unless you have a reason; it is the safe default. (Native non-English speech is not active; when it lands, `TTSLanguage` is the code that will re-drive it.)

---

## Voice Reference

### Voice aliases (all in the `SupertonicConfig.xml` VoiceMap)

The authoritative, generated list of every shipped voice and which personalities use it is `docs/VOICES.md` (regenerated by `tools/gen-voices-doc.js`; not shipped in the mod folder). The aliases below are the ones a personality's `<MaleVoice>`/`<FemaleVoice>` may name, and they are the same on both TTS engines:

- **Ten originals** - `M1` `M2` `M3` `M4` `M5` (male), `F1` `F2` `F3` `F4` `F5` (female).
- **Forty VoiceLab pack voices** - `P00` through `P39` (added 2026-09-05).
- **Friendly names** - one human-readable alias per pack voice (e.g. `Nate` to `P00`, `Nora` to `P01`, `Daisy` to `P02`, ... `Dylan` to `P39`). The full list is in the VoiceMap.
- **Legacy first names** - `Alex`, `James`, `Robert`, `Sam`, `Daniel`, `Sarah`, `Lily`, `Jessica`, `Olivia`, `Emily`, each mapping to one of the ten originals.

**Retired on 2026-09-05:** the 54 Kokoro-era aliases (`af_*`, `am_*`, `bf_*`, `bm_*`, `zf_*`, `zm_*`, `jf_*`, `jm_*`, `ef_*`, `em_*`, `hf_*`, `hm_*`, `if_*`, `im_*`, `ff_*`, `pf_*`, `pm_*`) were **removed from the VoiceMap** and no longer resolve. A personality or hand-edited config that still names one does not crash: the mod logs a warning naming the personality, the alias and the voice actually used, and the NPC speaks with the default voice - or, on the default `audiocpp` engine, refuses to speak that line. Update such configs to one of the aliases above.

### Custom voices

Create a custom voice with the **Voice Builder** (`NPCCore-VoiceBuilder-<version>.zip`, a separate download on the mod's page): it trains a voice from a recording and installs it for you. The Voice Builder needs the Complete add-on (the Voice add-on does not include the speech model it trains against). By hand: copy the voice's JSON into `bin/AudioCppServer/voice_styles/` of your add-on (the default engine reads only this folder: a voice missing from it is rejected at startup, `[TTS-AUDIOCPP] VoiceMap row unusable` in the log) and also into `Resources/Supertonic/voice_styles/` when your add-on has that folder, then register an alias in `SupertonicConfig.xml`:
```xml
<Voice alias="my_voice" file="my_voice.json" sex="female" />
```

`sex` (`male` or `female`, optional) puts the voice in that sex's pool for a personality whose voice is `random`. A voice without it is never picked at random, except the ten originals, which count by their letter (M male, F female).

`pitch` (semitones, -12 to 12, optional) and `grit` (0 to 1, optional) are per-voice effects the mod applies to every line the voice speaks:
```xml
<Voice alias="orc_embed" file="orc_embed.json" sex="male" pitch="-3" grit="0.4" />
```

- `pitch` is tape style: the pitch and the speed change together, so `pitch="-3"` is three semitones deeper AND about 16% slower (negative = deeper and slower, positive = higher and faster).
- `grit` adds a saturated, rough edge without changing the voice's loudness: 0 is none, 1 is the harshest.
- Updating the voice add-on replaces `SupertonicConfig.xml`: add your voice rows again afterwards (the Voice Builder's Install does it).

A row without the attributes is spoken exactly as before (no processing). The mod applies the effects at delivery (after synthesis, cache hit included), so every consumer - including the multiplayer relay - hears the processed voice; the values take effect at next launch.

Voice aliases must match an entry in the `SupertonicConfig.xml` VoiceMap. If a personality names a voice that is not there, it does not resolve - the mod warns loudly (as above) rather than silently giving the NPC another NPC's voice.

---

## modconfig.xml — Full Settings Reference

All runtime settings live in one file. Most can be adjusted in the in-game config menu (NPC VoiceControl Settings). Changes that take effect on next launch are noted in the UI.

### TTS Section

| Setting | Default | Description |
|---------|---------|-------------|
| `<Enabled>` | `true` | Turn TTS on/off globally |
| `<TtsEngine>` | `audiocpp` | TTS engine: `audiocpp` (default since 3.2.00, port 5057) or `supertonic` (port 5054). Any other value (e.g. the retired `sherpa`) logs a warning and is normalised to `audiocpp`, which is also the code default when the line is deleted. |
| `<TimeoutSeconds>` | `30` | How long to wait for TTS response before giving up |
| `<UseGpu>` | `false` | Legacy flag, no longer used. Kept so old configs load cleanly |
| `<Volume>` | `0.8` | Playback volume (0.0-1.0). A value saved in the settings window overrides this file value at launch |
| `<MaxDistance>` | `40` | Maximum distance NPC voice can be heard (meters) |
| `<MinDistance>` | `2` | Minimum distance for volume falloff (meters) |
| `<SpeechRate>` | `1.0` | Playback speed multiplier (0.5-2.0). A value saved in the settings window overrides this file value at launch |

#### Voice Defaults (overridden per-NPC by personalities.xml)

| Setting | Default | Description |
|---------|---------|-------------|
| `<DefaultVoice>` | `F4` | Fallback voice when no personality voice is assigned |
| `<TraderVoice>` | `M1` | Default voice for trader NPCs |
| `<CompanionVoice>` | `F5` | Default voice for companion NPCs |
| `<BanditVoice>` | `M3` | Default voice for bandit NPCs |

#### Face Lip-Sync (blendshape driver)

| Setting | Default | Description |
|---------|---------|-------------|
| `<FaceLipSyncEnabled>` | `true` | Enable face lip-sync |
| `<FaceLipSyncMode>` | `auto` | Mode: `auto`, `off`, `blendshape`, `animator`, `procedural` |
| `<FaceLipSyncGain>` | `7` | Blendshape weight gain |
| `<FaceLipSyncAttack>` | `35` | Attack time (ms) |
| `<FaceLipSyncRelease>` | `12` | Release time (ms) |
| `<FaceLipSyncNoiseGate>` | `0.015` | Noise gate threshold |
| `<FaceLipSyncMaxWeight>` | `100` | Maximum blendshape weight |
| `<FaceLipSyncBlinkEnabled>` | `true` | Enable procedural blinking |
| `<FaceLipSyncBlinkIntervalMin>` | `2.0` | Min blink interval (seconds) |
| `<FaceLipSyncBlinkIntervalMax>` | `6.0` | Max blink interval (seconds) |
| `<FaceLipSyncBlinkDurationMs>` | `120` | Blink duration (ms) |
| `<FaceLipSyncAnimParam>` | `IsTalking` | Animator float parameter name (animator mode) |

#### Head Gestures (nod on accept, shake on reject)

| Setting | Default | Description |
|---------|---------|-------------|
| `<HeadGesturesEnabled>` | `true` | Enable head nod/shake gestures |
| `<HeadGestureParam>` | `IdleVar` | Animator parameter name |
| `<HeadGestureHoldSeconds>` | `1.5` | How long to hold the gesture |

### STT Section

| Setting | Default | Description |
|---------|---------|-------------|
| `<Enabled>` | `true` | Turn STT on/off globally |
| `<ServerUrl>` | `http://127.0.0.1:5052` | Whisper-server endpoint |
| `<TimeoutSeconds>` | `60` | How long to wait for transcription |
| `<BeamSize>` | `1` | Recognition accuracy (1-5, higher = more accurate but slower). A value saved in the settings window overrides this file value at launch |
| `<LanguageLocked>` | `false` | Force English-only recognition |
| `<Language>` | `en` | ISO language code for Whisper. Use `auto` for auto-detection (less reliable on short clips). |
| `<Model>` | `ggml-small-q8_0.bin` | Whisper model filename. One model ships: `ggml-small-q8_0.bin` (the old `ggml-base.en.bin` Fast tier and the f16 `ggml-small.bin` were removed from the package - do not point this at them). Empty = auto-detect, which prefers the shipped model, but keep the line set explicitly. |
| `<UseGpu>` | `true` | GPU acceleration for whisper-server (Vulkan) |
| `<Translate>` | `false` | Translate non-English speech to English text |
| `<PushToTalkKey>` | `V` | Key held while speaking |
| `<WakeWordEnabled>` | `false` | Hands-free wake-word mode. Off by default. Uses extra CPU while listening. |
| `<WakeWordModel>` | `hey_marvin_v0.1` | Wake-word model name (must exist in `Resources/WakeWord/models/`) |
| `<WakeWordThreshold>` | `0.5` | Sensitivity 0-1 (higher = fewer false triggers) |
| `<VadSilenceMs>` | `400` | Milliseconds of silence before recording stops |
| `<WakeWordThreadPriority>` | `Normal` | ONNX inference thread priority: AboveNormal, Normal, BelowNormal, Lowest |
| `<WakeWordThreads>` | `1` | ONNX threads for each wake-word model, with idle threads sleeping (1 to 4). 1 is plenty for these small models; more only adds CPU. |
| `<WakeWordMaxChunksPerSignal>` | `4` | Max audio chunks processed per wake-word signal burst (1-8) |
| `<NameMatchPhonetic>` | `true` | Soundex phonetic NPC-name matching (gates tier 3 of name resolver) |
| `<Prompt>` | *(see config)* | Keyword hints that improve Whisper accuracy for 7DTD context |
| `<SampleRate>` | `16000` | Audio sample rate for recording |
| `<MaxRecordingSeconds>` | `15` | Maximum recording duration |
| `<MicrophoneDevice>` | *(empty)* | Microphone device name. Empty = first available. ASCII hyphens only. |
| `<FormationRadialKey>` | - | Legacy. The key now lives in the top-level `<FormationRadial>` section as `<Key>`; an old `<STT>` entry is still read as a fallback. See the Order Wheel section. |
| `<WhisperGpuDevice>` | *(empty)* | GPU device index for whisper-server (Vulkan). Empty = default (device 0). |

### LLM Section

| Setting | Default | Description |
|---------|---------|-------------|
| `<Endpoint>` | `http://127.0.0.1:5055/v1/chat/completions` | LLM API endpoint (OpenAI-compatible) |
| `<ModelFilename>` | *(empty)* | Model filename override. Leave empty for auto-detect. |
| `<Model>` | `Qwen3-4B-Instruct-2507-Q4_K_M` | Model name. Used verbatim for an external API endpoint. For the local llama-server, autodetect of the `.gguf` in `Resources/` overrides it, so this value only appears in the log when no model file is found. |
| `<TimeoutSeconds>` | `30` | How long to wait for LLM response |
| `<MaxTokens>` | `200` | Maximum response length in tokens (64-2048). Use 1024-1536 for reasoning models. A value saved in the settings window overrides this file value at launch |
| `<Temperature>` | `0.8` | Randomness 0-1 (higher = more creative, lower = more focused) |
| `<SystemPrompt>` | *(see config)* | Base personality injected into every NPC conversation |
| `<ContextMemory>` | `5` | Number of exchanges kept in conversation history (3-20) |
| `<ShowTypingIndicator>` | `true` | Show "typing..." indicator while waiting for response |
| `<TypingDelayMs>` | `75` | Per-character delay for typewriter effect (50=fast, 75=natural, 150=dramatic) |
| `<MaxResponseLength>` | `300` | Character cap on NPC responses. 200-400 is the sweet spot. |
| `<ChatTimeout>` | `30` | Chat inactivity timeout in seconds before conversation resets. A value saved in the settings window overrides this file value at launch |
| `<ContextSize>` | `8192` | KV cache window in tokens. 4096 for low-VRAM (6 GB), 8192 default (8 GB+), 16384 for 12 GB+, 32768 for 16 GB+ |
| `<WildCombatChatCooldownSeconds>` | `5` | Seconds a wild NPC stays uninterruptible after its last combat target dies |

### RAG Section

| Setting | Default | Description |
|---------|---------|-------------|
| `<ExtractionPrompt>` | *(see config)* | Instruction sent to LLM to extract persistent facts from conversations |
| `<ExtractionUserPrefix>` | *(see config)* | Prepended to transcript before sending to extractor |
| `<ExtractionTemperature>` | `0.0` | Temperature for fact extraction (0.0 = deterministic) |
| `<MaxExtractedFacts>` | `10` | Cap on facts extracted per conversation turn |
| `<EmbeddingEndpoint>` | `http://127.0.0.1:5056/v1/embeddings` | Dedicated embedding server endpoint (port 5056). Falls back to chat server (5055) if embed server fails. |
| `<MinFlushMessages>` | `4` | Skip flush if buffer has fewer messages than this (prevents 2-message NONE flushes) |
| `<ExtractionIdleSeconds>` | `100` | Idle time before extraction fires. Higher = fewer extractions during active play. |

### DeathGuard Section (guard the dropped backpack)

When you die, your hires on the **Follow** order walk to your death spot and guard the backpack there. Hires on Stay, Guard, Patrol or a job (loot, craft, base keep) keep their order. There is no timeout: the state survives a save and a reload. They guard until you are alive again and within `<ResumeMetres>` of the spot (they resume **Follow** at once), or until you give them any order - "follow me", "stay", "guard here", the wheel, a dialog, the radial. Any order you give ends it straight away (their new order stands; the spot is no longer treated as a post).

| Setting | Default | Description |
|---------|---------|-------------|
| `<Enabled>` | `true` | Turn the death guard on/off |
| `<ResumeMetres>` | `10` | Flat distance (clamped 2 to 100) at which a live leader near the post wins his hires back to Follow |

### Subtitles Section (NPC text on screen)

One switch for the on-screen text of every NPC at once, for a clear screen. It caps each character's own `<Subtitles>` in `personalities.xml`: the stricter of the two wins, so `none` here silences every NPC's text whatever her own setting says. Her voice is untouched: TTS replies, greetings, order acknowledgements and voice clips still play. Display is per player: each player's own `modconfig.xml` decides on his screen. The boot log shows the value in force (`NPCText=`).

| Setting | Default | Description |
|---------|---------|-------------|
| `<NPCText>` | `all` | `all`: each character's own setting decides (today's behaviour). `clip`: only voice-clip captions, never the generic reply text. `none`: no NPC text at all. Any other value runs as `all`, with a warning in the log. ⚠️ With `none` and no voice package (no TTS, no clips) your NPCs give no feedback at all. The mod's own notices ("No NPC nearby", microphone errors, settings messages) and the book menu's greyed reason still show. |

### NPCToNPC Section (Ambient NPC-to-NPC idle chatter)

| Setting | Default | Description |
|---------|---------|-------------|
| `<Enabled>` | `true` | Turn NPC-to-NPC chatter on/off |
| `<Range>` | `5` | Detection range in meters |
| `<ScanIntervalSeconds>` | `30` | How often to scan for nearby NPC pairs |
| `<Chance>` | `0.25` | Probability per scan that a pair chats |
| `<MaxLinesPerNPC>` | `2` | Max lines per NPC per chat |
| `<GlobalCooldownMinutes>` | `5` | Cooldown before any NPC can chat again |
| `<PairCooldownMinutes>` | `15` | Cooldown before the same pair can chat again |

### Billing Section (Weekly hire billing)

| Setting | Default | Description |
|---------|---------|-------------|
| `<GrowthRate>` | `1.2` | Cost multiplier per week (1.2 = 20% increase) |
| `<MaxWeeks>` | `12` | Exponent cap so cost plateaus after this many weeks |
| `<GraceDays>` | `2` | Days after missed payment before auto-dismiss |
| `<ApprovalTimeoutDays>` | `3` | Days with no response before grace fallback |

Her contract carries over between sessions and ends when she is dismissed (a new hire starts at week 1). When she asks for her
pay, answer yes (or pay) or no; during the grace days, say **pay** to pay what you owe and end the grace.

### Formation, Follow & Movement Settings (root level)

| Setting | Default | Description |
|---------|---------|-------------|
| `<FollowMinSeparation>` | `1.0` | Personal-space floor for the follow servo. NPC backs off when leader closes within this distance. |
| `<RecoveryMinDist>` | `1.5` | Below this distance to its target the NPC never hands movement to A* (a few decimetres of shuffle is not a path). |
| `<TrailHandoverDist>` | `4.0` | Beyond this, a following NPC walks your breadcrumb trail instead of aiming straight at you. Must stay well above FollowMinSeparation. Also controls the far-follow handover to A* travel: `PathHandoverDist` has no key and is derived from this value (S6, 2026-09-05). |
| `<TrailVerticalDist>` | `2.0` | Also uses the trail when this far above or below you (catches switchback stairs). |
| `<FollowCatchupCeiling>` | `2.0` | Catch-up speed ceiling, as a multiple of run speed; applies only when more than 8 m behind, so close formation keeps normal pace. |
| `<FollowServoNearCeiling>` | `1.0` | Near-range direct-servo speed ceiling (multiple of run speed); also governs formation slots. 1.0 is your own pace; raising it makes close formation visibly faster, so raise it only if you mean it. |
| `<FollowPathNearCeiling>` | `1.4` | Speed ceiling on the A* path band below 8 m; ramps to FollowCatchupCeiling by 30 m. Never 1.25: that re-breaks catch-up. |
| `<RescueMinSeparation>` | `2.0` | Hard floor: minimum distance from the leader a rescue teleport may land. Without it rescues land on you. |
| `<FollowArriveY>` | `3.0` | Vertical arrival threshold: this far above or below the leader still counts as "arrived" horizontally. |
| `<VerticalStrandSeconds>` | `6` | Seconds of no vertical progress before a stranded NPC is rescued. |
| `<DropCheckAhead>` | `1.0` | Metres ahead the NPC scans for ground before each move. |
| `<DropCheckDepth>` | `1` | Deepest drop in blocks the NPC will take on its own: a 1-block step stays walkable, a drop of 2 or more is never her own choice. |
| `<MaxFollowDrop>` | `5` | The one exception: how much deeper the NPC follows when you just dropped (jump/fall), detected via a 2 s high-water mark. Also controls the trail hop down budget: there is no separate hop-down key (removed S6, 2026-09-05), so this value is the only knob for how deep a hop may fall. |
| `<LeaderDropGraceSeconds>` | `5` | How long the MaxFollowDrop allowance stays active after your descent. |
| `<TrailHopMaxHoriz>` / `<TrailHopMaxClimb>` | `2.5` / `1.6` | When the trail leads to a crumb the NPC cannot walk to, it hops (short teleport) to it within these limits. The down budget has no key (removed S6, 2026-09-05): it follows `<MaxFollowDrop>` - the NPC must not hop a fall it would refuse to walk. An old `<TrailHopMaxDrop>` key in a user config is silently ignored. |
| `<PathFirstFollow>` | `true` | Far-from-leader travel is A*-first; set false for the old servo-first behaviour (A* only on failure). |
| `<PathReleaseDist>` | `2.5` | Release side of the horizontal hysteresis pair for the path/servo handover (engage side has no key). `PathHandoverDist` is derived from `<TrailHandoverDist>` at load (removed S6, 2026-09-05); an old `<PathHandoverDist>` key in a user config is silently ignored. |
| `<PathVerticalDist>` / `<PathVerticalRelease>` | `1.0` / `1.0` | Vertical hysteresis pair for the path/servo handover. |
| `<PathPendingMaxSeconds>` | `2.0` | How long a follower lets a route search that is still running finish before she asks for a new one. Your hire re-aims her route about every half second while you walk; asking again throws the unfinished search away, so near a gap or a step she could stand still. Lower it to re-aim sooner; raise it to wait longer on a hard search. |
| `<LadderOrderedUseAstar>` | `true` | INERT key (nothing reads it any more): the ladder climb hands A* one exact path unconditionally. The loader still reads the value in case the flag is wired back. |
| `<PatrolAStarLeg>` | `true` | Recorded-patrol legs between waypoints are A*-owned. |
| `<PatrolStartMaxMetres>` | `40` | "Patrol your route" is refused out loud when her recorded route's nearest point is farther than this from her ("My patrol route is far from here"); with no route she says she has none yet. Either way she keeps her current order. Clamped 10 to 500. |
| `<ChatFollowEscapeDist>` | `6.0` | While mid-conversation the NPC holds its ground within this distance and resumes following beyond it. |
| `<JumpDownWaitSeconds>` | `10` | After you drop further below the NPC than MaxFollowDrop, it waits at the lip this long before the teleport backstop re-arms (you hold the JumpDown command; you decide). |
| `<TeleportMinGain>` | `1.5` | Refuses teleports that would move the NPC less than this (no-op guard). |
| `<EngageContactRadius>` | `2.5` | Defensive (Cover Me) also engages an enemy this close to you even if it targets nobody. |
| `<EngageNearRangedRadius>` | `20` | Defensive (Cover Me) with a gun: she fights any enemy she can see within this distance of herself, whatever it targets, without leaving you to chase it. Melee hires keep `EngageNearRadius` (5). Clamped 5 to 60. |
| `<EngageNearRangedReleaseRadius>` | `22` | An enemy she is already fighting that way stays her fight out to this distance (it must not be inside `EngageNearRangedRadius`). |
| `<DisengageWindow>` | `20` | Seconds a "disengage" NPC keeps self-defending against a live enemy but walks away the moment it is down. |
| `<DefaultTacticalMode>` | `false` | If true, every existing hire is put into tactical (command-only) mode once at the start of each session (re-applied at boot, not just new hires) |
| `<FormationDistances/Tier>` | `1=2m, 2=5m` | Slot distances keyed by index. Add tiers with no code change. |

The breadcrumb trail itself is recorded from your own movement (`TrailRingSize` 64 crumbs, `TrailSpacing` 1.0 m, `TrailSampleInterval` 0.1 s, `TrailRosterInterval` 2.0 s, `TrailSnapEpsilon` 0.2, `TrailSnapMaxDrop` 1.2, `TrailRecordMaxScan` 3, `TrailMaxAgeSeconds` 180). Tune those only with the log open - the inline comments in `modconfig.xml` carry the reasoning.

---

## Hired survivors: follow and loot

A hired survivor can loot while she follows you. Start it from the order wheel (**Jobs > Follow + loot**) or by
voice (**"follow and loot"**, **"loot around here"**, **"scavenge here"**). Her order stays **Follow**; looting is a mode on top of it.

- **When she loots:** only while you stand still (about a second without moving). While you walk she just follows; a trip she has already
  started is finished.
- **What she looks at:** containers within **15 m** of you (and within 3 m up or down), in the building you are in. Loot bags on the ground
  (zombie and bandit death bags nobody has opened) first, because they vanish, and from farther: within **25 m** of you on any floor she
  can reach (`BagRangeMetres`); then loose piles (ammo, medical, books, food, chem, liquor), then
  the rest, nearest first. She opens a bag as if you opened it, so its loot (and a bandit's weapon) is yours, rolled at your loot stage. She
  takes everything: usable items to her bag (up to its caps), the rest to her harvest window.
- **What she skips:** containers already opened (by her or anyone), locked or padlocked ones, player storage, anything behind a closed door or a
  wall, and anything another hired survivor has already claimed.
- **Reach:** she opens a container from up to **4 m** (the player's own reach), measured from her eyes to the nearest part of the container,
  and only with a clear line to it.
- **Giving up:** a container she cannot get to is left alone for 5 minutes. A trip is dropped if you walk more than **20 m** from that
  container. A container she has emptied is never picked again; an emptied loose pile disappears.
- **Danger:** she does not loot with an enemy within 15 m. A sleeper that spots her sends her back to you (follow); once it is dead she can go
  back to the room.
- **Ending it:** any other order, dismissing her, or a full harvest window.
- **Debug view:** with `VerboseLogging` on, a smoke marker rises over the container she is heading for. It is off in normal play.

The scan, reach and range numbers are on the hired loot action in `Config/utilityai.xml` (`leaderRange="15"`, `leaderAbortRange="20"`).

## Hired survivors: ammo

- **Choosing her ammo:** **E > Ammo...** lists each ammo type of her gun she carries, with how many; the one loaded is marked.
  Pick one and she loads it at once: the rounds in her gun go back to her pack first. While she talks to you her gun is put
  away: it takes the new type then, and she loads it when she draws it. Or say it: **"use armor
  piercing"**, **"use hollow points"**, **"use slugs"**, **"use breaching slugs"**, **"use standard ammo"**, **"use flaming
  arrows"**, **"use exploding arrows"**. The choice stays with that gun. She tells you when she has none of that type, holds no
  gun, or her gun takes no such ammo.
- **When it runs out:** she switches to any other type she has for that gun and stays on it until you choose again.
- **At your base:** inside your land claim she tops her ammo up from the base's containers (`<Ammo>` in the settings table).

## Hired survivors: crafting

Ask a hired survivor to make something: **E** on her, **Craft...** (or **Craft** on her hold-E ring). Her craft window opens, in
the look of your own crafting window:

- **Tabs:** each station in reach she can work at first (**Campfire**, **Forge**), then **All**, then your crafting window's own
  categories (Basics, Resources, Tools/Traps and so on). When the tabs do not all fit, the last one pages to the rest.
- **The list:** every recipe she has unlocked, with the most she can make of each right now. One she is short for is greyed ("Not
  enough", or "Tool missing" when the station lacks its tool). Two recipes for the same item (Cloth Fragment from cotton or from
  yucca) also show their main ingredient. Recipes she has not unlocked are not listed.
- **The panel:** the picked recipe's icon and name, and for each ingredient what she has against what she needs (or the item's
  description). Set how many with minus, plus and max (never more than she can make now) and press **Craft**. A greyed recipe
  cannot be crafted.
- **Her job:** the row at the bottom shows what she is making and how many are left ("Making Forged Iron: 3 left", or "Not
  crafting"). **Cancel** stops it, the same as saying "stop".
- The window closes with Escape, when she dies, or when you walk out of talking range. On a dedicated server it waits for the
  server's list before it shows her recipes.

A station recipe also needs the station's tool: at a campfire most recipes need a cooking pot (some a grill or a beaker) in the
campfire, so a bare campfire offers only charred meat (and baked potato once she has learnt it). A tool she carries (her pack, or
a chest of yours in reach) counts: she puts it into the station's own slot when she crafts, and it stays there. A new Cook knows
only the basics (boiled water, charred meat and a few more); she learns the rest by cooking, or from a book you give her (see
*Hired survivors: reading books*).

- **What she can make:** the recipes you can make in your own backpack, and the recipes of a workstation you stand at (below), and only
  the ones she has unlocked herself: the open recipes from the start; one that needs a schematic or a perk only once she has it,
  or once her crafting skill reaches it (*Learning her craft*, below). The
  server's crafting options apply to her too: backpack and workstation crafting, the highest tech tier, biome progression.
- **Materials:** shared (broadcast) storage inside your land claim when she works at your base, then any unlocked chest you or one of
  your allies placed within **15 m** of her spot (never another player's), then her own stores: her harvest window first (her main supply backpack, so hand her materials there), then her
  toolbelt, bag and pack. Never a stack with mods installed.
- **Cost, time, quality:** what a player pays with no crafting perks: the recipe's ingredients and time, scaled by the server's crafting
  settings. The quality is what her crafting skills give for that recipe, as yours are (she raises them by crafting at her role's
  stations, *Learning her craft*, and with skill magazines, *Hired survivors: reading books*, below); 6 for everyone when crafting progression is off.
- **Where and when:** she kneels where you gave the order (her post) and makes the items one at a time. An enemy she can see within 15 m
  pauses the item she is on (one behind a wall does not, as for base repair); after the fight she walks back to her post and carries on. She takes an item's ingredients when she starts it, so no one else can use them up while
  she works; stopping her mid-item gives them back to her harvest window (her pack when the window is full).
- **Output and XP:** the items go to her harvest window; she earns the crafting XP a player earns (less for each repeat of the same recipe).
- **Ending it:** the count is done ("I'm done crafting."), an ingredient runs out ("I need ... for that."), her window is full ("My pack is
  full.") or she can no longer make it ("I can't make that."). Any other order ends it at once. Then she follows you again; from more
  than 40 m away she appears beside you (`Settings.xml` `<Loot><ReturnTeleportMetres>`). An order she cannot take ("I can't make
  that.") changes nothing: she keeps her spot and her job.
- Characters that cannot do jobs (Harley) do not show the row.

### At a workstation

Stand within **5 m** of a placed workbench, forge, campfire, chemistry station or cement mixer (`Settings.xml`
`<Crafting><StationReachMetres>`) and open **Craft...**: its recipes show on the station's own tab of her craft window.
She uses the station the way you do. Every survivor crafts from her backpack anywhere; a workstation needs her role for it and a
land claim (a campfire works anywhere: it is a field station). When one in reach does not count (no claim, a trader's area, or
someone using it) and she has no other of that kind, she says so once as the window opens ("I can only use the Forge inside your land claim.").

- **Roles:** she uses a workstation only with its role, toggled on her from the order wheel (**Orders > Roles**) or by voice ("be the
  cook", "stop being the blacksmith"): **Cook** the campfire, **Blacksmith** the forge, **Chemist** the chemistry station,
  **Crafter** the workbench and cement mixer (`<Crafting><CookStations>` and the like: add a modded station's craft area to a list).
  A survivor may hold several roles, and several survivors the same one. Asked for a station recipe she has no role for: "I don't
  know how to do that." While she is hired her roles show after her name ("Xyth - Cook, Smith") on the Press E prompt, the talk
  window, captions, the order wheel and SCore's companions list; **E > Your progress** lists them with her level, her XP toward
  the next level and the quality she crafts at.

- **The station:** the nearest one in reach that makes the recipe and holds its tool (the cooking pot, the crucible...), of a kind
  her roles allow. Inside your or an ally's land claim, never another player's claim or a trader's area; a
  campfire also in the wild, as a field station (`<Crafting><FieldStations>`). Not one a player has open, not a fire beside water. At the forge she makes what the forge shows you: with the server's smelter on, the
  recipes paid in its smelted units.
- **Where she works:** a spot beside the station, in front of it first (the side it faces), then its sides, then its back; the
  nearest one she can stand on and see the station from (a wall or another block between them rules a spot out, a person does not).
  She walks there and faces it. No such spot: she works from where you gave the order if the station can be seen from there,
  otherwise "I can't make that." and nothing is taken. If she stops getting closer on the way, she works from where she got to.
  A modded workstation can name the side she works from with a block property, `<property name="NPCWorkSide" value="Back" />`
  (`Front`, the default, is the side that faced you as you placed it; `Back`, `Left`, `Right` as you see it from the front; `Any` for the
  nearest side).
- **Her home station:** the station she last crafted at is her home, remembered with her through a save, until she crafts at a
  different one (a station picked up or destroyed stops being her home). Order a station recipe while you are not near a station
  of that kind and her home station is used if it makes the recipe and she is within 60 m of it (`<Crafting><HomeStationReachMetres>`):
  she walks there, with the ingredients from her stores and the chests near it.
- **When she is done:** she stays at the station until you give her an order (**Follow me**), walking around it within 2 m
  (`<Crafting><StationWanderMetres>`; 0 = she stands at her spot beside it, facing it). A hire you tell to stay within that distance
  of any workstation walks around it the same way. A backpack order always brings her back.
- **The order:** she takes every ingredient when you give it (smelted units from the forge, the rest from her stores and your
  chests, as above) and holds them; the station waits for her. Once she is at her spot, facing it, the whole count goes into the
  station's queue as hers, as your craft does, and she lights a forge or campfire (at a campfire, a few seconds into striking her
  match); the station makes them at its own pace. Every queue full: "That station is busy." and nothing is taken.
- **The Blacksmith smelts:** a forge recipe whose units the forge does not hold yet starts with her ore: she puts raw ore in (scrap
  iron, brass and lead, crushed sand, clay lumps, small rocks: `<Crafting><SmeltOre>`) from her stores and your chests, keeps the
  forge burning, and queues her crafts once it has melted enough. She also keeps every material she has ore for at **1000** units
  (`<Crafting><ForgeStockUnits>`, `0` = only what an order needs): when she starts at the forge and again when her crafts are done.
  **Craft...** counts the ore in reach, so forge recipes show even at an empty forge.
- **Fuel:** a forge or campfire that would go out before her items are done gets fuel from her stores and your chests in reach (wood
  first, then coal: `<Crafting><StationFuel>`), and she lights it. No fuel anywhere: "I need fuel for that." and she waits; put fuel
  in and she carries on.
- **Her items:** as they are made she takes them out of the station into her harvest window; when that is full, into the nearest
  unlocked chest you or an ally placed near her. Only her own items: yours in the same station stay where they are. When nothing can
  take them: "My pack is full." once; she carries on and the station keeps them.
- **XP:** the crafting XP a player earns for a station craft.
- **While a player has the station open** she waits and touches nothing.
- **Ending it:** everything made and moved: "I'm done crafting." Any other order, or a new craft order, takes her items not yet made
  out of the queue and gives their ingredients back (units into the forge, the rest into her harvest window), and her ore the forge has not melted yet. Ingredients she
  took but had not queued yet (still on her way) go back to her the same way, also when the game saves. If she dies or the game
  is reloaded, what she queued still finishes in the station: take it from the station's output.

### Learning her craft

A specialist gets better by doing the work. Every item a workstation makes for her counts in each role she holds that uses that
station (`<Crafting><CookStations>` and the like). Her skill in the role is those crafts divided by **10**
(`<CraftSkill><CraftsPerLevel>`), and each of the role's crafting skills (the game's own, the ones you raise with magazines) is raised
to at least that level. The game then unlocks recipes and raises her craft quality from them exactly as it does for you.

| Role | Skills it raises (`<CraftSkill>`) |
|---|---|
| Cook | Food, Medical |
| Blacksmith | Workstations, Explosives |
| Chemist | Medical, Vehicles, Explosives, Electrician, Food |
| Crafter | the 21 skills of the workbench's recipes (armor, weapons, tools, electrician, traps, vehicles, workstations...) |

- The defaults are the skills of the recipes each role's own stations make; edit a role's list to change what it learns.
- A new Cook makes corn on the cob and baked potato after 20 crafts at a campfire; later recipes take more (the game's own levels).
- Only workstation crafts count, not backpack crafts. A skill never drops, and she keeps it if she gives up the role.
- **E > Your progress** shows each role she holds: "Cook skill 2: 20 / 30 crafts" (her skill, her crafts, the crafts the next level
  takes). The skill shown is her real one: the highest of the role's crafting skills, so a skill magazine she read shows there too.
- Her level still earns the game's crafting XP, as before.

### Modded workstations

A workstation from another mod (an ammo bench, a second forge) works with no tags and no code, if:

1. **It is a normal workstation block**, of the game's workstation block classes (`<property name="Class" value="Workstation"/>`,
   or `Forge` / `Campfire` as the game's own use, or a mod's own class built on them), with the game's workstation tile entity. A station built on another block class,
   such as a powered block with its own tile entity, is not seen at all.
2. **Its recipes use its crafting area**: the station's crafting area is its block property `WorkstationName`, or the block's own
   name when it has none, and its recipes' `craft_area` is that name. Everything below keys off the crafting area.
3. **A role lists it**: add the crafting area to one of `<Crafting><CookStations>`, `<BlacksmithStations>`, `<ChemistStations>` or
   `<CrafterStations>` (for example `<CrafterStations>workbench,cementMixer,ammoBench</CrafterStations>`). Until then no survivor uses it.
4. **It stands in your or an ally's land claim**, unless its area is also in `<Crafting><FieldStations>` (as the campfire is).

Optional, each with a fallback:

- **Tools:** list its tool slots in `<Crafting><StationTools>` (`ammoBench=toolA,toolB`) and she puts a tool she carries into an empty
  slot. Not listed: she never puts one in, but uses one already there.
- **Animations:** give it a clip set in `<Crafting><AnimStations>` (`ammoBench=bench`, or `use`, or `fire`). Not listed: she plays the
  rummage pose (one log line).
- **Fuel:** a station with a fuel slot is lit and fed like the forge, from `<Crafting><StationFuel>` (wood, coal). A station that burns
  anything else needs that item added there. A station with no fuel slot is never lit.
- **Smelting:** her ore loading follows the station's material input slot, not the name "forge" (not yet tested with a modded forge).
- **Recipe unlocks:** she crafts a recipe only when it is unlocked for her, through the crafting skills her roles raise
  (`<CraftSkill>`). A modded recipe on the game's crafting skills works as it is. A recipe unlocked by a modded crafting skill: add that
  skill's name to a role's list; any crafting skill the game has loaded is accepted, and a name that is not one is skipped with a
  warning in the log.
- **Power:** she cannot see electricity. A modded station that will not craft without power crafts only while it has power, and she
  does not wait for it.

## Hired survivors: reading books

A hire reads the same books you do: a **schematic** she has not learned, or a **crafting skill magazine**. Give her the book (her
harvest window or her pack), then:

- **E > Read...** (right after **Craft...**, for your own hire who does jobs) lists every book she carries, one row each. A book that
  would teach her nothing shows "(nothing new)". Pick one and she reads it. With no book: "I don't have anything to read."
- Or say **"read your books"** ("read the book", "read the books"): she reads every book she carries that teaches her something,
  then says one line.
- **A schematic** unlocks its recipe for her: it shows in her craft window at once. **A magazine** raises her crafting skill the way
  it raises yours (the server's points per magazine), so her craft quality and her unlocked recipes move with it; her progress page
  shows it.
- She says what she learned. Reading uses up one copy. A book that would teach her nothing is not used up, and neither is a perk
  book (the series books): she does not read those yet and says so. She will not read in a fight.
- She never reads on her own: a book you only store with her stays a book.
- The book's XP does not count toward her level while `<Progression><KillXpWorkaround>` is on (the default).

## Hired survivors: putting her weapon away

Tell a hire **"put your weapon away"** (or "use bare hands", "empty hands", or **Weapons > Empty hands** on the order wheel) and
she stays bare-handed, also through a save and reload: she comes back with empty hands instead of drawing the weapon she held last.
Bare hands are a weapon: in a fight she punches. Ordering her to a weapon ends it, and so does dismissing her.

## Hired survivors: the base keeper in an attack

A base keeper (**Jobs > Base Repair** or **Combat Repair**, or "repair the base") reacts to an attack on the base by her duty. The
base is **under attack** while a block of it that she tracks has been hit within the last **10 s**
(`<BaseKeeper><UnderAttackSeconds>`); that one window serves both duties.

- **Base repair** (most damaged block first): while the base is under attack she stops repairing and puts the repair tool away.
  She fights what she can reach or shoot, and goes back to her repairs once the base has been quiet for 10 s.
- **Combat repair** (a freshly hit block first): she keeps repairing the wall through the attack, the blocks hit within the window
  first, and fights only what hits her.
- **Nothing she can repair** (no damaged block within her repair skill, or no materials for one): she does not take the repair
  tool out at all and keeps her weapon. She checks this again every 2 s (`<BaseKeeper><BaseKeepRecheckSeconds>`).
- **No back and forth:** once she has turned to her repairs or to a fight, she stays with it for at least 3 s
  (`<BaseKeeper><JobFightDwellSeconds>`), unless an enemy that just hit her is in her reach or she has a gun in hand.

## Hired survivors: emotes

A hire can wave, salute, dance and more: a short animation she plays where she stands.

- **Order wheel > Orders > Emotes**: `Wave`, `Salute`, `Fist bump`, `Stretch`, `Warm up`, `Whatever`, then **Dances**
  (ten dances, seven a page plus More) and **More** (`Flip off`, `Insult`, `Loser`, `Threaten`). Release over one for the hire named
  in the middle of the wheel; Shift+click for the whole squad.
- **By voice**, the whole phrase: "wave at me", "salute me", "bump fists", "do a stretch", "do a warm up", "dance for me" or "do a
  dance" (a random dance), "flip me off", "do an insult", "do the loser", "do a threat". A sentence that only mentions dancing ("do
  you like to dance?") does not start one. Given to the squad, each hire picks her own dance.
- Only your own hire takes the order. She never starts one in a fight: she tells you so instead.
- **Her weapon** is put away for the emote and drawn again after; it never leaves her inventory. A weapon she had already put
  away stays away.
- **How long:** a gesture plays once; a dance loops for up to 15 s.
- **What ends it:** any order to her (opening her E menu counts), an attack or a threat, or her starting to walk.
- **Reactions** (`<Emotes><Reactions>` in the settings table): she salutes right after you hire her (a hire who is yours when the
  world loads never does; one you hire again after dismissing her does). Half the time she fist-bumps when a fight she was in is
  over and no enemy is near, at most once a minute. Half the time she waves with her greeting when it plays toward you, at most
  once every two minutes (single player and a listen host for now: not on a dedicated server). A reaction is skipped, never saved
  for later, while she fights, walks, plays another animation, works a job, sits in a vehicle or has an enemy near.
- **Idle moments** (`<Emotes><Idle>`): a hire who stands still and free (on Stay or Guard, or following you while you stand still)
  now and then stretches, warms up or shrugs: after 20 s still, then 2 to 5 minutes apart, each hire on her own clock. Never in a
  fight, with an enemy near, while she or the leader she follows walks, while she works a job or sits in a vehicle, or while you
  are talking with her.
- Other players on a server see it.
- **The clip list** is the base mod's `Config/SdcsAnimClips.xml`: one row per clip (`id`, the asset `bundle` in the base mod's
  `Resources` folder, the clip's `name` in it, `profile`, `loop`, `maxSeconds`). An id that starts with `dance_` is a dance: a
  random dance picks among them and the wheel lists it on its Dances page, under its clip name.
- **Console:** `vc anim play <clip id> [seconds]` plays a clip on the survivor in front of you; `vc anim off` stops it.

## Survivors' aim

A survivor shoots like a player who aims down the sights, made less steady by her inexperience: her spread with a gun is a
player's aimed spread with that gun (no perks) times a factor of 3 at level 1, falling to 1 at level 10
(`<Progression><AimErrorAtLevel1>` and `<AimErrorAtMaxLevel>`). A level-1 survivor's pipe pistol spreads about as wide as a
player firing it from the hip; a level-10 one shoots like a player aiming. Moving, running and crouching change it the way they
change a player's, and her shots spread more as she keeps firing, as yours do. Bandits use the same aim at their own level.
They aim at the chest of a person (you, a survivor, a bandit, a trader) and at the head of a zombie or an animal; bows and
launchers aim at the head, and their arc drops the shot onto the body.

**Defending you.** On the Defensive stance a hire goes for an enemy that is coming for you, before it hurts you. One with a
melee weapon goes for it within 10 m of you even when your body hides it from her; one with a gun whose line of fire you
stand in steps 2 m aside, away from you (`<Combat><ClearShotStepMetres>`), then shoots. A hire with a gun
also fights any enemy she can see within 20 m of herself, whatever it is after; cover is fine, but she engages. Through a gap
(a broken door, a window) she aims at the part of the target she can actually hit, and holds fire when she can hit none.
An enemy behind a closed door, window or bars is not hers on Defensive: she waits until the way opens. On Aggressive she opens
the door, or breaks through it with an axe or hammer and shoots through the gap (`<Combat><BreakThroughDoors>`).

**Looking for cover.** A survivor or bandit whose search for cover found nothing does not search again from the same place: she
waits until she, her enemy or the area she keeps to has moved 2 m (`<Cover><FruitlessRescanMoveMetres>`), she faces another enemy,
the kind of cover she is after or her order changes, or 9 s have passed (`<Cover><FruitlessRescanSeconds>`), and meanwhile does what she does when there is no cover.
In a big fight this saves most of the repeated searches; the cost is that a spot that frees up while nothing moves is found up to
9 s later.

**Starting guns and ammo.** A tier 1 survivor starts with a melee weapon in her hand and an empty pipe pistol in her pack:
she gets no ammo for it when she appears, not even ammo from her own trade goods, which stay for sale. Ammo she loots later,
or that you give her, is hers, and then she uses the pistol. Tier 2 and 3 survivors start with ammo for the gun they roll.

## Bandits

Bandits are survivors gone hostile, and they spawn in the biomes with the survivors' tier progression: tier 1 bandits where
tier 1 survivors live, tier 2 and 3 bandits where those tiers do (outside towns, day and night). A tier 1 bandit
(`npcSdcsBanditMelee`, a tier 1 melee weapon) comes alone; a tier 2 or 3 bandit (`npcSdcsBanditMaleT2`, `...FemaleT2`,
`...MaleT3`, `...FemaleT3`: that tier's survivor, with her looks, armour and weapons) comes with 1 or 2 mates of her tier. A
bandit roll comes a quarter as often as a survivor roll at tier 1, half as often at tier 2 and as often at tier 3. They spawn
like zombies: only with enemies on, never in a blood moon, under the enemy cap and at least 28 m from you. `npcSdcsBanditRanged`
(a tier 1 female with a pipe pistol or a pipe rifle, 60 rounds and 2 pipe bombs) never spawns in the open world.
In the console, `sdcsbandit group 2` (or `3`) spawns a group of that tier beside you.

**In buildings:** bandits also sleep in points of interest, in the bandit rooms the legacy Harley bandits used to fill (Harley no
longer spawns anywhere). The room's gamestage picks the tier: up to 50 tier 1 (the melee bandit, and the pipe-gun bandit in the
ranged rooms), 100-200 tier 2, 400 and up tier 3; boss rooms are always tier 3. A sleeping bandit wakes when you walk into her
room, and before that she hears and sees like a sleeping zombie, in any room: a loud noise near her (gunfire) or you within 6 m in
front of her in enough light wakes her, and so do your survivors if she counts them as enemies. Crouch in the dark and stay quiet
to sneak past. In an attack room she is already awake and waiting. A bandit in a building comes without mates. If you make your
own NPC pack, add your NPCs to these groups by appending to the group by name.

**Which tier:** a POI's bandits are picked by the party's gamestage, and the POI's difficulty sets a floor
(`<Bandits><PoiTierFloors>`): difficulty 1-2 gives at least tier 1 bandits, 3-4 tier 2, 5 tier 3.

**Sentries:** a POI builder can name `GroupNPCBanditsSentry` (or `GroupNPCBanditsSentryRanged`, guns only) in a sleeper volume
instead. A sentry spawns awake on her spot, facing the way it faces, and holds it: in a fight she steps at most about a block
(`<Bandits><SentryLeashMetres>`) for a shot or cover, never off her spot's level and never jumping down; she shoots, throws and
swings from there, and if something knocks her off she walks back. A bandit spawned by a guard spawn block (Stay or Guard) keeps
her spot the same way; out of a fight a Guard block's bandit walks a short patrol around it on her own floor (`<Guard><PatrolMetres>`),
while a Stay block's bandit and a sentry stand still.

- **Throwables:** none at tier 1, one molotov at tier 2, three at tier 3; they throw only at a group (see *Throwables*).
- **They fight** you, your survivors, zombies and hostile animals on sight; passive animals are left alone. A bandit runs at a
  target she sees but cannot reach with her gun yet, and walks toward a player she hears but cannot see (crouch to stay quiet).
- **They don't talk:** no E prompt, no hire, no trade, no voice chat or voice orders.
- **One cold line:** the moment a bandit locks onto you or one of your survivors to fight, she says one calm line ("Time to
  die.", "Nothing personal.") in a random voice of her sex, then not again for a minute, and only the first bandit of a group
  speaks. A zombie or an animal gets no line. Text to speech only: with TTS off (the Core alone) she stays silent, caption
  included. The lines are her personality's `<CombatLines>` in `personalities.xml` (`bandit_cold_male`, `bandit_cold_female`,
  named by each bandit class's `personality` property); the timing is `<Bandits><LineCooldownSeconds>`,
  `<LineGroupQuietSeconds>` and `<LineGroupMetres>`.
- **Loot:** dead, she always drops a zombie loot bag for her tier: plain at tier 1, blue at tier 2, red at tier 3. Per class in
  `entityclasses.xml`: `LootDropEntityClass` and `LootDropProb` (the same properties as on any zombie; the blue and red bags
  are `EntityLootContainerStrong` and `...Boss`). Half the time the bag also holds her weapon: one the game crafts from parts
  (most tier 2 and 3 guns, the steel melee weapons) as 1 or 2 of those parts, with a 1 in 10 chance of the whole weapon too; one
  with no parts (pipe guns, simple melee) whole, at quality 1-2 at tier 1, 2-3 at tier 2, 3-4 at tier 3, empty, no mods. It goes in
  when the bag is first opened; a bag left unopened over a save and reload keeps only the game's loot.
- **Damage to players:** `<Bandits><PlayerDamage>` in Settings.xml (half by default). In the console, `sdcsbandit` shows it and lists
  the bandits in the world; `sdcsbandit damage 0.75` changes it at once, until the game restarts.
- **Aim:** a bandit shoots like a survivor of her level (see *Survivors' aim*). `<Bandits><SpreadByTier>` can widen it by tier
  (0 by default).
- **One target at a time:** she keeps fighting the target she has until it is dead or out of her sight (the class property
  `SdcsKeepTarget`, 0.7: any other target scores that much of hers).
- **She hunts:** if she loses sight of the enemy she is fighting, or of whoever just hurt her, within 40 m, she goes after it
  until she sees it again (instead of wandering off or standing still). Within `<Bandits><SeekTrackMetres>` (20 m) she goes to
  where you are, not where she last saw you. A melee bandit with no cover to bound to charges at once. She opens a closed,
  unlocked door on her way; she cannot break blocks, so her path goes round walls, fences, shanty walls and railings.
- **Under fire:** she does not stop to bandage while someone with a gun has her in sight within `<Combat><HealGunThreatMetres>`
  (25 m); she keeps fighting. The same holds for your survivors.
- **XP:** 600 at tier 1, 1000 at tier 2, 1400 at tier 3. Your own custom characters never spawn as bandits.

## Your own NPC or bandit boss

Every piece a survivor or bandit spawns with is a property of her class in `entityclasses.xml`, and a class that `extends`
another keeps every property it does not set. So a special character is a class of your own that extends the nearest one and
overrides what should differ. A tier 3 bandit boss with a better loot bag, more XP, more molotovs and two mates:

```xml
<entity_class name="myBanditBoss" extends="npcSdcsBanditMaleT3">
    <property name="ExperienceGain" value="3000"/>
    <property name="SdcsThrowables" value="thrownAmmoMolotovCocktail:3,thrownAmmoPipeBomb:2"/>
    <property name="SdcsSpawnMates" value="npcSdcsBanditMatesT3:2-2"/>
</entity_class>
```

| Property | What it sets |
|---|---|
| `SdcsWeapons` | The pool her weapon is rolled from (comma separated item names). |
| `SdcsArmorTier` | Which armour sets her look may roll (1 light, 2, 3); a bandit's aim tier too. |
| `SdcsSpawnAmmo` / `SdcsSpawnAmmoCount` | Whether a rolled gun comes with ammo, and how many rounds. |
| `SdcsThrowables` | Throwables she spawns with, `item:count,...` (once per life). |
| `HarvestItems` | Fixed starting items in her pack, `item=count` (the bandages). |
| `SdcsSpawnStock` | The loot container a wild NPC's rolled starting stock comes from (blank: none). |
| `SdcsSpawnWater` | Water she spawns with, `drink:count` (blank: none). |
| `BagItems` / `ItemsOnEnterGame` | Her spawn kit (the same properties as any NPC). |
| `LootDropEntityClass` / `LootDropProb` | The loot bag she drops dead, and its chance. |
| `SdcsDropWeaponChance` | The chance (0 to 1) that bag also holds her weapon or its parts: 0 never (the default), the bandits 0.5. |
| `SdcsDropWholeChance` | For a weapon with parts, the separate chance the whole weapon is in the bag too (0.1). |
| `SdcsDropQuality` | A whole weapon's quality, `min-max` 1 to 6 (the bandits 1-2, 2-3, 3-4 by tier). |
| `SdcsDropPartsCount` | How many parts, `min-max` 1 to 10 (1-2). |
| `SdcsSpawnMates` | Mates a biome spawn brings, `group:min-max` (blank: none). |
| `ExperienceGain` | The XP for killing her. |
| `personality` | Her personality from `personalities.xml`; for a bandit, her `<CombatLines>` and her voice. |

## Trading with wild survivors

**Trade** on a wild survivor (hired ones have none: give them items instead):

- **Buy** her goods: the stock in her pack. What you pay goes into her purse.
- **Sell** her what she can use: the medical items her heals use (bandages, first aid kits, aloe cream, splints, casts, painkillers)
  until she carries 10 (`<Trade><BuyMaxMedical>`), and ammo for a gun she carries
  up to what she keeps in her bag (the `<Loot>` bag cap), at the game's normal price (your Better Barter counts), paid from her purse. Sell is greyed out for
  anything else, or for more than she has room for or can pay; choose a smaller count to sell part of a stack.
- A new wild survivor starts with 50 to 150 coins (`<Trade>`).

## Pickpockets

A survivor whose class sets `<property name="SdcsPickpocket" value="true"/>` picks pockets (no class does by default; the
test character is `npcSdcsPickpocket`). Wild only: a hired one never steals.

- **When:** you stand within `ReachMetres` (2 m) of her, at most once per pickpocket every `SurvivorMinutes` (10) and once per
  player every `VictimMinutes` (30).
- **Caught:** if she is in your sight and not hidden by the dark, she fails, says a line and backs off. No fight.
- **What she takes:** one stack from your backpack (never the toolbelt or what you wear): medical, ammo, food, drink, skill
  magazines (`ListedTags`), anything worth `MinValue` (100) a unit, or up to `MaxCoins` (100) dukes.
- **Where it goes:** like her loot: what she can use into her bag, the rest into her trade stock, so you can buy it back.
- Single player and the host's own player for now; players on a dedicated server are never robbed yet.

## Survivor Settings — `0-XNPCCore/Config/Settings.xml`

The survivor (SDCS) behaviour of the base mod is tuned in its own `Settings.xml`, not in `modconfig.xml`. Each value is read once at game start,
so restart after an edit. A missing or unreadable value falls back to its default and prints one warning in `Player.log`.

| Setting | Default | What it does |
|---|---|---|
| `<SdcsModelYOffset>` | `0.08` | Lifts the survivor's body (metres) so the soles sit on the ground; `0` = the game's stock placement. Clamped to ±1. |
| `<AnimFromMeasuredSpeed>` | `true` | The walk and run animation follows how fast the survivor really moves over the ground, so her feet do not slide. `false`: it follows the speed the AI asked for. |
| `<SleeperTrigger>` | `true` | A survivor walking into a building's sleeper volume wakes / spawns it as a player would (only while a player is within 96 m). Bandits and sleeping NPCs never trigger one. `false`: survivors never trigger sleepers. |
| `<GunBurstEnabled>` | `true` | Paces survivors' gunfire in bursts so automatic weapons do not empty a magazine in seconds. |
| `<GunBurstTargetRpsAuto>` | `2.0` | Average rounds per second for fully automatic guns. |
| `<GunBurstTargetRpsSemi>` | `1.5` | Average rounds per second for pistols, semi-automatic rifles and bows. |
| `<GunBurstWindow>` | `0.5` | Burst length in seconds (longer bursts, longer pauses). |
| `<HealRollSpeed>` | `1.6` | How fast the heal roll plays (1.0 = the clip's own pace). Clamped 0.5 to 3.0. |
| `<HealBandageSeconds>` | `1.5` | How long a survivor holds the bandage before the health comes back. Clamped 0.8 to 5.0. |
| `<AnyWeaponEnabled>` | `true` | A hired survivor switches to a weapon you put into (or take out of) her backpack. |
| `<AnyWeaponSettleSeconds>` | `2.0` | How long the backpack must stay unchanged before she equips (minimum about 1). |
| `<NoWear>` | `true` | Survivors' gear takes no wear (they have no way to repair it yet). |
| `<PlayerCtrlDeathWeapon>` | `vanish` | For player-controller NPCs only: on death the held weapon `vanish`es, or `fall`s with the body. |
| `<Progression>` | | Hired survivors level up: `<HealthPerLevel>` (0.03), `<DamagePerLevel>` (0.02), `<AimErrorAtLevel1>` (3) and `<AimErrorAtMaxLevel>` (1, her spread is a player's aimed spread times this factor, falling in a straight line from level 1 to `<MaxLevel>`; see *Survivors' aim*), `<MaxLevel>` (10), `<StartLevelDaysPerLevel>` (7, world days per start level: a survivor met on day 8 starts at level 2), `<HealthRegenPerSecond>` (0.2), `<HarvestPerLevel>` (0.10, extra yield per level above 1 from wood, stone, ore, clay, plants, crops and butchering). Her XP: kills, crafting and harvesting (butchering included) pay her what they pay a player. |
| `<Crafting><PostReachMetres>` | `4` | How close to her post a crafting survivor must stand; farther away (after a fight) she walks back first. Clamped 1 to 20. |
| `<Crafting><StationReachMetres>` | `5` | How close to you a workstation must be for its recipes to show in **Craft...**. Clamped 1 to 10. |
| `<Crafting><StationFuel>` | `resourceWood,resourceCoal` | What she feeds a forge or campfire, the first one found first (a comma list of item names). |
| `<Crafting><CookStations>` | `campfire` | The workstations (craft areas) a Cook may use; `BlacksmithStations` (`forge`), `ChemistStations` (`chemistryStation`) and `CrafterStations` (`workbench,cementMixer`) likewise. |
| `<Crafting><ForgeStockUnits>` | `1000` | A Blacksmith keeps each forge material she has ore for at this many units; `0` = only what an order needs. Clamped 0 to 30000. |
| `<Crafting><SmeltOre>` | the six raw scraps | The raw items a Blacksmith may put in a forge (a comma list of item names). |
| `<Crafting><FieldStations>` | `campfire` | The workstations she may also use where there is no land claim (never in another player's claim); empty = every workstation needs a claim. |
| `<Crafting><StationWanderMetres>` | `2` | After a workstation order, and when told to stay this close to one, she walks around the station within this many metres until you give her an order. `0` = she stands at her spot. Clamped 0 to 20. |
| `<Crafting><StationTools>` | the campfire's and forge's | Each workstation's tool slots in the game's own order (`area=tool,tool,tool`, stations separated by `;`): a tool she carries goes into its own slot when she crafts. A station or tool not listed: she never puts one in. |
| `<Crafting><StationStowMetres>` | `4` | While a workstation order runs, and while she stays at the station afterwards, her weapon stays stowed within this many metres of it; farther, a new order, a follow or the job's end brings it back. An enemy target draws it at once. Clamped 2 to 10. |
| `<Crafting><StationClearanceMetres>` | `0.3` | Her walk around a workstation keeps this far from any workstation block (her body's width), so she never walks over one. Clamped 0 to 1. |
| `<Crafting><StationArriveMetres>` | `0.3` | How close to her spot beside a workstation she walks before she starts working. Clamped 0.15 to 0.7. |
| `<Crafting><StationStandInMetres>` | `0.1` | Her spot beside a workstation sits this far in from the middle of its cell, toward the station. Clamped 0 to 0.2 (more would put her body into the station). |
| `<Crafting><StationWanderPathMetres>` | `4` | A walk around the station whose path strays farther than this from it (a long way round a wall) is refused; she waits for the next walk. Clamped 1 to 20. |
| `<Crafting><AnimLightAtSeconds>` | `4` | A forge or campfire is lit this many seconds into her light clip (0 = as it starts); a station with no light clip is lit as she arrives. Clamped 0 to 30. |
| `<Crafting><AnimAccentSeconds>` | `20,40` | The gap between her accent clips while she works (min,max seconds; the next is rolled in between). Clamped 5 to 600. |
| `<Crafting><AnimSets>` | the fire, bench, use and pack sets | The clip sets: a set id, then its clips by slot (`in`, `loop`, `accent`, `light`, `refuel`, `out`; any slot may be left out). `<AnimStations>` picks the set per workstation. A set id with no row plays the rummage pose. |
| `<Crafting><AnimBackpackSet>` | `pack` | The clip set a backpack order plays. |
| `<Sleeper><SightRangeMetres>` | `16` | How close a sleeping zombie must be to see one of your survivors. Clamped 4 to 29. |
| `<Sleeper><LeaveBeyondMetres>` | `30` | Once a house wakes on a wild survivor, how far past it she runs before she counts as away. Clamped 10 to 100. |
| `<Sleeper><HouseAvoidSeconds>` | `300` | How long she then stays out of that house (and away from a container she gave up on). Clamped 30 to 3600. |
| `<Vehicle><GunnerMaxRangeMetres>` | `40` | The farthest a seated gunner fires; each gun also stops at its own effective range (a pistol about 17 m). Clamped 5 to 150. |
| `<Vehicle><GunnerIdleHolsterSeconds>` | `5` | How long she keeps the gun out after the last enemy is gone. Clamped 0 to 60. |
| `<Vehicle><BoardRangeMetres>` | `30` | How far from the vehicle a hire walks over to get in. Clamped 5 to 100. |
| `<WorkSite><HarvestRadius>` | `30` | How far from her post the chop-wood, mine and pick scans reach. Clamped 20 to 40. |
| `<WorkSite><ClayRadius>` | `8` | The same for digging clay (clay is everywhere, so it stays smaller). Clamped 4 to 40. |
| `<WorkSite><MineVeinDepthLayers>` | `2` | How many layers of an ore vein she mines, down from the surface as she first saw it. Clamped 1 to 10. |
| `<WorkSite><SwingBudgetSeconds>` | `60` | How long she swings at one tree, rock, plant or corpse before skipping it. Clamped 5 to 600. |
| `<WorkSite><WalkBudgetSeconds>` | `15` | How long she walks to one target before skipping it (harvest, butcher, base keeper). Clamped 5 to 120. |
| `<BaseKeeper><DefaultMode>` | `base` | What a plain "repair the base" order fixes first: `base` (the most damaged block) or `combat` (a block damaged in the last few seconds). |
| `<BaseKeeper><RescanSeconds>` | `30` | How often she rescans the base for new damage. Clamped 5 to 300. |
| `<BaseKeeper><AreaRadiusMetres>` | `8` | How far her worked area reaches; two keepers stay this far apart. Clamped 2 to 40. |
| `<BaseKeeper><UnderAttackSeconds>` | `10` | The one under-attack window: in combat repair a block hit within this many seconds is worked first, and while the base has been hit within it the base-repair duty stands down with the weapon out. Clamped 1 to 60. |
| `<BaseKeeper><ChestReachMetres>` | `15` | How far a chest (not broadcast storage) may sit from her post and still feed repairs and crafting. Clamped 2 to 40. |
| `<BaseKeeper><BaseKeepRecheckSeconds>` | `2` | How often she checks again whether there is any damaged block she can actually repair (her repair skill and the materials); while there is none she keeps her weapon and no repair tool comes out. Clamped 0.5 to 10. |
| `<BaseKeeper><JobFightDwellSeconds>` | `3` | Once a keeper has turned to her repairs or to a fight, the least time before she turns to the other; broken at once by an enemy that just hit her in her reach, or a gun in her hand. Clamped 0 to 15; `0` = off. |
| `<Trade><StartCoinsMin>` / `<StartCoinsMax>` | `50` / `150` | The coins a new wild survivor starts with, an even roll between the two: her purse for what you sell her before anyone has bought from her. `0` and `0` = none. Clamped 0 to 20000. |
| `<Trade><BuyMaxMedical>` | `10` | She buys medical items while her bag holds fewer than this. Keep it above `<Loot><BagMaxMedical>`: her start kit fills her bag to that cap. Clamped 0 to 100. |
| `<Pickpocket>` | | Pickpockets (see *Pickpockets*): `<SurvivorMinutes>` (10), `<VictimMinutes>` (30), `<ReachMetres>` (2), `<MaxCoins>` (100), `<MinValue>` (100), `<ListedTags>` (medical,ammo,food,drinks,csm), `<BackOffSeconds>` (5, how long a caught pickpocket backs away). |
| `<Guard><SpacingBlocks>` | `2` | Survivors told to guard the same spot spread out this many blocks apart: a line beside the first, facing your way, then a rank behind (see *Order Wheel*). 1 to 5. |
| `<Wander><StuckMetres>` | `3` | A wandering survivor or bandit (no order, or Wander) counts as getting somewhere once she is this far from where she last was. Clamped 1 to 10. |
| `<Wander><StuckSeconds>` | `6` | She gives the wander up when she has not got that far for this long (pressed into a car wreck or a corner), rests 10 s (the `SdcsWanderRest` buff in `buffs.xml`), then wanders again. Clamped 2 to 60. |
| `<Guard><PatrolMetres>` | `6` | On `Guard and return` a survivor walks out to a random spot within this many metres of her post, looks outward 4 to 10 s, walks back, faces your way 4 to 10 s, and repeats; a fight or a talk sends her back to the post first. 0 = she holds her post. Clamped 0 to 20. |
| `<CraftSkill><Enabled>` | `true` | A specialist learns her craft by crafting (see *Learning her craft*). `false` stops the learning; what she has learned stays. |
| `<CraftSkill><CraftsPerLevel>` | `10` | The workstation crafts a skill level takes, clamped 1 to 1000. |
| `<CraftSkill><CookSkills>` / `<BlacksmithSkills>` / `<ChemistSkills>` / `<CrafterSkills>` | the skills of each role's stations | The game's crafting skills (progression.xml names, comma list) each role raises. A name that is not a crafting skill is skipped with a warning. |
| `<RepairSkill><Enabled>` | `true` | A keeper learns to repair by repairing: each paid repair and rebuilt block pays her the player's repair XP for its materials, into her repair XP and her level, and her repair XP sets the strongest building material she may work. `false` = every keeper repairs everything (the XP still counts). |
| `<RepairSkill><StartTier>` | `0` | What a new hire may repair: 0 wood, 1 cobblestone, 2 concrete, 3 steel. |
| `<RepairSkill><CobblestoneXp>` / `<ConcreteXp>` / `<SteelXp>` | `100` / `900` / `1700` | The repair XP each tier takes (a full wood repair pays about 4, cobblestone or concrete about 32). Shown on E > Your progress. |
| `<RepairSkill><WoodMaxDamage>` / `<CobblestoneMaxDamage>` / `<ConcreteMaxDamage>` | `500` / `1500` / `5000` | A block's tier is its material's strength: up to 500 wood (sandbags, frames, wood), up to 1500 cobblestone (brick, scrap metal), up to 5000 concrete, above that steel. Workstations, traps and electrics are open to every keeper. |
| `<Medic><InCombatHealthPercent>` | `50` | The medic treats a patient below this health while an enemy is visible, and also in the first seconds after a fight (lighter wounds wait so you can patch yourself). Clamped 1 to 99. |
| `<Medic><AfterCombatHealthPercent>` | `50` | The same line once the fight is over. Clamped 1 to 99. |
| `<Medic><SelfHealHealthPercent>` | `70` | A survivor heals herself below this health. Clamped 1 to 99. |
| `<Medic><DireHealthPercent>` | `25` | The dire line below it (the first aid kit and pain pills). Clamped 1 to 99, below the self-heal line. |
| `<Medic><SelfHealCooldownSeconds>` | `30` | The wait between low-health self-heals. |
| `<Medic><BleedRecheckSeconds>` | `10` | The wait before a new bleed can start another treatment. |
| `<Medic><WalkRetrySeconds>` | `10` | A medic whose walk to a patient gets stuck skips that patient this long before trying again. 0 = no wait. |
| `<Combat><WildTargetedRadiusMetres>` | `15` | How far away a zombie that targets a wild (unhired) survivor still makes her fight it. Clamped 1 to 60. |
| `<Combat><CloseMeleeEnterMetres>` | `3` | A survivor holding a ranged weapon swaps to melee at or below this distance. Clamped 0.5 to 20. |
| `<Combat><CloseMeleeExitMetres>` | `5` | She swaps back to the ranged weapon beyond this distance (it must be above the enter distance). |
| `<Combat><CloseMeleeMinHoldSeconds>` | `3` | The least time on the melee swap before she swaps back. Clamped 0 to 30. |
| `<Combat><BreakThroughDoors>` | `true` | An enemy behind a door, window, glass or bars: an Aggressive hire or a bandit opens an unlocked door, otherwise breaks through with her best block-breaking melee item and switches to her gun the moment the opening gives a line of fire; a Defensive or Hold fire hire and an unhired survivor leave it alone until the way is open. Nobody swings at a wall or at another player's door. false = nobody breaks through. |
| `<Combat><MeleeArcRays>` | `5` | How a swinging weapon's arc is probed: when its centre ray is clear, this many rays (clamped 3 to 9) spread across the weapon's own `SwingDegrees`, each cast with the weapon's own sphere thickness; any ray that meets a solid block blocks the swing the way a wall does (the blocked-swing log line names the weapon and the ray count). A thrusting weapon keeps the single centre ray. |
| `<Combat><MeleeThrustTags>` | `perkJavelinMaster,perkDeepCuts,perkBrawler` | Weapons carrying one of these tags are thrusting (a straight hit that goes through a 1-block opening). A weapon's own `SdcsMeleeStyle` item property (`"thrust"` or `"swing"` in its item XML) beats both tag lists; `MeleeSwingTags` beats this one; with neither, a weapon swings. |
| `<Combat><MeleeSwingTags>` | `` | Weapons carrying one of these tags are forced to swing, beating `MeleeThrustTags`. Empty by default. |
| `<Combat><MeleeSwingHitsBlocks>` | `false` | When a swing is blocked by a solid block and the held weapon's block damage is at least `MeleeSwingBlockDamageMin`, she swings at the block instead of standing back (the hit damages it, so a strong swing can break the block); otherwise a wall is a no-swing and the target counts as unreachable. `false` = nobody swings at walls. |
| `<Combat><MeleeSwingBlockDamageMin>` | `30` | The minimum block damage a held weapon needs for `MeleeSwingHitsBlocks` to turn a blocked swing into a swing at the block. |
| `<Combat><NpcThrustPiercesBars>` | `true` | For a thrusting weapon in an NPC's hand, both the reach probe and the real swing hit with the arrow hit mask, so bars and openings the arrow passes through do not stop it (the probe casts the arrow mask, the swing's own hit mask is swapped to it for the length of the cast and restored after). A swinging weapon still stops at bars, though its arc probe can find a solid block at the arc's edge. A weapon that declares its own `Hitmask_override` is never touched. |
| `<Combat><PlayerThrustPiercesBars>` | `false` | The same bar-piercing for the LOCAL player's thrusting weapon (both his main hit and the game's own arc-sweep pass). Off by default: it makes base defence much easier. |
| `<Combat><HoldUnseenGiveUpSeconds>` | `5` | A hire holding on an enemy she cannot see (behind a wall or floor) lets it go after this long and follows you again; she takes it up again when she sees it or it hits her. Clamped 2 to 60. |
| `<Combat><CrawlerAimHeadMetres>` | `0.45` | Dedicated server: where she aims at a crawler's head, above its root. Clamped 0.05 to 1.5. |
| `<Combat><CrawlerAimBellyMetres>` | `0.25` | The same for its belly (it must be below the head height). |
| `<UaiDecisionSeconds>` (top level) | `0.6` | How often a survivor or bandit re-decides what to do, in seconds, the same in single player and on a dedicated server (the game alone ties it to the frame rate: a server re-decided about three times as often, at three times the cost). Lower reacts faster and costs more. `0` = the game's own pace. At most `10` (a larger value reads as 10). |
| `<Combat><TimidFleeDistanceMetres>` | `15` | A timid character (a class that sets the `$Timid` cvar, such as `npcSdcsSurvivorFemaleTimid`) runs from an enemy she can see within this many metres instead of fighting, hired or not, then follows you again. `0` turns fleeing off. |
| `<Combat><MeleeUnreachableSeconds>` | `3` | A melee fighter who can see her target but gets no closer (stuck at a gap too small to walk through, say) gives up on that target after this many seconds and fights an enemy she can reach. A hire with no other enemy goes back to following. `0` turns this off. |
| `<Combat><MeleeUnreachableForgetSeconds>` | `20` | How long she ignores a target she could not reach before trying it again. |
| `<Combat><MeleeUnreachableNearMetres>` | `3` | Within this many metres a melee fighter who sees her target but cannot reach it (standing at a gap in a wall, say) counts as stuck even while she waits, so she gives up on it after `MeleeUnreachableSeconds`. `0` counts only the time she walks toward it. |
| `<Combat><AttackerPrioritySeconds>` | `3` | How long after a hit the enemy that did the damage beats the target a keep-target fighter was holding: while within this window, an attacker within her melee reach (or one with a gun, which shoots through the gap) wins the target choice, so she turns and fights back instead of holding on a target that is not hitting her. A target marked unreachable (the row above) never keeps the preference. `0` turns this off. Clamped 0 to 15. |
| `<Combat><CloseMeleeClosingMetres>` | `0.5` | A gunner swaps to her melee weapon for a close enemy only when it is within her melee reach or closing on her by at least this many metres over `CloseMeleeClosingSeconds`. An enemy standing still at a gap in a wall does not make her swap. `0` swaps only for an enemy already in reach. |
| `<Combat><CloseMeleeClosingSeconds>` | `1` | The time window for `CloseMeleeClosingMetres`. |
| `<Combat><CloseMeleeWallReenterSeconds>` | `1` | After a wall between her and the enemy made a gunner put her melee weapon away, she does not swap back to it for this many seconds. Stops her switching weapons over and over when the enemy stands at the edge of a wall. `0` turns the wait off. |
| `<Combat><SightStealthOnlyWhileLeaderCrouches>` | `true` | A hired survivor holds fire on an enemy that cannot see you (too far or too dark for it) only while you crouch, so a sneak stays quiet. Standing, she fights anything she can see. `false` = she holds fire whenever the enemy cannot see you, crouched or not. |
| `<Combat><SightThroughPlants>` | `true` | Survivors and bandits see past grass, bushes, flowers and crops (up to three in a row) instead of the plant blocking the view; a tree trunk still blocks. With `SightStealth` on, each plant still makes the target harder to see (below). `false` = a plant blocks sight. |
| `<Combat><SightStealth>` | `true` | Survivors and bandits use the game's stealth value (the one your HUD stealth meter shows: light, crouching, moving) when they look at you or at another survivor or bandit, also while choosing whom to fight. How sharp-eyed each one is comes from its class's `SightLightThreshold` in entityclasses.xml. Zombies are not affected. `false` = they ignore stealth the way SCore does. |
| `<Combat><SightPlantStealthFactor>` | `0.7` | Each plant between the watcher and the target multiplies the target's stealth value by this, so crouching behind a bush hides you sooner. `1` = plants do not help; `0` = any plant hides completely. |
| `<Combat><SightPlantStealthFactorHidden>` | `0.35` | The same, but for a character marked `$HidesInPlants` while he crouches (see POI-BUILDERS). |
| `<Combat><ClearShotStepMetres>` | `2` | A hire with a gun steps this far aside, away from you, when you stand in her line of fire to an enemy that is going for you, then shoots. Clamped 1 to 5. |
| `<Combat><ClearShotLeaderRadiusMetres>` | `20` | ...for an enemy within this distance of you. Clamped 2 to 60. |
| `<Combat><HealGunThreatMetres>` | `25` | A survivor or bandit does not start a self-heal while an enemy holding a gun has her in sight this close: she keeps fighting and heals once it is gone. A melee threat (a zombie) still counts only within arm's reach, as before. A burning one is never held. 0 = never held. Clamped 0 to 60. |
| `<Combat><DeathGuardLeashMetres>` | `8` | A death-guard hire (her leader died, she guards his dropped bag) keeps every fight move within this distance of his death spot and within `<Bandits><SentryLevelMetres>` of its floor; her cover and back-up reach counts to it too. Bandits keep `<Bandits><SentryLeashMetres>`. Clamped 2 to 30. |
| `<Cover><FruitlessRescanMoveMetres>` | `2` | After a cover search that found nothing, a survivor or bandit does not search again until she, her enemy or the centre of the area she keeps to has moved this far (or her enemy, the kind of cover she is after or her order changed, or `FruitlessRescanSeconds` passed); meanwhile she does what she does when there is no cover. Clamped 0.5 to 10. |
| `<Cover><FruitlessRescanSeconds>` | `9` | The longest a cover search that found nothing is trusted while nothing moves. It saves searches only when it is longer than `<Cover><NoSpotCooldownSeconds>` and `<ShelterRescanSeconds>` (3 each): a spot that frees up while nothing moves is found up to this much later. Clamped 0.5 to 15. |
| `<Follow><PaceNearDistMetres>` | `8` | At or below this distance from you she walks. Clamped 1 to 60. |
| `<Follow><PaceFarDistMetres>` | `10` | At or above it she moves at full speed; between the two her speed blends (it must be above the near distance). |
| `<Follow><PaceNearWalkSpeedMps>` | `2` | Her walking speed near you, in metres per second (0.5 up to below 2.75, where the run starts). |
| `<Loot><BagMaxMedical>` | `5` | Medical items she keeps in her own bag from looting; the rest goes to her harvest window. Clamped 0 to 100. |
| `<Loot><BagMaxAmmoStacks>` | `2` | Full stacks of each carried gun's ammo she keeps in her bag. Clamped 0 to 20. |
| `<Loot><VerticalReachMetres>` | `3` | How far above or below you she loots (one storey). Clamped 1 to 20. |
| `<Loot><BagRangeMetres>` | `25` | How far from you, flat, she still takes a loot bag on the ground; any height she can reach. Clamped 1 to 100. |
| `<Loot><ReturnTeleportMetres>` | `40` | After "loot around here" ends: within this distance she walks back to you, beyond it she teleports. Clamped 5 to 44. |
| `<Loot><LowTopMetres>` | `1.1` | A container whose top is at most this high above her feet is looted kneeling; a taller one she leans into standing. A loot bag on the ground is always looted kneeling. Clamped 0.3 to 3. |
| `<Loot><ChainMetres>` | `1` | When her next bag or low container is this close to where she knelt, she stays kneeling and turns to it; she stands up only when the chain ends. 0 turns chaining off. Clamped 0 to 4. |
| `<Loot><ChainHoldSeconds>` | `3` | After a rummage with a close target left, how long she stays kneeling for it before standing up after all. Clamped 0 to 10. |
| `<Loot><DrawGraceSeconds>` | `3` | After a rummage her weapon stays stowed this long, so it does not flash out between containers; a fight draws at once. 0 draws as soon as the rummage ends. Clamped 0 to 30. |
| `<Loot><FaceWaitSeconds>` | `1.5` | The longest she turns toward a bag or container before her rummage starts anyway. Clamped 0 to 5. |
| `<Loot><TurnDegreesPerSecond>` | `360` | How fast she turns to face what she loots. Clamped 90 to 1080. |
| `<Loot><BagSightLift>` | `0.5` | A loot bag's eye line is checked to a point this far above it, so a bag in a dip is still seen; a wall or hill between still hides it. Clamped 0 to 1.5. |
| `<Loot><RummageBreakMetres>` | `12` | While she follows you and rummages, she stays at it while you are within this distance; farther, she breaks off and follows (the items stay put). Clamped 4 to 40. |
| `<Loot><ResumeFreeSlots>` | `5` | When her pack fills, a loot, harvest, mine, clay, pick or butcher job pauses instead of ending: she comes back to you, and once you empty her pack to at least this many free slots she goes back to the job. Clamped 1 to 100. |
| `<Loot><ResumeMinutes>` | `10` | How long that pause waits for an emptied pack; after it, or on any other order, she just follows. 0 turns the pause off (a full pack ends the job). Clamped 0 to 240. |
| `<Ammo><RestockTarget>` | `120` | Inside your (or an ally's) land claim, a hired survivor whose spare rounds for her gun drop below one magazine tops up to this many from the base: broadcast containers anywhere in the claims and your unlocked chests within `<BaseKeeper><ChestReachMetres>` of her, never her own pack or another player's box. `0` turns it off. Clamped 0 to 5000. |
| `<Ammo><RestockCheckSeconds>` | `5` | How often she checks her ammo. Clamped 1 to 120. |
| `<Ammo><RestockRetrySeconds>` | `30` | After a check that took nothing (no ammo in the base, or she is outside it), the wait before the next. Clamped 5 to 600. |
| `<Leadership><RegenPerSecond>` | `0.5` | Your Charismatic Nature rank 1 and up: health your hired survivors regain per second. Clamped 0 to 5. |
| `<Leadership><DamageBonus>` | `0.2` | Rank 2 and up: the extra damage they deal (0.2 = 20%), to blocks too with a melee weapon. Clamped 0 to 1. |
| `<Leadership><DamageResist>` | `0.1` | Rank 3 and up: the share of damage they do not take (0.1 = 10%). Clamped 0 to 0.9. |
| `<Bandits><PlayerDamage>` | `0.5` | The damage a bandit deals to PLAYERS, as a multiplier on her weapon's (1 = full, 0.5 = half); survivors, zombies and animals take full damage. Change it while playing with `sdcsbandit damage <x>` (lasts until the game restarts). Clamped 0 to 5. |
| `<Bandits><SpreadByTier>` | `0,0,0` | An extra spread for a bandit by her tier (T1, T2, T3), added to her hip spread on top of the aim every survivor of her level has (see *Survivors' aim*): 0 = she shoots like a survivor of her level, 1 = one more hip spread of her gun. Clamped 0 to 10 each. |
| `<Bandits><MatesMinMetres>` | `2` | The nearest a tier 2 or 3 bandit's mates spawn to her. |
| `<Bandits><MatesMaxMetres>` | `6` | The farthest her mates spawn from her. |
| `<Bandits><MatesMinPlayerMetres>` | `20` | A mate never spawns closer than this to a player (the `sdcsbandit group` test command ignores it). |
| `<Bandits><LineCooldownSeconds>` | `60` | After a bandit's one cold line (the moment she locks onto a person to fight: a player or an NPC, never a zombie or an animal), the wait before she can say another. Text to speech only. Clamped 0 to 3600. |
| `<Bandits><LineGroupQuietSeconds>` | `10` | After a bandit's line, every other bandit within `<LineGroupMetres>` of where she said it stays quiet this long, so only the first of a group speaks. Clamped 0 to 3600. |
| `<Bandits><LineGroupMetres>` | `32` | The reach of that quiet window (the distance a line is heard at). Clamped 0 to 200. |
| `<Bandits><SleepWake>` | `true` | A sleeping bandit hears and sees like a sleeping zombie, in any room (her class's `Sleeper*` values on `npcSdcsBase`): a player's noise heard at her distance over her wake threshold, or a player within `<WakeSightMetres>` in her `<WakeSightAngle>` cone in enough light, wakes her and makes him her target; an enemy survivor's sounds and light count too. Walking into her room still wakes it. `false`: she wakes only with her room, a hit or a blast. |
| `<Bandits><WakeCheckSeconds>` | `0.5` | How often a sleeping bandit listens and looks. Clamped 0.25 to 2. |
| `<Bandits><WakeSightMetres>` | `6` | Her sleeping sight range. Clamped 2 to 20. |
| `<Bandits><WakeSightAngle>` | `180` | Her sleeping sight cone in degrees. Clamped 30 to 360. |
| `<Bandits><WakeSightCurveMetres>` | `30` | The distance over which the light she needs rises from point blank to far (a sleeping zombie's curve). Clamped 10 to 60, never below `<WakeSightMetres>`. |
| `<Bandits><NoCoverRetrySeconds>` | `3` | A melee bandit who finds no cover to bound to charges at once, and does not look for cover again for this long. Clamped 0.5 to 30. |
| `<Bandits><SeekTrackMetres>` | `20` | A bandit who cannot see her target within this distance goes to where he is, not where she last saw him. 0 = always the last-seen spot. Clamped 0 to 40. |
| `<Bandits><SentryLeashMetres>` | `1.5` | A posted bandit (a sentry, or a guard spawn block's Stay or Guard bandit) keeps every fight move within this distance of her post, and within `<SentryLevelMetres>` of its floor; no move drops her off it. Clamped 0.5 to 6. |
| `<Bandits><SentryLevelMetres>` | `0.6` | How far above or below her post's floor still counts as on it for a posted bandit: her cover spot, the path to it, her step out and her dodge step all stay within it. At the default only her own floor counts. Clamped 0.25 to 2.5. |
| `<Bandits><PoiTierFloors>` | `1:1,2:1,3:100,4:100,5:400,6:400` | POI difficulty:gamestage pairs: a bandit sleeper group in a POI of that difficulty picks its bandits at no less than that gamestage (100 = tier 2, 400 = tier 3); the party's own gamestage still raises it. Zombies are unchanged. |
| `<Bandits><SeekArriveHeightMetres>` | `2.5` | A seek (a bandit's, or a hire's) has arrived only within this height of its spot, so a seeker on the floor above or below keeps walking her path to the stairs. Clamped 1 to 10. |
| `<Progression><KillXpWorkaround>` | `true` | Works around an SCore error on NPC kills that also costs the player the kill's XP; leave on. |
| `<AwakeStealth>` | `true` | An awake zombie that is after a survivor sees her through the player's own stealth test: crouching, a held light and speed count, as they do for you. |
| `<HitBoxFix>` | `true` | Keeps the oversized invisible box around survivors, NPCs and pets switched off, so hits land on the body. `false`: the game's behaviour (hits up to 30 cm off the body). |
| `<SdcsArmorExcludeGroups>` | `groupPreacher` | Outfit groups (comma separated) never rolled for an NPC, e.g. coats whose cloth only animates on players. |
| `<SdcsCustomCharacterChance>` | `0.1` | Chance that a new wild survivor is dressed as one of an online player's own creator characters of the same sex; `0` turns it off. |
| `<NpcPathRules>` | `true` | Survivors plan their paths with the mod's own rules (the hazard and drop settings below). `false`: the game's own paths. |
| `<NpcPathSpikeCost>` | `1000000` | Path cost of a spike / barbed wire / cactus cell: crossed only when there is no other way. |
| `<NpcPathHazardMargin>` | `4000` | Extra path cost next to a hazard, so paths keep one cell off spikes where there is room. `0` disables it. |
| `<NpcPathWallCost>` | `1000000` | Extra path cost, on every survivor's and bandit's path (hired or not), of a step into a block at head height (a wall she can neither walk through nor hop): she takes an opening or a way round, and goes at the wall only when there is no other way. A thin barrier (a shanty wall, a railing, a thin fence) is never crossed at all: her path goes round it, through a door or an opening. None of them can break blocks. `0` disables both. |
| `<NpcPathMaxDropBlocks>` | `5` | A path never drops more than this many blocks in one step. `0` disables the cap. |
| `<NpcPathMaxSpeed>` | `6` | Top speed (m/s) while she follows a path, so she cannot run past its turns (on open ground she keeps her full sprint). `0` disables it. |
| `<NpcHazardWalkSpeed>` | `1.0` | Speed cap (m/s) while a hazard is right next to her feet. `0` disables it. |
| `<NpcHazardStepOut>` | `true` | A survivor standing in contact with a hazard steps to the nearest safe spot. |
| `<NpcHazardFirstTouchDamage>` | `1` | Damage of her first brush against a hazard; a second touch within 3 s deals normal damage. `-1` turns the rule off. |
| `<Emotes><Reactions><Hire><Enabled>` / `<Chance>` / `<CooldownSeconds>` / `<PendingSeconds>` | `true` / `1` / `0` / `3` | The salute right after you hire her (see *Hired survivors: emotes*). `Chance` is how often it plays when it may (0 to 1), `CooldownSeconds` the least time between two of hers (0 = none), `PendingSeconds` how long it waits for her to be free after the hire before it is dropped (0 to 10). A survivor already hired when the world loads never salutes. |
| `<Emotes><Reactions><FightEnd><Enabled>` / `<Chance>` / `<CooldownSeconds>` | `true` / `0.5` / `60` | The fist bump when a fight she was in is over (hires only, never bandits). |
| `<Emotes><Reactions><Greet><Enabled>` / `<Chance>` / `<CooldownSeconds>` | `true` / `0.5` / `120` | The wave with her greeting when it plays toward you. Every reaction is skipped, never saved for later, while she fights, walks, plays another animation, works a job, sits in a vehicle or has an enemy near. |
| `<Emotes><Idle><Enabled>` | `true` | Idle moments: a hire who stands still and free now and then plays a short idle clip. `false` = never. |
| `<Emotes><Idle><StillSeconds>` | `20` | How long she stands still and free before the first one. |
| `<Emotes><Idle><MinIntervalSeconds>` / `<MaxIntervalSeconds>` | `120` / `300` | The random wait between two of her idle clips; each survivor rolls her own. |
| `<Emotes><Idle><StillMetres>` | `0.5` | A move of more than this between two looks a second apart (hers, or her leader's while she follows) counts as walking. |
| `<Emotes><Idle><Clips>` | `stretch,warmup,whatever` | The idle clips (ids from the base mod's `Config/SdcsAnimClips.xml`, comma separated); an unknown id is skipped with a warning. |
| `<VerboseLogging>`, `<JitterWindow>`, `<SdcsPitchPin>`, `<PlayerCtrlOwnController>` | | Diagnostics and engine switches; leave as shipped. |

Set elsewhere: wild and hired survivors walk at the same pace (`MoveSpeed` 1.5 on `npcSdcsBase` in `Config/entityclasses.xml`); about half as many survivors spawn in the open world as in 3.3.02 (`Config/entitygroups.xml`), and an
unhired survivor loots at most one container a minute (`Config/buffs.xml`, `SdcsLootWildCooldown`), taking every item tagged `medical`,
`ammo`, `food`, `drinks` or `csm` (skill magazines) plus anything worth 100 or more, and leaving the rest (the wild loot action in `Config/utilityai.xml`,
`listedTags` and `minValue`). Survivors ignore a zombie that is still
asleep, and an unhired survivor carries on with what she is doing until a zombie actually threatens her.

## Stories ("tell me a story")

Each character's stories live in `personalities.xml`, inside her `<Personality>`:

```xml
<Stories>
    <Story>
        <Text lang="en">I found a working radio once. Spent three days trying to reach someone.</Text>
        <Text lang="de">Ich habe mal ein funktionierendes Funkgerät gefunden. Drei Tage lang habe ich versucht, jemanden zu erreichen.</Text>
    </Story>
</Stories>
```

When the player asks for a story ("tell me a story", or any alias of a `CustomDialog` trigger with an empty `<Response>`), she
picks one of hers at random. The `en` text is what she speaks; the subtitle shows the player's language (`lang` is one of the
12 subtitle codes: en, de, es, fr, it, ja, ko, pl, pt, ru, tr, zh), falling back to `en`. The older plain form
`<Story>text</Story>` still works, English only. A character with no stories tells her `<Backstory>` instead.

The `<Stories>` block in `phrasetriggers.xml` is not read: stories belong to each character.
