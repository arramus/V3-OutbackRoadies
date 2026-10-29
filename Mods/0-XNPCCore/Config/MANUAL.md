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
- **Optional: ForceLanguage** — Set `<ForceLanguage>ja</ForceLanguage>` (or any ISO code) to force this NPC's subtitle language regardless of the player's game language setting.
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
| `BaseRepairHere` | Yes | The base keeper's standing duty: keeps the leader's base repaired, most damaged block first (SdcsLootMode 8). Voice: "base repair", "repair the worst first"; wheel: Actions > Base Repair |
| `CombatRepairHere` | Yes | The same standing duty, a block damaged in the last few seconds first (SdcsLootMode 9). Voice: "combat repair", "repair under fire"; wheel: Actions > Combat Repair |
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

**Pages.** The wheel always opens on page 0:

| Page | Sockets |
|------|---------|
| **0 (root)** | `Follow`, `Stay`, `Guard` commit directly; `Orders`, `Actions`, `Stance`, `Position`, `Formation` open a drill-down page. Only the `Actions` page (Harvest / Mine) and the `Stance` page (Weapons) open a third level - every order is one, two or three clicks deep |
| **Orders** | `Crouch` (the 0/1 toggle - Crouch / Resume resolved from her state, exactly as before), `Stand up` (the voice `stand up`, SetCrouch arg 2), `Scout` (the voice wander order), `Guard and return` (guard the spot, then come back to you), `Record patrol` / `Run patrol` / `Cancel patrol` (the patrol trio - one survivor's route; a click while held goes to the target, not the squad) |
| **Actions** | `Follow + loot` (starts **follow and loot**, exactly the voice "loot" command - see *Hired survivors: follow and loot* below), `Loot here`, the `Harvest` and `Mine` drill-downs (the wood / plants / crops / butcher orders and the mine / clay orders), and the two base-keeper duties `Base Repair` (most damaged block first, the wheel arg `baserepair`) and `Combat Repair` (a block damaged in the last few seconds first, the wheel arg `combatrepair`). **Look first, then open the wheel:** `Go there` goes to the spot under your crosshair *when the wheel opened*, not where you aim when you pick it. |
| **Stance** | `Aggressive` (1), `Defensive` (0), `Hold fire` (2), `Disengage` (3) - the four SetEngage args verbatim - plus the `Weapons` drill-down |
| **Weapons** | `Guns`, `Rifles`, `Shotguns`, `Machine guns`, `Melee`, `Blades`, `Empty hands` - the voice weapon-category swaps, the same resolved swap as speaking them (she equips the first weapon of that category she actually carries, and says so; a click while held goes to the target, not the squad) |
| **Position** | `Close` (1 m), `Normal` (2.5 m), `Far` (5 m) - the real metres are drawn in the icon slot - and `Point` (the voice "take point": the companion leads 2 m ahead of you) |
| **Formation** | The 8 compass bearings, N at top and E to the right. Direction only: each NPC keeps its own distance |

**Commit vs navigate.**
- **Release the key** while hovering a commit socket - the order goes to the **targeted companion**.
- **Click** while still holding the key - the order goes to the **whole squad** (every formation socket is squad-capable; a non-squad socket falls back to the target, so a click always does something).
- **Navigation sockets** (the drill-downs) never commit. They are marked with a small chevron and the navigation border tint. Open one by clicking it, or simply by **resting the cursor on it for the dwell** (0.4 s default) - a faster pass over it does nothing, so a transit can never flip the page by accident.
- **Back out:** rest the cursor on the wheel's **centre** and it returns to page 0. Clicking or releasing over the centre does nothing.

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

### Formation, Follow & Movement Settings (root level)

| Setting | Default | Description |
|---------|---------|-------------|
| `<FollowMinSeparation>` | `1.0` | Personal-space floor for the follow servo. NPC backs off when leader closes within this distance. |
| `<RecoveryMinDist>` | `1.5` | Below this distance to its target the NPC never hands movement to A* (a few decimetres of shuffle is not a path). |
| `<TrailHandoverDist>` | `4.0` | Beyond this, a following NPC walks your breadcrumb trail instead of aiming straight at you. Must stay well above FollowMinSeparation. Also controls the far-follow handover to A* travel: `PathHandoverDist` has no key and is derived from this value (S6, 2026-09-05). |
| `<TrailVerticalDist>` | `2.0` | Also uses the trail when this far above or below you (catches switchback stairs). |
| `<FollowCatchupCeiling>` | `2.0` | Catch-up speed ceiling, as a multiple of run speed; applies only when more than 8 m behind, so close formation keeps normal pace. |
| `<FollowServoNearCeiling>` | `1.25` | Near-range direct-servo speed ceiling (multiple of run speed); also governs formation slots. Leave at 1.25 unless you mean it. |
| `<FollowPathNearCeiling>` | `1.4` | Speed ceiling on the A* path band below 8 m; ramps to FollowCatchupCeiling by 30 m. Never 1.25: that re-breaks catch-up. |
| `<RescueMinSeparation>` | `2.0` | Hard floor: minimum distance from the leader a rescue teleport may land. Without it rescues land on you. |
| `<FollowArriveY>` | `3.0` | Vertical arrival threshold: this far above or below the leader still counts as "arrived" horizontally. |
| `<VerticalStrandSeconds>` | `6` | Seconds of no vertical progress before a stranded NPC is rescued. |
| `<DropCheckAhead>` | `1.0` | Metres ahead the NPC scans for ground before each move. |
| `<DropCheckDepth>` | `2` | Deepest drop in blocks the NPC will take on its own (2 = it can never injure itself by its own choice). |
| `<MaxFollowDrop>` | `4` | The one exception: how much deeper the NPC follows when you just dropped (jump/fall), detected via a 2 s high-water mark. Also controls the trail hop down budget: there is no separate hop-down key (removed S6, 2026-09-05), so this value is the only knob for how deep a hop may fall. |
| `<LeaderDropGraceSeconds>` | `5` | How long the MaxFollowDrop allowance stays active after your descent. |
| `<TrailHopMaxHoriz>` / `<TrailHopMaxClimb>` | `2.5` / `1.6` | When the trail leads to a crumb the NPC cannot walk to, it hops (short teleport) to it within these limits. The down budget has no key (removed S6, 2026-09-05): it follows `<MaxFollowDrop>` - the NPC must not hop a fall it would refuse to walk. An old `<TrailHopMaxDrop>` key in a user config is silently ignored. |
| `<PathFirstFollow>` | `true` | Far-from-leader travel is A*-first; set false for the old servo-first behaviour (A* only on failure). |
| `<PathReleaseDist>` | `3.0` | Release side of the horizontal hysteresis pair for the path/servo handover (engage side has no key). `PathHandoverDist` is derived from `<TrailHandoverDist>` at load (removed S6, 2026-09-05); an old `<PathHandoverDist>` key in a user config is silently ignored. |
| `<PathVerticalDist>` / `<PathVerticalRelease>` | `2.0` / `1.0` | Vertical hysteresis pair for the path/servo handover. |
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

A hired survivor can loot while she follows you. Start it from the order wheel (**Actions > Loot**) or by
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
| `<Progression>` | | Hired survivors level up: `<HealthPerLevel>` (0.03), `<DamagePerLevel>` (0.02), `<SpreadPerLevel>` (0.02, tighter aim per level), `<MaxLevel>` (10), `<HealthRegenPerSecond>` (0.2). |
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

## Stories (Custom Dialog Pool)

In `phrasetriggers.xml`, the `CustomDialog` action with an empty `<Response>` block picks from a `<Stories>` pool:

```xml
<Stories>
    <Story>I found a working radio once. Spent three days trying to reach someone. Finally got a signal, turned out to be a pre-recorded loop from a station that went dark in year two.</Story>
    <Story>There was a library on the east side. Spent a whole winter there. Warmest I've been since everything went sideways.</Story>
</Stories>
```

Stories are picked randomly when the player asks the NPC to "tell me a story" (or any alias that maps to a `CustomDialog` trigger with empty `<Response>`).

**Note:** Stories are English-only. There is no `lang` attribute support for `<Story>` elements — all stories display in English regardless of the player's UI language.
