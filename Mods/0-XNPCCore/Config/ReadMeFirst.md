# NPCCore — Read Me First

**Upgrading from 3.2.04 or earlier? DELETE your old `Mods/1-XNPCVoiceControl` folder (and any old `0-XNPCCore` / `0-XNPCCore2`) before installing - do not unzip over them.** The mod now installs as `0-XNPCCore`, and an old copy left in `Mods` still loads and clashes with it.

NPCCore fills your world with **survivors you can hire and lead**, and **bandits** who hunt you. Your hires follow you, fight beside
you, loot, harvest, mine, craft at your workstations, read books, guard your base and ride in your vehicles. Talking to them by
**voice**, with **AI chat**, is an optional add-on: everything below works without it.

---

## Installing

Unzip into your `Mods` folder:

1. **`0-XNPCCore-v3.4.04.zip` - required.** The survivors, the bandits, every order and menu, and the mod's code. On its own it runs
   with every voice and AI feature off.
2. **Plus at most ONE add-on, if you want voice:**
   - **`1-XNPCVoiceControl-Complete-v3.4.04.zip`** - AI chat, voice in and voice out. **Most players want this one.**
   - **`1-XNPCVoiceControl-Voice-v3.4.04.zip`** - voice in and voice out only, no AI chat.

You should end up with `Mods/0-XNPCCore` and, if you took an add-on, `Mods/1-XNPCVoiceControl-Complete` or `Mods/1-XNPCVoiceControl-Voice` beside it. The add-on holds only models and helper servers; it does nothing without `0-XNPCCore`.

**Requirements:** **7 Days to Die 3.4** and **0-SCore 3.4.14 or newer** (on a server: the server and every player). A newer SCore can rename something the mod patches: the log then names each patch that did not apply (`[SDCS-PATCH] ... NOT applied`) and the rest of the mod still runs. If you are still on game 3.2, use mod version 3.2.06 instead. It does NOT need the older `0-XNPCCore` (v3.1.00) - this mod replaces it (see the top of this file).

**On a dedicated server:**

- **The server installs `0-XNPCCore` only.** The two add-ons are **client side only**: they hold the voice and AI models and their
  helper servers, which run on each player's own machine. Never put an add-on on the server; it needs no GPU and starts no AI servers.
- **Every player needs `0-XNPCCore`,** the same version as the server. A player without it is refused at join with what looks like a
  version mismatch (see *Multiplayer and Dedicated Servers* below).
- **The add-on is each player's own choice:** one player can run Complete, another Voice, another none.

---

## Getting Started

### Find and hire a survivor

Survivors live in towns and out in the wild. Walk up to one and press **E** to talk; hire them from that menu. A hire is paid weekly:
the cost grows each week up to a cap, they ask you when it is due, and a missed payment ends the contract after a few grace days
(`<Billing>` in modconfig.xml). Only their leader can give them orders.

### How to command your hires

Three menus, each with one job, so every command has one home:

| Method | Reach | Use it for |
|--------|-------|------------|
| **Press E** on them | Up close: the game's own "Press E" reach, about 3.5 m in a survival game (a game value, not a mod setting) | The **top menu**: talk; hire or dismiss; **Follow me**; **Come here**; **Show me your inventory**; **Craft...**; and two links: **Order wheel** (opens the **;** wheel on them) and **Jobs and gear** (their hold-E menu, as a wheel page, for players who never hold E) |
| **Hold E** on them | The same reach | **Work and gear**, one hire at a time: **Jobs**, **Roles**, **Weapons** (carried weapons, each class, empty hands, what they carry, ammo), **Emotes**, **Read** (books), **Progress**, **Inventory**, **Pick me up**. Rest the cursor on an entry or let go of E over it; a quick tap of E still talks. Jobs, Roles and Read show only on a survivor who can do that work |
| **Order wheel**: the **;** key | Any distance | **Tactics**, for one hire or the whole squad: follow, stay, guard, come here, stance, **positioning** (follow distance, take point, go there, jump down, get in / out of your vehicle), formation, crouch, scout, guard and return, patrols |
| **Voice** (Voice / Complete add-on) | A hire within 40 m, a stranger within 15 m | Everything by speech (see *Voice and AI add-on*) |
| **Text chat** | The nearest NPC | Press **T** and type `@your message` in global chat (the `@` targets the nearest NPC) |

### The Order Wheel

