# NPCCore - Changelog

Release notes, newest first. How to install and use the mod is in `ReadMeFirst.md`, next to this file.

## What's New in 3.4.05

### New
- **More gestures.** Hires have five more talk gestures, three more nods, three more head shakes and one more idle (from Mixamo), on SDCS and humanoid characters.
- **More idle moments.** A hire standing around now also looks around, yawns or looks bored (new animations, on SDCS and humanoid
  characters).
- **Choose the armour survivors wear.** New Settings.xml lists exclude armour sets or single pieces (vanilla or from mod packs), and armour made for
  one sex is no longer worn by the other: detected automatically, with female-only / male-only lists to correct it.
- **E reaches your hires on Far spacing.** The E key now reaches your own hires out to 6 m (Settings.xml `<Follow>
  <HireActivateReachMetres>`), so a hire set to Far still opens her menu. Everyone else keeps the game's 3.5 m.
- **Gun flashlights in the dark.** A hire whose gun carries a flashlight mod switches it on at night and in dark rooms,
  keeps it on in a fight, and it goes out when you crouch (she sneaks with you) (Settings.xml `<Light>`).
- **Torches in the dark.** A hire with a torch (or a flashlight) in her bag holds it at night and in dark rooms when her gun has no light,
  puts it away for a fight (back 10 s after), by day, when you crouch, or for the night once you order her a weapon.
  New survivors start with one (Settings.xml `<Light>`). A hire with a one-handed weapon holds the torch in her left hand (her arm
  posed to hold it, the torch upright in her fist, with every one-handed weapon, pistols included) and keeps her weapon, also in a
  fight (Settings.xml `<Light><TorchLeftHand>`; the pose: `<TorchLeftHandTarget>`, `<TorchLeftElbowHint>`, `<TorchLeftGripPosition>`,
  `<TorchLeftGripRotation>`). While she holds a torch she only makes small gestures; an emote you order puts it away until it ends.
- **Dress your hires.** Hold E on a hire, Wardrobe: pick which armour piece from her pack she wears in each slot (it protects her as
  worn armour). Taking it out of her pack puts her own piece back.
- **Power attacks.** A survivor's melee every 3 to 6 swings (random) is a power attack (the weapon's own heavy swing and damage; Settings.xml
  `<Combat><PowerAttackMinSwings>`/`<PowerAttackMaxSwings>`, 0 in either turns it off).
- **POI residents (first step).** POI makers can place friendly survivors with the new sleeper group; each claims a bed and a workstation in her POI.
- **Residents keep a day:** they work at their station by day, go home at dusk and stay home on horde nights
  (hours from the world's dusk and dawn, Settings.xml `<Routine>`; they walk between their places, never leave their POI, and a fight,
  an order or a chat still comes first).
- **Residents sleep on their beds:** at night a resident steps to the floor beside her bed, lies down flat on it (or settles onto a
  bedroll or floor mattress) and sleeps until dawn, then stirs and gets up; with no free bed she sleeps on a bedroll from her pack
  (only a look, gone at dawn). A hostile close by, an attack or an order gets her up at once. Talking to her (E, or by voice) wakes
  her at once: she gets up and answers a couple of seconds later, and lies down again 15 s after the talk if it is still night. Any
  loud noise nearby wakes her too (a gunshot or an explosion, yours included; footsteps do not). Settings.xml `<Routine>`: the bed
  heights, the wake distances, `<TalkWakeSpeakSeconds>`, `<TalkResleepSeconds>`, `<WakeNoiseToWake>`, `<WakeLoudNoise>`. On SDCS and
  humanoid characters, in multiplayer too.
- **Residents work at their station by day:** at her workstation a resident faces it and works it with the same moves a hire
  crafts with (kneeling at a campfire and tossing logs on, hammering and chiselling at a workbench or forge, working a chemistry
  station or cement mixer), with short breaks in between when she idles nearby (Settings.xml `<Routine><WorkSeconds>`,
  `<BreakSeconds>`). She makes nothing and uses no fuel, and a player can use the station as usual. A fight, an order, a talk
  or hiring her stops the work at once; she goes back to it later.
- **Residents move between their POI's stations:** a resident who is the only one working in her POI moves to another free station
  each work spell (campfire, forge, cement mixer, ...; Settings.xml `<Routine><RotateStations>`); with two or more residents each
  keeps her own. She steps right up to the station's front on the floor (never onto a bed, counter or table), stands to work a
  station raised on a counter (`<Crafting><RaisedStationSet>`, per station `<StationPose>` auto / kneel / stand), and the station
  shows as running while she works (a campfire or forge burns, the cement mixer's drum turns; no fuel is used).
- **Residents use doors:** on her way to her bed or station a resident opens a closed (unlocked) door, walks through it (up or down
  a doorstep) and it closes behind her (Settings.xml `<PointWalk>`: `<DoorSeekMetres>` 16, `<DoorSeekSpotMetres>`,
  `<DoorPassMetres>`, `<DoorHoldMaxSeconds>`). She never gets stuck on a bed or other furniture.
- **Residents walk their whole POI.** Now and then, at the start of a break or once in the evening, a resident strolls to another
  part of her POI, lingers a little and walks back (Settings.xml `<Routine>`: `<StrollChance>`, `<StrollMinMetres>`,
  `<StrollLingerSeconds>`; not on horde nights). Her other break wandering stays near her spot.
- **Subtitles stay until she stops speaking.** A spoken line's subtitle now stays on screen while her voice is still playing, and a
  moment after (modconfig.xml `<TTS>`: `<SubtitleWordsPerSecond>`, `<SubtitleMinSeconds>`, `<SubtitleMaxSeconds>`,
  `<SubtitleTailSeconds>`, `<SubtitleHoldCapSeconds>`). Another survivor's voice ending no longer clears her subtitle.
- **More drinks put out a fire.** A hire on fire can also drink coffee, beer, moonshine, Megacrush or Grandpa's drinks to put it
  out (every fire-out drink in game 3.4). Plain water goes first, and the valuable buff drinks only when nothing cheaper is left.
- **Weapon fit for humanoid and VRoid characters.** A character author can now set a weapon's size, angle and position in the hand
  separately from the hand spread (entity class `PlayerCtrlWeaponScale`, `PlayerCtrlGripRotation`, `PlayerCtrlGripOffset`), or place an empty
  named `NpcRightWeapon` under the model's right hand (`PlayerCtrlWeaponSocket` renames it). A gun swapped in mid-game keeps the right size.
  The console command `sdcsgrip` tunes the survivor you look at live (`forward`, `up`, `right` nudge the gun along its own barrel) and prints the XML lines to paste. Guides: the humanoid and VRoid
  author guides.
- **Several blink shapes blink together.** A character author can name every blendshape the model blinks with (an eyelid and an
  eyelash shape, say) in the entity class property `FaceBlinkShapes` (comma separated, exact names, any mesh); they all move
  together, in place of the one shape picked automatically. Needs the voice add-on. Guide: the humanoid author guide, "Several
  blink shapes".

### Fixes
- **An ordered weapon shows before the talk puts it away.** Picking a weapon in a hire's menu while she held her torch put it straight back into her pack. It now shows for a few seconds first, as with any other weapon order.
- **Followers keep out of parked vehicles.** A hire whose follow spot fell against a parked vehicle could wedge into it
  and be teleported into its body, flipping it. Her spot and any rescue teleport now stay outside a vehicle's body.
- **Follow me from the menu ends an old post.** A hire who was told to stay or go somewhere, then told Follow me
  from her E menu, kept standing on the old spot. The menu's Follow me now ends it, like the voice and wheel orders.
- **The melee bandit groups are melee only again.** Their tier 2 and 3 bandits carried guns, and a POI's
  tier raises its bandits to those tiers, so most POI entrances built with the melee group spawned gunmen. They now spawn melee-only
  bandits at every tier.
- **Legacy NPCs no longer get stuck in a vehicle seat.** Legacy (V5) NPCs never take a seat: while you
  drive they are handled the way SCore does it, as before 3.4.03. An NPC already stuck in a seat gets out
  the next time you drive.
- **She talks with her hands through the whole reply.** A new talk gesture starts with each sentence she
  speaks (`<Emotes><Reactions><Talk>` Chance 1, CooldownSeconds 2), not one per reply.
- **Hires stop aiming at the ground after a fight.** A hire kept pointing her gun down at a dead zombie (or
  at one that vanished in a hitch) for a long time; with no live target her aim now stays level.
- **A misnamed SCore folder is reported.** If SCore loads after NPCCore (for example its folder was renamed to `SCore`), the log now
  says so in one line and names the fix, instead of only thousands of XML errors.
- **Patrols walk straight.** A recorded route was stored block by block, so a survivor zig-zagged along any diagonal. The route is now straightened when she starts patrolling (real corners, stairs and ledges kept). `<PatrolStraightenToleranceMetres>` 0 turns it off.
- **Recording a patrol again works.** Recording a new route over ground she had already recorded that session saved no points, so she answered that she had no route. Each recording now starts fresh.
- **A patrol no longer errors when its route changes.** If her route got shorter while she was patrolling, the patrol threw an error every tick until she got another order. She now picks up at the nearest leg.
- **Hires get out clear of the vehicle.** A hire getting out was placed against the vehicle's side, where the vehicle could clip her and fling her.
  She is now placed outside its body (Settings.xml `<Vehicle><ExitClearanceMetres>`, default 1 m).
- **A torch is not a weapon.** A torch or a flashlight in a hire's bag was picked as her melee weapon. Light tools are never picked as weapons now. New survivors start with a torch.
- **Other radial mods work beside the order wheel.** The closed order wheel was updated every frame, which could stop another mod's radial menus (a hover-to-open dance menu, for one)
  from working. It now stays idle while closed.
- **A hire keeps her loaded rounds when you close her pack.** Closing a hire's inventory could hand her gun back empty, and putting it away then saved the empty count over her loaded rounds. The saved rounds now stay; and a torch taken from her hand no longer leaves her stuck in torch mode.
- **Patrolling hires keep their place.** A hire on a patrol (or told to Wander) was pulled to you when you loaded the game or got
  out of a vehicle. She now stays where she was.
- **Hires' crossbows, bows and other launchers hit.** A hire's shots could stop on her own arm and do no damage, and after her
  reload a launcher fired nothing. Both fixed, for every launcher, modded ones included.
- **The heal roll goes somewhere.** A hire who rolls away to bandage now rolls away from the enemy, only where there is room;
  otherwise she bandages where she stands.
- **A read book shows in a humanoid character's hand.**
- **Six old test survivor classes are hidden** from the spawn menu, and their fixed outfit can no longer be swapped out.
- **Survivors no longer plan a path through a thin wall** or pole that the game's path grid misses, and a short walk now ends on
  its spot instead of a metre short.
- **Crouch holds.** A hire ordered to crouch no longer stands back up when an idle or a talk gesture plays; she stays down until
  you order her up.
- **Stand up does what it says.** The Stand up order no longer makes her stand, crouch and stand again. She stays standing until
  you next crouch or stand up yourself, then copies you again as before.
- **Melee enemies no longer hit you from behind.** A melee bandit fighting a zombie could hit a player standing behind him with
  his back turned. A survivor's melee swing now only lands in front of her.

## What's New in 3.4.04

Targets 7 Days to Die **3.4**. **Requires 0-SCore 3.4.14 or newer** (built and tested on 3.4.14.1542). Everyone on a server needs the
same version, server included.

### Chat server update
- **The bundled llama.cpp chat server is updated.** The chat model stays Qwen3-4B. Its thinking is switched off by the new
  `<LLM><Reasoning>` setting (off by default), and `<ModelFilename>` now names the shipped model, so an extra `.gguf` in
  `Resources/Models/Llama/` is never loaded by accident.

### Three menus, one home per command
- **Press E** is the top menu: talk, hire or dismiss, Follow me, Come here, Show me your inventory, Craft..., and two links:
  **Order wheel** (the **;** wheel, on them) and **Jobs and gear** (their hold-E menu as a wheel page).
- **Hold E** is their work and gear: Jobs, Roles, Weapons (the full weapon menu, with Ammo inside it), Emotes, Read, Progress,
  Inventory, Pick me up.
- The **;** order wheel is tactics only: Emotes and Weapons left it, and the ring's Orders and Craft entries are gone (Craft is on
  Press E). Voice and typed commands are unchanged.

