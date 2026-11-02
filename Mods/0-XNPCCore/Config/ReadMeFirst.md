# NPCCore — Read Me First

**Upgrading from 3.2.04 or earlier? DELETE your old `Mods/1-XNPCVoiceControl` folder (and any old `0-XNPCCore` / `0-XNPCCore2`) before installing - do not unzip over them.** The mod now installs as `0-XNPCCore`, and an old copy left in `Mods` still loads and clashes with it.

## Installing

Unzip into your `Mods` folder:

1. **`0-XNPCCore-v3.4.02.zip` - required.** The NPCs, orders, voice commands and the mod's code. On its own it runs with every AI feature off.
2. **Plus at most ONE add-on:**
   - **`1-XNPCVoiceControl-Complete-v3.4.02.zip`** - AI chat, voice in and voice out. **Most players want this one.**
   - **`1-XNPCVoiceControl-Voice-v3.4.02.zip`** - voice in and voice out only, no AI chat.

You should end up with `Mods/0-XNPCCore` and, if you took an add-on, `Mods/1-XNPCVoiceControl-Complete` or `Mods/1-XNPCVoiceControl-Voice` beside it. The add-on holds only models and helper servers; it does nothing without `0-XNPCCore`.

You've installed the mod. Here's how to actually use it, what to expect, and how to fix common problems.

---

## Quick Start

### Talking to NPCs

| Method | How |
|--------|-----|
| **Push-to-Talk** (default) | Walk up to an NPC, hold **V**, speak your message, release |
| **Text Chat** | Press **T**, type `@Your message here` in global chat (the `@` targets the nearest NPC) |

### What Happens

1. Your speech is transcribed to text (Whisper)
2. If it matches a known command ("follow me", "open inventory"), it executes **instantly**
3. If it doesn't match a command, the local AI thinks about it and responds with voice + subtitles

### Command Card

**`voice-command-card.html`**, in this folder, lists every command and its alternate
phrasings on one page. Open it in any browser and press **Ctrl-P** — it prints to a single
landscape sheet, handy beside the keyboard.

You do not need to memorise exact wording. Phrases match as **substrings**, so
"hey Brody, follow me would you" fires the same command as "follow me".

### The Order Wheel (Formation Radial)

Hold **;** (default; remappable in `modconfig.xml` under `<FormationRadial><Key>`) to open the
order wheel - a multi-page radial:

- **Page 0** - Follow, Stay, Guard, plus five submenus: **Jobs**, **Orders**, **Stance**, **Position**
  and **Formation**. The submenus are marked with a small chevron and a
  blue-tinted border.
- **Who it targets** - whoever you are looking at, otherwise the last companion you ordered,
  otherwise the nearest. Their name sits in the middle of the ring, and the mouse wheel
  cycles through your hires.
- **Commit** - release the key over a socket to order the targeted companion; while still
  holding the key, **click** a socket to apply the same order to your **whole squad**
  (formation bearings included). Everything on the **Jobs** page, the patrol trio, the **Roles** and the
  weapon swaps always go to the targeted companion only, click or release.
- **Open a submenu** - click it, or simply rest the cursor on it for a beat (the dwell,
  0.4 s by default - a quick pass over it does nothing).
- **Back out** - rest the cursor on the middle of the wheel and it returns to page 0.
- **Jobs** - Follow + loot, Loot here, Harvest (wood, plants, crops, butcher), Mine (all, rocks,
  iron, coal, lead, shale, nitrate, clay), Base Repair, Combat Repair and Remember base.
- **Orders** - Crouch (a toggle: it reads **Crouch** or **Resume** depending on whether the
  companion is crouch-locked), Stand up, Scout, Guard and return, the patrol trio (Record / Run /
  Cancel patrol) and **Roles**: Medic, Cook, Blacksmith, Chemist and Crafter, each a toggle (**Cook**
  or **Stop cook**...). A survivor uses a workstation only with its role (Cook the campfire,
  Blacksmith the forge, Chemist the chemistry station, Crafter the workbench and cement mixer) and
  inside your land claim (a campfire anywhere); she crafts from her backpack anywhere.
- **Stance** - Aggressive / Defensive / Hold fire / Disengage, plus **Weapons** (guns, rifles,
  shotguns, machine guns, melee, blades, empty hands).
- **Position** - Close (1 m), Normal (2.5 m) and Far (5 m), drawn as the real metres; Point (she
  leads 2 m ahead of you); Go there (the spot under your crosshair when the wheel opened); Jump
  down; Get in / Get out (your vehicle).
- **Formation** - the eight compass bearings (north at the top). Each companion keeps its own
  distance; the socket sets the direction only.

### Customising the mod

