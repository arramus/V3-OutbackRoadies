# 1-XNPCVoiceControl - Changelog

Historical release notes, newest first. The release being shipped (3.3.03) keeps its notes
in `ReadMeFirst.md`, next to this file.

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