Press **;** (default; remappable in `modconfig.xml` under `<FormationRadial><Key>`; free keys: Semicolon, Quote, Comma, Period, Slash,
RightBracket, Backslash; `None` turns it off) and the order wheel opens and stays open - no key to hold. It works at any distance,
so it is how you give orders to hires out of reach and call them back:

- **Who it targets** - whoever you are looking at, otherwise the last companion you ordered, otherwise the nearest. Their name sits
  in the middle of the ring, and the mouse wheel cycles through your hires.
- **Order** - **click** an order for the targeted companion; **Shift+click** gives it to your **whole squad** (formation bearings
  included). Patrol, weapon and job orders always go to the targeted companion only.
- **Pages** - click an entry marked with a chevron (blue-tinted border) to open its page; click the middle of the wheel to go back.
  Press **;** again, right-click or Esc to close it without an order.
- **First page** - Follow, Stay, Guard, **Come here** (calls them back from any job), then the **Orders**, **Stance**, **Position**
  and **Formation** pages.
- **Orders** - Crouch (a toggle: **Crouch** or **Resume**), Stand up, Scout, Guard and return, and the patrol trio (Record / Run /
  Cancel patrol).
- **Stance** - Aggressive / Defensive / Hold fire / Disengage.
- **Position** - Close (1 m), Normal (2.5 m) and Far (5 m), drawn as the real metres; Point (they lead 2 m ahead of you); Go there
  (the spot under your crosshair when the wheel opened); Jump down; Get in / Get out (your vehicle).
- **Formation** - the eight compass bearings (north at the top). Each companion keeps its own distance; the socket sets the direction
  only.
- **One hire's work and gear live on their hold-E ring** (or **Press E > Jobs and gear**), not on this wheel: **Jobs** (Follow + loot,
  Loot here, Harvest, Mine, Base Repair, Combat Repair, Remember base), **Roles** (Medic, Cook, Blacksmith, Chemist, Crafter: a survivor
  uses a workstation only with its role), **Weapons**, **Emotes**, **Read**, **Progress**, **Inventory** and **Pick me up**. Their craft
  window is **Press E > Craft...**.

---

## What Your Hires Can Do

Each line names the `MANUAL.md` section with the details.

- **Follow, stay and guard** - they keep their distance (Close, Normal, Far), take a formation bearing, take point, go where you
  point, patrol a route you record, and guard a spot (several guards on one spot spread out). *Order Wheel.*
- **Fight** - four stances (Aggressive, Defensive, Hold fire, Disengage); they take cover and peek, swap to melee up close, heal
  themselves with what they carry, and get more accurate as they level up. *Survivors' aim; Survivor Settings.*
- **Jobs** (hold E > Jobs) - follow and loot as you go, loot around a spot (bags, containers, and fresh kills they butcher), harvest
  wood, plants and crops, mine ore, clay and stone, and keep your base repaired. A full pack pauses the job; empty it and they go back.
  *Hired survivors: follow and loot.*
- **Craft** (E > Craft...) - their own crafting window: their recipes, what they have and need, a count, Craft. At a
  workstation they need its **role** (hold E > Roles): Cook the campfire, Blacksmith the forge, Chemist the chemistry station, Crafter the
  workbench and cement mixer. They get better at the work they do. *Hired survivors: crafting.*
- **Read** (hold E > Read) - give them a schematic or a crafting skill magazine and they learn it. **Hold E > Progress** shows their
  level, roles and skills. *Hired survivors: reading books.*
- **Medic** (hold E > Roles > Medic) - they treat you and the rest of your squad.
- **Ride** - on Follow they get in your vehicle when you drive, and out when you stop (Get in / Get out on the wheel too). They get out
  before you log off or the world saves, so they are never lost from the save.
- **Guard your backpack** - when you die, your following hires guard your dropped pack until you come back.
- **Emotes** (hold E > Emotes) - waves, salutes, dances. *Hired survivors: emotes.*

Not hired: **wild survivors** trade with you (*Trading with wild survivors*) and some pick pockets (*Pickpockets*). **Bandits** spawn by
biome tier, hold posts and fight in groups (*Bandits*; POI builders: `POI-BUILDERS.md`).

---

## Settings

The mod has three places to change things:

| Where | What |
|-------|------|
| **`Mods/0-XNPCCore/Config/Settings.xml`** | Everything about the survivors and bandits: combat, aim, jobs, crafting, looting, emotes, progression. Every key is described in the file and in `MANUAL.md` (*Survivor Settings*). Restart the game after a change. |
| **`modconfig.xml`** | Voice and AI, the order wheel key, hire billing, the NPC text switch (`<Subtitles><NPCText>`: `all`, `clip` or `none`). With an add-on installed, edit **the add-on's** copy, `Mods/1-XNPCVoiceControl-Complete/Config/modconfig.xml` (or `-Voice`): the mod then reads ONLY that file, and the copy inside `0-XNPCCore` is used only when no add-on is installed. Restart the game after a change. |
| **In-game menu** | **Esc > NPC VoiceControl Settings**: voice, chat and distance sliders, debug logging (see *Voice and AI add-on*). |

**`MANUAL.md`**, also in this folder, is the modder's guide: adding a personality or a voice command, every setting, the order wheel,
the voice list, voice clips, modded workstations (`NPCWorkSide`). You need no C# for any of it. **`UAI-MODDING.md`** covers the NPC
behaviour packages and **`POI-BUILDERS.md`** the bandit and guard spawn blocks.

---

## Multiplayer and Dedicated Servers

Multiplayer status: Singleplayer and Listen-host are fully supported. Dedicated servers are now supported — the mod runs NPC AI server-side, and voice commands (orders, follow distance, patrol, formations, squad commands) route correctly to the server. Dedicated server operators install **`0-XNPCCore` only**, with no add-on — the server needs no models or helper servers, because all AI runs on the players' machines. Players install `0-XNPCCore` plus the add-on they want (see Installing above). Both server and clients must be the same version; a mismatch is logged as a warning at join and multiplayer features may misbehave. Listen-host is unaffected — the host installs the same as any player. Lines a survivor builds on the server (their answer to "What are you carrying?", out of ammo, pack full) are spoken by every player's own client within earshot.

> ### ⚠️ Every player must have the mod installed
>
> **This is not an optional client mod.** A player without it is **denied at join**, with a message that reads like a *version mismatch* — and confusingly it shows two identical version numbers, because the versions are not the problem.
>
> The mod adds custom network packet types. When you connect, the server sends its packet table and the client must recognise every entry; one it does not know causes an immediate disconnect. That is a 7 Days to Die rule, not a choice we made, and it is why this mod cannot be server-only the way XML-only mods can.
>
> **If a player reports "version mismatch" but the versions match — they are missing the mod.**
>
> Plan for it: tell your players to install `0-XNPCCore` (plus an add-on if they want AI or voice) before they connect. Support for letting non-users join a modded server is being looked at, but is not available today.

Each player runs their own AI locally — the LLM, speech-to-text, and text-to-speech all run on your own machine, so every player needs the sidecar servers. The dedicated server itself needs no GPU and starts no AI servers; it only relays and arbitrates.