### Includes the 3.4.03 hotfix: hires keep their loaded rounds
- **Fixed:** a hire who put her gun away to talk to you, or in a vehicle gunner seat, drew it back empty, so her next reload took
  another magazine from her pack. The rounds in her gun are kept now.

### Survivors
- **Survivors shoot straighter than bandits.** A level-1 survivor, hired or not, now starts at twice a player's aimed spread
  instead of three times; bandits keep three times. Both still reach a player's aimed spread at the top level. New setting
  `<Progression><BanditAimErrorAtLevel1>` (3); `<AimErrorAtLevel1>` (now 2) is the survivors' value.
- **Kneeling to shoot.** A survivor or bandit with a gun who stands still to shoot at a long-range target sometimes kneels for it,
  with her gun's own crouch accuracy, and stands up when she moves or an enemy comes near. New settings `<Combat><KneelChance>`
  (0.5), `<KneelMinTargetMetres>` (15) and `<KneelNoEnemyWithinMetres>` (10).
- **Friends no longer stand in the way.** When a friend stands in a shooter's line of fire, the one in front kneels so the shot
  passes over her (`<Combat><KneelForAllySeconds>`, 3), or the shooter steps aside. Before, only you made a hire step aside, and
  bandits never did.

### Fixes
- **Legacy NPCs shoot their guns again.** Since 3.4.03 a legacy (V5) NPC with a rifle swung it like a club at range instead of
  shooting. She shoots again, and swings only up close.
- **Hires walk round parked vehicles.** A following hire plans her path round a parked truck instead of into it, and finds a way
  round when she gets wedged against one.
- **Hires are never invisible in a vehicle seat.** On a dedicated server a hire who boarded with you could ride hidden; she now
  shows in her seat.
- **Shorter replies, no stray tags.** A hire says at most three sentences (`<LLM><MaxSpokenSentences>`), and a stray `</think>` from the chat model is never spoken or shown.
- **Dismiss asks first.** Dismiss on the Press E menu now asks you to confirm, so a hire is never let go
  by a slip of the mouse.
- **Hold E -> Weapons, Read and Progress open their page** every time; after a talk they could open the top of the talk menu.
- **The Progress page says "My skills..."** (it said "Her skills..." on every hire).
- **A dismissed survivor offers Hire right away.** Talking to her just after a dismissal could show Dismiss again instead.
- **NPCs from other NPC packs can be talked to and hit again.** Since 3.4.02 the invisible box around every NPC was
  switched off; an NPC model without body-part colliders then had nothing to interact with or hit. Such NPCs now keep the box.
- **Humanoid characters can emote and dance.** The emotes and dances that were greyed out for humanoid characters (for example
  the VRoid characters of other packs) now play; only Flip Off stays greyed for them.
- **Idle gestures come more often.** A hire standing still plays an idle gesture after 20 seconds, then about once a minute.
- **For modders: VRoid characters hold their guns in the hand with `PlayerCtrlGripScale` 0.6.** A VRoid hand is smaller than the
  rig the weapon animations are built on, so the gun sits in front of it. Add `<property name="PlayerCtrlGripScale" value="0.6"/>`
  to the character's entity classes (fitted in game on a VRoid pack; allowed 0.5 to 2.0, lower pulls the gun back into the hand).
- **Talking to a hire keeps her weapon away.** It stays put away through the whole conversation (`<ChatWeaponGraceSeconds>`, 20 s after her last line)
  and her talk gestures no longer repeat the same one twice in a row; a fight still draws it at once.
- **"I'll" no longer calls Jill.** A word with an apostrophe is never taken loosely as a name, so the reply goes to the hire
  in front of you.

## What's New in 3.4.03

Targets 7 Days to Die **3.4**. **Requires 0-SCore 3.4.14 or newer** (built and tested on 3.4.14.1542). Everyone on a server needs the
same version, server included.

### Start here: three ways to command your hires
The menus were reorganised this release. How you reach a hire depends on how far away they are:

| | Reach | Use it for |
|---|---|---|
| **Press E** on them | Up close: the game's own "Press E" reach, 3.5 m in a survival game (a game value, not a mod setting) | Talking, **Craft...**, **Read...**, **Your progress**, their inventory and trade, hiring and dismissing |
| **Hold E** on them | The same reach | Their ring: Talk, **Orders**, **Jobs**, **Roles**, **Weapons**, **Inventory**, **Craft**. One hire's jobs, roles and crafting live here (Jobs, Roles and Craft only on a survivor who can do that work) |
| **Order wheel**: the **;** key | Any distance | Orders at range; **positioning** (follow distance, formation bearing, take point, go there, jump down, get in / out of your vehicle); and calling a hire back. Press **;** and it stays open: it addresses the hire you look at, else the last one you ordered, else the nearest (their name is in the middle; the mouse wheel cycles your hires). **Click** an order for them, **Shift+click** for the whole squad; click the middle to go back a page; **;** again, right-click or Esc closes it. First page: Follow, Stay, Guard, **Come here** (calls them back from any job), then Orders, Stance, Position, Formation |
| **Voice** (Voice / Complete packs) | A hire within 40 m, a stranger within 15 m (sliders in the settings menu) | Everything by speech |

The wheel's key is `<FormationRadial><Key>` in modconfig.xml (free keys: Semicolon, Quote, Comma, Period, Slash, RightBracket,
Backslash; `None` turns the wheel off). Full details: `MANUAL.md`, Order Wheel.

### Includes the 3.4.02 hotfix: high CPU with a voice pack
- **Fixed:** with the Voice or Complete pack installed and the hands-free wake word on, the game used most of your CPU: the
  wake word's listener kept a thread on every core busy. It now runs on one thread, a few percent of a core. New setting
  `<WakeWordThreads>` in modconfig.xml (default 1). The Voice and Complete packs now ship with the wake word OFF; turn it on in
  the settings menu.

### New: a crafting window for your hires
- **E > Craft...** now opens a crafting window in the game's own look instead of dialog pages: their stations as tabs first, then the
  categories. Every recipe they have unlocked is listed; the ones they are short for are greyed, and the panel shows what they have and
  needs for each ingredient. Pick a count (minus, plus, max) and **Craft**. A row at the bottom shows what they are making and how many
  are left, with **Cancel**. The hold-E ring's **Craft** opens the same window. It works on a dedicated server.

### New: hires read books
- Give a hire a **schematic** they have not learned or a **crafting skill magazine**, then **E > Read...** (right after Craft...), or
  say "read your books". They read it and say what they learned; a schematic's recipe shows in their craft window at once, a magazine
  raises their crafting skill the way it raises yours. A book that would teach them nothing is not used up. Perk books (the series
  books) are not read yet. They will not read in a fight.
- **Their progress page shows their real crafting skill**, so a magazine shows there too.
- **The Read list offers only books that help them**; their skills have their own page in **Your progress**, grouped (Weapons,
  Tools and gear, Survival, Building, Other).

### New: emotes
- **Order wheel > Orders > Emotes:** wave, salute, fist bump, stretch, warm up, whatever, ten dances, and a More page. Shift+click
  for the whole squad (each picks their own dance). By voice: "wave at me", "salute me", "do a dance", "dance for me", "do a
  stretch", "bump fists" and the rest (whole phrases: a sentence that only mentions dancing does not start one).
- **Reactions:** a hire salutes when you hire them, sometimes fist-bumps you after a fight once it is quiet, and survivors sometimes
  wave with their greeting. Each can be switched off or tuned in Settings.xml (`<Emotes><Reactions>`).
- **Idle moments:** a hire standing idle (on Stay, Guard, at their post or beside you) now and then stretches, warms up or shrugs,
  minutes apart (`<Emotes><Idle>` in Settings.xml: how long still first, the wait between, which clips, or off).
- Their weapon is put away for the emote and comes back after. Any order, any threat or their starting to walk ends an emote at once,
  and they never start one in a fight. Other players on a server see it.
- Emote, dance and pose animations by Mumpfy and LittleRedSonja, used with their permission.

### New: hires look like they are working
- **Animated work:** at a campfire they kneel, strike a match, toss on a log and stand back up; at a workbench or forge they
  hammer (with a claw hammer in hand) and chisel; at other stations they work with their hands; a backpack craft is done kneeling.
  They read a book holding it in their hand. They loot low containers and bags kneeling and reach into taller ones standing, and pick
  plants with their hands. They talk with their hands, and nod or shake their head at orders. Each clip eases in from their pose and back.
  Humanoid survivors play their own versions of all of these. The work animations come from licensed packs, credited in the mod's README.
- **At a workstation:** the station waits for them: they walk to a spot in front of it (the side it faced you as you placed it) that
  can see it, face it, and light a forge or campfire themselves; the forge shows its fire. They walk round workstations, never over
  them, also on the way there. Their weapon stays put away at the station unless an enemy is near. When their order ends they put out a
  station they lit, unless something else needs it. Modders: a workstation block can name the side they work from with
  `<property name="NPCWorkSide" value="Back" />` (`Front`, `Back`, `Left`, `Right`, `Any`).

### Looting
- **Loot around here** also butchers fresh kills, corpses first, and keeps their post for it. A **full pack pauses** a loot, harvest,
  mine, clay, pick or butcher job: they come back to you, and go back to the job once you empty their pack to 5 free slots.
- **Steadier looting:** bags are looted kneeling, a bag or low container within 1 m keeps them kneeling for the next, they face what
  they loot, a bag stays until they are done with it, a partly taken bag stays a target, bags in a dip are seen, and their weapon no
  longer flashes out between containers (also right after a fight). Looted weapons are not equipped on their own.
- **Dead NPCs' bags** can be looted. **Come here** calls a survivor back from any job (order wheel, first page, or voice).

### Bandits
- **They spawn by biome tier:** tier 1 alone, tiers 2 and 3 with one or two mates; tier 1 at a quarter of the rate, tier 2 at half.
  The old fixed bandit is replaced by the tiers in every entity group. `sdcsbandit group` spawns a group for testing.
