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

Any other value (e.g. the retired `sherpa`) loads with a warning and is normalised to `supertonic`, so old configs never crash. Deleting the line gives the same result (the code default is `supertonic`); the shipped `modconfig.xml` ships `audiocpp`.

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
- **Optional: VoiceMode** — Set `<VoiceMode>clips</VoiceMode>` to use pre-recorded OGG clips instead of TTS. See [Voice Clips Mode](#voice-clips-mode) below.
- **Optional: ForceLanguage** — Set `<ForceLanguage>ja</ForceLanguage>` (or any ISO code) to force this NPC's subtitle language regardless of the player's game language setting. Every caption she shows follows it: her replies, order acknowledgements and canned lines ("I'm done crafting."). An item named in a line ("I need Wood for that.") keeps the name the player's game shows.
- **Optional: Subtitles** — What text this character shows on screen: `all` (every line: replies, chat, order acknowledgements, clip captions), `clip` (only a voice clip's own caption, never the generic reply text) or `none` (no text at all). Omit it for the default: `clip` for a clips character, `all` for everyone else.

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
| `Pickup` | Yes | Picks up the NPC (SCore PickUpNPC) |
| `OpenInventory` | Yes* | Opens NPC's trade/inventory window. *Traders are the one exception: opening a trader's inventory needs no leader (it redirects to the trade dialog). |
| `SwapWeapon` | Yes | Swaps NPC weapon. See SwapWeapon section below. |
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
| `0` | Cover Me (defensive, default) | Fights what threatens the leader or a squad mate, and anything within the contact radius (`EngageContactRadius`, default 2.5 m) that is literally on top of you. May walk toward an already-authorised enemy within the 8 m advance leash, but holds position instead of chasing. |
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

Hold the radial key (default **Semicolon**) to open the order wheel - a multi-page radial, not the old single 8-socket formation picker. It is a menu feature: it works with STT/TTS/LLM all off.

**Targeting.** The wheel commands the companion you are looking at, otherwise the last one you gave an order to, otherwise the nearest. Their name sits in the middle of the ring and the mouse wheel cycles through your hires.

**From the E ring.** Hold **E** on one of your hires and the ring shows **Orders** next to Talk. Keep holding E and click Orders (or rest the cursor on it): the order wheel opens on her and E works like the radial key until it closes - release E over an order to give it to her, click while holding for the whole squad. If you let go of E on Orders instead, the wheel stays open and your next E press picks. Only her leader sees the entry. (The game's own **remove** entry still shows only with the debug menu on.)

**Pages.** The wheel always opens on page 0:

| Page | Sockets |
|------|---------|
| **0 (root)** | `Follow`, `Stay`, `Guard` commit directly; `Jobs`, `Orders`, `Stance`, `Position`, `Formation` open a drill-down page. Only the `Jobs` page (Harvest / Mine), the `Stance` page (Weapons) and the `Orders` page (Roles) open a third level - every order is one, two or three clicks deep |
| **Orders** | `Crouch` (the 0/1 toggle - Crouch / Resume resolved from her state, exactly as before), `Stand up` (the voice `stand up`, SetCrouch arg 2), `Scout` (the voice wander order), `Guard and return` (patrol the spot: walk out within `<Guard><PatrolMetres>` (6 m), look around, come back, face your way, repeat; fight freely, then back to the spot), `Record patrol` / `Run patrol` / `Cancel patrol` (the patrol trio - one survivor's route; a click while held goes to the target, not the squad), `Roles` (a drill-down: the role toggles, below) |
| **Roles** | One survivor's roles, each a toggle that reads **Stop ...** while she has it (target only, never the squad): `Medic` (the designated medic role), `Cook` (the campfire), `Blacksmith` (the forge), `Chemist` (the chemistry station), `Crafter` (the workbench and cement mixer). A survivor uses a workstation only with its role - see *Hired survivors: crafting* |
| **Jobs** | `Follow + loot` (starts **follow and loot**, exactly the voice "loot" command - see *Hired survivors: follow and loot* below), `Loot here`, the `Harvest` and `Mine` drill-downs (the wood / plants / crops / butcher orders and the mine / clay orders), the two base-keeper duties `Base Repair` (most damaged block first, the wheel arg `baserepair`) and `Combat Repair` (a block damaged in the last few seconds first, the wheel arg `combatrepair`), and `Remember base` |
| **Stance** | `Aggressive` (1), `Defensive` (0), `Hold fire` (2), `Disengage` (3) - the four SetEngage args verbatim - plus the `Weapons` drill-down |
| **Weapons** | `Guns`, `Rifles`, `Shotguns`, `Machine guns`, `Melee`, `Blades`, `Empty hands` - the voice weapon-category swaps, the same resolved swap as speaking them (she equips the first weapon of that category she actually carries, and says so; a click while held goes to the target, not the squad) |
| **Position** | `Close` (1 m), `Normal` (2.5 m), `Far` (5 m) - the real metres are drawn in the icon slot - `Point` (the voice "take point": the companion leads 2 m ahead of you), `Go there`, `Jump down`, `Get in` / `Get out` (your vehicle). **Look first, then open the wheel:** `Go there` goes to the spot under your crosshair *when the wheel opened*, not where you aim when you pick it. |
| **Formation** | The 8 compass bearings, N at top and E to the right. Direction only: each NPC keeps its own distance |

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

Every socket carries the exact router arg of the equivalent voice command, so the wheel and the voice path cannot disagree - the one deliberate exception is the crouch toggle on the Orders page, which offers Crouch **or** Resume depending on her current state (arg 1 or 0) instead of one fixed arg.

### Order Wheel settings (modconfig.xml, top-level `<FormationRadial>` section)

| Setting | Default | Description |
|---------|---------|-------------|
| `<Key>` | `Semicolon` | Unity KeyCode name that opens the wheel. Every letter A-Z has a vanilla default binding, so letters collide. Free keys: Semicolon, Quote, Comma, Period, Slash, RightBracket, Backslash. Empty value disables the wheel. An old `<STT><FormationRadialKey>` entry is still read as a fallback. |
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

Create a custom voice with the voice builder tool, add the resulting JSON to `Resources/Supertonic/voice_styles/`, and register an alias in `SupertonicConfig.xml`:
```xml
<Voice alias="my_voice" file="my_voice.json" />
```

Voice aliases must match an entry in the `SupertonicConfig.xml` VoiceMap. If a personality names a voice that is not there, it does not resolve - the mod warns loudly (as above) rather than silently giving the NPC another NPC's voice.

---

## modconfig.xml — Full Settings Reference

All runtime settings live in one file. Most can be adjusted in the in-game config menu (NPC VoiceControl Settings). Changes that take effect on next launch are noted in the UI.

### TTS Section

| Setting | Default | Description |
|---------|---------|-------------|
| `<Enabled>` | `true` | Turn TTS on/off globally |
| `<TtsEngine>` | `audiocpp` | TTS engine: `audiocpp` (default since 3.2.00, port 5057) or `supertonic` (port 5054). Any other value (e.g. the retired `sherpa`) logs a warning and is normalised to `supertonic`, which is also the code default when the line is deleted. |
| `<TimeoutSeconds>` | `30` | How long to wait for TTS response before giving up |
| `<UseGpu>` | `false` | Legacy flag, no longer used. Kept so old configs load cleanly |
| `<Volume>` | `0.8` | Playback volume (0.0-1.0) |
| `<MaxDistance>` | `40` | Maximum distance NPC voice can be heard (meters) |
| `<MinDistance>` | `2` | Minimum distance for volume falloff (meters) |
| `<SpeechRate>` | `1.0` | Playback speed multiplier (0.5-2.0) |

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
| `<BeamSize>` | `1` | Recognition accuracy (1-5, higher = more accurate but slower) |
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
| `<MaxTokens>` | `200` | Maximum response length in tokens (64-2048). Use 1024-1536 for reasoning models. |
| `<Temperature>` | `0.8` | Randomness 0-1 (higher = more creative, lower = more focused) |
| `<SystemPrompt>` | *(see config)* | Base personality injected into every NPC conversation |
| `<ContextMemory>` | `5` | Number of exchanges kept in conversation history (3-20) |
| `<ShowTypingIndicator>` | `true` | Show "typing..." indicator while waiting for response |
| `<TypingDelayMs>` | `75` | Per-character delay for typewriter effect (50=fast, 75=natural, 150=dramatic) |
| `<MaxResponseLength>` | `300` | Character cap on NPC responses. 200-400 is the sweet spot. |
| `<ChatTimeout>` | `30` | Chat inactivity timeout in seconds before conversation resets |
| `<ContextSize>` | `8192` | KV cache window in tokens. 4096 for low-VRAM (6 GB), 8192 default (8 GB+), 16384 for 12 GB+, 32768 for 16 GB+ |
| `<WildCombatChatCooldownSeconds>` | `5` | Seconds a wild NPC stays uninterruptible after its last combat target dies |

### Prompt Section

| Setting | Default | Description |
|---------|---------|-------------|
| `<BasePrompt>` | *(see config)* | Fallback system prompt if SystemPrompt above is empty |
| `<TtsRule>` | *(see config)* | Injected into prompt when TTSLanguage is Japanese — forces native script output |
| `<ChineseTtsRule>` | *(see config)* | Same as TtsRule but for Simplified Chinese — forces Hanzi output |

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
| `<LadderOrderedUseAstar>` | `true` | INERT key (nothing reads it any more): the ladder climb hands A* one exact path unconditionally. The loader still reads the value in case the flag is wired back. |
| `<PatrolAStarLeg>` | `true` | Recorded-patrol legs between waypoints are A*-owned. |
| `<ChatFollowEscapeDist>` | `6.0` | While mid-conversation the NPC holds its ground within this distance and resumes following beyond it. |
| `<JumpDownWaitSeconds>` | `10` | After you drop further below the NPC than MaxFollowDrop, it waits at the lip this long before the teleport backstop re-arms (you hold the JumpDown command; you decide). |
| `<TeleportMinGain>` | `1.5` | Refuses teleports that would move the NPC less than this (no-op guard). |
| `<EngageContactRadius>` | `2.5` | Defensive (Cover Me) also engages an enemy this close to you even if it targets nobody. |
| `<DisengageWindow>` | `20` | Seconds a "disengage" NPC keeps self-defending against a live enemy but walks away the moment it is down. |
| `<DefaultTacticalMode>` | `false` | If true, new hires start in tactical (command-only) mode |
| `<FormationDistances/Tier>` | `1=2m, 2=5m` | Slot distances keyed by index. Add tiers with no code change. |

The breadcrumb trail itself is recorded from your own movement (`TrailRingSize` 64 crumbs, `TrailSpacing` 1.0 m, `TrailSampleInterval` 0.1 s, `TrailRosterInterval` 2.0 s, `TrailSnapEpsilon` 0.2, `TrailSnapMaxDrop` 1.2, `TrailRecordMaxScan` 3, `TrailMaxAgeSeconds` 180). Tune those only with the log open - the inline comments in `modconfig.xml` carry the reasoning.

---

## Hired survivors: follow and loot

A hired survivor can loot while she follows you. Start it from the order wheel (**Jobs > Follow + loot**) or by
voice (**"follow and loot"**, **"loot around here"**, **"scavenge here"**). Her order stays **Follow**; looting is a mode on top of it.

- **When she loots:** only while you stand still (about a second without moving). While you walk she just follows; a trip she has already
  started is finished.
- **What she looks at:** containers within **15 m** of you (and within 3 m up or down), in the building you are in. Loose piles first (ammo,
  medical, books, food, chem, liquor), then the rest, nearest first. She takes everything: usable items to her bag (up to its caps), the rest to
  her harvest window.
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

Ask a hired survivor to make something: **E** on her, **Craft...**, pick a category, then a recipe, then how many (1, 5, 10 or
**All**). The categories are your crafting window's own (Basics, Resources, Tools/Traps and so on; at a station, that station's,
such as the campfire's Food and Drink): **All** first, then only the ones she can make something in now, each with how many
recipes it holds. **Back** on a recipe page returns to the categories. The list shows only
what she can make right now, with the most she can make of each. A station recipe also needs the station's tool: at a
campfire most recipes need a cooking pot (some a grill or a beaker) in the campfire, so a bare campfire offers only
charred meat (and baked potato once she has learnt it). A tool she carries (her pack, or a chest of yours in reach) counts:
she puts it into the station's own slot when she crafts, and it stays there. Each station she can make something at has its
own tab, listed first (**Campfire**, **Forge**), with every recipe she knows at that station and how many she can make now. One
she can't make yet shows `(0)`: pick it and she says what it needs (an ingredient, or the station's tool). A new Cook knows only
the basics (boiled water, charred meat and a few more); she learns the rest by cooking. Two recipes for the same item (Cloth Fragment from cotton or from yucca)
also show their main ingredient.

- **What she can make:** the recipes you can make in your own backpack, and the recipes of a workstation you stand at (below), and only
  the ones she has unlocked herself: the open recipes from the start; one that needs a schematic or a perk only once she has it,
  or once her crafting skill reaches it (*Learning her craft*, below). The
  server's crafting options apply to her too: backpack and workstation crafting, the highest tech tier, biome progression.
- **Materials:** shared (broadcast) storage inside your land claim when she works at your base, then any unlocked chest you or one of
  your allies placed within **15 m** of her spot (never another player's), then her own stores: her harvest window first (her main supply backpack, so hand her materials there), then her
  toolbelt, bag and pack. Never a stack with mods installed.
- **Cost, time, quality:** what a player pays with no crafting perks: the recipe's ingredients and time, scaled by the server's crafting
  settings. The quality is what her crafting skills give for that recipe, as yours are (she raises them by crafting at her role's
  stations: *Learning her craft*, below); 6 for everyone when crafting progression is off.
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
`<Crafting><StationReachMetres>`) and open **Craft...**: its recipes join the list, named with the station ("Forge: Forged Iron (4)").
She uses the station the way you do. Every survivor crafts from her backpack anywhere; a workstation needs her role for it and a
land claim (a campfire works anywhere: it is a field station).

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
- **Where she works:** she walks to the nearest spot beside the station she can stand on and kneels there, facing it. If every
  side is blocked she works from where you gave the order; if she cannot get there within 10 s, from where she got to.
- **When she is done:** she stays at the station until you give her an order (**Follow me**), walking around it within 5 m
  (`<Crafting><StationWanderMetres>`; 0 = she stands at her spot beside it, facing it). A hire you tell to stay within that distance
  of any workstation walks around it the same way. A backpack order always brings her back.
- **The order:** she takes every ingredient at once (smelted units from the forge, the rest from her stores and your chests, as above)
  and queues the whole count in the station's queue as hers, as your craft does; the station makes them at its own pace. Every queue
  full: "That station is busy." and nothing is taken.
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
  out of the queue and gives their ingredients back (units into the forge, the rest into her harvest window), and her ore the forge has not melted yet. If she dies or the game
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
- **E > Your progress** shows each role she holds: "Cook skill 2: 20 / 30 crafts" (her skill, her crafts, the crafts the next level takes).
- Her level still earns the game's crafting XP, as before.

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
stand in steps 2 m aside, away from you (`<Combat><ClearShotStepMetres>`), then shoots.

**Starting guns and ammo.** A tier 1 survivor starts with a melee weapon in her hand and an empty pipe pistol in her pack:
she gets no ammo for it when she appears, not even ammo from her own trade goods, which stay for sale. Ammo she loots later,
or that you give her, is hers, and then she uses the pistol. Tier 2 and 3 survivors start with ammo for the gun they roll.

## Bandits

Two hostile test characters in the spawn menu (no spawn lines yet): `npcSdcsBanditMelee`, a tier 1 male with a tier 1 melee
weapon, and `npcSdcsBanditRanged`, a tier 1 female with a pipe pistol or a pipe rifle. Each looks like a tier 1 survivor and
carries 60 rounds for her gun.

- **They fight** you, your survivors, zombies and hostile animals on sight; passive animals are left alone. A bandit runs at a
  target she sees but cannot reach with her gun yet, and walks toward a player she hears but cannot see (crouch to stay quiet).
- **They don't talk:** no E prompt, no hire, no trade, no voice.
- **Loot:** dead, she always drops the game's plain zombie loot bag. Per class in `entityclasses.xml`: `LootDropEntityClass` and
  `LootDropProb` (the same properties as on any zombie; the blue and red bags are `EntityLootContainerStrong` and `...Boss`).
- **Damage to players:** `<Bandits><PlayerDamage>` in Settings.xml (half by default). In the console, `sdcsbandit` shows it and lists
  the bandits in the world; `sdcsbandit damage 0.75` changes it at once, until the game restarts.
- **Aim:** a bandit shoots like a survivor of her level (see *Survivors' aim*). `<Bandits><SpreadByTier>` can widen it by tier
  (0 by default).
- **One target at a time:** she keeps fighting the target she has until it is dead or out of her sight (the class property
  `SdcsKeepTarget`, 0.7: any other target scores that much of hers).
- **She hunts:** if she loses sight of the enemy she is fighting, or of whoever just hurt her, within 40 m, she goes after it
  until she sees it again (instead of wandering off or standing still).
- **XP:** 600 each. Your own custom characters never spawn as bandits.

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
| `<SleeperTrigger>` | `true` | A survivor walking into a building's sleeper volume wakes / spawns it as a player would (only while a player is within 96 m). `false`: survivors never trigger sleepers. |
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
| `<Crafting><StationWanderMetres>` | `5` | After a workstation order, and when told to stay this close to one, she walks around the station within this many metres until you give her an order. `0` = she stands at her spot. Clamped 0 to 20. |
| `<Crafting><StationTools>` | the campfire's and forge's | Each workstation's tool slots in the game's own order (`area=tool,tool,tool`, stations separated by `;`): a tool she carries goes into its own slot when she crafts. A station or tool not listed: she never puts one in. |
| `<Sleeper><SightRangeMetres>` | `16` | How close a sleeping zombie must be to see one of your survivors. Clamped 4 to 29. |
| `<Sleeper><LeaveBeyondMetres>` | `30` | Once a house wakes on a wild survivor, how far past it she runs before she counts as away. Clamped 10 to 100. |
| `<Sleeper><HouseAvoidSeconds>` | `300` | How long she then stays out of that house (and away from a container she gave up on). Clamped 30 to 3600. |
| `<Vehicle><GunnerMaxRangeMetres>` | `40` | The farthest a seated gunner fires; each gun also stops at its own effective range (a pistol about 17 m). Clamped 5 to 150. |
| `<Vehicle><GunnerIdleHolsterSeconds>` | `5` | How long she keeps the gun out after the last enemy is gone. Clamped 0 to 60. |
| `<Vehicle><BoardRangeMetres>` | `30` | How far from the vehicle a hire walks over to get in. Clamped 5 to 100. |
| `<WorkSite><HarvestRadius>` | `20` | How far from her post the chop-wood, mine and pick scans reach. Clamped 4 to 40. |
| `<WorkSite><ClayRadius>` | `8` | The same for digging clay (clay is everywhere, so it stays smaller). Clamped 4 to 40. |
| `<WorkSite><MineVeinDepthLayers>` | `2` | How many layers of an ore vein she mines, down from the surface as she first saw it. Clamped 1 to 10. |
| `<WorkSite><SwingBudgetSeconds>` | `60` | How long she swings at one tree, rock, plant or corpse before skipping it. Clamped 5 to 600. |
| `<WorkSite><WalkBudgetSeconds>` | `15` | How long she walks to one target before skipping it (harvest, butcher, base keeper). Clamped 5 to 120. |
| `<BaseKeeper><DefaultMode>` | `base` | What a plain "repair the base" order fixes first: `base` (the most damaged block) or `combat` (a block damaged in the last few seconds). |
| `<BaseKeeper><RescanSeconds>` | `30` | How often she rescans the base for new damage. Clamped 5 to 300. |
| `<BaseKeeper><AreaRadiusMetres>` | `8` | How far her worked area reaches; two keepers stay this far apart. Clamped 2 to 40. |
| `<BaseKeeper><UnderAttackSeconds>` | `5` | In combat repair, a block hit within this many seconds comes first. Clamped 1 to 60. |
| `<BaseKeeper><ChestReachMetres>` | `15` | How far a chest (not broadcast storage) may sit from her post and still feed repairs and crafting. Clamped 2 to 40. |
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
| `<Medic><InCombatHealthPercent>` | `50` | The medic treats a patient below this health while an enemy is visible. Clamped 1 to 99. |
| `<Medic><AfterCombatHealthPercent>` | `50` | The same line once the fight is over. Clamped 1 to 99. |
| `<Medic><SelfHealHealthPercent>` | `70` | A survivor heals herself below this health. Clamped 1 to 99. |
| `<Medic><DireHealthPercent>` | `25` | The dire line below it (the first aid kit and pain pills). Clamped 1 to 99, below the self-heal line. |
| `<Medic><SelfHealCooldownSeconds>` | `30` | The wait between low-health self-heals. |
| `<Medic><BleedRecheckSeconds>` | `10` | The wait before a new bleed can start another treatment. |
| `<Combat><WildTargetedRadiusMetres>` | `15` | How far away a zombie that targets a wild (unhired) survivor still makes her fight it. Clamped 1 to 60. |
| `<Combat><CloseMeleeEnterMetres>` | `3` | A survivor holding a ranged weapon swaps to melee at or below this distance. Clamped 0.5 to 20. |
| `<Combat><CloseMeleeExitMetres>` | `5` | She swaps back to the ranged weapon beyond this distance (it must be above the enter distance). |
| `<Combat><CloseMeleeMinHoldSeconds>` | `3` | The least time on the melee swap before she swaps back. Clamped 0 to 30. |
| `<Combat><CrawlerAimHeadMetres>` | `0.45` | Dedicated server: where she aims at a crawler's head, above its root. Clamped 0.05 to 1.5. |
| `<Combat><CrawlerAimBellyMetres>` | `0.25` | The same for its belly (it must be below the head height). |
| `<Combat><ClearShotStepMetres>` | `2` | A hire with a gun steps this far aside, away from you, when you stand in her line of fire to an enemy that is going for you, then shoots. Clamped 1 to 5. |
| `<Combat><ClearShotLeaderRadiusMetres>` | `20` | ...for an enemy within this distance of you. Clamped 2 to 60. |
| `<Follow><PaceNearDistMetres>` | `8` | At or below this distance from you she walks. Clamped 1 to 60. |
| `<Follow><PaceFarDistMetres>` | `10` | At or above it she moves at full speed; between the two her speed blends (it must be above the near distance). |
| `<Follow><PaceNearWalkSpeedMps>` | `2` | Her walking speed near you, in metres per second (0.5 up to below 2.75, where the run starts). |
| `<Loot><BagMaxMedical>` | `5` | Medical items she keeps in her own bag from looting; the rest goes to her harvest window. Clamped 0 to 100. |
| `<Loot><BagMaxAmmoStacks>` | `2` | Full stacks of each carried gun's ammo she keeps in her bag. Clamped 0 to 20. |
| `<Loot><VerticalReachMetres>` | `3` | How far above or below you she loots (one storey). Clamped 1 to 20. |
| `<Loot><ReturnTeleportMetres>` | `40` | After "loot around here" ends: within this distance she walks back to you, beyond it she teleports. Clamped 5 to 44. |
| `<Ammo><RestockTarget>` | `120` | Inside your (or an ally's) land claim, a hired survivor whose spare rounds for her gun drop below one magazine tops up to this many from the base: broadcast containers anywhere in the claims and your unlocked chests within `<BaseKeeper><ChestReachMetres>` of her, never her own pack or another player's box. `0` turns it off. Clamped 0 to 5000. |
| `<Ammo><RestockCheckSeconds>` | `5` | How often she checks her ammo. Clamped 1 to 120. |
| `<Ammo><RestockRetrySeconds>` | `30` | After a check that took nothing (no ammo in the base, or she is outside it), the wait before the next. Clamped 5 to 600. |
| `<Leadership><RegenPerSecond>` | `0.5` | Your Charismatic Nature rank 1 and up: health your hired survivors regain per second. Clamped 0 to 5. |
| `<Leadership><DamageBonus>` | `0.2` | Rank 2 and up: the extra damage they deal (0.2 = 20%), to blocks too with a melee weapon. Clamped 0 to 1. |
| `<Leadership><DamageResist>` | `0.1` | Rank 3 and up: the share of damage they do not take (0.1 = 10%). Clamped 0 to 0.9. |
| `<Bandits><PlayerDamage>` | `0.5` | The damage a bandit deals to PLAYERS, as a multiplier on her weapon's (1 = full, 0.5 = half); survivors, zombies and animals take full damage. Change it while playing with `sdcsbandit damage <x>` (lasts until the game restarts). Clamped 0 to 5. |
| `<Bandits><SpreadByTier>` | `0,0,0` | An extra spread for a bandit by her tier (T1, T2, T3), added to her hip spread on top of the aim every survivor of her level has (see *Survivors' aim*): 0 = she shoots like a survivor of her level, 1 = one more hip spread of her gun. Clamped 0 to 10 each. |
| `<Progression><KillXpWorkaround>` | `true` | Works around an SCore error on NPC kills that also costs the player the kill's XP; leave on. |
| `<AwakeStealth>` | `true` | An awake zombie that is after a survivor sees her through the player's own stealth test: crouching, a held light and speed count, as they do for you. |
| `<HitBoxFix>` | `true` | Keeps the oversized invisible box around survivors, NPCs and pets switched off, so hits land on the body. `false`: the game's behaviour (hits up to 30 cm off the body). |
| `<SdcsArmorExcludeGroups>` | `groupPreacher` | Outfit groups (comma separated) never rolled for an NPC, e.g. coats whose cloth only animates on players. |
| `<SdcsCustomCharacterChance>` | `0.1` | Chance that a new wild survivor is dressed as one of an online player's own creator characters of the same sex; `0` turns it off. |
| `<NpcPathRules>` | `true` | Survivors plan their paths with the mod's own rules (the hazard and drop settings below). `false`: the game's own paths. |
| `<NpcPathSpikeCost>` | `1000000` | Path cost of a spike / barbed wire / cactus cell: crossed only when there is no other way. |
| `<NpcPathHazardMargin>` | `4000` | Extra path cost next to a hazard, so paths keep one cell off spikes where there is room. `0` disables it. |
| `<NpcPathMaxDropBlocks>` | `5` | A path never drops more than this many blocks in one step. `0` disables the cap. |
| `<NpcHazardWalkSpeed>` | `1.0` | Speed cap (m/s) while a hazard is right next to her feet. `0` disables it. |
| `<NpcHazardStepOut>` | `true` | A survivor standing in contact with a hazard steps to the nearest safe spot. |
| `<NpcHazardFirstTouchDamage>` | `1` | Damage of her first brush against a hazard; a second touch within 3 s deals normal damage. `-1` turns the rule off. |
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
