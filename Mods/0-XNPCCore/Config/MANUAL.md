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
added them and removed the NPC Mod's legacy patrol rows. The SDCS survivors' talk menu no longer
lists them (patrols are on the order wheel's Orders page); the generic NPC dialog keeps them flat in its
Command Menu. They are hidden for a `no_voice`
character because patrol cannot work without the Voice Control component the tag withholds.
A tagged character can be hired and ordered normally, just not put on a recorded patrol.
Ordered to patrol with no recorded route, or with a route recorded somewhere far away, they say so instead of
acknowledging, and keep their current order: show them a route where you are first.

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
- **A random voice:** `<MaleVoice>random</MaleVoice>` (or `<FemaleVoice>random</FemaleVoice>`) gives each NPC one of the voices of their own sex that this install can speak (the VoiceMap rows with a `sex` attribute, see [Custom voices](#custom-voices)), picked from their entity id: they keep it every time they speak, and every machine with the same voices picks the same. The bandits use it. Name a voice instead to pin one.
- **Optional: CombatLines** — `<CombatLines><Line>Time to die.</Line> ...</CombatLines>`: fixed lines a bandit says, unchanged, the moment they lock onto a person to fight (see [Bandits](#bandits)). Only a bandit says them; text to speech only, English. The shipped bandit personalities are `bandit_cold_male` and `bandit_cold_female`.
- **Optional: VoiceMode** — Set `<VoiceMode>clips</VoiceMode>` to use pre-recorded OGG clips instead of TTS. See [Voice Clips Mode](#voice-clips-mode) below.
- **Optional: ForceLanguage** — Set `<ForceLanguage>ja</ForceLanguage>` (or any ISO code) to force this NPC's subtitle language regardless of the player's game language setting. Every caption they show follows it: their replies, order acknowledgements and canned lines ("I'm done crafting."). An item named in a line ("I need Wood for that.") keeps the name the player's game shows.
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
| `Pickup` | Yes | Picks up the NPC (SCore PickUpNPC). Refused while their pack holds anything, the way a vehicle with items in its storage is |
| `OpenInventory` | Yes* | Opens NPC's trade/inventory window. *Traders are the one exception: opening a trader's inventory needs no leader (it redirects to the trade dialog). |
| `SwapWeapon` | Yes | Swaps NPC weapon. See SwapWeapon section below. |
| `ReadBooks` | Yes | The hire reads every schematic and crafting skill magazine they carry that teaches them something, then says one line ("read your books"). No `<ActionParam>`. See *Hired survivors: reading books* |
| `Emote` | Yes | The hire plays a clip of the base mod's clip list (`Config/SdcsAnimClips.xml`) where they stand. `<ActionParam>` = the clip id (`wave`, `salute`, `fistbump`...), or `dance` for a random dance. A squad order makes each member play it. Keep an emote phrase to one content word plus the matcher's stop words ("do a dance", "salute me"), so it fires only when the whole phrase is said. See *Hired survivors: emotes* |
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

Four persistent stances plus one action. Controls hired NPC autonomous combat via the `vcEngage` cvar and `MayEngage` UAI gating.

| ActionParam | Mode | Behavior |
|------------|------|----------|
| `0` | Defensive (default) | Fights what threatens the leader or a squad mate, and anything within the contact radius (`EngageContactRadius`, default 2.5 m) that is literally on top of you. They do not fire first: an enemy that has not noticed the party is left alone until it targets you or a squad mate. One holding a melee weapon steps out to a gunman within `<Cover><DefensiveCloseInMetres>` (5 m) of them, or to one shooting at the party within `<Cover><DefensiveAssistMetres>` (15 m) of you, instead of sheltering at your side. May walk toward an already-authorised enemy within the 8 m advance leash, but holds position instead of chasing. Following you, they take cover or move for a shot only at a spot you can see from where you stand, so they never slip into a building ahead of you; with none they crouch where they are. (Hold fire too; Aggressive and Hunting are not limited.) A Defensive group throws only at enemies that target the party. |
| `1` | Aggressive | Hunts and attacks any enemy on sight; closes freely. |
| `4` | Hunting | Goes after any enemy within `<Cover><HuntingLeashMetres>` (20 m) of you and starts the fight, with every Defensive limit otherwise; never chases past the leash, and a whistled hunter comes back still hunting. With a melee weapon they make stealth kills (see *Hired survivors: stealth kills*). Voice: "hunting mode", "hunt them", "go hunting", "stalk" (the bare word "hunting" is not an alias). |
| `2` | Hold Fire | Self-defense only. No leader-assist. Clears attack targets when set. (Renamed from "full control" on 2026-08-26; "full control" remains a legacy spoken alias for this same stance.) |
| `3` | Disengage (ACTION, not a stance) | Breaks off now: clears targets and sets the stance to `2` (hold fire); their order, their post and their place in the formation stay (the whistle is the break-off that also brings them back). For the break-off window (`DisengageWindow`, default 20 s) the NPC still defends themselves against a live enemy, but walks away the moment that enemy is stunned, ragdolled or dead. |

**Self-defense outranks every stance** — the gate matches `revengeTarget` against the enemy being scored, so an NPC always fights back against whatever damages them, but Hold Fire does not degenerate into free-fight when it happens.

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

**All.** The **All** button above the wheel sends your next order to the whole squad, the same as Shift+click; it is not shown on a wheel opened from a hire's E ring.

**Targeting.** The wheel commands the companion you are looking at, otherwise the last one you gave an order to, otherwise the nearest. Their name sits in the middle of the ring and the mouse wheel cycles through your hires.

**Three menus, one home per command.** The wheel is for **tactics** (how the squad moves and fights). **Press E** on a hire is the
**top menu**: talk, hire or dismiss (asks you to confirm first), **Follow me**, **Come here**, **Equipment...** (the equipment window, below), **Craft...**, and two links: **Order
wheel** opens this wheel on the squad's first page with them shown, and **Jobs and gear** opens it locked on them, on a page that lists their
hold-E ring's entries (the same entries, for players who never hold E). **Hold E** is their **work and gear**.

**From the E ring.** Hold **E** on one of your hires and the ring shows their own menu after Talk: **Jobs**, **Roles**, **Equipment**,
**Emotes**, **Read**, **Progress**, **Inventory** and **Pick me up** (Jobs, Roles and Read only on a survivor who can do that work). Pick an
entry by resting the cursor on it, or by letting go of E over it; a quick tap of E still talks. **Jobs**, **Roles** and **Emotes** open the
wheel straight on that page, locked on them: every pick is theirs, the mouse wheel does not switch to another hire, and resting on the centre
steps back up their pages, then closes the wheel. **Equipment** opens the equipment window (below). **Read** and **Progress** open their talk
menu straight on that page: their books, their progress. Their progress page always
shows their stance, "Stance: Defensive" (or Aggressive, Hunting, or Hold fire), so a hire who will not join a fight can be checked. **Inventory** opens their pack;
**Pick me up** picks them up. A wheel opened from the ring ignores the spot your cursor was resting on when it appeared: move onto a socket to
pick it. Only their leader sees these entries. (The game's own **remove** entry still shows only with the debug menu on.)

**Pages.** The wheel always opens on page 0:

| Page | Sockets |
|------|---------|
| **0 (root)** | `Follow`, `Stay`, `Come here` commit directly; `Equipment` opens the equipment window for the aimed hire (Guard is the voice order and the Orders page's `Guard and return`); `Orders`, `Stance`, `Position`, `Formation`, `Jobs` open a drill-down page. Jobs, Roles, Emotes and their gear are one survivor's menus: they open from their hold-E ring (above), not from this wheel |
| **Orders** | `Crouch` (the 0/1 toggle - Crouch / Resume resolved from their state, exactly as before), `Stand up` (the voice `stand up`, SetCrouch arg 2), `Scout` (the voice wander order), `Guard and return` (patrol the spot: walk out within `<Guard><PatrolMetres>` (6 m), look around, come back, face your way, repeat; fight freely, then back to the spot), `Record patrol` / `Run patrol` / `Cancel patrol` (the patrol trio - one survivor's route; a Shift+click goes to the target, not the squad) |
| **Roles** (from their ring) | One survivor's roles, each a toggle that reads **Stop ...** while they have it (target only, never the squad): `Medic` (the designated medic role), `Cook` (the campfire), `Blacksmith` (the forge), `Chemist` (the chemistry station), `Crafter` (the workbench and cement mixer). A survivor uses a workstation only with its role - see *Hired survivors: crafting* |
| **Their ring** (Press E > Jobs and gear) | Their hold-E ring's entries for you, in the ring's order; a pick does what the ring entry does |
| **Jobs** (from their ring) | `Follow + loot` (starts **follow and loot**, exactly the voice "loot" command - see *Hired survivors: follow and loot* below), `Loot here`, the `Harvest` and `Mine` drill-downs (the wood / plants / crops / butcher orders and the mine / clay orders), the two base-keeper duties `Base Repair` (most damaged block first, the wheel arg `baserepair`) and `Combat Repair` (a block damaged in the last few seconds first, the wheel arg `combatrepair`), and `Remember base` |
| **Stance** | `Aggressive` (1), `Hunting` (4), `Defensive` (0), `Hold fire` (2), `Disengage` (3) - the SetEngage args verbatim |
| **Position** | `Close` (1 m), `Normal` (2.5 m), `Far` (5 m) - the real metres are drawn in the icon slot - `Point` (the voice "take point": the companion leads 2 m ahead of you), `Go there` (aimed at an enemy: the quiet-kill order, see *Hired survivors: stealth kills*), `Jump down`, `Get in` / `Get out` (your vehicle). **Look first, then open the wheel:** `Go there` goes to the spot under your crosshair *when the wheel opened*, not where you aim when you pick it. |
| **Formation** | The 8 compass bearings, N at top and E to the right. Direction only: each NPC keeps its own distance |
| **Emotes** (from their ring) | `Wave`, `Salute`, `Fist bump`, `Stretch`, `Warm up`, `Whatever`, then the `Dances` drill-down (seven dances a page plus More) and the `More` drill-down (`Flip off`, `Insult`, `Loser`, `Threaten`). Only the clips the base mod's clip list holds show - see *Hired survivors: emotes* |

**Commit vs navigate.**
- **Release the key** while hovering a commit socket - the order goes to the **targeted companion**.
- **Click** while still holding the key - the order goes to the **whole squad** (every formation socket is squad-capable; a non-squad socket falls back to the target, so a click always does something).
- **Navigation sockets** (the drill-downs) never commit. They are marked with a small chevron and the navigation border tint. Open one by clicking it, or simply by **resting the cursor on it for the dwell** (0.4 s default) - a faster pass over it does nothing, so a transit can never flip the page by accident.
- **Back out:** rest the cursor on the wheel's **centre** and it returns to page 0. Clicking or releasing over the centre does nothing.

**Stay, Guard, Guard and return.** `Stay` holds them where they stand; `Guard` holds them on the spot where you stand. On
both they shoot from their spot and never back off or run in to melee (the AI shows the `Stay` action for both). `Guard and
return` patrols your spot: out to a random spot within 6 m, a look outward, back to the spot facing your way, again; in a fight
they move freely, then go back to the spot first (the AI shows the `Guard` action).

**Guarding together.** Survivors told to guard the same spot (a squad click on `Guard`, or Guard given to one after another
from where you stand) spread out instead of crowding onto it: the later ones step aside into a line beside the first, facing your
way, then a rank behind, `<Guard><SpacingBlocks>` (2) blocks apart, on spots they can stand on and walk to. A job's post (loot
here, a station, the base) is never moved.

**A recorded patrol across a save.** A save taken while they are recording a patrol brings them back following; the points recorded before the save are kept, so say `set patrol` again to continue recording (or `start patrol` to walk the points already recorded).

Every socket carries the exact router arg of the equivalent voice command, so the wheel and the voice path cannot disagree - the one deliberate exception is the crouch toggle on the Orders page, which offers Crouch **or** Resume depending on their current state (arg 1 or 0) instead of one fixed arg.

### Order Wheel settings (modconfig.xml, top-level `<FormationRadial>` section)

| Setting | Default | Description |
|---------|---------|-------------|
| `<Key>` | `Semicolon` | Unity KeyCode name that opens the wheel. Every letter A-Z has a vanilla default binding, so letters collide. Free keys: Semicolon, Quote (the whistle key's default, see *Whistle*), Comma, Period, Slash, RightBracket, Backslash. Set the key to `None` to disable the wheel (an empty value falls back to the default, Semicolon). An old `<STT><FormationRadialKey>` entry is still read as a fallback. |
| `<TargetLockSeconds>` | `3` | How long the last kill target your crosshair crossed stays locked, so a Go there click that slipped off it onto the ground still takes it. Clamped 0 to 10; `0` turns the lock off. |
| `<MarkKey>` | `Mouse2` | The key that marks the enemy under your crosshair (Mouse2 is the middle mouse button): a red skull goes over it and it is the Go there and Stealth kill target until it dies or is farther than `<MarkRangeMetres>` from you. The key on empty space clears it. It does nothing while you aim down sights. `None` turns it off. |
| `<MarkReachMetres>` | `50` | How far the mark key's own ray reaches (the game's gun ray, so grass and plants do not shorten it). Your crosshair's kill target is always preferred. Clamped 10 to 80. |
| `<MarkAssistMetres>` | `0.6` | The mark key's aim assist: when its ray finds no enemy, the enemy whose chest or head sits nearest your aim line is marked if it is within this many metres of the line (or the `<MarkAssistDegrees>` cone, whichever is wider) and in clear view. Clamped 0 to 2; both at `0` turn the assist off. |
| `<MarkAssistDegrees>` | `1.5` | The aim assist's cone in degrees (at 40 m, 1.5 degrees is about 1 m off the line). Clamped 0 to 5. |
| `<MarkRangeMetres>` | `60` | How far from you a marked enemy may get before the mark ends. Wider than the reach on purpose: the marked target may walk a little before the mark ends. Clamped 10 to 80. |
| `<LookPickDegrees>` | `8` | How close to a hire your aim must be for the combat wheel to open on them alone. On a hire under the crosshair it always does; with an enemy under the crosshair, marked or locked it never does. Clamped 0 to 30. |
| `<Dwell>` | `0.4` | Seconds the cursor must stay on a navigation socket (or the centre) before it navigates. 0 makes hover navigate on the first frame; use 0.3-1.0. |
| `<NavBorderColor>` | `80,180,255,255` | Border ring tint (r,g,b,a) for the navigation sockets (the ones carrying the chevron). |
| `<CommandBorderColor>` | `0,0,0,128` | Border ring tint for the commit sockets. Matches the window XML default, so sockets look unchanged unless you set one. |

## Whistle

The whistle key (default **Quote**, the `'` key beside Semicolon) is for the chaos of a fight: one key, no aiming at anyone. Like the
order wheel it is a menu feature and works with STT/TTS/LLM all off. The order wheel and its key are not changed by it.

- **Tap** the key: every hire breaks off the fight and comes to you, and you whistle. It is Disengage and Come here in one: they drop
  their target, go to hold fire (they still defend themselves), their job ends, and they walk back (or are brought to you when they are beyond
  the return distance).
- **A hire's stance comes back.** A whistle means "come here now", not "hold fire forever": once they are back with you (or the 20 s break-off
  window, `<DisengageWindow>`, runs out) they get the stance they had before the whistle. A second tap before then keeps the first
  stance; any stance order you give them in between (Aggressive, Defensive, Hold fire, Disengage) wins and nothing is given back.
- **Hold** the key (`<HoldSeconds>`, 0.30 s): the **combat wheel** opens and stays open after you let go. One page, nine one-click
  orders: `Follow`, `Stay`, `Aggressive`, `Hunting`, `Defensive`, `Hold fire`, `Disengage`, the **weapon toggle** and `Go there`
  (a tap of the whistle is the recall).
- **The weapon toggle** tells the target, or the whole squad with **All**, to fight with their melee weapon or their gun. Its label shows
  what they hold now and what a click does: `Melee -> Ranged`, `Ranged -> Melee`, or `Mixed -> Ranged` for a mixed squad. The choice
  sticks through fights, weapon swaps and a save; with no usable gun they stay on melee and say "I have no ammo for a gun." Voice:
  "go ranged", "go melee".
- **Go there** sends the aimed hire where you look; aimed at an enemy it is the quiet-kill order (see *Hired survivors: stealth
  kills*). It goes to one hire even with **All** picked.
- **The names row.** Above the ring, one button per hire (up to six) and **All**. Click a name and they are the target (their name shows
  in the centre; without a click the wheel picks as the order wheel does: the hire you look at, else the last one, else the nearest,
  and the mouse wheel cycles). Click **All** and your next order goes to the whole squad; **Shift+click** an order does the same.
  A hire's name shows their stance: "Emily (Hold fire)", "Jill (Aggressive)", "Bob (Defensive)".
- The whistle key again, a right click or Esc closes the combat wheel. The key is ignored while a window holds the cursor, the chat
  box is open, or the order wheel is open.

### Whistle settings (modconfig.xml, top-level `<Whistle>` section)

| Setting | Default | Description |
|---------|---------|-------------|
| `<Key>` | `Quote` | Unity KeyCode name of the whistle key. Every letter A-Z, 0-9 and the F-keys carry a game default binding. `None` disables it (an empty value falls back to Quote). |
| `<HoldSeconds>` | `0.30` | A press held this long opens the combat wheel; released sooner it is a tap (the whistle). |
| `<Sound>` | `true` | The tap plays a whistle at you (a base-game trader whistle). |

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
- A clips character shows only their clips' own captions by default (`<Subtitles>clip</Subtitles>`): the generic text for an order ("On my way") is not shown, because it would not match the recorded line. A clip without a caption plays with no text. Set `<Subtitles>none</Subtitles>` for no text at all, or `<Subtitles>all</Subtitles>` to show the generic text as well
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
| `<ModelFilename>` | `Qwen3-4B-Instruct-2507-Q4_K_M.gguf` | The .gguf in Resources/Models/Llama to load. Empty = the first .gguf found. |
| `<Model>` | `Qwen3-4B-Instruct-2507-Q4_K_M` | Model name. Used verbatim for an external API endpoint. For the local llama-server, autodetect of the `.gguf` in `Resources/` overrides it, so this value only appears in the log when no model file is found. |
| `<TimeoutSeconds>` | `30` | How long to wait for LLM response |
| `<MaxTokens>` | `200` | Maximum response length in tokens (64-2048). Use 1024-1536 for reasoning models. A value saved in the settings window overrides this file value at launch |
| `<Temperature>` | `0.8` | Randomness 0-1 (higher = more creative, lower = more focused) |
| `<Reasoning>` | `off` | The bundled server's thinking switch: off, on or auto. Keep it off unless you raise MaxTokens for a thinking model. |
| `<MaxSpokenSentences>` | `3` | At most this many sentences of a reply are spoken and shown; 0 is no cap. A stray think-close tag from the model is never spoken or shown. |
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
| `<ResumeMetres>` | `10` | Flat distance (clamped 2 to 100) at which a live leader near the post wins their hires back to Follow |

### Subtitles Section (NPC text on screen)

One switch for the on-screen text of every NPC at once, for a clear screen. It caps each character's own `<Subtitles>` in `personalities.xml`: the stricter of the two wins, so `none` here silences every NPC's text whatever their own setting says. The NPC's voice is untouched: TTS replies, greetings, order acknowledgements and voice clips still play. Display is per player: each player's own `modconfig.xml` decides on their screen. The boot log shows the value in force (`NPCText=`).

| Setting | Default | Description |
|---------|---------|-------------|
| `<NPCText>` | `all` | `all`: each character's own setting decides (today's behaviour). `clip`: only voice-clip captions, never the generic reply text. `none`: no NPC text at all. Any other value runs as `all`, with a warning in the log. ⚠️ With `none` and no voice package (no TTS, no clips) your NPCs give no feedback at all. The mod's own notices ("No NPC nearby", microphone errors, settings messages) and the book menu's greyed reason still show. |

### NPCToNPC Section (Ambient NPC-to-NPC idle chatter)

| Setting | Default | Description |
|---------|---------|-------------|
| `<Enabled>` | `false` | Turn the old LLM NPC-to-NPC chatter on/off. Off by default: POI residents have their own canned small talk (see "POI residents: small talk, greetings and the evening campfire"). |
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

A hire's contract carries over between sessions and ends when they are dismissed (a new hire starts at week 1). When they ask for their
pay, answer yes (or pay) or no; during the grace days, say **pay** to pay what you owe and end the grace.
Or tap **E** on them and pick **Pay / Gift...** (its **Give a gift...** row opens the gift window, see Gifts below): while their week is due it shows **Pay this week (N coins)**, the same as saying
pay; otherwise a greyed **Paid up. Next pay day: day D (N coins)** line shows where you stand. A payment takes nothing unless
you can cover the whole week: short of it, you keep every coin you had.

### Relationship Section (trust, affinity, the price, gifts)

| Setting | Default | Description |
|---------|---------|-------------|
| `<Enabled>` | `true` | Master switch: false makes the whole feature inert (no state, no price effect, no gifts). |
| `<OptOutTag>` | `norelationship` | An entity class tag: a hire whose class carries it is fully opted out of relationships. |
| `<GlobalShare>` | `0.25` | Share of a per-hire trust/affinity move that also ripples to your global value (so mistreating one hire affects the rest). |
| `<RecoveryRate>` | `0.5` | A positive move while the value is below 0 is multiplied by this (recovery is slower than damage). |
| `<ScarThreshold>` | `-50` | The hire's effective trust at or below this scars them. |
| `<ScarCeiling>` | `50` | A scarred trust can never again raise their effective trust above this. |
| `<PriceMinMultiplier>` | `0.85` | The weekly price multiplier at +100 trust (the floor). |
| `<PriceMaxMultiplier>` | `1.50` | The weekly price multiplier at -100 trust (the top). |
| `<PaidTrust>` | `2` | Trust on a paid week. |
| `<DownedTrust>` | `-4` | Trust when they go down. |
| `<RevivedTrust>` | `3` | Trust when their leader revives them. |
| `<RevivedAffinity>` | `3` | Affinity when their leader revives them. |
| `<HireDiedGlobalTrust>` | `-5` | Global trust when one of your hires dies (your global, not one hire). |
| `<AttackerKilledTrust>` | `1` | Trust when you kill an enemy that was targeting them. |
| `<BloodMoonAffinity>` | `4` | Affinity for a blood moon survived together. |
| `<GiftValueUnit>` | `50` | Divisor for gift points (diminishing returns on the gift's economic value). |
| `<GiftDailyCap>` | `6` | The per-hire daily cap on gift points. |
| `<GiftItems>` | `medicalFirstAidBandage:2,medicalFirstAidKit:2,drugPainkillers:1` | Shipped as three `<Item name="..." kind="medical" stacks="N"/>` entries (that is the list they read as). The medical gifts their own bag keeps, `stacks` per item (besides their drinks, their ammo and the hire currency). Add an `<Item>` for a custom gift; `kind="medical"` is the only item kind today. The older text form `name:stacks,...` still reads, and so does an old `<GiftMedical>` name list (one stack each, with a warning). |

See the *Hired survivors: relationships* section for what each value does in play.

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
| `<DropCheckDepth>` | `1` | Deepest drop in blocks the NPC will take on its own: a 1-block step stays walkable, a drop of 2 or more is never their own choice. |
| `<MaxFollowDrop>` | `5` | The one exception: how much deeper the NPC follows when you just dropped (jump/fall), detected via a 2 s high-water mark. Also controls the trail hop down budget: there is no separate hop-down key (removed S6, 2026-09-05), so this value is the only knob for how deep a hop may fall. |
| `<LeaderDropGraceSeconds>` | `5` | How long the MaxFollowDrop allowance stays active after your descent. |
| `<ChatWeaponGraceSeconds>` | `20` | Seconds a hire keeps their weapon put away after a chat line, so it stays away through a conversation; a fight draws it at once. |
| `<TrailHopMaxHoriz>` / `<TrailHopMaxClimb>` | `2.5` / `1.6` | When the trail leads to a crumb the NPC cannot walk to, it hops (short teleport) to it within these limits. The down budget has no key (removed S6, 2026-09-05): it follows `<MaxFollowDrop>` - the NPC must not hop a fall it would refuse to walk. An old `<TrailHopMaxDrop>` key in a user config is silently ignored. |
| `<PathFirstFollow>` | `true` | Far-from-leader travel is A*-first; set false for the old servo-first behaviour (A* only on failure). |
| `<PathReleaseDist>` | `2.5` | Release side of the horizontal hysteresis pair for the path/servo handover (engage side has no key). `PathHandoverDist` is derived from `<TrailHandoverDist>` at load (removed S6, 2026-09-05); an old `<PathHandoverDist>` key in a user config is silently ignored. |
| `<PathVerticalDist>` / `<PathVerticalRelease>` | `1.0` / `1.0` | Vertical hysteresis pair for the path/servo handover. |
| `<PathPendingMaxSeconds>` | `2.0` | How long a follower lets a route search that is still running finish before they ask for a new one. Your hire re-aims their route about every half second while you walk; asking again throws the unfinished search away, so near a gap or a step they could stand still. Lower it to re-aim sooner; raise it to wait longer on a hard search. |
| `<LadderOrderedUseAstar>` | `true` | INERT key (nothing reads it any more): the ladder climb hands A* one exact path unconditionally. The loader still reads the value in case the flag is wired back. |
| `<PatrolAStarLeg>` | `true` | Recorded-patrol legs between waypoints are A*-owned. |
| `<PatrolStartMaxMetres>` | `40` | "Patrol your route" is refused out loud when their recorded route's nearest point is farther than this from them ("My patrol route is far from here"); with no route they say they have none yet. Either way they keep their current order. Clamped 10 to 500. |
| `<PatrolStraightenToleranceMetres>` | `0.9` | A recorded route is stored block by block, so a diagonal walk is a staircase of 1 m steps. When they start patrolling, those runs are merged into straight segments; this is how far a recorded point may sit off the straight line and still be dropped. Real corners, stairs and ledges are kept. `0` turns it off (they walk every recorded point). Clamped 0 to 2. |
| `<PatrolStraightenMaxSegmentMetres>` | `8` | The longest straight segment the merge may make, in metres. Clamped 2 to 30. |
| `<ChatFollowEscapeDist>` | `6.0` | While mid-conversation the NPC holds its ground within this distance and resumes following beyond it. |
| `<JumpDownWaitSeconds>` | `10` | After you drop further below the NPC than MaxFollowDrop, it waits at the lip this long before the teleport backstop re-arms (you hold the JumpDown command; you decide). |
| `<TeleportMinGain>` | `1.5` | Refuses teleports that would move the NPC less than this (no-op guard). |
| `<EngageContactRadius>` | `2.5` | Defensive (Cover Me) also engages an enemy this close to you even if it targets nobody. |
| `<EngageNearRangedRadius>` | `20` | Defensive (Cover Me) with a gun: they fight any enemy they can see within this distance of themselves, whatever it targets, without leaving you to chase it. Melee hires keep `EngageNearRadius` (5). Clamped 5 to 60. |
| `<EngageNearRangedReleaseRadius>` | `22` | An enemy they are already fighting that way stays their fight out to this distance (it must not be inside `EngageNearRangedRadius`). |
| `<DisengageWindow>` | `20` | Seconds a "disengage" NPC keeps self-defending against a live enemy but walks away the moment it is down. |
| `<DefaultTacticalMode>` | `false` | If true, every existing hire is put into tactical (command-only) mode once at the start of each session (re-applied at boot, not just new hires) |
| `<FormationDistances/Tier>` | `1=2m, 2=5m` | Slot distances keyed by index. Add tiers with no code change. |

The breadcrumb trail itself is recorded from your own movement (`TrailRingSize` 64 crumbs, `TrailSpacing` 1.0 m, `TrailSampleInterval` 0.1 s, `TrailRosterInterval` 2.0 s, `TrailSnapEpsilon` 0.2, `TrailSnapMaxDrop` 1.2, `TrailRecordMaxScan` 3, `TrailMaxAgeSeconds` 180). Tune those only with the log open - the inline comments in `modconfig.xml` carry the reasoning.

---

## Party (the 8-member cap)

Your hires form a party of at most **8** members (the base mod's `Settings.xml`,
`<PartyMaxMembers>`). A hire is **in** the party while their order is **Follow** or
**follow-and-loot**; **Stay**, **Guard**, **Patrol** and the base jobs (loot here,
harvest, mine, craft, base keep) take them out of it, and **Follow** puts them back -
so a hire working at base or on patrol does not count against the 8. The
companion list in the top-left corner shows the party, and now has room for all
8.

When the party is full, a 9th member is refused, with a "Your party is full
(8/8)." tooltip (the count follows `<PartyMaxMembers>`):

- **Hiring** a new survivor is declined - the dialog's confirm button and the
  voice **"hire"** both refuse before anything is hired.
- **Follow**, **Come here**, **follow-and-loot** and the **whistle** on a hire
  who is not already in the party are refused - they keep their current order.
  A hire who is already in the party is never refused by these orders (their
  seat is already counted).

To make room, put a member on **Stay**, **Guard**, **Patrol** or a base job -
they are out of the party while they are on that order.

## Hired survivors: follow and loot

A hired survivor can loot while they follow you. Start it from the order wheel (**Jobs > Follow + loot**) or by
voice (**"follow and loot"**, **"loot around here"**, **"scavenge here"**). Their order stays **Follow**; looting is a mode on top of it.

- **When they loot:** only while you stand still (about a second without moving). While you walk they just follow; a trip they have already
  started is finished.
- **What they look at:** containers within **15 m** of you (and within 3 m up or down), in the building you are in. Loot bags on the ground
  (zombie and bandit death bags nobody has opened) first, because they vanish, and from farther: within **25 m** of you on any floor they
  can reach (`BagRangeMetres`); then loose piles (ammo, medical, books, food, chem, liquor), then
  the rest, nearest first. They open a bag as if you opened it, so its loot (and a bandit's weapon) is yours, rolled at your loot stage. They
  take everything: usable items to their bag (up to its caps), the rest to their harvest window.
- **What they skip:** containers already opened (by them or anyone), locked or padlocked ones, player storage, anything behind a closed door or a
  wall, and anything another hired survivor has already claimed.
- **Reach:** they open a container from up to **4 m** (the player's own reach), measured from their eyes to the nearest part of the container,
  and only with a clear line to it.
- **Giving up:** a container they cannot get to is left alone for 5 minutes. A trip is dropped if you walk more than **20 m** from that
  container. A container they have emptied is never picked again; an emptied loose pile disappears.
- **Danger:** they do not loot with an enemy within 15 m. A sleeper that spots them sends them back to you (follow); once it is dead they can go
  back to the room.
- **Ending it:** any other order, dismissing them, or a full harvest window.
- **Debug view:** with `VerboseLogging` on, a smoke marker rises over the container they are heading for. It is off in normal play.

The scan, reach and range numbers are on the hired loot action in `Config/utilityai.xml` (`leaderRange="15"`, `leaderAbortRange="20"`).

## Hired survivors: the equipment window

**E > Equipment...** (or hold E > **Equipment**) opens one window for a hire's gear. On the left their loadout: **Head**, **Chest**, **Hands**,
**Feet** (the icon and name of the armour piece they wear; the piece they came with reads "(own piece)"), **Weapon** (the one in their hand, or the one they put away to
talk; "Bare hands" when none) and **Ammo** (the type loaded; not shown for a melee weapon). Pick a row and its choices show on the right,
all from **their own pack**: armour that fits that slot (a piece that would protect them but has no model to show, or does not fit them, is
greyed with the reason), every weapon they carry plus bare hands, every ammo type for their gun with how many they have. One click wears,
swaps or loads it ("Own piece" puts the piece they came with back on), the same as the old menu rows did, and works the same on a
dedicated server. **Show backpack** opens their pack to move things in or out (the window comes back when you close it). With a debug
switch on (the mod's, or the game's `dm`) a second button, **Show bag**, lists what is in their private bag (meds, ammo, drinks, coins)
with counts. The window
replaced the old Weapons, Ammo and Wardrobe pages; voice and the order wheel still swap weapons and ammo as before.

## Hired survivors: ammo

- **Choosing their ammo:** **E > Equipment...**, then the **Ammo** row, lists each ammo type of their gun they carry, with how many; the one loaded is marked.
  Pick one and they load it at once: the rounds in their gun go back to their pack first. While they talk to you their gun is put
  away: it takes the new type then, and they load it when they draw it. Or say it: **"use armor
  piercing"**, **"use hollow points"**, **"use slugs"**, **"use breaching slugs"**, **"use standard ammo"**, **"use flaming
  arrows"**, **"use exploding arrows"**. The choice stays with that gun. They tell you when they have none of that type, hold no
  gun, or their gun takes no such ammo.
- **When it runs out:** they switch to any other type they have for that gun and stay on it until you choose again.
- **At your base:** inside your land claim they top their ammo up from the base's containers (`<Ammo>` in the settings table).

## Hired survivors: crafting

Ask a hired survivor to make something: **E** on them, **Craft...** (or **Craft** on their hold-E ring). Their craft window opens, in
the look of your own crafting window:

- **Tabs:** each station in reach they can work at first (**Campfire**, **Forge**), then **All**, then your crafting window's own
  categories (Basics, Resources, Tools/Traps and so on). When the tabs do not all fit, the last one pages to the rest.
- **The list:** every recipe they have unlocked, with the most they can make of each right now. One they are short for is greyed ("Not
  enough", or "Tool missing" when the station lacks its tool). Two recipes for the same item (Cloth Fragment from cotton or from
  yucca) also show their main ingredient. Recipes they have not unlocked are not listed.
- **The panel:** the picked recipe's icon and name, and for each ingredient what they have against what they need (or the item's
  description). Set how many with minus, plus and max (never more than they can make now) and press **Craft**. A greyed recipe
  cannot be crafted.
- **Their job:** the row at the bottom shows what they are making and how many are left ("Making Forged Iron: 3 left", or "Not
  crafting"). **Cancel** stops it, the same as saying "stop".
- The window closes with Escape, when they die, or when you walk out of talking range. On a dedicated server it waits for the
  server's list before it shows their recipes.

A station recipe also needs the station's tool: at a campfire most recipes need a cooking pot (some a grill or a beaker) in the
campfire, so a bare campfire offers only charred meat (and baked potato once they have learnt it). A tool they carry (their pack, or
a chest of yours in reach) counts: they put it into the station's own slot when they craft, and it stays there. A new Cook knows
only the basics (boiled water, charred meat and a few more); they learn the rest by cooking, or from a book you give them (see
*Hired survivors: reading books*).

- **What they can make:** the recipes you can make in your own backpack, and the recipes of a workstation you stand at (below), and only
  the ones they have unlocked themselves: the open recipes from the start; one that needs a schematic or a perk only once they have it,
  or once their crafting skill reaches it (*Learning a craft*, below). The
  server's crafting options apply to them too: backpack and workstation crafting, the highest tech tier, biome progression.
- **Materials:** shared (broadcast) storage inside your land claim when they work at your base, then any unlocked chest you or one of
  your allies placed within **15 m** of their spot (never another player's), then their own stores: their harvest window first (their main supply backpack, so hand their materials there), then their
  toolbelt, bag and pack. Never a stack with mods installed.
- **Cost, time, quality:** what a player pays with no crafting perks: the recipe's ingredients and time, scaled by the server's crafting
  settings. The quality is what their crafting skills give for that recipe, as yours are (they raise them by crafting at their role's
  stations, *Learning a craft*, and with skill magazines, *Hired survivors: reading books*, below); 6 for everyone when crafting progression is off.
- **Where and when:** they kneel where you gave the order (their post) and make the items one at a time. An enemy they can see within 15 m
  pauses the item they are on (one behind a wall does not, as for base repair); after the fight they walk back to their post and carry on. They take an item's ingredients when they start it, so no one else can use them up while
  they work; stopping them mid-item gives them back to their harvest window (their pack when the window is full).
- **Output and XP:** the items go to their harvest window; they earn the crafting XP a player earns (less for each repeat of the same recipe).
- **Ending it:** the count is done ("I'm done crafting."), an ingredient runs out ("I need ... for that."), their window is full ("My pack is
  full.") or they can no longer make it ("I can't make that."). Any other order ends it at once. Then they follow you again; from more
  than 40 m away they appear beside you (`Settings.xml` `<Loot><ReturnTeleportMetres>`). An order they cannot take ("I can't make
  that.") changes nothing: they keep their spot and their job.
- Characters that cannot do jobs (Harley) do not show the row.

### At a workstation

Stand within **5 m** of a placed workbench, forge, campfire, chemistry station or cement mixer (`Settings.xml`
`<Crafting><StationReachMetres>`) and open **Craft...**: its recipes show on the station's own tab of the hire's craft window.
They use the station the way you do. Every survivor crafts from their backpack anywhere; a workstation needs their role for it and a
land claim (a campfire works anywhere: it is a field station). When one in reach does not count (no claim, a trader's area, or
someone using it) and they have no other of that kind, they say so once as the window opens ("I can only use the Forge inside your land claim.").

- **Roles:** they use a workstation only with its role, toggled on them from the order wheel (**Orders > Roles**) or by voice ("be the
  cook", "stop being the blacksmith"): **Cook** the campfire, **Blacksmith** the forge, **Chemist** the chemistry station,
  **Crafter** the workbench and cement mixer (`<Crafting><CookStations>` and the like: add a modded station's craft area to a list).
  A survivor may hold several roles, and several survivors the same one. Asked for a station recipe they have no role for: "I don't
  know how to do that." While they are hired their roles show after their name ("Xyth - Cook, Smith") on the Press E prompt, the talk
  window, captions, the order wheel and SCore's companions list; **Hold E > Progress** lists them with their level, their XP toward
  the next level and the quality they craft at.

- **The station:** the nearest one in reach that makes the recipe and holds its tool (the cooking pot, the crucible...), of a kind
  their roles allow. Inside your or an ally's land claim, never another player's claim or a trader's area; a
  campfire also in the wild, as a field station (`<Crafting><FieldStations>`). Not one a player has open, not a fire beside water. At the forge they make what the forge shows you: with the server's smelter on, the
  recipes paid in its smelted units.
- **Where they work:** a spot beside the station, in front of it first (the side it faces), then its sides, then its back; the
  nearest one they can stand on and see the station from (a wall or another block between them rules a spot out, a person does not).
  They walk there and face it. No such spot: they work from where you gave the order if the station can be seen from there,
  otherwise "I can't make that." and nothing is taken. If they stop getting closer on the way, they work from where they got to.
  A modded workstation can name the side they work from with a block property, `<property name="NPCWorkSide" value="Back" />`
  (`Front`, the default, is the side that faced you as you placed it; `Back`, `Left`, `Right` as you see it from the front; `Any` for the
  nearest side).
- **Their home station:** the station they last crafted at is their home, remembered with them through a save, until they craft at a
  different one (a station picked up or destroyed stops being their home). Order a station recipe while you are not near a station
  of that kind and their home station is used if it makes the recipe and they are within 60 m of it (`<Crafting><HomeStationReachMetres>`):
  they walk there, with the ingredients from their stores and the chests near it.
- **When they are done:** they stay at the station until you give them an order (**Follow me**), walking around it within 2 m
  (`<Crafting><StationWanderMetres>`; 0 = they stand at their spot beside it, facing it). A hire you tell to stay within that distance
  of any workstation walks around it the same way. A backpack order always brings them back.
- **The order:** they take every ingredient when you give it (smelted units from the forge, the rest from their stores and your
  chests, as above) and hold them; the station waits for them. Once they are at their spot, facing it, the whole count goes into the
  station's queue as theirs, as your craft does, and they light a forge or campfire (at a campfire, a few seconds into striking their
  match); the station makes them at its own pace. Every queue full: "That station is busy." and nothing is taken.
- **The Blacksmith smelts:** a forge recipe whose units the forge does not hold yet starts with their ore: they put raw ore in (scrap
  iron, brass and lead, crushed sand, clay lumps, small rocks: `<Crafting><SmeltOre>`) from their stores and your chests, keep the
  forge burning, and queue their crafts once it has melted enough. They also keep every material they have ore for at **1000** units
  (`<Crafting><ForgeStockUnits>`, `0` = only what an order needs): when they start at the forge and again when their crafts are done.
  **Craft...** counts the ore in reach, so forge recipes show even at an empty forge.
- **Fuel:** a forge or campfire that would go out before their items are done gets fuel from their stores and your chests in reach (wood
  first, then coal: `<Crafting><StationFuel>`), and they light it. No fuel anywhere: "I need fuel for that." and they wait; put fuel
  in and they carry on.
- **Their items:** as they are made they take them out of the station into their harvest window; when that is full, into the nearest
  unlocked chest you or an ally placed near them. Only their own items: yours in the same station stay where they are. When nothing can
  take them: "My pack is full." once; they carry on and the station keeps them.
- **XP:** the crafting XP a player earns for a station craft.
- **While a player has the station open** they wait and touch nothing.
- **Ending it:** everything made and moved: "I'm done crafting." Any other order, or a new craft order, takes their items not yet made
  out of the queue and gives their ingredients back (units into the forge, the rest into their harvest window), and their ore the forge has not melted yet. Ingredients they
  took but had not queued yet (still on their way) go back to them the same way, also when the game saves. If they die or the game
  is reloaded, what they queued still finishes in the station: take it from the station's output.

### Learning a craft

A specialist gets better by doing the work. Every item a workstation makes for them counts in each role they hold that uses that
station (`<Crafting><CookStations>` and the like). Their skill in the role is those crafts divided by **10**
(`<CraftSkill><CraftsPerLevel>`), and each of the role's crafting skills (the game's own, the ones you raise with magazines) is raised
to at least that level. The game then unlocks recipes and raises their craft quality from them exactly as it does for you.

| Role | Skills it raises (`<CraftSkill>`) |
|---|---|
| Cook | Food, Medical |
| Blacksmith | Workstations, Explosives |
| Chemist | Medical, Vehicles, Explosives, Electrician, Food |
| Crafter | the 21 skills of the workbench's recipes (armor, weapons, tools, electrician, traps, vehicles, workstations...) |

- The defaults are the skills of the recipes each role's own stations make; edit a role's list to change what it learns.
- A new Cook makes corn on the cob and baked potato after 20 crafts at a campfire; later recipes take more (the game's own levels).
- Only workstation crafts count, not backpack crafts. A skill never drops, and they keep it if they give up the role.
- **Hold E > Progress** shows each role they hold: "Cook skill 2: 20 / 30 crafts" (their skill, their crafts, the crafts the next level
  takes). The skill shown is their real one: the highest of the role's crafting skills, so a skill magazine they read shows there too.
- Their level still earns the game's crafting XP, as before.

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

- **Tools:** list its tool slots in `<Crafting><StationTools>` (`ammoBench=toolA,toolB`) and they put a tool they carry into an empty
  slot. Not listed: they never put one in, but use one already there.
- **Animations:** give it a clip set in `<Crafting><AnimStations>` (`ammoBench=bench`, or `use`, or `fire`). Not listed: they play the
  rummage pose (one log line).
- **Fuel:** a station with a fuel slot is lit and fed like the forge, from `<Crafting><StationFuel>` (wood, coal). A station that burns
  anything else needs that item added there. A station with no fuel slot is never lit.
- **Smelting:** their ore loading follows the station's material input slot, not the name "forge" (not yet tested with a modded forge).
- **Recipe unlocks:** they craft a recipe only when it is unlocked for them, through the crafting skills their roles raise
  (`<CraftSkill>`). A modded recipe on the game's crafting skills works as it is. A recipe unlocked by a modded crafting skill: add that
  skill's name to a role's list; any crafting skill the game has loaded is accepted, and a name that is not one is skipped with a
  warning in the log.
- **Power:** they cannot see electricity. A modded station that will not craft without power crafts only while it has power, and they
  do not wait for it.

## Hired survivors: reading books

A hire reads the same books you do: a **schematic** they have not learned, or a **crafting skill magazine**. Give them the book (their
harvest window or their pack), then:

- **Hold E > Read** (for your own hire who does jobs) lists every book they carry, one row each. A book that
  would teach them nothing shows "(nothing new)". Pick one and they read it. With no book: "I don't have anything to read."
  With two or more books that would teach them something, the list starts with **Read all (N)**: picking it reads them
  all, the same as saying "read your books".
- Or say **"read your books"** ("read the book", "read the books"): they read every book they carry that teaches them something,
  then say one line.
- **A schematic** unlocks its recipe for them: it shows in their craft window at once. **A magazine** raises their crafting skill the way
  it raises yours (the server's points per magazine), so their craft quality and their unlocked recipes move with it; their progress page
  shows it.
- They say what they learned. Reading uses up one copy. A book that would teach them nothing is not used up, and neither is a perk
  book (the series books): they do not read those yet and say so. They will not read in a fight.
- They never read on their own: a book you only store with them stays a book.
- The book's XP does not count toward their level while `<Progression><KillXpWorkaround>` is on (the default).

## Hired survivors: putting the weapon away

Tell a hire **"put your weapon away"** (or "use bare hands", "empty hands", or **Weapons > Empty hands** on the order wheel) and
they stay bare-handed, also through a save and reload: they come back with empty hands instead of drawing the weapon they held last.
Bare hands are a weapon: in a fight they punch. Ordering them to a weapon ends it, and so does dismissing them.

## Hired survivors: the gun's flashlight

Their gun's flashlight comes on by itself in the dark, stays on in a fight, goes out when you crouch (they sneak with you) (Settings.xml `<Light>`).

## Hired survivors: the torch

When their gun has no light of its own, a hire holds a light tool (a torch or a flashlight) from their bag in the dark: at night, and by day in a room the sky does not reach (a lamp does not count). A fight puts it away at once (their weapon back), and it stays away for 10 s after the fight (`<Light><TorchFightCooldownSeconds>`). By day it goes away after a moment (`<Light><HoldSeconds>`), your crouch puts it away (as the gun light), and a weapon order at night stops it for the rest of the night. The tool comes from their window or bag only, never the spawn kit; a hire whose gun carries a flashlight mod keeps their gun. New survivors start with one torch in their window.

## Hired survivors: the base keeper in an attack

A base keeper (**Jobs > Base Repair** or **Combat Repair**, or "repair the base") reacts to an attack on the base by their duty. The
base is **under attack** while a block of it that they track has been hit within the last **10 s**
(`<BaseKeeper><UnderAttackSeconds>`); that one window serves both duties.

- **Base repair** (most damaged block first): while the base is under attack they stop repairing and put the repair tool away.
  They fight what they can reach or shoot, and go back to their repairs once the base has been quiet for 10 s.
- **Combat repair** (a freshly hit block first): they keep repairing the wall through the attack, the blocks hit within the window
  first, and fight only what hits them.
- **Nothing they can repair** (no damaged block within their repair skill, or no materials for one): they do not take the repair
  tool out at all and keep their weapon. They check this again every 2 s (`<BaseKeeper><BaseKeepRecheckSeconds>`).
- **No back and forth:** once they have turned to their repairs or to a fight, they stay with it for at least 3 s
  (`<BaseKeeper><JobFightDwellSeconds>`), unless an enemy that just hit them is in their reach or they have a gun in hand.

## Hired survivors: downed and revive

A killing blow on a survivor who may go down does not kill them: they go down (the game's own
knockdown, held) and bleed out over a timer instead. A **gib** never downs them (it is a real
death, as is a **second down** within `<Downed><RedownWindowSeconds>` of their last revive).
Who may go down: **a hire with a player leader**, or **any NPC whose class carries the
`reviveable` tag** (a wild survivor without either dies as usual). While they are down they say one
pained line ("Argh... I'm down! Help me!"); they do no chat and take no orders until they are up.

- **The bleed-out:** their health holds at `<Downed><FloorHealth>` and the seconds left count down
  (`<Downed><SurviveSeconds>`). For a hire the countdown pauses while their leader is away (a dead
  or gone leader pauses the bleed); an NPC with the tag always counts down. At zero they die for
  real (the bag drops, the hire ends, normally).
- **The marker:** while they are down, every player sees a red medical marker above them (on screen
  and on the compass) with the seconds left counting down; it is gone when they are revived or
  bleed out. A client that joins late sees them lying after a moment
  (`<Downed><LateShowSeconds>`).
- **The leader messages:** their leader gets a HUD message when they go down and again at each
  `<Downed><WarnSeconds>` seconds-left mark (30 and 10 s by default).
- **How to revive them:**
  - **Use a revive item on them:** one of `<Downed><ReviveItems>` (the first-aid bandage and kit
    by default) used on them while they are down revives them. The item is consumed exactly like a
    normal use; its heal is not applied to them. Any player within `<Downed><ReviveRangeMetres>`
    of them may do it.
  - **E on them:** while they are down, E on them offers ONLY Revive - the prompt reads
    "[E] Revive <name>" with the seconds left counting down, and the press consumes one carried
    revive item from your bag (with none carried it tells you which item you need and uses
    nothing). Any player within the revive range may do it.
  - **A hire:** any hire carrying a revive item in their OWN pack revives a downed squad mate who
    shares their leader (`<Downed><AnyHireRevives>`, true; false = only a hire with the medic role),
    and a downed squad mate overrides their defend-first rule, so they go to the patient mid-fight.
- **While they are up again:** they are invulnerable for `<Downed><GraceSeconds>` (a second hit in
  the grace does not re-down them), and a second down within `<Downed><RedownWindowSeconds>` of
  the revive is a real death. The revive is remembered in their leader's record, naming who got them
  back up (the memory, so it needs the memory / RAG option on).
- **What they will not do while down:** nothing else starts - no animation clip, no self-heal (their
  health holds at the floor), no dodge, no kneel, no loot pose, no torch, no vehicle auto-board,
  no pickup, no team teleport, no chat or orders.
- **Save / quit while down** loads them standing: the boot clears the downed flag and the
  post-revive grace on load (the redown window, if one is running, survives the save).
- Settings (Settings.xml `<Downed>`, base mod): `<Enabled>` (true), `<FloorHealth>` (10),
  `<SurviveSeconds>` (60), `<GraceSeconds>` (60), `<RedownWindowSeconds>` (600),
  `<HoldSeconds>` (9998), `<LateShowSeconds>` (1.0), `<ReviveItems>`
  (medicalFirstAidBandage, medicalFirstAidKit), `<ReviveRangeMetres>` (4), `<WarnSeconds>`
  (30,10).

## Hired survivors: relationships

A hire keeps a **relationship** with you: two numbers, **trust** and **affinity**, each on a
-100 to +100 scale (0 is neutral). Each has a **global** value on you and a **per-hire** modifier on them;
what matters is the **effective** value (global + their modifier, clamped). The relationship is deterministic:
no AI, just the events below moving the numbers. It is **off** when modconfig `<Relationship><Enabled>`
is false or their entity class carries the `<OptOutTag>` tag (default `norelationship`), in which case
nothing here happens.

- **What moves trust:**
  - A **paid week**: `<PaidTrust>` (+2).
  - **They go down**: `<DownedTrust>` (-4).
  - **You revive them**: `<RevivedTrust>` (+3) trust and `<RevivedAffinity>` (+3) affinity.
  - **You kill an enemy that was targeting them**: `<AttackerKilledTrust>` (+1).
  - **One of your hires dies**: `<HireDiedGlobalTrust>` (-5) to your **global** trust (it ripples, it is not one hire).
- **What moves affinity:** you reviving them (+3) and a **blood moon survived together** (`<BloodMoonAffinity>`, +4).
- **The price:** their weekly pay is multiplied by their effective trust: at +100 they cost `<PriceMinMultiplier>` (0.85)
  times, at 0 they cost full, at -100 they cost `<PriceMaxMultiplier>` (1.50) times. A billing failure changes nothing.
- **Recovery is slower than damage:** a positive move while the effective value is below 0 is multiplied by
  `<RecoveryRate>` (0.5).
- **A scar:** when their effective trust reaches `<ScarThreshold>` (-50) or below, they are scarred, and their trust can
  never again raise their effective trust above `<ScarCeiling>` (+50). A new leader starts clean.
- **Gifts:** hand them something they can use and it goes into their private bag, within their caps, and moves their
  affinity by the gift's value. They take the **medical** items on `<GiftItems>` (First Aid Bandages and First Aid Kits up to 2 stacks each, Painkillers 1 stack; plain bandages are not a gift), **drinks** they can douse a fire with (up to
  `<Loot><BagMaxDrinks>`), **ammo for a gun they carry**, and the **hire currency**. A gift's points are `ceil(log2(1 + value / <GiftValueUnit>))`, capped at `<GiftDailyCap>` (6)
  per game day per hire. The gift window works in single player, on a host and from a client on a dedicated server; the
  held-item Gift (ring or voice) is single player and host only. If a week is due when you hand them
  coins, the coins **pay the week first**; only the rest is a gift (too few for the week: nothing is taken, they tell you the cost).
- **How to give:** hold the item, then use **Gift** on their hold-E ring, or say **"a gift for you"**. The ring's Gift entry
  shows only while what you hold is a gift for them. Something they cannot use is refused and stays in your hand.
  Or tap **E** on them, pick **Pay / Gift...** and then **Give a gift...**: a window lists every gift you carry (backpack or
  toolbelt) with how many you have. Pick one to see what they hold against their cap ("Held: 2 / 4"; coins have no limit), set
  the amount with minus / plus / max (max = what fits in their bag) and press **Give** (their daily gift cap still applies to the
  points). Coins in the window never pay the week: while a week is due the coin row says to pay their week first (the Pay row).
  On a dedicated server your client takes the items from your pack and the server puts what fits in their bag; anything that
  does not fit lands at your feet. Asking them to **give** you back a bandage, ammo or coins never takes them from their private bag.
- **Where you see it:** their **Progress** page shows a line, "Trust: <word>  Feels: <word>" (trust is *wary / steady /
  loyal*, affinity is *cold / neutral / warm*, turning at -25 / +25), and `vc standing` prints the full readout
  (global, theirs, effective, the words, the scar, the price multiplier, gifts today).

## Hired survivors: emotes

A hire can wave, salute, dance and more: a short animation they play where they stand.

- **Hold E > Emotes** (or **Press E > Jobs and gear > Emotes**): `Wave`, `Salute`, `Fist bump`, `Stretch`, `Warm up`, `Whatever`, then **Dances**
  (ten dances, seven a page plus More) and **More** (`Flip off`, `Insult`, `Loser`, `Threaten`). Pick one and they play it.
- **By voice**, the whole phrase: "wave at me", "salute me", "bump fists", "do a stretch", "do a warm up", "dance for me" or "do a
  dance" (a random dance), "flip me off", "do an insult", "do the loser", "do a threat". A sentence that only mentions dancing ("do
  you like to dance?") does not start one. Given to the squad, each hire picks their own dance.
- Only your own hire takes the order. They never start one in a fight: they tell you so instead.
- **Their weapon** is put away for the emote and drawn again after; it never leaves their inventory. A weapon they had already put
  away stays away.
- **How long:** a gesture plays once; a dance loops for up to 15 s.
- **What ends it:** any order to them (opening their E menu counts), an attack or a threat, or their starting to walk.
- **Reactions** (`<Emotes><Reactions>` in the settings table): they salute right after you hire them (a hire who is yours when the
  world loads never does; one you hire again after dismissing them does). Half the time they fist-bump when a fight they were in is
  over and no enemy is near, at most once a minute. Half the time they wave with their greeting when it plays toward you, at most
  once every two minutes (single player and a listen host for now: not on a dedicated server). A reaction is skipped, never saved
  for later, while they fight, walk, play another animation, work a job, sit in a vehicle or have an enemy near.
  While they speak a chat reply they talk with their hands, a new gesture as each sentence starts.
- **Idle moments** (`<Emotes><Idle>`): a hire who stands still and free (on Stay or Guard, or following you while you stand still)
  now and then stretches, warms up, shrugs, looks around, yawns or looks bored: after 20 s still, then about once a minute, each hire on their own clock. Never in a
  fight, with an enemy near, while they or the leader they follow walks, while they work a job or sit in a vehicle, or while you
  are talking with them.
- Other players on a server see it.
- **The clip list** is the base mod's `Config/SdcsAnimClips.xml`: one row per clip (`id`, the asset `bundle` in the base mod's
  `Resources` folder, the clip's `name` in it, `profile`, `loop`, `maxSeconds`). An id that starts with `dance_` is a dance: a
  random dance picks among them and the wheel lists it on its Dances page, under its clip name.
- **Console:** `vc anim play <clip id> [seconds] [all]` plays a clip on the survivor in front of you (`all`: every survivor within 10 m); `vc anim idles [all] [gap]` plays each idle clip in turn with a pause between (default 2 s) and `vc anim idles stop` ends it; `vc anim off` stops a clip.

## Survivors' aim

A survivor shoots like a player who aims down the sights, made less steady by their inexperience: their spread with a gun is a
player's aimed spread with that gun (no perks) times a factor that falls in a straight line from level 1 to 1 at level 10
(`<AimErrorAtMaxLevel>`). Survivors, hired or not, start at 2 (`<Progression><AimErrorAtLevel1>`): they had practice before anyone
hired them. Bandits start at 3 (`<BanditAimErrorAtLevel1>`): little practice and scarce ammo. A level-1 bandit's pipe pistol
spreads about as wide as a player firing it from the hip; a level-10 survivor or bandit shoots like a player aiming. Moving,
running and crouching change it the way they change a player's, and their shots spread more as they keep firing, as yours do. A
bandit's tier can add more spread on top (`<Bandits><SpreadByTier>`).
They aim at the chest of a person (you, a survivor, a bandit, a trader) and at the head of a zombie or an animal; bows and
launchers aim at the head, and their arc drops the shot onto the body.

**Defending you.** On the Defensive stance a hire goes for an enemy that is coming for you, before it hurts you. One with a
melee weapon goes for it within 10 m of you even when your body hides it from them; one with a gun whose line of fire you
stand in steps 2 m aside, away from you (`<Combat><ClearShotStepMetres>`), then shoots. A hire with a gun
also fights any enemy they can see within 20 m of themselves, whatever it is after; cover is fine, but they engage. Through a gap
(a broken door, a window) they aim at the part of the target they can actually hit, and hold fire when they can hit none.
An enemy behind a closed door, window or bars is not theirs on Defensive: they wait until the way opens. On Aggressive they open
the door, or break through it with an axe or hammer and shoot through the gap (`<Combat><BreakThroughDoors>`).

**Kneeling to shoot.** A survivor or bandit with a gun who stands still to shoot at a target at least 15 m away sometimes kneels
for it (half their fights, `<Combat><KneelChance>`), but never while another enemy is within 10 m of them. Kneeling uses their gun's
own crouch accuracy, as it does for you, and their aim steadies as they stay down. They stand up when they move, when the target
closes in or changes, or when an enemy comes near. When a friend (a survivor, a bandit mate, you) stands in their line of fire,
the one in front kneels for about 3 seconds if the shot then passes over them (`<Combat><KneelForAllySeconds>`); if it would not,
or the friend is moving, the shooter steps aside instead. Your crouch or stand order always wins.

**Looking for cover.** A survivor or bandit whose search for cover found nothing does not search again from the same place: they
wait until they, their enemy or the area they keep to has moved 2 m (`<Cover><FruitlessRescanMoveMetres>`), they face another enemy,
the kind of cover they are after or their order changes, or 9 s have passed (`<Cover><FruitlessRescanSeconds>`), and meanwhile do what they do when there is no cover.
In a big fight this saves most of the repeated searches; the cost is that a spot that frees up while nothing moves is found up to
9 s later.

**Starting guns and ammo.** A tier 1 survivor starts with a melee weapon in their hand and an empty pipe pistol in their pack:
they get no ammo for it when they appear, not even ammo from their own trade goods, which stay for sale. Ammo they loot later,
or that you give them, is theirs, and then they use the pistol. Tier 2 and 3 survivors start with ammo for the gun they roll.

## Hired survivors: stealth kills

**On their own (Hunting).** A hire on the Hunting stance who holds a melee weapon picks an enemy within the hunting leash that has not
noticed the party (or, alert from an earlier fight, cannot see them now: behind it, in the dark, behind cover) - a zombie anywhere, a bandit outside an untouched bandit room, never a sleeping bandit and never anything already
fighting you - crouch-walks up to it and strikes crouched. Their hit takes the same sneak-attack bonus as yours: blades carry it
themselves, the other melee weapons through Hidden Strike, which a hire learns like any perk. It works while you crouch too: an enemy
that cannot see you is still one they go after. They give up without reaching it after `<Stealth><GiveUpSeconds>` (30 s) plus
`<Stealth><GiveUpPerMetreSeconds>` (1 s) for every metre the target was beyond the 20 m free allowance at the start of the approach:
a far stalk gets time to arrive.

**Spotted on the way in.** If the enemy sees them and turns on the party before they strike, they act on their personality: a timid hire backs off to
you, a veteran, hunter or soldier charges and finishes the attack, anyone else fights by their stance. The choice is the optional
`<Spotted>` element of their personality in `personalities.xml` (`backoff`, `fight` or `charge`); a personality without it uses
Settings.xml `<Stealth><SpottedDefault>` (`fight`).

**The kill order: Go there on a target.** Aim at a hostile (a zombie, a bandit, a hostile animal) or a wild animal (a deer, a wolf) and open
the wheel: its **Go there** button reads **Stealth kill** with a skull. Picking it is a kill order: they stalk the target quietly until it notices
them, then fight it. Aimed at a block, the ground or a vehicle, Go there is a plain move; aimed at a friend (you, a hire, a trader, a survivor),
they walk to them. A click that slips off the target still takes it: the last target your crosshair crossed stays locked for
`<FormationRadial><TargetLockSeconds>` (3 s). Spotted on the way in, a hire with a bow or crossbow gets their shot in first; a melee hire reacts
by their personality, except that on Hold fire they always back off and on Hunting or Aggressive they never do.

**Who goes.** The combat wheel opens on **All** (the centre reads All): a Stealth kill on All goes to your best hire for it - the nearest one
with a loaded bow or crossbow, otherwise the nearest one with a melee weapon. To send one hire, aim at them as you open the wheel (the crosshair
on them, or within `<FormationRadial><LookPickDegrees>` 8 degrees of them), open it from their own ring, or pick them on the wheel; that choice lasts
until the wheel closes, and the next open is All again. While an enemy is under your crosshair, marked or locked, the wheel always opens on All.

**Mark a target.** Press the middle mouse button (`<FormationRadial><MarkKey>`) with your crosshair on an enemy: a red skull appears over it
- the enemy your crosshair holds if it is on one, else the enemy the key's own ray hits within `<FormationRadial><MarkReachMetres>` (50 m; the ray is the game's
gun ray, so grass and plants do not shorten it) - and Go there and Stealth kill take it as the target wherever you aim, until it dies or is more than
`<FormationRadial><MarkRangeMetres>` (60 m) from you. Press the key on empty
space to clear the mark (do that before sending a plain move). Marking a new enemy replaces the old mark. A Hunting or Aggressive hire also
fights a marked enemy from range, whatever it targets - the mark is their hunt's target, and the marked enemy is their preferred target in melee, ahead
of a nearer one (self-defence still wins). While you aim down sights the key
stays the game's scope zoom; in third person the game also flips your camera shoulder.

**A sneak hit hits as hard as yours.** A hire's first strike, crouched, on an enemy that has not noticed them deals what your hit with the same
weapon would: the world's Melee (or Ranged) Damage setting applies to it as it does to yours (`<Stealth><SneakHitParity>`).

**The quiet-kill order.** Aim at an enemy and pick **Go there** (order wheel or combat wheel), or say "take him out quietly" or
"quiet kill": your hire stalks it and kills it quietly in any stance, within `<Stealth><OrderRangeMetres>` (60 m) of you. An enemy
that has already noticed you gets a normal attack instead. Given to the squad, the nearest hire with a melee weapon goes. A hire
holding a lit torch never stalks on their own and is never picked for the squad order; ordered by name, they swap to their melee weapon
first. A hire holding a gun swaps to their melee weapon too (with none, they go with bare hands). You and their squad mates standing in their
line of sight do not stop them, and if they lose sight of the target for a moment they keep going for `<Stealth><SightGraceSeconds>` (10 s), counted from the last time they saw or heard it.

**A bow or crossbow: the quiet shot.** A hire carrying a bow or crossbow with arrows or bolts takes it for a quiet-kill order (a gun in
their hand is swapped for it). They crouch-walk in and stop just outside the distance at which the target would notice them - worked out
from how lit they are - plus `<Stealth><ShotMarginMetres>` (3 m), kept between `<Stealth><ShotMinMetres>` (6 m) and `<ShotMaxMetres>`
(20 m). With a clear line they fire one full-draw shot, still crouched, and stay crouched for `<Stealth><ShotHoldSeconds>` (2 s) so the
arrow lands with the sneak-attack bonus. A Hunting bow hire on their own keeps shooting normally; the quiet shot is for the order.

**At bow range.** They fire at the first clear line within `<Stealth><ShotMaxMetres>` (20 m), even before they reach their shooting spot, and never walk inside it. A wall or fence in their line at their shooting spot makes them step up to `<Stealth><ShotSideStepMetres>` (3 m) to the side for a clear line; with none they hold. While the target still has not noticed them they shoot again (a miss does not give them away); once it notices, their stance takes over.

**Waiting while it looks their way.** On the way in, if the target faces them and would notice them at that distance and light, they stop
where they are, and move on once it looks away. Their light is worked out the way yours is, and the stealth perk From the Shadows (which a
hire learns as they level, like their weapon perks), the Silent Night book and the rogue and assassin outfits lower it for them as they do
for you.

**The log (verbose):** `[SDCS-STEALTH] ... stalk` when they start, `strike ... crouched=True notAlerted=1` at the hit, and while an
order waits, once a second, `ordered <target> waiting: <reason>` naming what holds them back.

## Bandits

Bandits are survivors gone hostile, and they spawn in the biomes with the survivors' tier progression: tier 1 bandits where
tier 1 survivors live, tier 2 and 3 bandits where those tiers do (outside towns, day and night). A tier 1 bandit
(`npcSdcsBanditMelee`, a tier 1 melee weapon) comes alone; a tier 2 or 3 bandit (`npcSdcsBanditMaleT2`, `...FemaleT2`,
`...MaleT3`, `...FemaleT3`: that tier's survivor, with their looks, armour and weapons) comes with 1 or 2 mates of their tier. A
bandit roll comes a quarter as often as a survivor roll at tier 1, half as often at tier 2 and as often at tier 3. They spawn
like zombies: only with enemies on, never in a blood moon, under the enemy cap and at least 28 m from you. `npcSdcsBanditRanged`
(a tier 1 female with a pipe pistol or a pipe rifle, 60 rounds and 2 pipe bombs) never spawns in the open world.
In the console, `sdcsbandit group 2` (or `3`) spawns a group of that tier beside you.

**In buildings:** bandits also sleep in points of interest, in the bandit rooms the legacy Harley bandits used to fill (Harley no
longer spawns anywhere). The room's gamestage picks the tier: up to 50 tier 1 (the melee bandit, and the pipe-gun bandit in the
ranged rooms), 100-200 tier 2, 400 and up tier 3; boss rooms are always tier 3. A sleeping bandit wakes when you walk into their
room, and before that they hear and see like a sleeping zombie, in any room: a loud noise near them (gunfire) or you within 6 m in
front of them in enough light wakes them, and so do your survivors if they count them as enemies. Crouch in the dark and stay quiet
to sneak past. In an attack room they are already awake and waiting. A bandit in a building comes without mates. If you make your
own NPC pack, add your NPCs to these groups by appending to the group by name.

**Awake, they see what is in front of them:** survivors and bandits see a 220 degree cone (front and the sides, not behind). Sneak up
behind one and they do not see you until they turn, hear you, or you hurt them; whoever hurts them is seen whichever way they face for
a few seconds (`<Combat><SightAttackerExemptSeconds>`), so they turn and fight back. The width is the class's `MaxViewAngle` in
entityclasses.xml (360 = all round). Your hires see the same way: a zombie coming up behind a hire is noticed when they turn, hear
it, or it hits them.

**They hear footsteps too:** an awake survivor or bandit hears an enemy's footsteps (zombies, bandits, enemy survivors) and, for a
few seconds, sees that enemy whichever way they face, so they turn and fight if nothing blocks the view. A walk is heard about 2 to 5 m
away depending on the ground, a run 5 to 11 m, a crouching survivor only within 1 to 2 m, so your hires can still sneak up on a bandit
if they crouch (`<Combat><HearingLevel>`, `<Combat><HearingSeenSeconds>`). A zombie's groans and its alert or attack cries are heard too (a crawler makes no footstep noise, but its voice carries; `<Combat><HearingVoiceQuietNoise>`, `<Combat><HearingVoiceLoudNoise>`). Bandits hear you, the player, the way they always have.

**Which tier:** a POI's bandits are picked by the party's gamestage, and the POI's difficulty sets a floor
(`<Bandits><PoiTierFloors>`): difficulty 1-2 gives at least tier 1 bandits, 3-4 tier 2, 5 tier 3.

**Sentries:** a POI builder can name `GroupNPCBanditsSentry` (or `GroupNPCBanditsSentryRanged`) in a sleeper volume instead. A
sentry spawns awake on their spot, facing the way it faces, and holds it. Every sentry rolls from their tier's guns and spawns with 120
rounds for it; the tier 1 bandit melee sentry (`npcSdcsBanditSentryMelee`) is the one melee sentry. In a fight a sentry with a gun
steps at most about a block (`<Bandits><SentryLeashMetres>`) for a shot or cover; a sentry holding a melee weapon (or one whose gun
ran dry, with their club out) steps out to fight an enemy within `<Bandits><SentryMeleeLeashMetres>` (6 m) of their spot and walks back
when the fight is over; if their way to the enemy drops off a ledge they leave that enemy for a while (`<Combat><MeleeUnreachableForgetSeconds>`)
and walk back instead. Either way they never leave their spot's level and never jump down, and if something knocks them off they
walk back.

**Placing a sentry:** put their sleeper spot at least 1.5 blocks back from any drop, or give their perch a low wall or lip. Their leash only
limits their OWN moves: an enemy that climbs up and presses against them in a melee fight can push their body off the edge (the game's own
body push between two NPCs). When they are shoved while standing still they step back to where they stood (`<Bandits><SentryShoveReturnMetres>`), and if they fall they walk back,
but a perch they cannot be pushed off is the reliable answer.

A bandit spawned by a guard spawn block (Stay or Guard) keeps their spot the same way; out of a fight a Guard block's bandit
walks a short patrol around it on their own floor (`<Guard><PatrolMetres>`), while a Stay block's bandit and a sentry stand still.

**Friendly sentries:** `S_-Group_NPC_Survivors_Sentry` in a sleeper volume spawns friendly survivors who hold their spot the same
way (the classes `npcSdcsSurvivorSentryMaleT1` .. `npcSdcsSurvivorSentryFemaleT3`, one per tier and sex, by gamestage like the POI
residents). What makes a sentry is the class property `<property name="SdcsSentry" value="true"/>`: give it to your own survivor
or bandit class and any sleeper volume spawns them as a sentry. They fight their faction's enemies, and never the player unless their
faction is hostile to the player: an unhired friendly sentry fights any enemy within `<Bandits><SentryEngageMetres>` (20 m) of their spot, not
only one that comes for them (from their spot, on their leash). Hiring a friendly sentry ends their post for good: they follow you like any hire, and dismissing them later
does not post them again. A sentry is never made a POI resident.

- **Throwables:** none at tier 1, one molotov at tier 2, three at tier 3; they throw only at a group (see *Throwables*).
- **They fight** you, your survivors, zombies and hostile animals on sight; passive animals are left alone. A bandit runs at a
  target they see but cannot reach with their gun yet, and walks toward a player they hear but cannot see (crouch to stay quiet).
- **They don't talk:** no E prompt, no hire, no trade, no voice chat or voice orders.
- **One cold line:** the moment a bandit locks onto you or one of your survivors to fight, they say one calm line ("Time to
  die.", "Nothing personal.") in a random voice of their sex, then not again for a minute, and only the first bandit of a group
  speaks. A zombie or an animal gets no line. Text to speech only: with TTS off (the Core alone) they stay silent, caption
  included. The lines are their personality's `<CombatLines>` in `personalities.xml` (`bandit_cold_male`, `bandit_cold_female`,
  named by each bandit class's `personality` property); the timing is `<Bandits><LineCooldownSeconds>`,
  `<LineGroupQuietSeconds>` and `<LineGroupMetres>`.
- **Loot:** dead, they always drop a zombie loot bag for their tier: plain at tier 1, blue at tier 2, red at tier 3. Per class in
  `entityclasses.xml`: `LootDropEntityClass` and `LootDropProb` (the same properties as on any zombie; the blue and red bags
  are `EntityLootContainerStrong` and `...Boss`). Half the time the bag also holds their weapon: one the game crafts from parts
  (most tier 2 and 3 guns, the steel melee weapons) as 1 or 2 of those parts, with a 1 in 10 chance of the whole weapon too; one
  with no parts (pipe guns, simple melee) whole, at quality 1-2 at tier 1, 2-3 at tier 2, 3-4 at tier 3, empty, no mods. It goes in
  when the bag is first opened; a bag left unopened over a save and reload keeps only the game's loot.
- **Damage to players:** `<Bandits><PlayerDamage>` in Settings.xml (half by default). In the console, `sdcsbandit` shows it and lists
  the bandits in the world; `sdcsbandit damage 0.75` changes it at once, until the game restarts.
- **Aim:** a bandit shoots like a survivor of their level (see *Survivors' aim*). `<Bandits><SpreadByTier>` can widen it by tier
  (0 by default).
- **One target at a time:** they keep fighting the target they have until it is dead or out of their sight (the class property
  `SdcsKeepTarget`, 0.7: any other target scores that much of theirs).
- **They hunt:** if they lose sight of the enemy they are fighting, or of whoever just hurt them, within 40 m, they go after it
  until they see it again (instead of wandering off or standing still): they walk to where they last saw or heard you and
  search there. A melee bandit with no cover to bound to charges at once. They open a closed,
  unlocked door on their way; they cannot break blocks, so their path goes round walls, fences, shanty walls and railings.
- **Under fire:** they do not stop to bandage while someone with a gun has them in sight within `<Combat><HealGunThreatMetres>`
  (25 m); they keep fighting. The same holds for your survivors.
- **XP:** 600 at tier 1, 1000 at tier 2, 1400 at tier 3. Your own custom characters never spawn as bandits.

## Your own NPC or bandit boss

Every piece a survivor or bandit spawns with is a property of their class in `entityclasses.xml`, and a class that `extends`
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
| `SdcsWeapons` | The pool their weapon is rolled from (comma separated item names). |
| `SdcsArmorTier` | Which armour sets their look may roll (1 light, 2, 3); a bandit's aim tier too. |
| `SdcsSpawnAmmo` / `SdcsSpawnAmmoCount` | Whether a rolled gun comes with ammo, and how many rounds. |
| `SdcsThrowables` | Throwables they spawn with, `item:count,...` (once per life). |
| `HarvestItems` | Fixed starting items in their pack, `item=count` (the bandages). |
| `SdcsSpawnStock` | The loot container a wild NPC's rolled starting stock comes from (blank: none). |
| `SdcsSpawnWater` | Water they spawn with, `drink:count` (blank: none). |
| `SdcsHarvestItems` | Starting items with a rolled count for a wild NPC, once: `item=min-max`, `item=count` or `item` (a full stack), comma separated. Do not write a range in `HarvestItems`. |
| `BagItems` / `ItemsOnEnterGame` | Their spawn kit (the same properties as any NPC). |
| `LootDropEntityClass` / `LootDropProb` | The loot bag they drop dead, and its chance. |
| `SdcsDropWeaponChance` | The chance (0 to 1) that bag also holds their weapon or its parts: 0 never (the default), the bandits 0.5. |
| `SdcsDropWholeChance` | For a weapon with parts, the separate chance the whole weapon is in the bag too (0.1). |
| `SdcsDropQuality` | A whole weapon's quality, `min-max` 1 to 6 (the bandits 1-2, 2-3, 3-4 by tier). |
| `SdcsDropPartsCount` | How many parts, `min-max` 1 to 10 (1-2). |
| `SdcsSpawnMates` | Mates a biome spawn brings, `group:min-max` (blank: none). |
| `ExperienceGain` | The XP for killing them. |
| `personality` | Their personality from `personalities.xml`; for a bandit, their `<CombatLines>` and their voice. |

## Trading with wild survivors

**Trade** on a wild survivor (hired ones have none: give them items instead):

- **Buy** their goods: the stock in their pack. What you pay goes into their purse.
- **Sell** them what they can use: the medical items their heals use (bandages, first aid kits, aloe cream, splints, casts, painkillers)
  until they carry 10 (`<Trade><BuyMaxMedical>`), and ammo for a gun they carry
  up to what they keep in their bag (the `<Loot>` bag cap), at the game's normal price (your Better Barter counts), paid from their purse. Sell is greyed out for
  anything else, or for more than they have room for or can pay; choose a smaller count to sell part of a stack.
- A new wild survivor starts with 50 to 150 coins (`<Trade>`).

## Pickpockets

A survivor whose class sets `<property name="SdcsPickpocket" value="true"/>` picks pockets (no class does by default; the
test character is `npcSdcsPickpocket`). Wild only: a hired one never steals.

- **When:** you stand within `ReachMetres` (2 m) of them, at most once per pickpocket every `SurvivorMinutes` (10) and once per
  player every `VictimMinutes` (30).
- **Caught:** if they are in your sight and not hidden by the dark, they fail, say a line and back off. No fight.
- **What they take:** one stack from your backpack (never the toolbelt or what you wear): medical, ammo, food, drink, skill
  magazines (`ListedTags`), anything worth `MinValue` (100) a unit, or up to `MaxCoins` (100) dukes.
- **Where it goes:** like their loot: what they can use into their bag, the rest into their trade stock, so you can buy it back.
- Single player and the host's own player for now; players on a dedicated server are never robbed yet.

## POI residents: small talk, greetings and the evening campfire

The survivors who live in a POI (its residents: they work its stations by day, go home before dusk and sleep in its beds) now have a
small social life of their own. All of it runs on the server, so it works the same in single player, on a listen host and on a
dedicated server; each player in earshot hears the lines in the resident's voice (captioned like any other line, and caption only on
an install without text to speech).

- **Small talk.** Two free residents of the same POI who stand within `<ChatMetres>` (4 m) of each other may exchange a short canned
  pair: one says an opener, the other answers a moment later, both turn toward each other and gesture as they talk. It can happen when
  one of them reaches a stroll spot, starts a break, or settles in for the evening. At most one exchange starts within `<ChatQuietMetres>`
  (40 m) every `<ChatPoiQuietSeconds>` (3 minutes), the same two talk at most once every `<ChatPairCooldownSeconds>` (10 minutes), and
  each chance passes `<ChatChance>` (50%). A resident who is working, walking, fighting, downed or chatting with a player does not join
  in, and a fight ends an exchange at once. Nobody talks when no player is within 32 m to hear it, and a player walking up (inside the
  greeting range) stops the exchange before the answer: their greeting wins.
- **They walk over to talk.** When a resident's stroll rolls, they sometimes (`<ChatMeetChance>`, 50%) walk over to stand beside another
  free resident of their POI instead of a random spot, so the small talk actually gets a chance.
- **One greeting per POI.** Walking into a POI, one resident greets you; the others stay quiet for that visit instead of greeting one
  after another. Your own hires always greet you.
- **The evening campfire.** In the evening (not on a horde night), residents of a POI with a campfire walk over and stand around it for
  30 to 60 seconds, facing it, with its fire shown lit, then go back to their evening; standing together, they chat. A POI without a
  campfire has no gathering.
- **The lines** live in `personalities.xml`: a `<SmallTalk>` block per personality (one `<Line>` per pair, `opener|reply`) and a
  `<SmallTalkDefault>` pool used by every personality without its own. Add or reword pairs there.
- **Test it:** `sdcsresident chat` (single player / listen host) starts an exchange now between the resident you look at and the
  nearest free resident of their POI, skipping the chance, the quiet window, the pair cooldown and the player distance.

| Setting | Default | Description |
|---|---|---|
| `<Routine><ChatChance>` | `0.5` | The chance a small-talk moment (a stroll spot, a break start, the evening) starts an exchange. 0 to 1. |
| `<Routine><ChatMetres>` | `4` | How close the partner must stand. 0.5 to 20. |
| `<Routine><ChatPoiQuietSeconds>` | `180` | After an exchange starts, no other starts within `<ChatQuietMetres>` for this long. 0 to 3600. |
| `<Routine><ChatQuietMetres>` | `40` | The reach of that quiet window. 0 to 200. |
| `<Routine><ChatPairCooldownSeconds>` | `600` | The same two residents talk at most once per this long. 0 to 86400. |
| `<Routine><ChatReplyGapSeconds>` | `0.5` | The pause between the end of the opener and the answer. 0 to 10. |
| `<Routine><ChatHoldAfterReplySeconds>` | `5` | Both stay put this long after the answer. 0 to 60. |
| `<Routine><ChatTurnDegrees>` | `45` | How fast they turn toward each other, degrees a second. 0 to 180. |
| `<Routine><ChatMeetChance>` | `0.5` | The chance a rolled stroll walks to another free resident instead of a random spot. 0 to 1. |
| `<Routine><ChatMeetStandMetres>` | `1.5` | How far from the other resident they stop. 0.5 to 5. |
| `<Routine><GatherChance>` | `1` | The chance a resident joins their POI's campfire gathering each evening. 0 turns the gathering off. |
| `<Routine><GatherBlocks>` | `campfire` | The blocks they gather at (a list; each must also be in `<WorkBlocks>`). |
| `<Routine><GatherSeconds>` | `30,60` | How long each stands there (min,max seconds). |
| `<Routine><GatherRingMetres>` | `2` | How far from the fire they stand. 1 to 5. |
| modconfig `<GreetGroupMetres>` | `30` | After a greeting plays, no stranger within this many metres greets for `<GreetGroupQuietSeconds>`. 0 turns it off. 0 to 200. |
| modconfig `<GreetGroupQuietSeconds>` | `120` | How long that quiet lasts. 0 to 3600. |

## Survivor Settings — `0-XNPCCore/Config/Settings.xml`

The survivor (SDCS) behaviour of the base mod is tuned in its own `Settings.xml`, not in `modconfig.xml`. Each value is read once at game start,
so restart after an edit. A missing or unreadable value falls back to its default and prints one warning in `Player.log`.

| Setting | Default | What it does |
|---|---|---|
| `<SdcsModelYOffset>` | `0.08` | Lifts the survivor's body (metres) so the soles sit on the ground; `0` = the game's stock placement. Clamped to ±1. |
| `<AnimFromMeasuredSpeed>` | `true` | The walk and run animation follows how fast the survivor really moves over the ground, so their feet do not slide. `false`: it follows the speed the AI asked for. |
| `<SleeperTrigger>` | `true` | A survivor walking into a building's sleeper volume wakes / spawns it as a player would (only while a player is within 96 m). Bandits and sleeping NPCs never trigger one. `false`: survivors never trigger sleepers. |
| `<PartyMaxMembers>` | `8` | The party cap: the most of a leader's hires that may be in their party at once (in it while on **Follow** or **follow-and-loot**; **Stay**, **Guard**, **Patrol** and the base jobs are out of it). At the cap a 9th is refused - the hire, and the Follow / Come here / follow-and-loot / whistle orders on a non-member - with a "party full" tooltip. `1` or more. The companion list's HUD rows track this value. |
| `<GunBurstEnabled>` | `true` | Paces survivors' gunfire in bursts so automatic weapons do not empty a magazine in seconds. |
| `<GunBurstTargetRpsAuto>` | `2.0` | Average rounds per second for fully automatic guns. |
| `<GunBurstTargetRpsSemi>` | `1.5` | Average rounds per second for pistols, semi-automatic rifles and bows. |
| `<GunBurstWindow>` | `0.5` | Burst length in seconds (longer bursts, longer pauses). |
| `<HealRollSpeed>` | `1.6` | How fast the heal roll plays (1.0 = the clip's own pace). Clamped 0.5 to 3.0. |
| `<HealBandageSeconds>` | `1.5` | How long a survivor holds the bandage before the health comes back. Clamped 0.8 to 5.0. |
| `<AnyWeaponEnabled>` | `true` | A hired survivor switches to a weapon you put into (or take out of) their backpack. |
| `<AnyWeaponSettleSeconds>` | `2.0` | How long the backpack must stay unchanged before they equip (minimum about 1). |
| `<NoWear>` | `true` | Survivors' gear takes no wear (they have no way to repair it yet). |
| `<PlayerCtrlDeathWeapon>` | `vanish` | For player-controller NPCs only: on death the held weapon `vanish`es, or `fall`s with the body. |
| `<Progression>` | | Hired survivors level up: `<HealthPerLevel>` (0.03), `<DamagePerLevel>` (0.02), `<AimErrorAtLevel1>` (2, survivors), `<BanditAimErrorAtLevel1>` (3, bandits) and `<AimErrorAtMaxLevel>` (1, their spread is a player's aimed spread times this factor, falling in a straight line from level 1 to `<MaxLevel>`; see *Survivors' aim*), `<MaxLevel>` (10), `<StartLevelDaysPerLevel>` (7, world days per start level: a survivor met on day 8 starts at level 2), `<HealthRegenPerSecond>` (0.2), `<HarvestPerLevel>` (0.10, extra yield per level above 1 from wood, stone, ore, clay, plants, crops and butchering). Their XP: kills, crafting and harvesting (butchering included) pay them what they pay a player. |
| `<Crafting><PostReachMetres>` | `4` | How close to their post a crafting survivor must stand; farther away (after a fight) they walk back first. Clamped 1 to 20. |
| `<Crafting><StationReachMetres>` | `5` | How close to you a workstation must be for its recipes to show in **Craft...**. Clamped 1 to 10. |
| `<Crafting><StationFuel>` | `resourceWood,resourceCoal` | What they feed a forge or campfire, the first one found first (a comma list of item names). |
| `<Crafting><CookStations>` | `campfire` | The workstations (craft areas) a Cook may use; `BlacksmithStations` (`forge`), `ChemistStations` (`chemistryStation`) and `CrafterStations` (`workbench,cementMixer`) likewise. |
| `<Crafting><ForgeStockUnits>` | `1000` | A Blacksmith keeps each forge material they have ore for at this many units; `0` = only what an order needs. Clamped 0 to 30000. |
| `<Crafting><SmeltOre>` | the six raw scraps | The raw items a Blacksmith may put in a forge (a comma list of item names). |
| `<Crafting><FieldStations>` | `campfire` | The workstations they may also use where there is no land claim (never in another player's claim); empty = every workstation needs a claim. |
| `<Crafting><StationWanderMetres>` | `2` | After a workstation order, and when told to stay this close to one, they walk around the station within this many metres until you give them an order. `0` = they stand at their spot. Clamped 0 to 20. |
| `<Crafting><StationTools>` | the campfire's and forge's | Each workstation's tool slots in the game's own order (`area=tool,tool,tool`, stations separated by `;`): a tool they carry goes into its own slot when they craft. A station or tool not listed: they never put one in. |
| `<Crafting><StationStowMetres>` | `4` | While a workstation order runs, and while they stay at the station afterwards, their weapon stays stowed within this many metres of it; farther, a new order, a follow or the job's end brings it back. An enemy target draws it at once. Clamped 2 to 10. |
| `<Crafting><StationClearanceMetres>` | `0.3` | Their walk around a workstation keeps this far from any workstation block (their body's width), so they never walk over one. Clamped 0 to 1. |
| `<Crafting><StationArriveMetres>` | `0.3` | How close to their spot beside a workstation they walk before they start working. Clamped 0.15 to 0.7. |
| `<Crafting><StationStandInMetres>` | `0.1` | Their spot beside a workstation sits this far in from the middle of its cell, toward the station. Clamped 0 to 0.2 (more would put their body into the station). |
| `<Crafting><StationWanderPathMetres>` | `4` | A walk around the station whose path strays farther than this from it (a long way round a wall) is refused; they wait for the next walk. Clamped 1 to 20. |
| `<Crafting><AnimLightAtSeconds>` | `4` | A forge or campfire is lit this many seconds into their light clip (0 = as it starts); a station with no light clip is lit as they arrive. Clamped 0 to 30. |
| `<Crafting><AnimAccentSeconds>` | `20,40` | The gap between their accent clips while they work (min,max seconds; the next is rolled in between). Clamped 5 to 600. |
| `<Crafting><AnimSets>` | the fire, bench, use and pack sets | The clip sets: a set id, then its clips by slot (`in`, `loop`, `accent`, `light`, `refuel`, `out`; any slot may be left out). `<AnimStations>` picks the set per workstation. A set id with no row plays the rummage pose. |
| `<Crafting><AnimBackpackSet>` | `pack` | The clip set a backpack order plays. |
| `<Crafting><StationWalkSeconds>` | `10` | How long their walk to their spot beside a workstation goes on without getting closer before they give up. Clamped 3 to 60. |
| `<Crafting><StationWorkFromMetres>` | `0.7` | When that walk gives up, they still work the station from where they stopped if they are this close to their spot; farther, the order ends. Clamped 0.3 to 1.5. |
| `<Crafting><StationSettleSeconds>` | `2` | Within `StationWorkFromMetres` of their spot and no closer for this long, they stop and work from there. Clamped 0.5 to 10. |
| `<Crafting><StationWalkProgressMetres>` | `0.5` | The walk counts as getting closer only when they are at least this much nearer their spot than before. Clamped 0.1 to 2. |
| `<Crafting><StationWalkAskSeconds>` | `2` | While they walk to their spot their path is asked for again this often (a dropped path is planned again). Clamped 0.5 to 10. |
| `<Crafting><FacingDegrees>` | `20` | They face a workstation when their yaw is within this many degrees of the direction to it; they turn until then and their work clip waits for it. Clamped 5 to 90. |
| `<Crafting><WanderMinFromStationMetres>` | `1.5` | Each spot they wander to at a station is at least this far from its centre (at most `StationWanderMetres`). Clamped 0.5 to 5. |
| `<Routine><ArriveLevelMetres>` | `2` | A POI resident more than this far above or below their bed or work spot is on another floor: their walk there has not arrived and they count as away from it. Clamped 0.5 to 4. |
| `<Sleeper><SightRangeMetres>` | `16` | How close a sleeping zombie must be to see one of your survivors. Clamped 4 to 29. |
| `<Sleeper><LeaveBeyondMetres>` | `30` | Once a house wakes on a wild survivor, how far past it they run before they count as away. Clamped 10 to 100. |
| `<Sleeper><HouseAvoidSeconds>` | `300` | How long they then stay out of that house (and away from a container they gave up on). Clamped 30 to 3600. |
| `<Vehicle><GunnerMaxRangeMetres>` | `40` | The farthest a seated gunner fires; each gun also stops at its own effective range (a pistol about 17 m). Clamped 5 to 150. |
| `<Vehicle><GunnerIdleHolsterSeconds>` | `5` | How long they keep the gun out after the last enemy is gone. Clamped 0 to 60. |
| `<Vehicle><BoardRangeMetres>` | `30` | How far from the vehicle a hire walks over to get in. Clamped 5 to 100. |
| `<Vehicle><ExitClearanceMetres>` | `1` | How far outside the vehicle's body a hire is placed when they get out, so the vehicle cannot clip them; 0 = keep the game's exit spot. Clamped 0 to 3. |
| `<Light><GunLightEnabled>` | `true` | A hire whose held gun carries a flashlight mod switches it on by themselves in the dark, keeps it on in a fight, and it goes out when you crouch (they sneak with you). `false` = they never switch it. |
| `<Light><IndoorDarkBelow>` | `0.3` | By day, a room counts as dark when the sky light at their head (0 to 1: 1 under the open sky, 0 in a sealed room) is below this. At night it is always dark; a lamp does not count. Clamped 0 to 1. |
| `<Light><HoldSeconds>` | `2` | How long the dark or light must last before they switch the light, so a doorway does not flicker it. Clamped 0 to 10. |
| `<Light><GunLightRangeMetres>` | `30` | The gun light beam's reach. The game's gun light prefab is a flood (a long range, a wide cone and a glow dome around the muzzle, visible from far away, by day too), so every machine that draws the survivor clamps the light to these values. The default is the player's own gun light value; lower it for a shorter reach. The light is made a spot and aligned once to the gun's `Muzzle` when the model is found (the model's own forward in our rigs can be their back); being parented to the gun it follows it through every animation. Its shadow near plane keeps the gun, the housing and their hand from shadowing the beam (the prefab seats the light node 0.23 m behind the housing). Clamped 1 to 30. |
| `<Light><GunLightSpotAngle>` | `60` | The gun light beam's cone, degrees: the light is made a spot, pointing along the barrel (its hard inner core is 0.75 of this, derived). The default is the player's own value; lower it for a narrower beam. Clamped 5 to 151. |
| `<Light><GunLightIntensity>` | `1.3` | The gun light beam's brightness. The default is the player's own value; lower it for a dimmer beam. Clamped 0 to 1.3. |
| `<Light><GunLightGlow>` | `false` | `false` hides the muzzle's glow dome and its dust, so from far away the gun reads as a beam, not a lamp. `true` keeps the prefab's glow parts. The lens (`glow_tip`, the visible glass at the beam's tip) always stays: it is the light you see on the weapon itself. |
| `<Light><TorchEnabled>` | `true` | A hire holds a light tool (torch or flashlight) from their window or bag (never the spawn kit) in the dark when their gun has no light of its own. A fight puts it away at once and it stays away for `<TorchFightCooldownSeconds>` after; by day it goes away after `<HoldSeconds>`, your crouch puts it away, and a weapon order at night stops it for the rest of the night. `false` = they never hold the torch. |
| `<Light><TorchFightCooldownSeconds>` | `10` | After the last hostile is gone, how long they keep the torch away. Clamped 0 to 60. |
| `<Light><TorchNoToolRecheckSeconds>` | `5` | After a search of their window and bag finds no light tool, how long they wait before searching again (a torch handed to them shows up within this time). `0` searches every second. Clamped 0 to 60. |
| `<WorkSite><HarvestRadius>` | `30` | How far from their post the chop-wood, mine and pick scans reach. Clamped 20 to 40. |
| `<WorkSite><ClayRadius>` | `8` | The same for digging clay (clay is everywhere, so it stays smaller). Clamped 4 to 40. |
| `<WorkSite><MineVeinDepthLayers>` | `2` | How many layers of an ore vein they mine, down from the surface as they first saw it. Clamped 1 to 10. |
| `<WorkSite><SwingBudgetSeconds>` | `60` | How long they swing at one tree, rock, plant or corpse before skipping it. Clamped 5 to 600. |
| `<WorkSite><WalkBudgetSeconds>` | `15` | How long they walk to one target before skipping it (harvest, butcher, base keeper). Clamped 5 to 120. |
| `<WorkSite><KeepAreaLoadedMax>` | `-1` | How many hires on a working order (harvest, loot, mine, butcher, craft, base keep) keep their area loaded and working while you are away: -1 every one, 0 none, N at most N. Each keeps about 7 x 7 chunks loaded. Clamped -1 to 32. |
| `<BaseKeeper><DefaultMode>` | `base` | What a plain "repair the base" order fixes first: `base` (the most damaged block) or `combat` (a block damaged in the last few seconds). |
| `<BaseKeeper><RescanSeconds>` | `30` | How often they rescan the base for new damage. Clamped 5 to 300. |
| `<BaseKeeper><AreaRadiusMetres>` | `8` | How far their worked area reaches; two keepers stay this far apart. Clamped 2 to 40. |
| `<BaseKeeper><UnderAttackSeconds>` | `10` | The one under-attack window: in combat repair a block hit within this many seconds is worked first, and while the base has been hit within it the base-repair duty stands down with the weapon out. Clamped 1 to 60. |
| `<BaseKeeper><ChestReachMetres>` | `15` | How far a chest (not broadcast storage) may sit from their post and still feed repairs and crafting. Clamped 2 to 40. |
| `<BaseKeeper><BaseKeepRecheckSeconds>` | `2` | How often they check again whether there is any damaged block they can actually repair (their repair skill and the materials); while there is none they keep their weapon and no repair tool comes out. Clamped 0.5 to 10. |
| `<BaseKeeper><JobFightDwellSeconds>` | `3` | Once a keeper has turned to their repairs or to a fight, the least time before they turn to the other; broken at once by an enemy that just hit them in their reach, or a gun in their hand. Clamped 0 to 15; `0` = off. |
| `<Trade><StartCoinsMin>` / `<StartCoinsMax>` | `50` / `150` | The coins a new wild survivor starts with, an even roll between the two: their purse for what you sell them before anyone has bought from them. `0` and `0` = none. Clamped 0 to 20000. |
| `<Trade><BuyMaxMedical>` | `10` | They buy medical items while their bag holds fewer than this. Keep it above `<Loot><BagMaxMedical>`: their start kit fills their bag to that cap. Clamped 0 to 100. |
| `<Pickpocket>` | | Pickpockets (see *Pickpockets*): `<SurvivorMinutes>` (10), `<VictimMinutes>` (30), `<ReachMetres>` (2), `<MaxCoins>` (100), `<MinValue>` (100), `<ListedTags>` (medical,ammo,food,drinks,csm), `<BackOffSeconds>` (5, how long a caught pickpocket backs away). |
| `<Guard><SpacingBlocks>` | `2` | Survivors told to guard the same spot spread out this many blocks apart: a line beside the first, facing your way, then a rank behind (see *Order Wheel*). 1 to 5. |
| `<Wander><StuckMetres>` | `3` | A wandering survivor or bandit (no order, or Wander) counts as getting somewhere once they are this far from where they last were. Clamped 1 to 10. |
| `<Wander><StuckSeconds>` | `6` | They give the wander up when they have not got that far for this long (pressed into a car wreck or a corner), rest 10 s (the `SdcsWanderRest` buff in `buffs.xml`), then wander again. Clamped 2 to 60. |
| `<Guard><PatrolMetres>` | `6` | On `Guard and return` a survivor walks out to a random spot within this many metres of their post, looks outward 4 to 10 s, walks back, faces your way 4 to 10 s, and repeats; a fight or a talk sends them back to the post first. 0 = they hold their post. Clamped 0 to 20. |
| `<CraftSkill><Enabled>` | `true` | A specialist learns their craft by crafting (see *Learning a craft*). `false` stops the learning; what they have learned stays. |
| `<CraftSkill><CraftsPerLevel>` | `10` | The workstation crafts a skill level takes, clamped 1 to 1000. |
| `<CraftSkill><CookSkills>` / `<BlacksmithSkills>` / `<ChemistSkills>` / `<CrafterSkills>` | the skills of each role's stations | The game's crafting skills (progression.xml names, comma list) each role raises. A name that is not a crafting skill is skipped with a warning. |
| `<RepairSkill><Enabled>` | `true` | A keeper learns to repair by repairing: each paid repair and rebuilt block pays them the player's repair XP for its materials, into their repair XP and their level, and their repair XP sets the strongest building material they may work. `false` = every keeper repairs everything (the XP still counts). |
| `<RepairSkill><StartTier>` | `0` | What a new hire may repair: 0 wood, 1 cobblestone, 2 concrete, 3 steel. |
| `<RepairSkill><CobblestoneXp>` / `<ConcreteXp>` / `<SteelXp>` | `100` / `900` / `1700` | The repair XP each tier takes (a full wood repair pays about 4, cobblestone or concrete about 32). Shown on hold E > Progress. |
| `<RepairSkill><WoodMaxDamage>` / `<CobblestoneMaxDamage>` / `<ConcreteMaxDamage>` | `500` / `1500` / `5000` | A block's tier is its material's strength: up to 500 wood (sandbags, frames, wood), up to 1500 cobblestone (brick, scrap metal), up to 5000 concrete, above that steel. Workstations, traps and electrics are open to every keeper. |
| `<Medic><InCombatHealthPercent>` | `50` | The medic treats a patient below this health while an enemy is visible, and also in the first seconds after a fight (lighter wounds wait so you can patch yourself). Clamped 1 to 99. |
| `<Medic><AfterCombatHealthPercent>` | `50` | The same line once the fight is over. Clamped 1 to 99. |
| `<Medic><SelfHealHealthPercent>` | `70` | A survivor heals themselves below this health. Clamped 1 to 99. |
| `<Medic><DireHealthPercent>` | `25` | The dire line below it (the first aid kit and pain pills). Clamped 1 to 99, below the self-heal line. |
| `<Medic><SelfHealCooldownSeconds>` | `30` | The wait between low-health self-heals. |
| `<Medic><BleedRecheckSeconds>` | `10` | The wait before a new bleed can start another treatment. |
| `<Medic><WalkRetrySeconds>` | `10` | A medic whose walk to a patient gets stuck skips that patient this long before trying again. 0 = no wait. |
| `<Downed>` | | Downed and revive (see *Hired survivors: downed and revive*): `<Enabled>` (true, the downed state on), `<FloorHealth>` (10, the health they hold at while down), `<SurviveSeconds>` (60, the bleed-out), `<GraceSeconds>` (60, invulnerable after a revive), `<RedownWindowSeconds>` (600, a second down within it is a real death), `<HoldSeconds>` (9998, the ragdoll hold), `<LateShowSeconds>` (1.0, how long a late-joining client waits before it shows them lying), `<ReviveItems>` (medicalFirstAidBandage, medicalFirstAidKit), `<ReviveRangeMetres>` (4, how close the reviver must be), `<WarnSeconds>` (30,10, the seconds-left marks for the leader's bleed warning). |
| `<Downed><ClearNegativeBuffs>` | `true` | A down clears their negative status effects (the game's own definition: a buff whose class has a damage type, so fire, shock, bleeding and the long-term ones), so a hire who goes down burning does not keep burning; a revive is a treatment. false = they keep them until the usual expiry. |
| `<Downed><ClearKeepBuffs>` | *(blank)* | The buff names that survive the down's clear (comma separated, any case); blank = nothing survives. |
| `<Downed><SettleRagdoll>` | `true` | Steadies a downed survivor's live ragdoll the way the game steadies a dead body, so they do not drift or twitch while they lie; their own values come back when they get up. `false` = the live ragdoll as the game leaves it. |
| `<Downed><SettleAfterSeconds>` | `1` | How long after they go down the settle starts, so the blow that downed them plays out first. Clamped 0 to 10. |
| `<Downed><SettleMaxAngularVelocity>` | `1` | The fastest each ragdoll body may spin while settled, radians per second (1 is the game's dead body value). Clamped 0.1 to 50. |
| `<Downed><SettleMaxDepenetrationVelocity>` | `2` | The fastest the physics may push overlapping ragdoll bodies apart while settled, m/s (2 is the game's dead body value). Clamped 0.1 to 50. |
| `<Downed><LateShowAfterRagdollSeconds>` | `6` | Multiplayer: a client never starts its own catch-up downed ragdoll this soon after one ended on that client (so a hire who was just revived is not laid down again). Clamped 0 to 30. |
| `<Combat><WildTargetedRadiusMetres>` | `15` | How far away a zombie that targets a wild (unhired) survivor still makes them fight it. Clamped 1 to 60. |
| `<Combat><WildHelpMetres>` | `15` | A wild (unhired) survivor fights an enemy that is attacking you or another survivor within this distance of them. 0 turns it off. Clamped 0 to 40. |
| `<Combat><DefensiveZombieMetres>` | `10` | A hire on Defensive fights a zombie or hostile animal they hear or can see within this distance of them, whatever it is attacking. Bandits and other humans still have to come for the squad first. 0 turns it off. Clamped 0 to 30. |
| `<Combat><MarkWalkRetrySeconds>` | `10` | A Hunting or Aggressive gun hire runs at your marked enemy until their gun reaches it; when that run stalls (a wall, no way through) they leave that target alone for this long. A new mark is not affected. Clamped 0 to 60. |
| `<Combat><CloseMeleeEnterMetres>` | `3` | A survivor holding a ranged weapon swaps to melee at or below this distance. Clamped 0.5 to 20. |
| `<Combat><MeleeDefendMateMetres>` | `15` | A hire on a melee weapon walks to an enemy they cannot see when it is attacking one of their squad mates within this distance of them. 0 turns it off. Clamped 0 to 40. |
| `<Combat><CloseMeleeExitMetres>` | `5` | The hire swaps back to the ranged weapon beyond this distance (it must be above the enter distance). |
| `<Combat><CloseMeleeMinHoldSeconds>` | `3` | The least time on the melee swap before they swap back. Clamped 0 to 30. |
| `<Combat><BreakThroughDoors>` | `true` | An enemy behind a door, window, glass or bars: an Aggressive hire or a bandit opens an unlocked door, otherwise breaks through with their best block-breaking melee item and switches to their gun the moment the opening gives a line of fire; a Defensive or Hold fire hire and an unhired survivor leave it alone until the way is open. Nobody swings at a wall or at another player's door. false = nobody breaks through. |
| `<Combat><MeleeArcRays>` | `5` | How a swinging weapon's arc is probed: when its centre ray is clear, this many rays (clamped 3 to 9) spread across the weapon's own `SwingDegrees`, each cast with the weapon's own sphere thickness; any ray that meets a solid block blocks the swing the way a wall does (the blocked-swing log line names the weapon and the ray count). A thrusting weapon keeps the single centre ray. |
| `<Combat><MeleeArcFloorClearance>` | `0.5` | Metres above the higher fighter's feet below which an arc hit is the ground and never blocks a swing. Raise it if low kerbs or slopes still stop swings; `0` counts only blocks at or below the feet as floor. |
| `<Combat><MeleeThrustSwapMinHold>` | `3` | Seconds a survivor keeps the thrusting weapon they drew because their swing was blocked (bars, a one-block gap the stab can reach through) before their own weapon comes back once it can swing clear again. It also comes back at once when that target dies or they change target. |
| `<Combat><MeleeThrustTags>` | `perkJavelinMaster,perkDeepCuts,perkBrawler` | Weapons carrying one of these tags are thrusting (a straight hit that goes through a 1-block opening). A weapon's own `SdcsMeleeStyle` item property (`"thrust"` or `"swing"` in its item XML) beats both tag lists; `MeleeSwingTags` beats this one; with neither, a weapon swings. |
| `<Combat><MeleeSwingTags>` | `` | Weapons carrying one of these tags are forced to swing, beating `MeleeThrustTags`. Empty by default. |
| `<Combat><MeleeSwingHitsBlocks>` | `false` | When a swing is blocked by a solid block and the held weapon's block damage is at least `MeleeSwingBlockDamageMin`, they swing at the block instead of standing back (the hit damages it, so a strong swing can break the block); otherwise a wall is a no-swing and the target counts as unreachable. `false` = nobody swings at walls. |
| `<Combat><MeleeSwingBlockDamageMin>` | `30` | The minimum block damage a held weapon needs for `MeleeSwingHitsBlocks` to turn a blocked swing into a swing at the block. |
| `<Combat><NpcThrustPiercesBars>` | `true` | For a thrusting weapon in an NPC's hand, both the reach probe and the real swing hit with the arrow hit mask, so bars and openings the arrow passes through do not stop it (the probe casts the arrow mask, the swing's own hit mask is swapped to it for the length of the cast and restored after). A swinging weapon still stops at bars, though its arc probe can find a solid block at the arc's edge. A weapon that declares its own `Hitmask_override` is never touched. |
| `<Combat><PlayerThrustPiercesBars>` | `false` | The same bar-piercing for the LOCAL player's thrusting weapon (both their main hit and the game's own arc-sweep pass). Off by default: it makes base defence much easier. |
| `<Combat><HoldUnseenGiveUpSeconds>` | `10` | A hire holding on an enemy they cannot see (behind a wall or floor) lets it go after this long unseen AND unheard and follows you again; they take it up again when they see or hear it, or it hits them. Clamped 2 to 60. |
| `<Combat><CrawlerAimHeadMetres>` | `0.45` | Dedicated server: where they aim at a crawler's head, above its root. Clamped 0.05 to 1.5. |
| `<Combat><CrawlerAimBellyMetres>` | `0.25` | The same for its belly (it must be below the head height). |
| `<UaiDecisionSeconds>` (top level) | `0.6` | How often a survivor or bandit re-decides what to do, in seconds, the same in single player and on a dedicated server (the game alone ties it to the frame rate: a server re-decided about three times as often, at three times the cost). Lower reacts faster and costs more. `0` = the game's own pace. At most `10` (a larger value reads as 10). |
| `<UaiTaskEndRescoreSeconds>` (top level) | `0.2` | When a hire's action ends on its own (a stalk it was spotted on, a hold it gave up), it decides again at once instead of briefly restarting the same action; this is the least time between two such re-decisions, in seconds (0 to 2; 0 turns it off). |
| `<Combat><TimidFleeDistanceMetres>` | `15` | A timid character (a class that sets the `$Timid` cvar, such as `npcSdcsSurvivorFemaleTimid`) runs from an enemy they can see within this many metres instead of fighting, hired or not, then follows you again. `0` turns fleeing off. |
| `<Combat><MeleeUnreachableSeconds>` | `3` | A melee fighter who can see their target but gets no closer (stuck at a gap too small to walk through, say) gives up on that target after this many seconds and fights an enemy they can reach. A hire with no other enemy goes back to following. `0` turns this off. |
| `<Combat><MeleeUnreachableForgetSeconds>` | `20` | How long they ignore a target they could not reach before trying it again. |
| `<Combat><MeleeUnreachableNearMetres>` | `3` | Within this many metres a melee fighter who sees their target but cannot reach it (standing at a gap in a wall, say) counts as stuck even while they wait, so they give up on it after `MeleeUnreachableSeconds`. `0` counts only the time they walk toward it. |
| `<Combat><AttackerPrioritySeconds>` | `3` | How long after a hit the enemy that did the damage beats the target a keep-target fighter was holding: while within this window, an attacker within their melee reach (or one with a gun, which shoots through the gap) wins the target choice, so they turn and fight back instead of holding on a target that is not hitting them. A target marked unreachable (the row above) never keeps the preference. `0` turns this off. Clamped 0 to 15. |
| `<Combat><CloseMeleeClosingMetres>` | `0.5` | A gunner swaps to their melee weapon for a close enemy only when it is within their melee reach or closing on them by at least this many metres over `CloseMeleeClosingSeconds`. An enemy standing still at a gap in a wall does not make them swap. `0` swaps only for an enemy already in reach. |
| `<Combat><CloseMeleeClosingSeconds>` | `1` | The time window for `CloseMeleeClosingMetres`. |
| `<Combat><CloseMeleeWallReenterSeconds>` | `1` | After a wall between them and the enemy made a gunner put their melee weapon away, they do not swap back to it for this many seconds. Stops their switching weapons over and over when the enemy stands at the edge of a wall. `0` turns the wait off. |
| `<Combat><SightStealthOnlyWhileLeaderCrouches>` | `true` | A hired survivor holds fire on an enemy that cannot see you (too far or too dark for it) only while you crouch, so a sneak stays quiet. Standing, they fight anything they can see. `false` = they hold fire whenever the enemy cannot see you, crouched or not. |
| `<Combat><SightThroughPlants>` | `true` | Survivors and bandits see past grass, bushes, flowers and crops (up to three in a row) instead of the plant blocking the view; a tree trunk still blocks. With `SightStealth` on, each plant still makes the target harder to see (below). `false` = a plant blocks sight. |
| `<Combat><SightStealth>` | `true` | Survivors and bandits use the game's stealth value (the one your HUD stealth meter shows: light, crouching, moving) when they look at you or at another survivor or bandit, also while choosing whom to fight. How sharp-eyed each one is comes from its class's `SightLightThreshold` in entityclasses.xml. Zombies are not affected. `false` = they ignore stealth the way SCore does. |
| `<Combat><SightPlantStealthFactor>` | `0.7` | Each plant between the watcher and the target multiplies the target's stealth value by this, so crouching behind a bush hides you sooner. `1` = plants do not help; `0` = any plant hides completely. |
| `<Combat><SightPlantStealthFactorHidden>` | `0.35` | The same, but for a character marked `$HidesInPlants` while they crouch (see POI-BUILDERS). |
| `<Combat><SightAttackerExemptSeconds>` | `5` | For this many seconds after an enemy hurts a survivor or bandit, that enemy is seen whichever way they face, so they turn and fight back at any range. Their view cone is their class's `MaxViewAngle` (220 for survivors and bandits). 0 = no exemption. 0 to 25. |
| `<Combat><HearingLevel>` | `3` | How loud an enemy's footstep must be where a survivor or bandit stands for them to hear it. A step counts 3 to 15 by ground (grass 6, wood 10, gravel 11), doubled when running, less than half when a survivor crouches, and fades with distance. At 3 a walk is heard about 2 to 5 m away, a run 5 to 11 m, a crouched survivor within 2 m. Higher = duller. 0 = no hearing. 0 to 20. |
| `<Combat><HearingSeenSeconds>` | `5` | For this many seconds after they hear an enemy, it counts as in front of them, so they notice it and turn if nothing blocks the view and it is not too dark. 0 = hearing changes nothing. 0 to 25. |
| `<Combat><HearingVoiceQuietNoise>` | `8` | The noise of an enemy's quiet voice (its roam groan, its sense sound) for an awake survivor's or bandit's hearing, on the same scale as a footstep (grass 6, gravel 11): 8 is heard about 4 m away. 0 = not heard. Clamped 0 to 60. |
| `<Combat><HearingVoiceLoudNoise>` | `15` | The same for its alert and attack sounds: 15 is heard about 8 m away. 0 = not heard. Clamped 0 to 60. |
| `<Combat><ClearShotStepMetres>` | `2` | A survivor or bandit with a gun steps this far aside, away from the one in their way, when you or a friend who cannot kneel stand in their line of fire, then shoots. Clamped 1 to 5. |
| `<Combat><ClearShotLeaderRadiusMetres>` | `20` | ...and when it is you, for an enemy within this distance of you that is going for you. Clamped 2 to 60. |
| `<Combat><KneelChance>` | `0.5` | A gun holder standing still to shoot kneels with this chance per target (0 never, 1 always). Clamped 0 to 1. |
| `<Combat><KneelMinTargetMetres>` | `15` | ...only for a target at least this far away. Clamped 3 to 100. |
| `<Combat><KneelNoEnemyWithinMetres>` | `10` | ...and never while another enemy is this close to them. Clamped 2 to 40. |
| `<Combat><KneelForAllySeconds>` | `3` | A friend standing still in a shooter's line kneels this long when the shot then passes over them. Clamped 1 to 10. |
| `<Combat><HealGunThreatMetres>` | `25` | A survivor or bandit does not start a self-heal while an enemy holding a gun has them in sight this close: they keep fighting and heal once it is gone. A melee threat (a zombie) still counts only within arm's reach, as before. A burning one is never held. 0 = never held. Clamped 0 to 60. |
| `<Combat><DeathGuardLeashMetres>` | `8` | A death-guard hire (their leader died, they guard the leader's dropped bag) keeps every fight move within this distance of the leader's death spot and within `<Bandits><SentryLevelMetres>` of its floor; their cover and back-up reach counts to it too. Bandits keep `<Bandits><SentryLeashMetres>`. Clamped 2 to 30. |
| `<Combat><InViewReloadDeadlineSeconds>` | `8` | A hire in view whose reload was asked for but never started (they stand clicking an empty gun with ammo in their pack) gets it completed after this many seconds, without the reload animation, and the log prints one `[SDCS-RELOAD] ... reload completed in view` line naming the likely cause. Above the reload clip length, so a working reload always wins. Clamped 5 to 30. |
| `<Cover><MaxScansPerFrame>` | `2` | The most cover searches that run in one frame across all NPCs, so a squad's first gunshot does not stall a frame; the rest wait a frame. Clamped 1 to 16. |
| `<Cover><DefensiveCloseInMetres>` | `5` | A Defensive melee hire steps out of cover toward a gunman this close to them, with or without a clear line. Clamped 1 to 15. |
| `<Cover><DefensiveAssistMetres>` | `15` | A Defensive melee hire steps out toward a gunman that targets the party (them, their leader or a squad mate) within this distance of their leader. Clamped 5 to 40. |
| `<Cover><HuntingLeashMetres>` | `20` | A Hunting hire engages any enemy they see within this distance of their leader, and never chases past it. Clamped 8 to 40. |
| `<Cover><FruitlessRescanMoveMetres>` | `2` | After a cover search that found nothing, a survivor or bandit does not search again until they, their enemy or the centre of the area they keep to has moved this far (or their enemy, the kind of cover they are after or their order changed, or `FruitlessRescanSeconds` passed); meanwhile they do what they do when there is no cover. Clamped 0.5 to 10. |
| `<Cover><FruitlessRescanSeconds>` | `9` | The longest a cover search that found nothing is trusted while nothing moves. It saves searches only when it is longer than `<Cover><NoSpotCooldownSeconds>` and `<ShelterRescanSeconds>` (3 each): a spot that frees up while nothing moves is found up to this much later. Clamped 0.5 to 15. |
| `<Follow><PaceNearDistMetres>` | `8` | At or below this distance from you they walk. Clamped 1 to 60. |
| `<Follow><PaceFarDistMetres>` | `10` | At or above it they move at full speed; between the two their speed blends (it must be above the near distance). |
| `<Follow><PaceNearWalkSpeedMps>` | `2` | Their walking speed near you, in metres per second (0.5 up to below 2.75, where the run starts). |
| `<Follow><HireActivateReachMetres>` | `6` | How far the E key reaches YOUR OWN hires (the game's reach, 3.5 m in survival, stays for everyone else), so a hire on Far spacing still answers E. 2 to 10, default 6. |
| `<Loot><BagMaxMedical>` | `5` | Medical items they keep in their own bag from looting; the rest goes to their harvest window. Clamped 0 to 100. |
| `<Loot><BagMaxAmmoStacks>` | `2` | Full stacks of each carried gun's ammo they keep in their bag. Clamped 0 to 20. |
| `<Loot><BagMaxDrinks>` | `10` | Drinks a gift puts in their own bag (a hired survivor needs a drink to douse themselves when they burn). Clamped 0 to 100. |
| `<Loot><VerticalReachMetres>` | `3` | How far above or below you they loot (one storey). Clamped 1 to 20. |
| `<Loot><BagRangeMetres>` | `25` | How far from you, flat, they still take a loot bag on the ground; any height they can reach. Clamped 1 to 100. |
| `<Loot><ReturnTeleportMetres>` | `40` | After "loot around here" ends: within this distance they walk back to you, beyond it they teleport. Clamped 5 to 44. |
| `<Loot><LowTopMetres>` | `1.1` | A container whose top is at most this high above their feet is looted kneeling; a taller one they lean into standing. A loot bag on the ground is always looted kneeling. Clamped 0.3 to 3. |
| `<Loot><ChainMetres>` | `1` | When their next bag or low container is this close to where they knelt, they stay kneeling and turn to it; they stand up only when the chain ends. 0 turns chaining off. Clamped 0 to 4. |
| `<Loot><ChainHoldSeconds>` | `3` | After a rummage with a close target left, how long they stay kneeling for it before standing up after all. Clamped 0 to 10. |
| `<Loot><DrawGraceSeconds>` | `3` | After a rummage their weapon stays stowed this long, so it does not flash out between containers; a fight draws at once. 0 draws as soon as the rummage ends. Clamped 0 to 30. |
| `<Loot><FaceWaitSeconds>` | `1.5` | The longest they turn toward a bag or container before their rummage starts anyway. Clamped 0 to 5. |
| `<Loot><TurnDegreesPerSecond>` | `360` | How fast they turn to face what they loot. Clamped 90 to 1080. |
| `<Loot><BagSightLift>` | `0.5` | A loot bag's eye line is checked to a point this far above it, so a bag in a dip is still seen; a wall or hill between still hides it. Clamped 0 to 1.5. |
| `<Loot><RummageBreakMetres>` | `12` | While they follow you and rummage, they stay at it while you are within this distance; farther, they break off and follow (the items stay put). Clamped 4 to 40. |
| `<Loot><ResumeFreeSlots>` | `5` | When their pack fills, a loot, harvest, mine, clay, pick or butcher job pauses instead of ending: they come back to you, and once you empty their pack to at least this many free slots they go back to the job. Clamped 1 to 100. |
| `<Loot><ResumeMinutes>` | `10` | How long that pause waits for an emptied pack; after it, or on any other order, they just follow. 0 turns the pause off (a full pack ends the job). Clamped 0 to 240. |
| `<Ammo><RestockTarget>` | `120` | Inside your (or an ally's) land claim, a hired survivor whose spare rounds for their gun drop below one magazine tops up to this many from the base: broadcast containers anywhere in the claims and your unlocked chests within `<BaseKeeper><ChestReachMetres>` of them, never their own pack or another player's box. `0` turns it off. Clamped 0 to 5000. |
| `<Ammo><RestockCheckSeconds>` | `5` | How often they check their ammo. Clamped 1 to 120. |
| `<Ammo><RestockRetrySeconds>` | `30` | After a check that took nothing (no ammo in the base, or they are outside it), the wait before the next. Clamped 5 to 600. |
| `<Leadership><RegenPerSecond>` | `0.5` | Your Charismatic Nature rank 1 and up: health your hired survivors regain per second. Clamped 0 to 5. |
| `<Leadership><DamageBonus>` | `0.2` | Rank 2 and up: the extra damage they deal (0.2 = 20%), to blocks too with a melee weapon. Clamped 0 to 1. |
| `<Leadership><DamageResist>` | `0.1` | Rank 3 and up: the share of damage they do not take (0.1 = 10%). Clamped 0 to 0.9. |
| `<Bandits><PlayerDamage>` | `0.5` | The damage a bandit deals to PLAYERS, as a multiplier on their weapon's (1 = full, 0.5 = half); survivors, zombies and animals take full damage. Change it while playing with `sdcsbandit damage <x>` (lasts until the game restarts). Clamped 0 to 5. |
| `<Bandits><SpreadByTier>` | `0,0,0` | An extra spread for a bandit by their tier (T1, T2, T3), added to their hip spread on top of the aim every survivor of their level has (see *Survivors' aim*): 0 = they shoot like a survivor of their level, 1 = one more hip spread of their gun. Clamped 0 to 10 each. |
| `<Bandits><MatesMinMetres>` | `2` | The nearest a tier 2 or 3 bandit's mates spawn to them. |
| `<Bandits><MatesMaxMetres>` | `6` | The farthest their mates spawn from them. |
| `<Bandits><MatesMinPlayerMetres>` | `20` | A mate never spawns closer than this to a player (the `sdcsbandit group` test command ignores it). |
| `<Bandits><LineCooldownSeconds>` | `60` | After a bandit's one cold line (the moment they lock onto a person to fight: a player or an NPC, never a zombie or an animal), the wait before they can say another. Text to speech only. Clamped 0 to 3600. |
| `<Bandits><LineGroupQuietSeconds>` | `10` | After a bandit's line, every other bandit within `<LineGroupMetres>` of where they said it stays quiet this long, so only the first of a group speaks. Clamped 0 to 3600. |
| `<Bandits><LineGroupMetres>` | `32` | The reach of that quiet window (the distance a line is heard at). Clamped 0 to 200. |
| `<Bandits><SleepWake>` | `true` | A sleeping bandit hears and sees like a sleeping zombie, in any room (their class's `Sleeper*` values on `npcSdcsBase`): a player's noise heard at their distance over their wake threshold, or a player within `<WakeSightMetres>` in their `<WakeSightAngle>` cone in enough light, wakes them and makes the player their target; an enemy survivor's sounds and light count too. Walking into their room still wakes it. `false`: they wake only with their room, a hit or a blast. |
| `<Bandits><WakeCheckSeconds>` | `0.5` | How often a sleeping bandit listens and looks. Clamped 0.25 to 2. |
| `<Bandits><WakeSightMetres>` | `6` | Their sleeping sight range. Clamped 2 to 20. |
| `<Bandits><WakeSightAngle>` | `180` | Their sleeping sight cone in degrees. Clamped 30 to 360. |
| `<Bandits><WakeSightCurveMetres>` | `30` | The distance over which the light they need rises from point blank to far (a sleeping zombie's curve). Clamped 10 to 60, never below `<WakeSightMetres>`. |
| `<Bandits><NoCoverRetrySeconds>` | `3` | A melee bandit who finds no cover to bound to charges at once, and does not look for cover again for this long. Clamped 0.5 to 30. |
| `<Bandits><DropUnseenSeconds>` | `10` | A bandit holds on to an enemy only while they have seen it, heard it or been hurt by it within this many seconds; after that they drop it and go back to their post or their patrol. Clamped 2 to 60. |
| `<Bandits><SentryLeashMetres>` | `1.5` | A posted bandit (a sentry, or a guard spawn block's Stay or Guard bandit) keeps every fight move within this distance of their post, and within `<SentryLevelMetres>` of its floor; no move drops them off it. Clamped 0.5 to 6. |
| `<Bandits><SentryLevelMetres>` | `0.6` | How far above or below their post's floor still counts as on it for a posted bandit: their cover spot, the path to it, their step out and their dodge step all stay within it. At the default only their own floor counts. Clamped 0.25 to 2.5. |
| `<Bandits><SentryMeleeLeashMetres>` | `6` | A sentry or guard-block bandit holding a melee weapon (or one whose gun ran dry) steps out to fight an enemy within this distance of their post, then walks back when the fight is over; a gun holder keeps `<SentryLeashMetres>`. The level band (`<SentryLevelMetres>`) is the same. Clamped 1.5 to 15. |
| `<Bandits><SentryEngageMetres>` | `20` | An unhired friendly sentry on their post fights any enemy within this distance of their post, not only one that hurts them, comes within 5 m or targets them; their leash still holds them on the post. A bandit sentry fights on sight anyway. Clamped 5 to 40. |
| `<Bandits><SentryShoveReturnMetres>` | `0.3` | A posted survivor pushed this far from the spot where they stopped, while they have no move of their own (an enemy pressing against them), steps back to that spot. Their own steps (cover, a clear shot, a melee guard stepping out) are never undone. Clamped 0.1 to 1.5. |
| `<Stealth><Enabled>` | `true` | The stealth kill and the quiet-kill order (see *Hired survivors: stealth kills*). `false` turns both off. |
| `<Stealth><SpottedDefault>` | `fight` | What a hire whose personality has no `<Spotted>` does when the target notices the party on their way in: `backoff`, `fight` or `charge` (anything else reads as `fight`). |
| `<Stealth><ApproachSpeedScale>` | `1.0` | Their crouched approach pace as a fraction of their walk speed. Clamped 0.3 to 1.0. |
| `<Stealth><GiveUpSeconds>` | `30` | The base time of the approach before they give up; see `<GiveUpPerMetreSeconds>` for the far-start part. Clamped 5 to 60. |
| `<Stealth><GiveUpPerMetreSeconds>` | `1` | Extra give-up time per metre the target was beyond 20 m at the start of the approach: a far stalk gets time to arrive (30 s + 1 s per metre past 20 m, so a 50 m target allows 50 s). Clamped 0 to 5. |
| `<Stealth><StalkRetrySeconds>` | `10` | After a stalk ends because the target spotted or noticed them, or their approach stalled or ran out of time, they leave that target alone this long before they stalk it again (they still fight it). A quiet-kill order is never held back by it. Clamped 0 to 60; 0 turns it off. |
| `<Stealth><SightGraceSeconds>` | `10` | An ordered quiet kill keeps stalking this long after they lose sight of the target. Clamped 0 to 20; 0 stops at once. |
| `<Stealth><SightGraceAllStalks>` | `true` | Every stalk the hire is on gets the sight grace above (its own Hunting stalk and one on your mark too), so a wall or a door hiding the target for a moment does not end it. `false`: only your Stealth kill order gets the grace. |
| `<Stealth><HoldCrouchesWithLeader>` | `true` | A hire holding on an enemy stays crouched while you crouch, instead of standing up beside you mid-sneak. `false`: the hold crouches only under gunfire. |
| `<Stealth><ShotMarginMetres>` | `3` | How far outside the target's notice distance a bow hire stops to take the quiet shot. Clamped 0 to 10. |
| `<Stealth><ShotMinMetres>` | `6` | The closest they take the quiet shot from. Clamped 3 to 15. |
| `<Stealth><ShotMaxMetres>` | `20` | The farthest they take the quiet shot from. Clamped 8 to 40. |
| `<Stealth><ShotHoldSeconds>` | `2` | How long they stay crouched after the quiet shot, so the arrow lands with the sneak bonus. Clamped 0.2 to 3. |
| `<Stealth><ShotSideStepMetres>` | `3` | A bow hire whose line is blocked at their shooting spot tries spots this far and half this far to either side for a clear line, never inside the target's notice distance. Clamped 0 to 5; 0 turns the side step off. |
| `<Stealth><ShotRepathMetres>` | `1.5` | A bow hire walking to their shot re-aims their walk when the target has moved this far. Clamped 0.5 to 5. |
| `<Stealth><SneakHitParity>` | `true` | A hire's sneak hit (they are crouched, the target has not noticed them) deals what your hit with the same weapon would: it is scaled by the world's Melee or Ranged Damage setting over its Entity Damage setting. `false` leaves it at the Entity Damage setting. |
| `<Stealth><OrderRangeMetres>` | `60` | How far from you the target of a quiet-kill order may be when the order lands. Clamped 10 to 80. |
| `<Stealth><StrikeHeightMetres>` | `1.0` | How far above or below the target a stalking hire may be and still strike (a raised bed, a stair landing). Clamped 0 to 2.5. |
| `<Bandits><PoiTierFloors>` | `1:1,2:1,3:100,4:100,5:400,6:400` | POI difficulty:gamestage pairs: a bandit sleeper group in a POI of that difficulty picks its bandits at no less than that gamestage (100 = tier 2, 400 = tier 3); the party's own gamestage still raises it. Zombies are unchanged. |
| `<Bandits><SeekArriveHeightMetres>` | `2.5` | A seek (a bandit's, or a hire's) has arrived only within this height of its spot, so a seeker on the floor above or below keeps walking their path to the stairs. Clamped 1 to 10. |
| `<Progression><KillXpWorkaround>` | `true` | Works around an SCore error on NPC kills that also costs the player the kill's XP; leave on. |
| `<AwakeStealth>` | `true` | An awake zombie that is after a survivor sees them through the player's own stealth test: crouching, a held light and speed count, as they do for you. |
| `<HitBoxFix>` | `true` | Keeps the oversized invisible box around survivors, NPCs and pets switched off, so hits land on the body. `false`: the game's behaviour (hits up to 30 cm off the body). An NPC whose model has no body-part colliders (some other NPC packs) keeps its box, so they can still be hit and talked to. |
| `<TraderAreaScanOff>` | `true` | Stops the game removing one of two same-class survivors (or hires) standing close together inside a trader compound, a check meant for duplicate traders. `false`: the game's behaviour. |
| `<SdcsArmorExcludeGroups>` | `groupPreacher` | Outfit groups (comma separated) never rolled for an NPC, e.g. coats whose cloth only animates on players. |
| `<SdcsArmorExcludeItems>` | `` | Single armour items (comma separated) never worn by an NPC; a set left with nothing in a slot is dropped. Empty by default. |
| `<SdcsArmorFemaleOnly>` | `` | Outfit groups or armour items (comma separated) only women wear. Beats the detection below. Empty by default. |
| `<SdcsArmorMaleOnly>` | `` | Outfit groups or armour items (comma separated) only men wear. Beats the detection below. Empty by default. |
| `<SdcsArmorSexDetect>` | `true` | Detects armour made for one sex from its model path, so a man never wears a dress. `false`: only the two lists above decide. |
| `<SdcsCustomCharacterChance>` | `0.1` | Chance that a new wild survivor is dressed as one of an online player's own creator characters of the same sex; `0` turns it off. |
| `<NpcPathRules>` | `true` | Survivors plan their paths with the mod's own rules (the hazard and drop settings below). `false`: the game's own paths. |
| `<NpcPathSpikeCost>` | `1000000` | Path cost of a spike / barbed wire / cactus cell: crossed only when there is no other way. |
| `<NpcPathHazardMargin>` | `4000` | Extra path cost next to a hazard, so paths keep one cell off spikes where there is room. `0` disables it. |
| `<NpcPathWallCost>` | `1000000` | Extra path cost, on every survivor's and bandit's path (hired or not), of a step into a block at head height (a wall they can neither walk through nor hop): they take an opening or a way round, and go at the wall only when there is no other way. A thin barrier (a shanty wall, a railing, a thin fence) is never crossed at all: their path goes round it, through a door or an opening. None of them can break blocks. `0` disables both. |
| `<NpcPathMaxDropBlocks>` | `5` | A path never drops more than this many blocks in one step. `0` disables the cap. |
| `<NpcPathMaxSpeed>` | `6` | Top speed (m/s) while they follow a path, so they cannot run past its turns (on open ground they keep their full sprint). `0` disables it. |
| `<NpcHazardWalkSpeed>` | `1.0` | Speed cap (m/s) while a hazard is right next to their feet. `0` disables it. |
| `<NpcHazardStepOut>` | `true` | A survivor standing in contact with a hazard steps to the nearest safe spot. |
| `<NpcHazardFirstTouchDamage>` | `1` | Damage of their first brush against a hazard; a second touch within 3 s deals normal damage. `-1` turns the rule off. |
| `<Emotes><Reactions><Hire><Enabled>` / `<Chance>` / `<CooldownSeconds>` / `<PendingSeconds>` | `true` / `1` / `0` / `3` | The salute right after you hire them (see *Hired survivors: emotes*). `Chance` is how often it plays when it may (0 to 1), `CooldownSeconds` the least time between two of theirs (0 = none), `PendingSeconds` how long it waits for them to be free after the hire before it is dropped (0 to 10). A survivor already hired when the world loads never salutes. |
| `<Emotes><Reactions><FightEnd><Enabled>` / `<Chance>` / `<CooldownSeconds>` | `true` / `0.5` / `60` | The fist bump when a fight they were in is over (hires only, never bandits). |
| `<Emotes><Reactions><Greet><Enabled>` / `<Chance>` / `<CooldownSeconds>` | `true` / `0.5` / `120` | The wave with their greeting when it plays toward you. Every reaction is skipped, never saved for later, while they fight, walk, play another animation, work a job, sit in a vehicle or have an enemy near. |
| `<Emotes><Reactions><Talk><Enabled>` / `<Chance>` / `<CooldownSeconds>` / `<Clips>` | `true` / `1` / `2` / `talk1,talk2,talk3,talk_happy,talk_dismiss,talk_lookaway,talk_ack,talk_cocky` | Their hands while they speak a chat reply: a gesture as each sentence starts, once the last one is over, never the same one twice in a row. |
| `<Emotes><Reactions><Nod><Enabled>` / `<Chance>` / `<CooldownSeconds>` / `<Clips>` / `<Lines>` | `true` / `1` / `3` / `nod1,nod_yes,nod_hard,nod_long` / `ammo.switched,basekeep.remembered,vehicle.wait,book.later,give.atfeet,comehere.ack,loot.resume` | A nod when they accept an order (an order wheel or voice order is answered, or they speak one of their `<Lines>`). `<Lines>` is a comma separated list of spoken line ids that nod; an entry ending in a colon covers every line id that starts with it. |
| `<Emotes><Reactions><Shake><Enabled>` / `<Chance>` / `<CooldownSeconds>` / `<Clips>` / `<Lines>` | `true` / `1` / `3` / `shake1,shake2,shake_no,shake_annoyed,shake_thoughtful` / `job.refused,patrol.none,patrol.far,craft.cant,craft.busy,craft.fuel,craft.noclaim:,craft.trader:,craft.inuse:,ammo.same,ammo.none,ammo.nogun,ammo.unknown,vehicle.noroom,vehicle.far,basekeep.nobase,book.nothing,book.notbook,book.none,harvest.notool,mine.notool,clay.notool,butcher.notool,basekeep.notool` | A head shake when they refuse one (the same, or one of its `<Lines>`). A line in both the Nod and Shake lists shakes. |
| `<Emotes><Idle><Enabled>` | `true` | Idle moments: a hire who stands still and free now and then plays a short idle clip. `false` = never. |
| `<Emotes><Idle><StillSeconds>` | `20` | How long they stand still and free before the first one. |
| `<Emotes><Idle><MinIntervalSeconds>` / `<MaxIntervalSeconds>` | `60` / `60` | The random wait between two of their idle clips; each survivor rolls their own. |
| `<Emotes><Idle><StillMetres>` | `0.5` | A move of more than this between two looks a second apart (theirs, or their leader's while they follow) counts as walking. |
| `<Emotes><Idle><Clips>` | `stretch,warmup,whatever,bored,lookaround,idle_weightshift,breathingidle,neckstretch,nervouslookaround,lookingbehind,yawn,neutralidle,standingshortidle,weightshiftidle,standingidlelookaround` | The idle clips (ids from the base mod's `Config/SdcsAnimClips.xml`, comma separated); an unknown id is skipped with a warning. |
| `<Emotes><Idle><FemaleOnlyClips>` | `weightshiftidle,standingshortidle` | Idle clip ids a male hire never plays (a female hire may); comma separated. |
| `<Emotes><Idle><MaleOnlyClips>` | (blank) | Idle clip ids a female hire never plays (a male hire may); comma separated. |
| `<VerboseLogging>`, `<JitterWindow>`, `<SdcsPitchPin>`, `<PlayerCtrlOwnController>` | | Diagnostics and engine switches; leave as shipped. `<VerboseLogging>` adds the per-survivor lines that repeat every second (heavy in a town); the in-game Mod Debug Logging toggle logs events only. |

Set elsewhere: wild and hired survivors walk at the same pace (`MoveSpeed` 1.5 on `npcSdcsBase` in `Config/entityclasses.xml`); about half as many survivors spawn in the open world as in 3.3.02 (`Config/entitygroups.xml`), and an
unhired survivor loots at most one container a minute (`Config/buffs.xml`, `SdcsLootWildCooldown`), taking every item tagged `medical`,
`ammo`, `food`, `drinks` or `csm` (skill magazines) plus anything worth 100 or more, and leaving the rest (the wild loot action in `Config/utilityai.xml`,
`listedTags` and `minValue`). Survivors ignore a zombie that is still
asleep, and an unhired survivor carries on with what they are doing until a zombie actually threatens them.

**Choosing the armour survivors wear.** A survivor wears a full armour set: every armour group (vanilla or from a mod pack) that has a
head, chest, hands and feet piece. To keep a whole set off survivors, add its group name (the item's `ArmorGroup`, e.g.
`groupPreacher`) to `<SdcsArmorExcludeGroups>`; to keep one piece off, add the item name to `<SdcsArmorExcludeItems>` (a set with no
other piece for that slot drops out). Armour made for one sex is worn by that sex only. It is detected from the item's SDCS `Prefab`
path: a path with the `{sex}` token (all vanilla gear) fits both; otherwise `female` in the path or the item name means women only, and
`male` means men only. Where the guess is wrong, name the group or item in `<SdcsArmorFemaleOnly>` or `<SdcsArmorMaleOnly>` (an item
name beats its group; a name in both lists counts as female). A one-sex piece that is one of several choices for its slot is just
skipped for the other sex. At game start `Player.log` prints `[SDCS-LOOK] armour pool: <n> sets (<f> female-only, <m> male-only,
<u> unisex), <x> excluded`; with `<VerboseLogging>` on it also names each one-sex set and the piece that decided it. A survivor already
wearing a set the lists now exclude is re-dressed the next time they load. In multiplayer every machine, server included, needs the same
lists.

## Stories ("tell me a story")

Each character's stories live in `personalities.xml`, inside their `<Personality>`:

```xml
<Stories>
    <Story>
        <Text lang="en">I found a working radio once. Spent three days trying to reach someone.</Text>
        <Text lang="de">Ich habe mal ein funktionierendes Funkgerät gefunden. Drei Tage lang habe ich versucht, jemanden zu erreichen.</Text>
    </Story>
</Stories>
```

When the player asks for a story ("tell me a story", or any alias of a `CustomDialog` trigger with an empty `<Response>`), the character
picks one of theirs at random. The `en` text is what they speak; the subtitle shows the player's language (`lang` is one of the
12 subtitle codes: en, de, es, fr, it, ja, ko, pl, pt, ru, tr, zh), falling back to `en`. The older plain form
`<Story>text</Story>` still works, English only. A character with no stories tells their `<Backstory>` instead.

The `<Stories>` block in `phrasetriggers.xml` is not read: stories belong to each character.