- **Loadouts by class**, molotovs by tier (none, one, three) thrown only at groups, ranged bandits advance cover to cover and melee
  bandits bound between covers. A death bag may hold the bandit's weapon or its parts.
- **Sleeping bandits hear and see you like zombies** in every room; no sleeper or bandit triggers a room's wake on its own.
- They close in, track you, and do not bandage under fire. They path around walls, fences and thin barriers and open doors on the
  way. Sentry groups hold their posts; six guard spawn blocks for POI builders (see `POI-BUILDERS.md`, new).

### Hires and survivors
- **Combat:** an enemy hitting them within reach beats the target they were keeping; a target they cannot reach is given up after 3 s;
  melee reach is the weapon's own range; swing weapons trace their arc and thrust weapons (spear, knife, knuckles) stab through bars
  and openings. Survivors' stealth sight scales with distance again, so bandits in the open are engaged. Cover: peeks and a
  step-out, a defensive hire takes cover only where you can see, and a cover search that found nothing is not repeated until
  something moves (less CPU in big fights). The dodge never glides, and there is no pause after a fight.
- **Following hires guard your backpack** when you die, until you come back for it.
- **"Put your weapon away" lasts** through a save and reload; dismissing them ends it.
- **Base keepers in an attack:** on **Base** repair they stop repairing while the base is under attack (no repair tool out): they
  fight what they can reach or shoot, and go back to repairs 10 s after it goes quiet. On **Combat** repair they keep repairing
  the wall and only fight what hits them. With nothing they can repair (skill or materials) they do not take the tool out at all.
  The under-attack window is one setting, `<BaseKeeper><UnderAttackSeconds>` (now 10, was 5).
- **Vehicles:** hires on Follow get in your vehicle on their own when you drive (a melee hire takes the front passenger seat);
  one in the middle of an emote stops it first. A seated hire now gets out before you log off or the world saves, so they are
  never lost from the save or left inside the vehicle.
- Follow-and-loot picks up loot bags; hires head-shot zombies again; a path speed cap (`NpcPathMaxSpeed`).

### Voice and chat
- **"Hey Nathan, stay"** now goes to Nathan (a greeting before the name used to send it to the nearest hire).
- **The order wheel** opens with a press: click orders them, Shift+click the squad; Back goes one level up.
- **One switch for NPC text:** `<Subtitles><NPCText>` in modconfig.xml caps every NPC's on-screen text (`all`, `clip` or `none`);
  their voice still plays. Vanilla traders are no longer part of the chat (no greeting, banter or speech from them).