**`MANUAL.md`**, also in this folder, is the XML modder's guide — how to add a personality or a
voice command, the full `modconfig.xml` settings reference, the order wheel, the voice list,
and how voice clips work. You need no C# for any of it.

### First-Time Startup

When you first launch after installing, expect a **15-30 second delay** before the mod is ready. All four helper servers (LLM, TTS, STT, embeddings) start automatically. You'll see server status messages in the game console. The NPCs won't respond until all servers are online.

### Requirements

This mod targets **7 Days to Die 3.4** and needs **0-SCore 3.4.3 or newer** (on a server: the server and every player). A newer SCore can rename something the mod patches: the log then names each patch that did not apply (`[SDCS-PATCH] ... NOT applied`) and the rest of the mod still runs. If you are still on game 3.2, use mod version 3.2.06 instead. It does NOT need the older `0-XNPCCore` (v3.1.00) - this mod replaces it (see the top of this file).

### Multiplayer Status

Multiplayer status: Singleplayer and Listen-host are fully supported. Dedicated servers are now supported — the mod runs NPC AI server-side, and voice commands (orders, follow distance, patrol, formations, squad commands) route correctly to the server. Dedicated server operators install **`0-XNPCCore` only**, with no add-on — the server needs no models or helper servers, because all AI runs on the players' machines. Players install `0-XNPCCore` plus the add-on they want (see Installing above). Both server and clients must be the same version; a mismatch is logged as a warning at join and multiplayer features may misbehave. Listen-host is unaffected — the host installs the same as any player. Lines a survivor builds on the server (her answer to "What are you carrying?", out of ammo, pack full) are spoken by every player's own client within earshot.

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

## In-Game Settings Menu

Open the escape menu (**Esc**) and click **NPC VoiceControl Settings**. Here's what each control does:

### Text-to-Speech (TTS)

| Setting | What It Does |
|---------|-------------|
| **Text-to-Speech** toggle | Turn NPC voice on/off entirely |
| **Volume** | How loud NPCs speak (0.0–1.0) |
| **Speech Rate** | Speed of NPC speech (0.5 = slow, 2.0 = fast). Default 1.0 is natural |

### Speech-to-Text (STT)

| Setting | What It Does |
|---------|-------------|
| **Speech-to-Text** toggle | Turn voice recognition on/off |
| **Wake Word (Hands-Free)** | Say "Hey Marvin" instead of holding V. **Off by default** — it uses extra CPU. See FAQ below |
| **Beam Size (1–5)** | How many candidate transcriptions Whisper weighs. **Ships at 1** — fastest, and in practice it recognises commands very well. **3 is the accuracy sweet spot** of the 1–5 range, but it samples three times, so it costs real time for a gain you usually will not notice. Raise it only if commands are being misheard |

### Conversation

| Setting | What It Does |
|---------|-------------|
| **Loaded AI Model** | Shows which LLM is loaded (read-only) |
| **Max Tokens (64–2048)** | Maximum response length. Higher = longer replies but slower. The shipped default is 200; raise it if replies cut off mid-sentence |
| **Context Window (3–20)** | How many recent exchanges the AI remembers in conversation. Higher = more context-aware but uses more VRAM |

### Distance & Range

| Setting | What It Does |
|---------|-------------|
| **Chat Distance (3–15m, default 15)** | How close you need to be to talk to an NPC you have NOT hired |
| **Hired Chat Distance (5–40m, default 40)** | How far away your own hired companions still answer you |
| **Voice Range (5–40m, default 40)** | How far NPCs' voices carry (audio volume drops off after this). Keep this at or above Hired Chat Distance, or you will be able to give an order you cannot hear the reply to. |

### Facial Expressions

| Setting | What It Does |
|---------|-------------|
| **Procedural blink** (automatic) | NPCs blink naturally at random intervals (2–6 seconds). Works on most NPC models with eyelid geometry. Some models split eye geometry across multiple meshes and won't blink — this is a model limitation, not a bug |

### Long-Term Memory

| Setting | What It Does |
|---------|-------------|
| **Long-term memory** (automatic) | The mod remembers facts about you across conversations and game sessions. If you tell an NPC your name, what you're building, or where you live, they'll recall it later. Stored per-NPC — each NPC has its own memory of you. Saved to `NPC_Memories.xml` in the Mods folder. High-importance memories (like shared events) persist longer than casual facts |

### Other

| Setting | What It Does |
|---------|-------------|
| **Chat Timeout (15–120s)** | How long the NPC stays "in conversation mode" after you speak. After this idle period, the NPC forgets recent context and you'd start a fresh conversation. Default 30s is good for most — higher if you like long pauses between exchanges |
| **Mod Debug Logging** | Turn on only if troubleshooting — adds lots of console spam. Saved on this PC: it stays on across mod updates until you turn it off (or type `vcdebug off`) |