Not yet synchronised with 2+ players on one server: NPC memory is per-player (each client's NPC remembers its own conversations), there's no conversation lock (two players talking to the same NPC at once will conflict), and hire billing isn't arbitrated between clients. Single-player-per-server and singleplayer are unaffected.

---

## Voice and AI Add-on

Everything in this section needs the **Voice** or **Complete** add-on (AI chat: **Complete** only).

### Talking by voice

| Method | How |
|--------|-----|
| **Push-to-Talk** (default) | Look at or walk up to an NPC, hold **V**, speak your message, release. Your own hires answer within 40 m, other NPCs within 15 m |
| **Wake word** (off by default) | Say "Hey Marvin" instead of holding V (see the FAQ below) |

What happens when you speak:

1. Your speech is transcribed to text (Whisper)
2. If it matches a known command ("follow me", "open inventory"), it executes **instantly**
3. If it doesn't match a command, the local AI thinks about it and responds with voice + subtitles (**Complete** add-on)

### Command Card

**`voice-command-card.html`**, in this folder, lists every command and its alternate
phrasings on one page. Open it in any browser and press **Ctrl-P** — it prints to a single
landscape sheet, handy beside the keyboard.

You do not need to memorise exact wording. Phrases match as **substrings**, so
"hey Brody, follow me would you" fires the same command as "follow me".

### First-Time Startup

When you first launch after installing an add-on, expect a **15-30 second delay** before voice and AI are ready. The helper servers (LLM, TTS, STT, embeddings) start automatically. You'll see server status messages in the game console. NPCs won't answer by voice until the servers are online.

### In-Game Settings Menu

Open the escape menu (**Esc**) and click **NPC VoiceControl Settings**. Here's what each control does:

#### Text-to-Speech (TTS)

| Setting | What It Does |
|---------|-------------|
| **Text-to-Speech** toggle | Turn NPC voice on/off entirely |
| **Volume** | How loud NPCs speak (0.0–1.0) |
| **Speech Rate** | Speed of NPC speech (0.5 = slow, 2.0 = fast). Default 1.0 is natural |

#### Speech-to-Text (STT)

| Setting | What It Does |
|---------|-------------|
| **Speech-to-Text** toggle | Turn voice recognition on/off |
| **Wake Word (Hands-Free)** | Say "Hey Marvin" instead of holding V. **Off by default** — it uses extra CPU. See FAQ below |
| **Beam Size (1–5)** | How many candidate transcriptions Whisper weighs. **Ships at 1** — fastest, and in practice it recognises commands very well. **3 is the accuracy sweet spot** of the 1–5 range, but it samples three times, so it costs real time for a gain you usually will not notice. Raise it only if commands are being misheard |

#### Conversation

| Setting | What It Does |
|---------|-------------|
| **Loaded AI Model** | Shows which LLM is loaded (read-only) |
| **Max Tokens (64–2048)** | Maximum response length. Higher = longer replies but slower. The shipped default is 200; raise it if replies cut off mid-sentence |
| **Context Window (3–20)** | How many recent exchanges the AI remembers in conversation. Higher = more context-aware but uses more VRAM |

#### Distance & Range

| Setting | What It Does |
|---------|-------------|
| **Chat Distance (3–15m, default 15)** | How close you need to be to talk to an NPC you have NOT hired |
| **Hired Chat Distance (5–40m, default 40)** | How far away your own hired companions still answer you |
| **Voice Range (5–40m, default 40)** | How far NPCs' voices carry (audio volume drops off after this). Keep this at or above Hired Chat Distance, or you will be able to give an order you cannot hear the reply to. |

#### Facial Expressions

| Setting | What It Does |
|---------|-------------|
| **Procedural blink** (automatic) | NPCs blink naturally at random intervals (2–6 seconds). Works on most NPC models with eyelid geometry. Some models split eye geometry across multiple meshes and won't blink — this is a model limitation, not a bug |

#### Long-Term Memory

| Setting | What It Does |
|---------|-------------|
| **Long-term memory** (automatic) | The mod remembers facts about you across conversations and game sessions. If you tell an NPC your name, what you're building, or where you live, they'll recall it later. Stored per-NPC — each NPC has its own memory of you. Saved to `NPC_Memories.xml` in the Mods folder. High-importance memories (like shared events) persist longer than casual facts |

#### Other

| Setting | What It Does |
|---------|-------------|
| **Chat Timeout (15–120s)** | How long the NPC stays "in conversation mode" after you speak. After this idle period, the NPC forgets recent context and you'd start a fresh conversation. Default 30s is good for most — higher if you like long pauses between exchanges |
| **Mod Debug Logging** | Turn on only if troubleshooting — adds lots of console spam. Saved on this PC: it stays on across mod updates until you turn it off (or type `vcdebug off`) |

#### Test Buttons

- **Test Voice** — plays a sample TTS line so you can hear the voice
- **Test Microphone** — records 3 seconds and shows what Whisper heard
- **Test AI Connection** — confirms the LLM server is responding
- **Clear All Conversations** — wipes chat history with all NPCs

### Common Voice and AI Config Changes (modconfig.xml)

The in-game menu covers most things. For advanced tweaks, edit **the add-on's** `Config/modconfig.xml` (see *Settings* above). **Changes here require a full game restart.**

#### TTS Engine (`<TtsEngine>`)

The voices all come from one model: **Supertonic 3**. Two server binaries can play it, and
`<TtsEngine>` picks the binary - the values are named after the binary, not the voices:

- **`audiocpp`** (the default) - the audio.cpp server, which runs the model from a GGUF file
  (`supertonic-3-f16.gguf`) on port 5057, with Vulkan and a CPU fallback.
- **`supertonic`** - the older ONNX server binary, which runs the same Supertonic 3 model from
  `Resources/Supertonic/onnx/` on port 5054. Kept as a fallback.

The shipped `modconfig.xml` sets `audiocpp` (since 3.2.00). The older Kokoro/sherpa engine was
removed. Set `<TtsEngine>` to `audiocpp` or `supertonic`, then restart. Any other value - for
example the retired `sherpa` - logs a warning at startup and is normalised to `supertonic`;
deleting the line gives the same result (the code default is `supertonic`), so keep the line set
to `audiocpp` to match the shipped config.

#### Speech Recognition Model (`<Model>`)

One model ships: `ggml-small-q8_0.bin` (the small model, quantized, near-lossless). The older
`ggml-base.en.bin` (a faster English-only tier) and the f16 `ggml-small.bin` were removed from
the package, so do not point `<Model>` at either of them.

Leave empty for auto-detect - it prefers the shipped `ggml-small-q8_0.bin`, but keeping the
explicit filename in the config is still better.

#### Push-to-Talk Key (`<PushToTalkKey>`)

Default is **V**. Change to any single key: `<PushToTalkKey>G</PushToTalkKey>`

#### Phonetic Name Matching (`<NameMatchPhonetic>`)

Default **true** — enables Soundex phonetic matching for NPC names (e.g. "bow" matches "Beau"). Set to `false` if you experience false name matches; exact, prefix and fuzzy matching still work.

#### Microphone Device (`<MicrophoneDevice>`)

If voice isn't working at all, your mic might not be the default device. Set it explicitly:

```xml
<MicrophoneDevice>Microphone (Realtek Audio)</MicrophoneDevice>
```

Check the boot log for your exact device name — it prints `[0] Microphone (device name)` on startup.

#### Max Tokens (`<MaxTokens>`)

Maximum response length in tokens. The shipped default is 200. If NPC responses cut off mid-sentence, increase it to 768 or 1024. If responses are too long and slow, lower it.

#### Max Response Length (`<MaxResponseLength>`)

Hard cap on response characters (default 300). Prevents the AI from generating very long speeches. Lower = snappier replies, higher = more detailed answers but slower TTS.

#### Typing Delay (`<TypingDelayMs>`)

Per-character delay for the typewriter subtitle effect (default 75ms). Values: 50 = fast, 75 = natural, 150 = dramatic. Set to 0 to disable the typing animation entirely.

#### Context Memory (`<ContextMemory>`)

How many recent exchanges the AI keeps in its short-term context window during conversation. The default is 5.
- **3–5** (default) — remembers last few things you said, lowest VRAM usage
- **8–10** — more context-aware, remembers longer conversations, uses more VRAM
- **15–20** — very context-aware but may repeat old topics and uses significant VRAM

When the conversation grows beyond this limit, older messages are trimmed from the LLM prompt (but facts you told NPCs are saved to long-term memory automatically).

#### KV Cache Size (`<ContextSize>`)

Controls how much VRAM the LLM server reserves for its context window. Lower values help on GPUs with limited VRAM. The default 8192 works for most cards with 8 GB+ VRAM.

| ContextSize | VRAM Used | When to Use |
|-------------|-----------|-------------|
| **2048** | ~224 MB | Bare minimum — very limited conversation memory, last resort for 4 GB cards |
| **4096** | ~448 MB | 6 GB VRAM cards; use with IQ4_NL model |
| **8192** (default) | ~896 MB | Recommended for 8 GB+ VRAM (RTX 3060 / 4060 class) |
| **16384** | ~1.75 GB | 12 GB+ cards (RTX 3080 / 4070 Ti class) |
| **32768** | ~3.5 GB | 16 GB+ cards (RTX 3090 / 4090 class) |

Values above 32768 waste VRAM with no gameplay benefit. If your game stutters or crashes on launch, try lowering this to 4096.

#### Supertonic TTS Speed (`SupertonicConfig.xml`)

This section only applies when `<TtsEngine>` is `supertonic`; the default `audiocpp` engine does not read `SupertonicConfig.xml`. If NPC voice synthesis feels slow on a weaker CPU with the supertonic engine, two settings in `Config/SupertonicConfig.xml` can help:

| Setting | Default | Faster Value | Trade-off |
|---------|---------|-------------|----------|
| `<TotalStep>` (denoising steps) | 6 | **4 or 5** | Slightly less polished audio, noticeably faster. At 4 most users won't notice quality loss |
| `<IntraOpNumThreads>` | 4 | **2** | Less CPU contention on quad-core CPUs; slower isolated synthesis but smoother overall game performance |

Neither is in the in-game menu — edit the XML directly and restart the game.

**Note:** The in-game **Speech Rate** slider controls how fast or slow the voice *sounds* (playback speed). These settings control how long it takes to *generate* the audio. They're separate.

---

## FAQ

### Why won't the NPC follow my orders?

The NPC must be **hired** (have you as their leader). Leaderless NPCs will chat with you but refuse orders like "follow me". Press **E** on them and hire them first (or say "hire you" with a voice add-on).

### The NPC keeps saying "I don't take orders from just anyone"

You're talking to a wild NPC that isn't hired yet. Hire them first (E, or say "hire you"). Traders are an exception — they'll always open their inventory regardless of leader status.

### What are "phrase triggers"?

Pre-written commands that bypass the AI entirely for instant execution: "follow me", "stay here", "open inventory", "use your knife", etc. They work by voice and by text chat (`@follow me`). They execute in under 1 second because no AI thinking is involved.

When you give a command, your NPC nods to confirm acceptance or shakes their head to refuse — subtle head gestures that respond to whether they can comply (hired status, item availability, etc.).

### What are aliases?

Each phrase trigger can have alternative names called **aliases**. For example, the "open inventory" trigger also matches "show me what you have" and "let me see your stuff" as phrases, and "backpack", "gear", "bag" or "inventory" as aliases — all defined in `phrasetriggers.xml`. You don't need to memorize exact phrases; if you say something close enough, it'll match. Modders can add their own aliases by editing the XML.

### Can I add my own voice commands?

Yes — edit `Config/phrasetriggers.xml`. Each trigger has a phrase pattern and a response. The XML is well-commented with examples. See `MANUAL.md`, *Quick Recipe: Add a New Voice Command*.

### "Tell me a story"

This phrase trigger plays a pre-written story from the NPC's personality pool — instant response with no AI thinking involved. Each personality has multiple stories, picked at random.

### Voice and AI questions

#### Why is speech recognition slow?

Whisper runs on your GPU via Vulkan. On older GPUs (GTX 10-series or older) or when the GPU is busy rendering, transcription can take several seconds.

**Mitigations:** lower **Beam Size** to 1 in the in-game settings (the default) — beam search is the main accuracy-vs-speed trade-off. If the server grabbed your integrated GPU instead of the discrete one, set `<WhisperGpuDevice>` in `modconfig.xml` to the discrete card's index (the `[whisper-server]` log lines list them).

#### Can I use wake-word mode?

Yes, but it uses extra CPU while listening. To enable: toggle **Wake Word (Hands-Free)** in the in-game settings, or set `<WakeWordEnabled>true</WakeWordEnabled>` in `modconfig.xml`. Say "Hey Marvin" to start recording — it stops after silence.

#### The voice sounds a little mechanical on long passages

On the `supertonic` engine this is default behavior on longer passages: it is a diffusion model with 6 denoising steps, so quality is good but can sound slightly mechanical on story-length text. This is expected and not a bug. Switching to the default `audiocpp` engine will not change the voice itself - both engines run the same Supertonic 3 model.

#### Can I use my own LLM model?

Yes. The mod ships with Qwen3-4B-Instruct (a 4B-parameter model) optimized for NPC dialogue. You can swap it for a larger or different model:

**1. Download a `.gguf` file** — any llama.cpp-compatible GGUF model works (HuggingFace, TheBloke, etc.). Quantized models (Q4_K_M, Q4_0, IQ4_NL) are recommended.

**2. Place it in the folder:**
```
1-XNPCVoiceControl/Resources/Models/Llama/
```
Remove or rename the old `.gguf` file first — only keep one model in the folder.

**3. (Optional) Specify by name in modconfig.xml:**
```xml
<ModelFilename>your-model.gguf</ModelFilename>
```
If omitted, the mod auto-discovers the first `.gguf` it finds.

**4. Restart the game.** The server loads the model on startup and prints its name to the console.

#### What model size should I use?

| Model Size | VRAM Needed | When to Use |
|------------|-------------|-------------|
| **4B** (included, Qwen3-4B) | ~2 GB | Default — fast, good enough for NPC dialogue |
| **8B Q4_K_M** | ~5 GB | Better reasoning and personality. Needs 8 GB+ VRAM card |
| **8B IQ4_NL** | ~5 GB | Higher quality quantization of 8B models |
| **13B Q4_K_M** | ~8 GB | Excellent quality, needs 12 GB+ VRAM card |

**Important tuning when upgrading to a larger model:**

- **Increase Max Tokens** — the shipped default of 200 is fine for the included 4B model. An 8B or 13B model can handle longer responses, so bump to **768–1024** in the in-game settings.
- **Increase ContextSize** — the default 8192 KV cache may bottleneck a larger model. Try **16384** for 8B models, **32768** for 13B (see VRAM table above).
- **Increase ContextMemory** — larger models benefit from more conversation history. Try **8–10** instead of the default 5.

#### The AI only speaks English

The included Qwen3-4B model is multilingual but strongest in English. For stronger multilingual output you can swap in a different model (see the section above).

**Licences of the bundled AI parts**

The chat model is Qwen3-4B-Instruct-2507 by the Qwen team (Alibaba Cloud), released under the Apache License 2.0; the full text is in `Resources/Models/Llama/LICENSE-Qwen3`. The chat server is llama.cpp (MIT licence; the notice ships in `bin/LlamaServer/LICENSE-llama.cpp`). Another model can go in at any time: put its `.gguf` in `Resources/Models/Llama/` and name it in `<ModelFilename>`.

---

## Troubleshooting

### Game hangs on exit

Known Steam/7DTD issue. Just wait or click Stop in Steam. Not caused by the mod.

### NPC doesn't respond to anything

1. Are they hired? Only their leader can give them orders (see the FAQ).
2. Up close, press **E**: does their menu open? If it does, the mod is running and the problem is voice or AI.
3. With a voice add-on: are you within chat distance? Default is 15m for an NPC you have not hired, 40m for your own hire — walk closer if you are beyond it
4. Is the LLM server running (Complete add-on)? Check console for `llama-server is accepting connections on port 5055`
   The LLM uses port 5055 (moved off 8080 to avoid collisions with web admin panels like AMP).
   Upgrading and hand-edited modconfig.xml? Update Endpoint to 5055.
5. Is there a `.gguf` file in `Resources/Models/Llama/`?
6. Try text chat (`@hello`) — if that works, it's a voice issue, not an AI issue

### Voice recognition isn't working at all

1. Check `bin/WhisperServer/` contains `whisper-server.exe`, `whisper.dll`, and `ggml*.dll` files
2. In-game settings → click **Test Microphone** — does it show what you said?
3. If mic test fails: check your `<MicrophoneDevice>` setting matches your actual device name (see boot log)
4. If whisper-server didn't start, look for error lines in the game console

### NPC voice isn't playing

1. Check TTS is enabled in the in-game settings
2. Click **Test Voice** — does it play?
3. Verify the TTS server started: the startup console log prints a server health summary that lists the active engine as ready (`audiocpp` by default, or `supertonic` if you switched)

### The LLM response cuts off mid-sentence

Increase **Max Tokens** in the in-game settings (try 768 or 1024). The shipped default of 200 is enough for most conversations, but long stories need more. Speech-recognition slowness is covered in the FAQ above.

### I hear two voices overlapping

This shouldn't happen in the current version. If it does, interrupting an NPC mid-speech should stop the old voice and start the new one immediately. Report this as a bug with your Player.log.

---

## Console Commands & Diagnostics

Press **F1** to open the console. Every mod command starts with `vc` (or the longer `voicecontrol`). `vcdebug` turns the mod's verbose logging on or off — turn it on before reproducing a problem. `<VerboseLogging>true</VerboseLogging>` in `Mods/0-XNPCCore/Config/Settings.xml` turns it on too; either one is enough, and `vcdebug status` shows both. (`dm` is the game's own debug menu, not this mod's logging.)

On a dedicated server, `vc` and `vcdebug` run on **your own** machine (your speech, captions, faces and log) and any player may use them; to see the server's side, use the server's own console. The `sdcs...` field tools are for single player or a listen host, except `sdcscustom spawn` (admins, from any machine).

**General**

| Command | What it does |
|---------|-------------|
| `vc status` | Mod state, server connections, request counts, response times, active NPCs |
| `vc test` | Test the connection to the LLM server |
| `vc talk <message>` | Send a message to the nearest NPC within 15 m |
| `vc action <action>` | Order the nearest NPC: `follow`, `stop`, `guard`, `wait` |
| `vc list` | List active NPC chat sessions |
| `vc clear` | Clear all conversation history |

**Speech out (TTS)**

| Command | What it does |
|---------|-------------|
| `vc tts` | Show TTS status |
| `vc tts test` | Speak a test line |
| `vc tts on` / `vc tts off` | Turn spoken replies on or off |
| `vc tts voices` | List available voices |

**Speech in (STT)**

| Command | What it does |
|---------|-------------|
| `vc stt` | Show STT status |
| `vc stt test` | Test your microphone and transcription |
| `vc stt on` / `vc stt off` | Turn voice input on or off |
| `vc stt devices` | List available microphones |

**When something is wrong**

| Command | What it does |
|---------|-------------|
| `vc selftest` | Full check: config, personalities, servers, STT/TTS/LLM |
| `vc npcdiag` | Dump the world's NPC roster with persistence fields |
| `vc patrol` | Show the nearest hired companion's patrol state |
| `vc patrol clear` | Clear recorded patrol points and stop patrolling |
| `vc reloadface` | Reload lip-sync config from `modconfig.xml`, no restart *(dev)* |
| `vc facescan` | Scan nearby NPC face rigs *(dev)* |
| `vc hires` | Print both halves of every hire link (your hire records and each NPC's leader): for a report about a hire who stopped obeying or vanished |

**Emotes**

| Command | What it does |
|---------|-------------|
| `vc anim play <clip id> [seconds]` | Play an emote on the survivor in front of you |
| `vc anim off` | Stop it |

### NPCCore commands (`sdcs...`)

These come with `0-XNPCCore` and work without any add-on. Most run on the host: single player, a listen host, or the dedicated
server's own console (each line says where).

**For players and server admins**

| Command | What it does |
|---------|-------------|
| `sdcsbandit` | Show the damage bandits deal to players, and the bandits in the world |
| `sdcsbandit damage <x>` | Set that damage live (a multiplier on the bandit's weapon damage, players only: 1 = full, 0.5 = half). Lasts until the game restarts; `<Bandits><PlayerDamage>` in Settings.xml is the value at start |
| `sdcsbandit group <2\|3>` | Spawn a bandit group of that tier beside you, for testing |
| `sdcscustom` | Custom characters: list or apply profiles (single player / listen host), or spawn one of your uploaded characters near you (`sdcscustom spawn`, admins, any machine) |
| `sdcsbase` | Base keeper: list the damaged blocks a keeper would repair at your position; `snapshot`, `holes` and `sight` show the remembered base, its missing blocks and what a keeper can see (single player / listen host) |
| `sdcsmedic` | The nearest hired survivor checks you for wounds and treats you now if they can (single player / listen host) |

**Test and diagnostic tools** (mostly for bug reports and modders)

| Command | What it does |
|---------|-------------|
| `sdcsperf on [seconds]` / `sdcsperf off` | Time the NPC AI: each survivor's update cost and the costliest actions; the report goes to the log |
| `sdcsbench` | A stress run: bandits ahead, zombies behind, the AI timed for the run, then the spawns removed |
| `sdcscover bench\|spots\|threats\|lean\|crouch ...` | Cover tools: the ray cost, the cover spots around you, nearby NPCs' threat lists, and test poses |
| `sdcsthrow list\|<pipebomb\|grenade\|contact\|charge\|molotov>\|give ...` | Throwables: what the nearest NPC carries, make them throw one where you look, or give test supplies |
| `sdcsdodge list` | The live explosives and each nearby NPC's dodge verdict |
| `sdcspose` | Capture or list seated weapon poses (single player / listen host) |

**Formation by console** — `vc formation <bearing> <metres>` sets a world-anchored socket on the nearest hired companion. Bearing is compass degrees (0=N, 90=E, 180=S, 270=W) or a cardinal name (`n`, `ne`, `e`, `se`, `s`, `sw`, `w`, `nw`). Metres is the distance, and **0 cancels the formation**.

- `vc formation e 5` — east socket at 5 m
- `vc formation ne 4` — northeast socket at 4 m
- `vc formation 45 3` — 45° bearing at 3 m
- `vc formation 0 0` — cancel

The game log location depends on how you launched the game:

| Launch Method | Log Location |
|---------------|-------------|
| **Steam / .exe** (default) | `%AppData%\..\LocalLow\The Fun Pimps\7 Days To Die\Player.log` |
| **Game Launcher** | `%AppData%\7DaysToDie\logs` |
| **Mod Launcher** | `<gamefolder>\7DaysToDie_Data\output.log` |

Search for `[1-XNPCVoiceControl]` to see mod-specific messages.

---

## Release Notes

What changed in each version, newest first, is in **`CHANGELOG.md`**, next to this file.