- "Back off", "stand down" and "at ease" each mean one thing now (they belonged to two commands and the second never fired).
- The settings window's Volume, Speech Rate, Chat Timeout and Max Tokens are kept between launches.
- With the AI chat off, your saved pending memories are left as they are at boot (they were rewritten every launch).
- Fixes from a full code review: crashes, a memory-file loss, the voice cache's size cap, the memory pipeline, and multiplayer
  authority (on a dedicated server a client's dismiss, give item, jump and billing answer now run on the server).

### New: voice effects and the Voice Builder
- **Pitch and grit per voice:** a voice's line in `SupertonicConfig.xml` can carry `pitch` (semitones, -12 to 12; lower is deeper
  and slower, like tape) and `grit` (0 to 1, a rough edge that keeps the voice's loudness). Every line that voice speaks gets them,
  and other players on a server hear the same. Voices without them sound exactly as before. See `MANUAL.md`, Custom voices.
- **Voice Builder, a separate download for modders:** make a new NPC voice from a recording. Pick the recording, trim it, set pitch
  and grit, preview a few lines and install it into the game, all in one window; silence in the recording is cut out for you.
  Needs the Complete add-on, Windows, an NVIDIA GPU and Python 3.10 or newer. `NPCCore-VoiceBuilder-v3.4.03.zip` on the mod's page; its README has the
  details. Only clone a voice you have the right to use.

### Content and settings
- Eleven NPC guns show their own colours (a second white tint was overriding them); the wild fox uses its own bite; names instead of
  raw ids in the creative menu; localization rows that showed English in every language.
- New settings: the `<Crafting>` station and clip keys (`StationStowMetres`, `StationClearanceMetres`, `StationArriveMetres`,
  `StationStandInMetres`, `StationWanderPathMetres`, `AnimSets` and the rest), the `<Loot>` kneel and pause keys (`ChainMetres`,
  `ResumeFreeSlots`, `ResumeMinutes` and the rest; all in MANUAL.md), modconfig.xml `<Subtitles><NPCText>`. Also: `<Emotes>` (reactions and idle moments), `<Cover><FruitlessRescanMoveMetres>` / `<FruitlessRescanSeconds>`, the base keeper's
  `<BaseKeepRecheckSeconds>` / `<JobFightDwellSeconds>` under `<BaseKeeper>`. Removed: the `sdcsperf scorefix` test switch (SCore 3.4.11 does
  this itself).
- Console: `vc anim play <clip id> [seconds]` plays an emote on the survivor in front of you; `vc anim off` stops it.

## What's New in 3.4.02

Targets 7 Days to Die **3.4**. **Requires 0-SCore 3.4.3 or newer** - an older SCore will not do: it is what keeps a survivor's
harvest window right on a dedicated server. Everyone on a server needs the same version, server included.

### New: Orders in the hold-E ring
- Hold **E** on one of your hires: the ring now has **Orders**. Keep holding E and click it (or rest on it) and the order wheel opens
  on her with E as its key: release E over an order to give it to her, click while holding for the squad. Let go of E on Orders and
  the wheel stays open; your next E press picks. Only her leader sees it.

### New: hired survivors craft
- **E > Craft...** on a hired survivor: pick a category (All, or one of the game's own crafting categories she can make something
  in now), then a recipe from what she can make right now, then how many. She kneels where you gave the order
  and makes them one at a time from her own pack and the chests around her, into her harvest window, and earns crafting XP the way you do.
  Her own unlocks, your server's crafting settings. See the manual, "Hired survivors: crafting".
- **...and at workstations, as specialists.** Give a survivor a role on the order wheel (**Orders > Roles**, where **Medic** now lives
  too) or by voice ("be the blacksmith"): **Cook** (campfire), **Blacksmith** (forge), **Chemist** (chemistry station) or **Crafter**
  (workbench, cement mixer). Stand at her station, inside your or an ally's land claim (a campfire anywhere: it is a field station), and
  open **Craft...**: its recipes join the
  list ("Forge: Forged Iron (4)"). She walks over, kneels beside the station facing it, queues them in it the way you do, keeps a forge or campfire fuelled and lit, and moves
  her finished items into her harvest window, or your nearest chest when it is full. When she is done she walks back to you if you
  are within 15 m; farther away she stays at the station until you call her. Another order takes her unmade items out of the
  queue and gives the ingredients back. Without a role she crafts from her backpack, anywhere.
- **The Blacksmith smelts.** A forge recipe whose units the forge does not hold yet starts with her ore: she puts scrap iron, clay and
  the rest into the forge, keeps it burning and makes the items once they have melted. She keeps every material she has ore for at
  1000 units (Settings.xml), restocking when her crafts are done; a cancel gives back her ore that has not melted.
- **She takes an item's ingredients when she starts it**, so another survivor crafting from the same chests cannot use them up
  while she works (a forge smelting the base's scrap iron used to leave her backpack item short at the finish, and she made
  nothing). An order that ends mid-item gives them back to her harvest window; quitting the game puts them in her pack.

### New: see how a survivor is progressing
- **E > Your progress** on one of your hires: her level, her XP toward the next level, the quality she crafts at, and her roles.
  It reads the server's numbers, so it is right on a dedicated server too.
- **Her roles show after her name**: "Xyth - Smith", "Xyth - Cook, Smith" (in the order of the wheel's Roles page). On the
  "Press E" prompt, the talk window, captions, the order wheel and SCore's companions list, only while she is hired. Her name
  itself does not change: her memories and voice keep it.
- On a dedicated server, the Craft... page no longer stays on "..." until you click it.

### Survivors at a workstation
- **She stays at the workstation after a job.** When a survivor finishes at a forge, campfire or other station she no longer walks
  back to you: she stays and walks around it, within 5 m (`<Crafting><StationWanderMetres>` in Settings.xml; 0 = she stands still
  at her spot) until you give her an order. A hire you tell to stay near a workstation walks around it the same way. This replaces
  `<Crafting><ReturnWithinMetres>`.
- **"What are you carrying?" now counts her ammo**: the rounds loaded in her guns (one she holsters to talk to you included) as
  well as the ammo in her pack and on her toolbelt, and she says "I have no ammo." when she carries a gun with nothing for it.
- **Her station lists every recipe she knows there**, with how many she can make now. One she can't make yet shows `(0)`: pick
  it and she says what it needs ("I need Murky Water for that.", "I need Cooking Pot for that.").
- **She brings her own tools.** A cooking pot, grill, beaker, anvil, bellows or crucible she carries (in her pack, or in a chest
  of yours in reach) counts for her station's recipes: she puts it into the station's own slot when she crafts, and it stays
  there (`<Crafting><StationTools>`).
- **Each station has its own tab** in **Craft...**, listed first: a Cook at a campfire finds her cooking under **Campfire**,
  apart from the backpack's categories.
- **The hold-E ring's Orders entry has an icon** (eight circles in a ring); it was blank.
- **Starting ammo is a setting**: a survivor whose class rolls a gun starts with `SdcsSpawnAmmoCount` rounds for it (entity class
  XML, default 120, 0 = none). Fixed starting items stay in the class's `HarvestItems`.
- A blood-moon memory now names the day the blood moon began, not the morning after.

### New: survivors learn their craft
- **A specialist gets better at her craft by doing it.** Every item a workstation makes for her counts toward her skill in the
  role that uses that station: a Cook's campfire meals, a Blacksmith's forge work, a Chemist's chemistry station, a Crafter's
  workbench. Every 10 crafts (`Settings.xml` `<CraftSkill><CraftsPerLevel>`) her role's crafting skills go up a level, the same
  skills a player raises with magazines, so she unlocks recipes and crafts better quality the way you do (a new Cook makes corn
  on the cob after 20 crafts). Her skill never drops and stays if she gives up the role. **E > Your progress** shows it per role
  ("Cook skill 2: 20 / 30 crafts"). Which skills each role learns is a list per role in `<CraftSkill>`.

### New: pickpockets
- **A survivor can be a pickpocket**, per character: a class that sets `SdcsPickpocket` to `true` (no survivor does by default;
  the test character is `npcSdcsPickpocket` in the spawn menu). Stand within 2 m of her without looking and she may lift one
  stack from your backpack: something worth having, or up to 100 dukes. Catch her at it and she apologises and backs off.
  What she takes goes where her loot goes, so you can buy it back. At most once every 10 minutes per pickpocket and every
  30 minutes per player (`<Pickpocket>`). Single player and the host for now.

### New: sell to wild survivors
- **A wild survivor now buys from you** in her Trade window: the medical items her heals use (bandages, first aid kits, aloe
  cream, splints, casts, painkillers) until she carries 10, and ammo for a gun she
  carries up to what she keeps in her bag, at the game's normal price (your Better Barter counts), paid from her own coins. **Sell** is greyed out
  for anything else, or for more than she has room for or can pay. A new wild survivor starts with 50 to 150 coins, and
  what you pay her for her goods goes into the same purse. Settings: `<Trade>`.

### New: keepers learn to repair
- **A keeper learns by repairing.** Each repair and each rebuilt block pays her the repair XP a player earns for the same
  materials, into her repair skill and her level. A new hire repairs wood-tier blocks (sandbags, frames, wood); about 25
  repairs a tier unlock cobblestone (brick, scrap metal), then concrete, then steel. Workstations, traps and electrics are
  open to every keeper, as before. Her tier and repair XP show on **E > Your progress**. Settings: `<RepairSkill>`.

### New: survivors learn from harvesting
- Chopping wood, mining, picking plants and crops and butchering now pay a hired survivor the XP a player earns for the same
  haul (each item's material XP), so a woodcutter or a miner levels up too. Looting and repairing pay nothing yet.

### New: bandits
- **Two test bandits** in the spawn menu for now: `npcSdcsBanditMelee` (tier 1, a melee weapon) and `npcSdcsBanditRanged` (tier
  1, a pipe pistol or a pipe rifle). They look like tier 1 survivors but are hostile: they attack you, your survivors and
  zombies on sight, run at a target their gun cannot reach yet, and come looking when they hear you. No talk, hire or trade.
- **Loot:** a dead bandit always drops the game's plain zombie loot bag.
- **Their damage to players** is a setting (`<Bandits><PlayerDamage>`, half by default) that you can change while playing:
  `sdcsbandit damage 0.75` in the console. `sdcsbandit` alone shows it and lists the bandits in the world.
- **Their aim is a survivor's at their level** (see *Survivors aim like players*); `<Bandits><SpreadByTier>` can widen it by
  tier (0 by default).
- **One target at a time:** a bandit keeps fighting the target she has until it is dead or out of her sight, instead of
  turning back and forth between you and a zombie.
- **She hunts:** a bandit who loses sight of the enemy she is fighting, or of whoever just shot her, within 40 m, goes to where she
  last saw it, searches there for a few seconds, then gives up (`<Bandits><SeekMemorySeconds>`, `<SeekSearchSeconds>`).
  Before, an enemy behind a wall but inside her gun's range left her nothing to do: she wandered off (and could get stuck on a
  wrecked car) or stood still.
- Spawn lines come after they have been tried in the field.

### New: survivors aim like players
- **A survivor's spread is a player's aimed spread with her gun, times a factor for her inexperience:** 3 at level 1, falling to 1
  at level 10 (`<Progression><AimErrorAtLevel1>` and `<AimErrorAtMaxLevel>`). A level-1 survivor's pipe pistol spreads about as
  wide as a player firing it from the hip; a level-10 one shoots like a player aiming. Before, a standing survivor's shots spread
  6 to 30 times less than a player aiming down the sights, from her first day. Moving, running, crouching and firing a burst
  change her spread the way they change yours.
- **They aim at the chest of people** (you, survivors, bandits, traders); zombies and animals are still shot in the head. Every
  shot used to go at the head, so most hits on a player were head hits and a fight between survivors and bandits ended in a
  moment.

### New: hires get a clear shot
- **A hire with a gun steps aside when you are in her line of fire.** When an enemy is coming for you and you stand between
  her and it, she steps 2 m to the side, away from you, then shoots, instead of standing still behind you
  (`<Combat><ClearShotStepMetres>`, for an enemy within `<ClearShotLeaderRadiusMetres>` 20 m of you).

### New: guards spread out
- **Survivors told to guard the same spot spread out** instead of crowding onto it: the later ones step aside into a line beside
  the first, facing your way, then a rank behind, 2 blocks apart (`<Guard><SpacingBlocks>`). A squad click on Guard, or Guard
  given to one after another from where you stand, used to send them all to the block you stood on.
- **Guard and return now patrols the spot:** she walks out to a random spot within 6 m of it, looks outward a few seconds, comes
  back, faces your way, and repeats (`<Guard><PatrolMetres>`, 0 to stand still). In a fight she moves freely, then goes back to
  the spot first. Plain **Guard** still holds the spot and fights from it.

### New: your leadership reaches your hires
- **Charismatic Nature now works on your hired survivors**, as it does on other players: rank 1 they regain health faster, rank 2
  they deal 20% more damage, rank 3 they take 10% less (`<Leadership>` in Settings.xml). It stacks with their own level bonus and
  stops while you are not in the world.

### New: choose a survivor's ammo
- **E > Ammo...** lists each ammo type of the gun in her hand she carries, with how many; pick one and she loads it now (the
  rounds in her gun go back to her pack first). Or say it: "use armor piercing", "use hollow points", "use slugs", "use
  breaching slugs", "use standard ammo", "use flaming arrows", "use exploding arrows". The choice stays with that gun; when that
  type runs out she uses any other she has for it.

### New: your squad's stance on the buff bar
- **The Engage stance your hires share shows on your buff bar**: Defensive, Aggressive or Hold fire, with the order wheel's own
  icons. Hires on different stances, or no hires: no icon. The old mode icons (Resume, Hunting and the rest) are gone; nothing
  had set them for a long time, so Resume and Hunting just sat there.

### New: survivors restock ammo at your base
- A hired survivor inside your land claim (or an ally's) whose spare ammo for her gun drops below one magazine takes more from the
  base, up to 120 rounds (`<Ammo><RestockTarget>` in Settings.xml; 0 = off): from containers you set to broadcast anywhere in your
  claims and from your unlocked chests within 15 m of her, never another player's box. She takes it from where she stands, even
  mid-fight, so a blood-moon defender no longer runs dry with ammo in the base. Away from the base she still loots it.

### The base keeper: doors and traps
- **A rebuilt door is yours again, and locked if it was.** A door the keeper rebuilt used to come back with no owner and unlocked.
  Now it comes back owned by you and, when it was locked as it fell, locked (no password).
- **She puts your spike traps and barbed wire back.** Wood and iron spikes, barbed wire and barbed fence come back when zombies
  break them, and when zombies only wear them down a stage she restores the stage you built, both at a new trap's full repair cost
  (10 wood for wood spikes, 2 forged iron for iron spikes, 10 scrap iron for barbed fence). Electrical traps, turrets, mines and
  workstations are still not rebuilt, and blocks that came with a POI stay as the zombies left them.
- The base check that finds destroyed blocks now runs only while a keeper is on duty for that base.

### Fixes
- **A survivor you pick up comes back exactly as she was.** Placing a picked-up survivor again could change her armor (about
  four armor sets in ten came back as another one), and on a dedicated server she stood without her armor for a few seconds
  before it appeared. She now keeps her own look and armor, and everyone sees it at once. A survivor picked up before this
  update almost always gets her own armor back when you place her.
- **A survivor you pick up and place again keeps her name, her hire and her weapon.** She could come back named and dressed as
  one of your custom characters, or with her weapon put away for good when you had picked her up from her dialog; on a
  dedicated server she could also come back under another name (over her head, in her dialog or in chat) and ask to be hired
  again although you were still her leader. A survivor already affected keeps that name and look.
- **A survivor with anything in her pack can no longer be picked up**, the way a vehicle with items in its storage cannot: you
  are told to empty her pack first. On a dedicated server a pickup used to lose everything in her pack and she came back with
  a fresh set of her starting items; in single player her pack travelled along as extra storage. Every survivor starts with two jars of water in her pack for putting out fires;
  take those out too and give them back once you have placed her. To free a stuck hire, order her to follow you.
- **The health bar shows a survivor's name.** With health bars on, looking at a survivor or a bandit showed her class name
  (such as `npcSdcsPickpocket`); it now shows her name.
- **Your hires no longer stand idle mid-fight.** When their shot was off for a moment (a bow reloading after every arrow, no
  clear line of fire, the enemy out of sight), nothing else told them what to do past 10 m, so they dropped to their idle. They
  now keep facing the enemy, ready, until they can shoot again. Bandits do the same.
- **A follower no longer freezes behind a gap or a step while you walk on.** Near awkward ground a route search can take longer
  than the half second after which she asked for a new one as you walked, which threw the unfinished search away, so she could
  stand for seconds with no route. She now lets a search finish (up to 2 s, `<PathPendingMaxSeconds>` in modconfig.xml).
- **A new voice setting takes effect even when only the core package is updated.** With the voice package installed, a
  setting it did not have yet used to run on a built-in default until the voice package was updated too; it now uses
  the core package's value. The boot log still names such settings.
- **A wandering survivor or bandit no longer loops forever when she is blocked.** Pressed into a car wreck or a corner,
  the wander kept picking new points and walking her back into it until something else came along. Now, when she has not
  got 3 m anywhere in 6 s, she gives that wander up and rests 10 s before she wanders again (`<Wander><StuckMetres>`,
  `<StuckSeconds>`). Hires on Wander get the same.
- **Your hires defend you when you are attacked from the front.** On the Defensive stance a melee hire is meant to go for an
  enemy that is coming for you, but following behind you, your own body blocked her view of it, and she stood by while you
  were hit until you moved or struck back. She now goes for an enemy that is after you within 10 m of you even when she
  cannot see it yet.
- **A tier 1 survivor no longer gets free ammo at spawn.** 9mm from her own trade goods went into her pack, and she spawned
  holding her starting pipe pistol instead of her melee weapon. Her trade goods' ammo now stays for sale; ammo she loots later
  is hers.
- **Survivors no longer pick up medical items they cannot use.** A sewing kit, vitamins, antibiotics, a blood bag or the candy
  drugs went into a survivor's bag as if they were bandages (looted, bought or stolen) and took the room of the ones she heals
  with. They now go to her trade stock, and she no longer buys them.
- **One damaged character profile no longer hides all your others.** When the game could not read one of your saved
  characters, none of them could appear as wild survivors. Now that one is skipped and named in the log.
- **Every survivor has a name.** A few survivor types (the test classes and npcSdcsXyth) showed their class name on the wheel
  and in memories. They now roll a name like the others, and one already saved without a name gets one at her next load.
- **A survivor no longer loses her weapon to a save.** Quitting while her weapon was put away (talking to you, crafting, looting)
  saved her empty-handed, lost her gun's loaded rounds, and she fought with her fists from then on. Her weapon now goes back in
  her hand before the world saves; a survivor who loads with empty hands and no gun she can fire takes her melee weapon. And a
  gun she spawned with no longer counts as empty after she switches to melee at close range, and keeps its loaded rounds when
  the game is saved while she has it put away.
- **Hired survivors' XP follows your server's XP multiplier**, as yours does (it ignored the setting). And the pace of the start
  level is now a setting: `<Progression><StartLevelDaysPerLevel>` (7 world days per level) in Settings.xml.
- **A hired survivor's weekly pay keeps counting between sessions.** Every load used to restart her contract at week 1, so the
  price never grew and a reload wiped the grace days. Now the contract carries on; dismissing her ends it, so a new hire starts at
  week 1. And during the grace days you can now pay: say **pay**, and the grace ends.
- **A survivor saved inside the floor stands on it after a load.** She used to fall through it each time the load guard caught her,
  then through the world, and the rescue left her badly hurt.
- **No more debug smoke over a survivor's target.** Smoke marking the container or block a survivor was heading for showed
  whenever a player had **Mod Debug Logging** on. It is a developer marker and no longer follows that switch. The switch itself
  is saved on your PC and stays on across updates until you turn it off (`vcdebug off`).
- **Players who joined a game see her put her weapon away to talk, and see her bandage herself.** On a dedicated server,
  or in another player's game, a joined player kept seeing her gun and her armed pose while she talked, and did not see her
  bandage herself or douse a fire with a water jar; they now see what the host sees.
- **A mine hurts a survivor as much as it hurts you.** Survivors took about two and a half times a player's blast damage, so a
  mine under a vehicle killed the riders sitting over it; they now take the player's share.
- **A dead survivor keeps her medical items.** Her self-heal check kept running on her body and, once its wait was over,
  could spend a bandage or kit from her pack on herself.
- **Quieter logs.** A weapon swap no longer warns "no model child" every time, and with verbose logging on, a dead survivor's
  body no longer repeats its diagnostic lines every second.
- **A survivor no longer falls through the world when a save loads slowly.** When the ground's collision was late, a
  survivor could sink into the ground before the load guard took hold; it then held her there, she fell below the world
  and was rescued beside you, inside your parked vehicle if you stood next to one. The guard now holds her where she was
  saved.
- **Her inventory window refreshes** when she adds an item while you have it open (single player): before, the window kept
  showing the old slots, and an item dropped onto a slot shown as empty replaced what she had just put there.
- **A voice pack's clips keep their own captions.** Another pack's clip with the same file name ("Laugh01") no longer shows
  Harley's line. Pack authors: a clip's caption row is keyed `<voice folder>_<file name>` (see the manual).
- **A program already using one of the AI helpers' ports is no longer taken for the helper.** That helper stays off and
  the log names the program; on a non-English Windows a helper left over from a crash is now found and closed at start. A
  leftover helper that no longer answers is restarted instead of reused.
- **No network clash with other mods:** the voice relay's network message has a name only this mod uses.
- **The order wheel and the voice settings window speak your game language** (12 languages), the settings window's test
  results and the "No NPC nearby" note too. The survivor still says the order in English; the caption under it is translated.
- **A character with her own caption language keeps it everywhere.** A character set to one language (`<ForceLanguage>`)
  still captioned her canned lines ("I'm done crafting.", "My pack is full.") and her squad order replies in your game
  language; they now follow her setting, like her replies.
- **"I don't know how to do that." is captioned in your language** (12 languages); it was English for everyone.
- **Multiplayer: "I need X for that." names the item in each player's own language.** On a dedicated server the item
  came out in the server's language for every player. The caption now uses the name your game shows; the spoken line
  uses the English name, as every spoken line does.
- **Multiplayer: her inventory no longer loses or duplicates items while you look into it.** On a dedicated server, what a survivor
  made, harvested or looted while a player had her inventory open vanished when the window closed, and what she used came back. With
  SCore 3.4.3 the server keeps its own changes when the window closes, and only one player at a time can open it; she stops
  and faces you while it is open, as in single player.
- **Multiplayer: switching worlds without restarting the game.** After quitting to the menu and loading another save or joining
  another server, survivors followed without their breadcrumb trail, a client that had played single player kept doing the
  server's trail work, your custom characters were not sent to the server you rejoined, and a second server got no version check.
  All of it now starts fresh with each world.
- **An SCore or game update can no longer switch off half the mod.** If an update renames something the mod patches, that one
  patch is skipped with a `[SDCS-PATCH] ... NOT applied` line in the log, and everything else still loads. Before, the first such
  patch stopped the rest of the survivor mod from starting. Each patch also names the exact method it changes, so an update that
  only adds a same-name method no longer trips it.
- **Survivor memories and the remembered base survive a crash.** Both files are written to a temporary file and swapped in, so a
  crash mid-save can no longer cut them short. An unreadable memory file is set aside (`NPC_Memories.xml.unreadable-<date>`) and
  the last good copy (`NPC_Memories.xml.bak`) is read instead, where before the next save overwrote it and every relationship's
  history was gone. A conversation not yet committed to memory (saved at quit) is written the same way.
- **Captions in your game language.** Order captions, reply subtitles, stories and the AI's subtitle language followed the game's
  Language option only when you had set it by hand; left at its default (your Steam language) they came out in English. They now
  follow the language the game shows, Traditional Chinese included (shown in Simplified).
- **The job tool goes away when the job ends.** A survivor whose harvest or butcher job finished kept the axe, pick or knife in hand
  while she followed and fought; she now goes back to her weapon within a second.
- **No duplicated items from a nearly full inventory.** When her inventory could take only part of a looted or harvested stack, that
  part was also left in the container or paid again later.
- **Console commands on a dedicated server.** `vc` and `vcdebug` now run on the player's own machine and any player may use them:
  before, a non-admin was refused and an admin got the server's state (no TTS or STT there), and several `vc` diagnostics never
  printed to the console at all. `vcdebug` now shows both switches (its own and `Settings.xml` `VerboseLogging`) and the result.
  `sdcscustom spawn` works from a client, as its help said (admins).
- **Clearer config warnings.** A broken `Settings.xml` now says the whole file was ignored (with the line the parser stopped at);
  an invalid order-wheel key says the wheel is off and lists valid key names; a character whose `personality` names no loaded
  personality is reported once; `<DefaultTacticalMode>` with a bad value warns and falls back instead of failing.
- **Survivors use only your chests.** Base repair (and the new crafting) take from an unlocked chest near her only when you or one
  of your allies placed it; a neighbour's open box no longer feeds them.
- **Base repair takes shared storage only inside your claims.** A broadcast chest between two of your land claims, outside both, no
  longer feeds her repairs.
- **An unsupported `TtsEngine` value falls back to audio.cpp**, the default, as `modconfig.xml`'s own comment says; it used to
  fall back to Supertonic. Ten more voice settings, when missing from both config copies, now use their shipped values instead
  of older built-in ones: the LLM timeout, conversation memory, typing indicator and typing delay; the lip-sync gain, noise gate
  and jaw angle; and the speech-to-text beam size, silence length and vocabulary hint.
- **A custom character wears her outfit for players who joined the game.** On a dedicated server (or in another player's
  game) a joined player could see a survivor made in the character creator in her underwear: her face arrived with her, her
  outfit a moment later, and only the face was applied. She now waits for the outfit and puts it on.

### Performance
- **A survivor walking around her workstation asks for a path at most once a second** while she has none, instead of on every
  AI tick (up to several hundred path searches a minute near a crowded station).
- **Survivors cost less per frame, with no change in what they do.** In a fight their checks test the cheap facts first (the
  order, the weapon's reach) and the line-of-sight test last. The first-aid check skips a survivor who carries no medical item
  and reads her packs once per frame instead of once per ally. A work-site scan (chop, mine, dig clay, pick) reads the ground a
  chunk at a time instead of thousands of single lookups in one frame. Several per-frame diagnostics and small memory
  allocations are gone while verbose logging is off.

## What's New in 3.4.01

Targets 7 Days to Die **3.4** with **0-SCore 3.4**, like 3.4.00. Everyone on a server needs the same version, server included.

### Humanoid characters ride as gunners
- **Mod-pack characters hold their gun in a vehicle.** Any Humanoid character (the Mixamo-format rigs mod packs use) now holds its weapon
  properly in a vehicle seat instead of the plain sitting pose, and turns to aim like the built-in survivors.
- **For pack authors:** `sdcspose captureall human` captures the hold poses on any Humanoid character in one run; one set serves every
  Humanoid rig. The shipped set lives in `Config/SeatPoses/Humanoid/`.

### Character authors: choose what text a character shows
- New `<Subtitles>` in a character's `personalities.xml` entry: `all` shows every line (replies, chat, order acknowledgements, clip
  captions), `clip` shows only a voice clip's own caption and never the generic reply text, `none` shows no text for that character.
- Unset, a clips character (`VoiceMode` clips, such as Harley) uses `clip`, so her recorded lines are no longer paired with generic
  order text that does not match them; set `all` for the old behaviour. Every other character keeps `all`.

### New settings: how survivors follow you, loot, work, keep your base, deal with sleepers and ride
- `Settings.xml` `<Follow>`: how close she walks beside you (`PaceNearDistMetres` 8), when she runs to catch up (`PaceFarDistMetres` 10)
  and her walking speed near you (`PaceNearWalkSpeedMps` 2).
- `modconfig.xml`: her default follow distance (`FollowDistanceDefault` 2.5), how far you may walk from a "stay here" post before she
  follows (`CommandLeashDist` 45), when she steps up to catch-up speed (`CatchupStartDist` 8), and how close before a survivor greets
  you unprompted (`ProactiveGreetRange` 10).
- `Settings.xml` `<Loot>`: how much medical (`BagMaxMedical` 5) and how many ammo stacks per gun (`BagMaxAmmoStacks` 2) she keeps
  in her own bag before the rest goes to your window, how far above or below you she loots (`VerticalReachMetres` 3, one storey), and
  the distance beyond which she teleports back after "loot around here" (`ReturnTeleportMetres` 40).
- `Settings.xml` `<WorkSite>`: how many layers of an ore vein she mines (`MineVeinDepthLayers` 2), how long she swings at one tree,
  rock or corpse (`SwingBudgetSeconds` 60) and how long she walks to one (`WalkBudgetSeconds` 15) before moving on.
- `Settings.xml` `<BaseKeeper>`: how often the base keeper rescans for new damage (`RescanSeconds` 30), how big an area she finishes
  before moving on (`AreaRadiusMetres` 8), how fresh a hit must be for combat repair to chase it (`UnderAttackSeconds` 5) and how far a
  chest that is not broadcast may sit from her post (`ChestReachMetres` 15).
- `Settings.xml` `<Sleeper>`: how close a sleeping zombie must be to see one of your survivors (`SightRangeMetres` 16), how far a wild
  survivor runs past a house that woke on her (`LeaveBeyondMetres` 30) and how long she stays out of it (`HouseAvoidSeconds` 300).
- `Settings.xml` `<Vehicle>`: the seated gunner's range cap (`GunnerMaxRangeMetres` 40), how long she keeps her gun out after the last
  enemy (`GunnerIdleHolsterSeconds` 5) and how far from the vehicle a hire walks over to get in (`BoardRangeMetres` 30).
- Every default is the old fixed value, so nothing changes until you edit one. A value that would break a linked distance (for example
  a leash beyond the 50 m follow teleport) is refused with a warning in the log, and the default stays.

### Fixes
- A seated gunner fires within her gun's own effective range (a pistol about 17 m, a magnum 27 m, rifles up to the 40 m cap)
  instead of firing every gun out to 40 m.
- Survivors no longer fall through the ground and come back badly hurt when you load a save; a survivor who was riding with you
  when you logged off now stands beside the vehicle, not inside or under it.
- No more red "Attachments not found" error when a survivor holding a scoped weapon puts out a fire or bandages herself.
- A wild survivor no longer stays crouched after looting.
- A survivor's pack window is titled "Backpack" (in every language) instead of `traderNPC`.
- A survivor whose weapon comes out mid-chat (you pick one from her menu, a roll, a bandage) puts it away again a few seconds later,
  instead of holding it for the rest of the chat.
- A medic whose own heal cannot act yet (waiting, or no item that fits) now treats the party instead of standing by.
- The Korean "stop being the medic" reply reads correctly.
- German "follow me" subtitles show the follow replies again (they always read "Schon erledigt."); the Spanish "weapons free" reply
  reads "Entro en combate.".
- A clips character (such as Harley) answers wheel and squad orders with her own clips instead of a text voice.
- Harley's voice clips show their captions (they sat in a file the game never loaded), and her laugh has one now.
- Harley and the other older characters cannot do jobs (loot, pick, chop, mine, dig, butcher, repair, keep the base, be the medic). A
  job order now gets "I don't know how to do that." (Harley answers with a clip) and she keeps following, instead of taking the order
  and standing guard at the spot. One still carrying a job from an older save drops it when the save loads, so she fights again
  instead of holding back to the old job spot.
- A hire keeps fighting the zombie she is on instead of flipping between two of them every time she closes in or swings: another
  one takes over only when it is 2 m closer to her, or when hers dies or drops out of the fight. New `modconfig.xml` settings:
  `TargetSwitchCloserMetres` (2; 0 = the nearest always wins, as before) and `TargetStickMemorySeconds` (3, how long she
  remembers her zombie through a step back or a pause between swings).
- A survivor's short status lines ("Base remembered.", "My pack is full.", "I need an axe for that." and the rest) now show their
  subtitles in your game language. They are still spoken in English, like every other line.
- A survivor who rode with you no longer ends up under the vehicle when you get out: the follow trail stops while you drive and
  starts fresh where you step out, so she can no longer hop back to a spot inside the vehicle.

## What's New in 3.4.00

Targets 7 Days to Die **3.4**. **Requires 0-SCore 3.4 or newer** - an older SCore will not do. Everyone on a server needs the same version,
server included.

### The designated medic
- **Be the medic.** Say "be the medic" or use the wheel toggle to appoint one survivor as your medic - one at a time; "stop being the
  medic" releases her. As medic she looks after the whole party, not just you.
- **Bleeding first, then the right item.** She stops a bleed before anything else, and heals with the item that fits the job - a cloth
  bandage, a medicated bandage, a first aid kit, or pain pills - spending that item and getting its own heal.
- **Aids you in a fight on her own.** A hired survivor treats a leader who drops low in a fight (and after, if he stays hurt): she walks
  to him first, never while an enemy is on her, and says "Applying first aid." when she starts.
- Survivors bleed in a fight again (legacy NPCs too), now that bandaging works - which is what all of the above treats.
- The role and the treatment choices are tunable in `Config/Settings.xml` (`<Medic>`).

### Steadier in a fight
- **She keeps her nerve.** A hire engages an enemy inside about 5 m, keeps fighting the one she is already on until it is beyond 7 m, and
  after the last kill she holds her spot for 3 seconds before resuming the follow - a bursty fight no longer turns her around halfway.
  Any order ends the wait at once. `Config/modconfig.xml` has `EngageReleaseRadius` and `CombatLingerSeconds` if you want them different.
- **No trap damage on your hires.** With 0-SCore 3.4 a hired survivor takes no damage from traps.

### The base keeper
- **"Remember the base."** Say it (or pick it on the wheel) and she takes a snapshot of the base; it then learns your own building and
  removals, while zombie, collapse, gun and explosion damage stays marked as holes. The snapshot loads with the world and saves with it.
- **Rebuild what is destroyed.** A base keeper walks to each destroyed block - lowest first - and puts the original block back at its
  full material cost, paid from the base chests and her pack. "Combat repair" fills fresh holes first; "base repair" works the damaged
  blocks first. Two or more keepers split the base between them, and a fresh hit in combat still draws everyone.
- A block's neighbours are re-checked a second after it falls, so a wall that goes down in a line is rebuilt as a line; destroyed traps
  and workstations are not left stuck in the hole list.

### Field jobs
- **The harvest list grows:** chop wood around here, mine stone and every ore, dig clay, pick wild plants and ripe crops, and butcher
  animals. "Butcher around here" sends a hire to the fresh corpses near you, and a following hire will also finish a fresh corpse within
  about 10 m of you on her own.
- **Loot around here.** A hire loots the area you stand in (the place, or 15 m), guards it on a Guard order, then comes back.
- **Tools wear for real:** the wear sticks, a broken tool is not used, she moves to her next one, and every survivor's kit now carries a
  stone axe, a stone shovel, a bone knife and 5 bandages.
- **She works harder as she levels:** harvest jobs pay +10% per level above 1.
- **The order wheel matches the voice:** a root Orders page (crouch, stand up, scout, guard and return, record / run / cancel patrol)
  and a Stance Weapons page (guns, rifles, shotguns, machine guns, melee, blades, empty hands); the jobs live on Harvest and Mine
  pages. The talk menu is cut to essentials, and the patrol rows fold into one entry.
- **Every voice command answers in all 12 languages** - the stance, formation, distance, crouch, patrol, jump and go-there replies that
  used to be English only are translated now.

### In the vehicle
- **She rides with you.** "Get in" / "get out", or she boards when you start driving - and you can see her in the seat. Ranged riders
  take the back seats, melee riders the front passenger seat.
- **Car gunner.** A seated ranged hire fires her own gun at what her stance allows (about 40 m, a 90 degree arc of the vehicle), holds a
  shot that would pass through an ally, reloads that can never stick, and her upper body twists toward her target; in defensive stance
  she answers anything coming for you or her squad.
- Getting out no longer sends her sliding off the vehicle, and a pillion gunner shoots past the driver's shoulders.

### Fire
- **A burning survivor puts herself out.** She rolls clear of the fire first, then drinks from a water jar (every survivor carries two);
  a heal waits until the fire is out.

### Fixes
- **The medic walk crash** - the medic no longer crashes when she walks to a patient.
- **npcSdcsXyth's eyes** - her eye colour was spelled a way the game does not ship; she now has the blue colour the game does.
- A repair hammer is never remembered as a survivor's weapon - she fights with her real weapon and the hammer comes back for the repair.
- After a restart she keeps the weapon you last gave or chose (when she still has it and the gun is loaded).
- She holds her weapon away for a few seconds after a chat window closes, and draws at once if a fight starts.
- Harvest and mine field fixes: small trees and shrubs count, a one-swing shrub pays its wood, she walks up close before swinging and
  aims into dug holes, and a leftover target no longer steers the swing.
- A seated gunner aims at a live target only - no more aiming at a dead target's head.
- Both debug switches (verbose logging) ship off.

### Under the hood
- Survivor decisions are grouped into priority bands (safety, defence, aid, engage, jobs, orders, ambient), with tests that hold every
  weight in its band - the base for steadier calls in a fight.
- A performance probe (`sdcsperf`, off by default) times each survivor's decision updates.
- The new `Config/Settings.xml` `<Combat>` block carries the fight-back radius and the close-melee distances at today's values.

## What's New in 3.3.04

Targets 7 Days to Die **3.3**. **Requires 0-SCore 3.3.28 or newer** - an older SCore will not do. Everyone on a server needs the same version,
server included.

### Spikes and traps (requested)
- **Survivors respect your traps.** They avoid spikes, barbed wire and blade traps, keep a cell's distance from spikes where there is room,
  slow to a walk next to them, and step off if they end up touching one. The first brush against a spike costs them 1 health, not a full hit.
- They are no longer dragged to a crawl by spikes, wire, webs or water, and they will not path off a drop of more than 5 blocks.
- Survivors are immune to ordinary land mines (as before); the one mine that ignores that, the Twitch candy tin mine, they now walk around.

### Follow and loot
- **A hired survivor loots while she follows you.** Order it from her menu ("Follow and loot"), from the order wheel (Actions > Loot) or by
  voice ("follow and loot" or "loot around here"). She stays with you; when you stop, she searches the containers within about 15 m of you and comes back; when you
  walk on, she follows. Any other order ends it, and so does a full pack.
- **Loose loot first.** Ammo, medical, book, food, chem and liquor piles are taken before cupboards and boxes, and a pile she empties
  disappears just like one you empty.
- **She reaches like you do.** She opens a container from up to 4 m away, as a player can, so a counter or a corner no longer stops her
  short. She still needs a clear line to it: no looting through walls or closed doors, and she skips locked boxes and player storage.
- **She finishes the room.** A container she has started for is not abandoned when you take a few steps; she only gives it up if you walk
  more than 20 m away. A container she just emptied is never picked again.
- **No looting under a zombie's nose.** She will not loot with an enemy close, and a sleeper that spots her sends her back to you. Once the
  zombie is dead she goes back to the room.
- She remembers a container she could not reach and leaves it alone for a few minutes.

### The order wheel
- **Two pages:** Actions (Go there, Jump down, Loot) and Position (Close, Normal, Far, Point).
- **Go there:** look at a spot, open the wheel and send her; your aim passes through your own hires. She holds the spot without spinning.
- **Point:** she leads the way ahead of you, takes stairs, and opens the door in front of her.

### Combat
- **Melee hires stay close.** A hire with a melee weapon only runs at an enemy within 10 m of you; hires with guns are unchanged.

### Custom characters
- **Your own characters as NPCs.** The characters you make in the character creator can be spawned as survivors (the F6 entity spawner's
  `npcSdcsCustomMale` / `npcSdcsCustomFemale`, or the console: `sdcscustom spawn <name>`), with their hair and eye colour kept.

### Fixes
- **No more crash** when looking at a hired survivor.
- **Hits land on the body.** An invisible box around every survivor, NPC and pet caught shots and swings up to 30 cm away from the body
  (blood in the air beside them). It is gone; `Config/Settings.xml` `<HitBoxFix>` turns the fix off if you ever need to.
- **Stunned pets** no longer flood the log with an error every tick.
- **NPCs that fall through the world** (usually on loading into a game before the ground under them is ready) are put back: this is
  SCore 3.3.28's own rescue - thanks to SphereII.

## What's New in 3.3.03

Targets 7 Days to Die **3.3**. Built and tested against 0-SCore **3.3.7.933** - use that or newer. Everyone on a server needs the same version, server included.

### Survivors live their own lives
- **Unhired survivors loot.** A survivor you have not hired searches containers near where she roams: usable items go to her bag, goods to her trade stock,
  at most one container a minute. She gives up on a house she cannot get into.
- **Hired survivors loot on order** (the "Loot" row in their dialog), kneel at the container and put their weapon away while they search.
- **Trade with a survivor** to buy what she has collected; your coins go into her bag.
- **Sleepers.** Survivors also spawn asleep inside buildings and wake on sight and noise. An unhired survivor slips away from a house that wakes up around her.
- **Defensive by default.** An unhired survivor fights only an enemy that hurt her, is right on top of her or is coming for her.
- **About half as many survivors** spawn in the open world.

### Hired survivors
- **They stay and face you** while you talk, trade or open their inventory, with their weapon put away.
- **They greet you as a companion**, never as a stranger, and again after a quiet spell.
- **They draw the gun you gave them** when a save loads, instead of their starting gun.
- **"I'm out of ammo."** A survivor tells you when her gun runs dry. Starting ammo and medical supplies go into her bag.
- **Weapon attachments show** (scope, flashlight, silencer) on add-on weapon models too.

### Movement
- **Feet on the ground.** Survivors no longer stand sunk into the floor.
- **No more foot slide.** The walk and run animation follows how fast a survivor really moves.
- **One pace.** Wild and hired survivors walk at the same speed.

### Dedicated servers
- **Server-built lines are heard.** "What are you carrying?", "I'm out of ammo." and "My pack is full." now reach every player within earshot.

### Source code
- Both mods are now published under the MPL 2.0 licence: <https://github.com/XythQ/NPCCore>.

## What's New in 3.3.02

Targets 7 Days to Die **3.3**. Built and tested against 0-SCore **3.3.7.933** - use that or newer. Everyone on a server needs the same version, server included.

### Survivors
- **A hurt survivor rolls clear before bandaging.** When a survivor needs to heal and an enemy is within reach, they turn and roll about five metres out of the
  way first: past the enemy when there is room, otherwise to the side or straight back. They then turn to face the enemy and bandage. With nowhere to roll,
  or with no enemy close, they bandage where they stand. The health comes back when the bandage is done.
- **Weapons are put away while you talk.** A survivor stows their weapon while your chat window or their dialog is open, and draws it again when you close it
  or a fight starts.
- **A gun you hand over keeps its scope and its loaded magazine.** The survivor holds the gun you gave them, not a plain copy of it.
- **DLC cosmetic outfits are no longer picked** when a survivor's look is rolled.

### Known issues
- The heal roll and its bandage have only been tested in single player. On a dedicated server the survivor still heals; the roll may not be shown.
- A survivor who rolls into something they could not see (a step, another body) finishes the roll on the spot.
- On a dedicated server, other players can still see a stowed weapon in a survivor's hand while you talk to them.
- A companion can walk off the edge of a sloped roof while following you on it.

### Credits
- Roll animation by Mumpfy.

## What's New in 3.3.01

Targets 7 Days to Die **3.3**. Built and tested against 0-SCore **3.3.7.933** - use that or newer. Everyone on a server needs the same version, server included.

### Weapons
- **Survivors use any weapon you give them.** Put a gun or a melee weapon in a hired survivor's backpack and they can use it: no special NPC copies, weapons from
  other mods included. A gun needs its ammo in the backpack.
- **Carried Weapons menu.** Talk to a hired survivor and pick "Carried Weapons" to choose from what they are actually carrying.
- **Swap by voice.** "Use your club", "switch to the pistol", or a kind of weapon ("use a rifle", "get your melee weapon out").
- **Each survivor spawns with one weapon of their own**, rolled from their tier's pool, plus a club to fall back on. They no longer carry every weapon of the pool.
  Survivors in an existing save keep the bags they already had.
- **Out of ammo:** a survivor switches to melee when the gun is dry, and back when you hand over ammo.
- **A survivor's weapon is put away when they die** (it no longer hangs in the air).

### Survivors
- **64 more looks.** Many more combinations of face, hair and colours among the survivors you meet.
- **Paced retreat.** A survivor backing away from a fight now steps back at a steady pace.

### Dedicated servers
- **Weapons now show and behave correctly on a dedicated server.** You see the weapon the server gave the survivor, swaps by menu, dialog row and voice hold,
  reloads work, and opening a survivor's backpack no longer changes what they hold.

### For character pack authors
- **Humanoid characters can run on the player's own animations and real player weapons.** This is on by default. A pack's classes extend one of two base classes;
  the guide is `docs/author-guide-humanoid-pack-conversion.md` in the mod's source.
- **A pack can carry its own `Config/personalities.xml`** (personality, voice, stories, face tuning). Nothing about your character has to live in this mod.
- **Gun models without a scope node no longer stop a character from spawning.**
- **Two class properties adjust the weapon grip** for a model's proportions (`PlayerCtrlGripOffset`, `PlayerCtrlGripScale`). New and not yet tuned on a pack.

### Known issues
- A companion can walk off the edge of a sloped roof while following you on it.
- A survivor that falls from a roof may not climb back up the ladder to you.
- A survivor's armour can be missing for a fraction of a second when you set them down again.
- Converted pack characters have been tested in single player only.

## What's New in 3.3.00

Targets 7 Days to Die **3.3**. Built and tested against 0-SCore **3.3.7.933** - use that or newer.

### Game 3.3 support
- **This release is for game 3.3 only.** Game 3.3 changed how items, held weapons and containers work, so every mod with its own code had to be updated.
  3.3.00 will not load on game 3.2: if you are still on 3.2, stay on 3.2.06.
- **Update 0-SCore first.** You need the 3.3 build of 0-SCore (3.3.7.933 or newer). An older 3.3 build of SCore throws an error when you open a companion's inventory.
- **Everything else is as it was in 3.2.06.** Hiring, following, orders, companion inventory, asking a companion for an item, combat and survivor levelling
  were all re-tested on 3.3. There are no new features in this release.

### Thanks
- To SphereII for the 3.3 fixes he sent to both halves of the mod, and for the quick SCore turnaround.

### Known issues (carried over from 3.2.06, being fixed next)
- A companion can walk off the edge of a sloped roof while following you on it.
- A survivor that falls from a roof may not climb back up the ladder to you.

## What's New in 3.2.06

Targets 7 Days to Die **3.2**. Built and tested against 0-SCore **3.2.36.628** - use that or newer.

### Following
- **Companions path to their place beside you.** Following now plans a proper route to its spot next to you instead of walking your exact footsteps, and hands back to
  close-range steering once it is level with you.
- **Stairs and ladders work much better.** Companions take stairs step by step a flight at a time, keep up while you climb, and no longer take big hops down a stairwell.
- **Routes up railed stairs.** Companions can now plan a route up stairs whose railings used to block every path.
- **Close follow at your pace.** Companions that used to jog beside you while you walked now walk.
- **No more looping.** A companion that used to swing back and forth while catching up now heads straight for you.

### Survivors
- **Survivors walk straight and keep up.** The survivor NPCs no longer circle or skate: they walk in a straight line, their legs match how fast they really move,
  they wander at a walk, and they can keep up with you at a sprint.

### Known issues (being fixed next)
- A companion can walk off the edge of a sloped roof while following you on it.
- A survivor that falls from a roof may not climb back up the ladder to you.

---

## What's New in 3.2.05

Targets 7 Days to Die **3.2**. Built and tested against 0-SCore **3.2.29.1709** - use that or newer.

### A new way to install
The mod is now **`0-XNPCCore`** plus an optional add-on for AI and voice. See **Installing** at the
top of this file, and **delete your old `1-XNPCVoiceControl` folder** first.

### Survivors
- **New survivor NPCs, male and female, in three tiers.** Each one rolls its own face, hair, outfit
  and weapon when it spawns, and keeps that look for good. They can be hired like any companion.
- **They spawn in the wild by biome**, tougher tiers in harder biomes, at the same rate the old
  friendly NPCs did. The old friendly human spawns are switched off; Harley's bandits and the
  animal companions still appear.
- **Armour is real.** Survivors wear vanilla armour that protects them the same way it protects
  you. Heavy armour only turns up on tier 3.
- **Weapons come with ammo.** Survivors carry a weapon suited to their tier and the ammo to fire
  it. Tier 1 survivors carry an empty pipe pistol and fight with melee.

### Performance
- A newly spawned survivor now builds its model once instead of four times.

---

## What's New in 3.2.04

Targets 7 Days to Die **3.2**. Requires 0-SCore **3.2.15.1547** or newer.

> Covers **3.2.03 as well** - that build went out without release notes, so everything since
> 3.2.02 is described here.

> **Since release: use 0-SCore 3.2.19.1009 or newer if you can.** 3.2.15 is the minimum this
> version was built against, but every SCore build before 3.2.19 carries a bug that could leave a
> companion parked on Stay permanently missing from your save - no error, nothing in the log. It is
> fixed upstream in 3.2.19.

The biggest movement release so far, and the companions finally sound like different people.

### Fifty Voices
- **Forty new voices**, on top of the ten you already had. Every personality now has its own
  voice, chosen to suit the character rather than picked from a short list. In a settlement you
  should no longer hear the same voice coming from three different people.
- Voices have **friendly names** now (Alex, Sarah and forty more), so a personality can name one
  in plain English.
- **The old Kokoro voice names have been retired.** They had not been real voices for a while -
  every one of them quietly resolved to one of ten. If a config still names one, the log now says
  so and tells you what to use instead, rather than silently substituting something else.

### They Stay Where You Put Them
- **Companions settle properly on slopes and ledges.** Assign one to a position on a hillside and
  it stands where you put it, at the right height, instead of drifting downhill, hovering, or
  walking on the spot.
- **The "Far" follow distance works.** Setting it to Far used to make a companion pace back and
  forth forever, never settling - it was trying to reach a spot that its own catch-up logic kept
  dragging it away from. Close, Normal and Far all hold their distance now.
- **They stop crowding you when they arrive.** A companion catching up used to walk all the way to
  your shoulder before backing off to its proper distance. It now stops where it belongs.
- **Follow releases a formation position.** Pressing Follow on a companion standing in an assigned
  spot used to do nothing at all. It now returns them to normal following, the same as saying
  "on me".

### When They Cannot Get There
- **A locked door ends the trip.** They try briefly, say they cannot get there, and come back,
  instead of grinding against it.
- **An unreachable perch is refused** rather than attempted.
- **A companion that cannot find a route gives up** instead of rerouting forever.
- **A genuinely stuck companion is recovered** - a short hop first, and only then a rescue.
- They can **stand on wedges, ramps and stairs** again, and will **look downhill** for a way to a
  spot below you.

### Talking
- **Your own companions answer from 40 metres.** Strangers only respond when you are close, within
  15, so walking into a settlement no longer sets off everyone in it.
- **Only the leader can give movement orders.** Anyone can still hire, chat, and ask for a story.

### Other Fixes
- **The backpack closes when a fight starts.**
- Refused move orders now **say why** they were refused.

### Notes
- **0-SCore 3.2.15.1547 or newer is required.** It carries two fixes reported from this mod:
  NPCs can open ordinary doors again, and the vision check no longer measures from the wrong
  place.
- The download is now **Core**, **Voice** or **Complete**. Core is the small one for servers and
  players who do not want the AI; Complete has everything.
- Server and clients must both be on 3.2.04.
- No configuration changes needed. If you have edited `personalities.xml` and used an old Kokoro
  voice name, the log will tell you which line to change.

## What's New in 3.2.02

Targets 7 Days to Die **3.2**. Requires 0-SCore **3.2.7.728** or newer.

### A New Brain
- The chat model is now **Qwen3-4B-Instruct**, replacing a Llama model from 2024. Companions
  follow instructions better and write with more character.

### Things That Were Quietly Broken
- **Greetings work again.** A companion decided it could not see you even face to face, and once
  you spoke to it, it would never greet you again until you walked away and came back.
- **You can read the whole reply.** The subtitle showed only the first sentence while the rest
  was spoken. It now follows the speech, three lines, larger text.
- **Orders are acknowledged even when a fight starts.** A long conversation is still cut short.
- **"Open inventory" tells the truth** when it cannot open.
- **Your first sentence is fast** - the AI warms up during loading.
- **Companions walk onto grass** instead of stopping a step short.

### Notes
- Server and clients must both be on 3.2.02.
- No configuration changes needed.

---

## What's New in 3.2.01

Targets 7 Days to Die **3.2**. Requires 0-SCore **3.2.7.728** or newer. A movement and
responsiveness release - everything here came out of field testing.

### They Go Where You Point
- **Grass is no longer a wall.** Pointing at a spot in a field and sending a companion there put
  them a step short of it, every time. Grass, weeds and small bushes were treated as obstacles -
  and then the same grass was treated as solid footing one block up, so the destination slid
  sideways and upward. Both halves are fixed.

### They Move More Sensibly
- **No more crowding.** A companion holding near you could drift into a slow circle and nudge you
  along the way.
- **No more surge on a reposition.** Swapping companions from one side of you to the other made
  them briefly sprint, because anything past eight metres jumped a full step in speed. They run to
  catch up only when you are genuinely moving away.
- **Standing at a post is not "stuck".** A companion doing exactly what it was told could still be
  teleported a few metres.

### Faster To Respond
- **Wake-word commands act about twice as fast** once you stop speaking - roughly 1.1 seconds
  instead of 2.2. Push-to-talk was never affected.

### Notes
- Server and clients must both be on 3.2.01; the version is part of the join handshake.
- No configuration changes needed.

---

## What's New in 3.2.00

Targets 7 Days to Die **3.2**. Requires 0-SCore **3.2.7.728** or newer.

### The Command Wheel Became a Menu
- **Pages, not one crowded ring.** The wheel opens on the orders you use most — follow, stay, guard, jump down, crouch — with `Stance`, `Distance` and `Formation` leading to their own pages.
- **Hover to open, sweep to ignore.** Rest on a page button for a moment and it opens. Move across it on your way to something else and it leaves you alone. Hover the centre to go back.
- **Release gives the order to the companion you are looking at; click gives it to everyone.**
- **Pages look different from orders** — a small arrow and a coloured ring. Both ring colours are yours to change in `modconfig.xml` under `<FormationRadial>`.
- **Every button has an icon.** The formation page shows eight arrows pointing where the companion will actually stand, and follow distance reads in real metres: `1m`, `2.5m`, `5m`.

### Crouch Is One Button
- It reads `Crouch` when a companion is free and `Resume` when it is already locked down, so one press does the obvious thing. There is no `Stand` button — resuming while you are standing already stands them up. Voice still has `stand up` if you want it explicitly.

### Fixes You Will Notice
- **Interrupting a companion no longer eats their reply.** Cutting a story short could leave the acknowledgement clipped or silent. It now plays in full every time, single player and dedicated server both.
- **Long stories can be interrupted on a dedicated server.** They already could in single player, but only by accident; it is now a deliberate rule that behaves the same in both.
- **The in-game "Accurate" speech setting works.** It pointed at a speech model no longer included in the package, so it could fail to start listening.

### Also
- `Engage` is now called `Stance`. The orders and voice commands behind it are unchanged.
- The printable command card and the manual have both been rewritten for the new wheel.

---

## What's New in 3.1.01

### Send a Companion Somewhere
- **"Go there"** — look at a spot and say `go there`, `move there`, `head over there`. Works on a floor, a doorway, a parked car, or a specific zombie. If you are pointing at something with a name, they say it back: *"Heading to the wooden door."*
- **They actually arrive now.** Short-range orders were being handed to the pathfinder as "walk to where you already are", so a companion sent three metres would simply stand there. That one fault was behind most of the "he won't move" reports, in following as well as commanded moves.
- **A companion sent upstairs stays upstairs** — commanded positions were being dragged down to your floor every tick, which is why they ran back down the stairs.
- **They will not be delivered somewhere they could not walk to.** If a spot is genuinely unreachable they say so instead of appearing inside it.

### They Move Like They Mean It
- **Companions run to catch up** and ease off as they arrive, instead of ambling the whole way at walking pace whenever you stood still.
- **Companions no longer crawl under railings and deck overhangs.** The game was ducking them into gaps you cannot fit through yourself, and they would then sit under a porch instead of walking round to the steps.
- **They get themselves free** when wedged — a short hop to solid ground and walk on, rather than teleporting.

### Combat: Three Stances
- **`weapons free`** (also `hunting mode`) — hunts and attacks on sight, and closes on enemies freely.
- **`cover me`** (the default) — defends you and anything that comes within about eight metres of you, but **holds position** rather than charging off across the map.
- **`full control`** (also `hold fire`, `weapons tight`) — self-defense only. They ignore threats to you until you say otherwise.
- **`disengage` is now an action, not a mode.** Say `disengage`, `break off` or `fall back` and they drop the fight, cancel their formation, run back to you and settle into `cover me`.
- **They always defend themselves**, whatever the stance. No order stops a companion fighting back.
- **They finish the fight they start.** Previously a companion would fire one shot and walk away: hitting an enemy made it turn on them, which cancelled the very condition that had permitted the attack. They now stay on an enemy until it is dead or the fight moves away from you.

### Also
- **A printable command card** — `voice-command-card.html` in this folder lists every command and its alternatives on one page. It marks which commands need a hired companion, which are not working yet, and which settings are the defaults.

### Known Issues (carried forward from 3.1.01)
- **`loot around here` does nothing.** Companions are not given any looting behaviour by the underlying NPC framework, so the order is accepted and then ignored. Marked on the command card.
- **Asking for items only works if you gave them the item first.** Companions carry nothing spare of their own, so `i'm thirsty` answers *"Sorry, I don't have that on me"* unless there is a jar of boiled water in their inventory. Open their pack with `show me what you have` and put things in. The card names the exact item and amount for each request.
- **`jump up` needs a ladder.** Without one nearby the command does nothing, despite the wording suggesting they will climb anything.
- **A companion cannot climb onto a raised porch** whose deck sits over an open gap. They will use the stairs instead. Solid, ground-reaching blocks are stepped up normally.
- **Companions will talk to you again promptly after a fight** instead of staying silent for a long stretch.
- **`go there` no longer sends a companion to where they already are** when they happen to drift across your aim.

---

## What's New in 3.1.00

*Version numbering now tracks the base game — this release targets 7 Days to Die 3.1.*

### Formation Radial
- **Press `;` for an eight-direction radial** — click a compass point to send a companion there. The four diagonals could not be reached by any means before.
- **Pick who you are commanding** — the radial targets whoever you are looking at, otherwise the last companion you gave an order to, otherwise the nearest. Their name shows in the middle of the ring, and the mouse wheel cycles through your hires.

### New Orders
- **Take point** — sends a companion ahead of you and *keeps* them ahead as you turn, unlike the compass orders which hold a fixed bearing. Cancel with `on me` or `break formation`.
- **Directions relative to your facing** — `cover the front`, `watch the rear`, `cover my left`, `cover behind me`. Resolved from where you are looking when you give the order, then held.
- **Shield formations** — `shield the right` fans the squad across five positions centred on that side instead of stacking them; `full shield` covers all eight; `shield wall` forms up across your front.
- **Diagonals by voice** — `cover northeast`, `hold the southwest`, in one-word and two-word forms.

### Companions Get Themselves Unstuck
- **Recovery actually works now.** When a companion was wedged, the mod asked the game for a route and then gave up before that route had finished computing, decided it had succeeded, and walked them back into the same obstacle indefinitely. Both escape hatches that should have rescued them were disabled by the same fault.
- **Stuck detection is about three times faster** — every timer in the follow logic was counting at roughly a third of real speed, so a delay meant to be a second and a half took nearly five.
- **Companions open doors when leading you**, not only when following.
- **A companion that genuinely cannot reach where you sent them** is moved there after a few attempts rather than grinding against a wall forever.

### Fixes
- **Squad direction orders only ever worked for north** — every other direction silently sent the whole squad north.
- **Distance sticks** — telling a companion to come closer and then giving any direction order no longer resets them to the default five metres.
- **Companions can talk about fifteen seconds sooner** after you load in.
- **Speech recognition is actually steered by companion names now** — the list of nearby names and command words was being sent to the recognition server in a form it never reads.
- **Two companions with the same name no longer cost you the order.**
- **A longer command is no longer swallowed by a shorter one** — "cover southeast" was resolving as "cover south".
- **Clicking the formation radial no longer swings your weapon.**
- **Orders are acknowledged on screen as well as aloud**, so the mod is fully usable with text-to-speech switched off. Subtitles are translated properly now, where before they were English regardless of your language.

### Known Issue
- **Companions approaching an enemy can run into walls.** This is in the shared NPC framework (SCore) rather than this mod — it uses a direct-line approach that never asks for a path when the target is standing still. Root-caused and written up for the framework author; a local workaround is planned.

---

## What's New in 3.0.08

### Tactical Mode
- **Player-wide command-only mode** — toggle with `go tactical`/`lock in` (ON) or `at ease`/`stand down`/`normal mode` (OFF). In tactical mode, unrecognized phrases get a rotating clarification ("Say again?", "What do you need?") instead of routing to LLM chat. Suppresses proactive greetings and RAG flush.
- **`<DefaultTacticalMode>`** config option — set `true` in `modconfig.xml` to start all hires in tactical mode (applied once per player per session; "at ease" then sticks).

### Formation System
- **World-anchored cardinal sockets** — `cover north`, `hold south`, `watch east`, `cover west`. Sockets reference map compass directions (not leader heading) — NPCs stay on their side when you turn.
- **Adjacency allocator** — when a second NPC is ordered to an occupied cardinal, it auto-spreads to the nearest free diagonal. "Billy, cover north" then "Sam, cover north" → N then NE.
- **Squad fan-out** — `squad, cover north` spreads the squad across N/NE/NW automatically.
- **Unified mode-aware distance** — one vocabulary ("come closer", "back off", "normal distance") works in both plain follow and formation mode. The system knows your mode and adjusts the right value.

### Combat
- **Engagement model** — `weapons free` (free-fight), `cover me`/`defend only` (assist-only default), `disengage`/`break off` (passive, clears targets + cancels formation).
- **Live crouch** — `kneel`/`get down`/`stand up` — SCore ships the CrouchOverride hook at both follow and idle chokepoints.

### Memory
- **Improved long-term memory** — global player identity (same memories across NPCs sharing your player ID), minimum flush threshold so short conversations don't waste extraction cycles, junk-fact filter removes placeholder contamination, and high-importance memories persist longer than casual facts.

### Settings
- **Hired chat distance** increased from 30m to 40m maximum.

### Bug Fixes
- TTS permanent-silence bug (queue hang after one response)
- Mid-sentence audio cutoffs (watchdog timeouts added)
- GiveItem over-removal from NPC stock
- RAG memory contamination (few-shot examples leaking as real facts)
- Streaming subtitle showed literal action tags (`[follow]`, `[trade]`)
- Asterisk emotes (`*shakes head*`) showing in subtitles
- First-interaction RAG delay (unrelated NPC backlog blocking active chat)
- Various dialogue cleaning and emote stripping fixes