### Test Buttons

- **Test Voice** — plays a sample TTS line so you can hear the voice
- **Test Microphone** — records 3 seconds and shows what Whisper heard
- **Test AI Connection** — confirms the LLM server is responding
- **Clear All Conversations** — wipes chat history with all NPCs

---

## Common Config Changes (modconfig.xml)

The in-game menu covers most things. For advanced tweaks, edit **the add-on's** `Config/modconfig.xml` - `Mods/1-XNPCVoiceControl-Complete/Config/modconfig.xml` (or `-Voice`). When an add-on is installed the mod reads ONLY that file; the `modconfig.xml` inside `0-XNPCCore` is used only when no add-on is installed, and editing it then does nothing. **Changes here require a full game restart.**

### TTS Engine (`<TtsEngine>`)

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

### Speech Recognition Model (`<Model>`)

One model ships: `ggml-small-q8_0.bin` (the small model, quantized, near-lossless). The older
`ggml-base.en.bin` (a faster English-only tier) and the f16 `ggml-small.bin` were removed from
the package, so do not point `<Model>` at either of them.

Leave empty for auto-detect - it prefers the shipped `ggml-small-q8_0.bin`, but keeping the
explicit filename in the config is still better.

### Push-to-Talk Key (`<PushToTalkKey>`)

Default is **V**. Change to any single key: `<PushToTalkKey>G</PushToTalkKey>`

### Phonetic Name Matching (`<NameMatchPhonetic>`)

Default **true** — enables Soundex phonetic matching for NPC names (e.g. "bow" matches "Beau"). Set to `false` if you experience false name matches; exact, prefix and fuzzy matching still work.

### Microphone Device (`<MicrophoneDevice>`)

If voice isn't working at all, your mic might not be the default device. Set it explicitly:

```xml
<MicrophoneDevice>Microphone (Realtek Audio)</MicrophoneDevice>
```

Check the boot log for your exact device name — it prints `[0] Microphone (device name)` on startup.

### Max Tokens (`<MaxTokens>`)

Maximum response length in tokens. The shipped default is 200. If NPC responses cut off mid-sentence, increase it to 768 or 1024. If responses are too long and slow, lower it.

### Max Response Length (`<MaxResponseLength>`)

Hard cap on response characters (default 300). Prevents the AI from generating very long speeches. Lower = snappier replies, higher = more detailed answers but slower TTS.

### Typing Delay (`<TypingDelayMs>`)

Per-character delay for the typewriter subtitle effect (default 75ms). Values: 50 = fast, 75 = natural, 150 = dramatic. Set to 0 to disable the typing animation entirely.

### Context Memory (`<ContextMemory>`)

How many recent exchanges the AI keeps in its short-term context window during conversation. The default is 5.
- **3–5** (default) — remembers last few things you said, lowest VRAM usage
- **8–10** — more context-aware, remembers longer conversations, uses more VRAM
- **15–20** — very context-aware but may repeat old topics and uses significant VRAM

When the conversation grows beyond this limit, older messages are trimmed from the LLM prompt (but facts you told NPCs are saved to long-term memory automatically).

### KV Cache Size (`<ContextSize>`)

Controls how much VRAM the LLM server reserves for its context window. Lower values help on GPUs with limited VRAM. The default 8192 works for most cards with 8 GB+ VRAM.

| ContextSize | VRAM Used | When to Use |
|-------------|-----------|-------------|
| **2048** | ~224 MB | Bare minimum — very limited conversation memory, last resort for 4 GB cards |
| **4096** | ~448 MB | 6 GB VRAM cards; use with IQ4_NL model |
| **8192** (default) | ~896 MB | Recommended for 8 GB+ VRAM (RTX 3060 / 4060 class) |
| **16384** | ~1.75 GB | 12 GB+ cards (RTX 3080 / 4070 Ti class) |
| **32768** | ~3.5 GB | 16 GB+ cards (RTX 3090 / 4090 class) |

Values above 32768 waste VRAM with no gameplay benefit. If your game stutters or crashes on launch, try lowering this to 4096.

### Supertonic TTS Speed (`SupertonicConfig.xml`)

This section only applies when `<TtsEngine>` is `supertonic`; the default `audiocpp` engine does not read `SupertonicConfig.xml`. If NPC voice synthesis feels slow on a weaker CPU with the supertonic engine, two settings in `Config/SupertonicConfig.xml` can help:

| Setting | Default | Faster Value | Trade-off |
|---------|---------|-------------|----------|
| `<TotalStep>` (denoising steps) | 6 | **4 or 5** | Slightly less polished audio, noticeably faster. At 4 most users won't notice quality loss |
| `<IntraOpNumThreads>` | 4 | **2** | Less CPU contention on quad-core CPUs; slower isolated synthesis but smoother overall game performance |

Neither is in the in-game menu — edit the XML directly and restart the game.

**Note:** The in-game **Speech Rate** slider controls how fast or slow the voice *sounds* (playback speed). These settings control how long it takes to *generate* the audio. They're separate.

---

## FAQ

### Why is speech recognition slow?

Whisper runs on your GPU via Vulkan. On older GPUs (GTX 10-series or older) or when the GPU is busy rendering, transcription can take several seconds.

**Mitigations:** lower **Beam Size** to 1 in the in-game settings (the default) — beam search is the main accuracy-vs-speed trade-off. If the server grabbed your integrated GPU instead of the discrete one, set `<WhisperGpuDevice>` in `modconfig.xml` to the discrete card's index (the `[whisper-server]` log lines list them).

### Can I use wake-word mode?

Yes, but it uses extra CPU while listening. To enable: toggle **Wake Word (Hands-Free)** in the in-game settings, or set `<WakeWordEnabled>true</WakeWordEnabled>` in `modconfig.xml`. Say "Hey Marvin" to start recording — it stops after silence.

### Why won't the NPC follow my orders?

The NPC must be **hired** (have you as their leader). Leaderless NPCs will chat with you but refuse commands like "follow me." Say "hire you" to open the hire dialog first.

### The NPC keeps saying "I don't take orders from just anyone"

You're talking to a wild NPC that isn't hired yet. Say "hire you" or use the vanilla hire system first. Traders are an exception — they'll always open their inventory regardless of leader status.

On the `supertonic` engine this is default behavior on longer passages: it is a diffusion model with 6 denoising steps, so quality is good but can sound slightly mechanical on story-length text. This is expected and not a bug. Switching to the default `audiocpp` engine will not change the voice itself - both engines run the same Supertonic 3 model.

### Can I use my own LLM model?

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

### What model size should I use?

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

### The AI only speaks English

The included Qwen3-4B model is multilingual but strongest in English. For stronger multilingual output you can swap in a different model (see the section above).

### What are "phrase triggers"?

Pre-written commands that bypass the AI entirely for instant execution: "follow me", "stay here", "open inventory", "use your knife", etc. These execute in under 1 second because no LLM thinking is involved.

When you give a command, your NPC nods to confirm acceptance or shakes their head to refuse — subtle head gestures that respond to whether they can comply (hired status, item availability, etc.).

### "Tell me a story"

This phrase trigger plays a pre-written story from the NPC's personality pool — instant response with no AI thinking involved. Each personality has multiple stories, picked at random.

### What are aliases?

Each phrase trigger can have alternative names called **aliases**. For example, the "open inventory" trigger also matches "show me what you have" and "let me see your stuff" as phrases, and "backpack", "gear", "bag" or "inventory" as aliases — all defined in `phrasetriggers.xml`. You don't need to memorize exact phrases; if you say something close enough, it'll match. Modders can add their own aliases by editing the XML.

### Can I add my own voice commands?

Yes — edit `Config/phrasetriggers.xml`. Each trigger has a phrase pattern and a response. The XML is well-commented with examples. See the mod page for documentation.

---

## Troubleshooting

### Voice recognition isn't working at all

1. Check `bin/WhisperServer/` contains `whisper-server.exe`, `whisper.dll`, and `ggml*.dll` files
2. In-game settings → click **Test Microphone** — does it show what you said?
3. If mic test fails: check your `<MicrophoneDevice>` setting matches your actual device name (see boot log)
4. If whisper-server didn't start, look for error lines in the game console

### NPC voice isn't playing

1. Check TTS is enabled in the in-game settings
2. Click **Test Voice** — does it play?
3. Verify the TTS server started: the startup console log prints a server health summary that lists the active engine as ready (`audiocpp` by default, or `supertonic` if you switched)

### NPC doesn't respond to anything

1. Are you within chat distance? Default is 15m for an NPC you have not hired, 40m for your own hire — walk closer if you are beyond it
2. Is the LLM server running? Check console for `llama-server is accepting connections on port 5055`
   The LLM uses port 5055 (moved off 8080 to avoid collisions with web admin panels like AMP).
   Upgrading and hand-edited modconfig.xml? Update Endpoint to 5055.
3. Is there a `.gguf` file in `Resources/Models/Llama/`?
4. Try text chat (`@hello`) — if that works, it's a voice issue, not an AI issue

### Game hangs on exit

Known Steam/7DTD issue. Just wait or click Stop in Steam. Not caused by the mod.

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
- **She hunts:** a bandit who loses sight of the enemy she is fighting, or of whoever just shot her, within 40 m, goes after it.
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

---

## Earlier Releases

Release notes for 3.4.01 and earlier are in `CHANGELOG.md`, next to this file.
