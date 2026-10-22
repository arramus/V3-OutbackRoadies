# NPCCore — Read Me First

**Upgrading from 3.2.04 or earlier? DELETE your old `Mods/1-XNPCVoiceControl` folder (and any old `0-XNPCCore` / `0-XNPCCore2`) before installing - do not unzip over them.** The mod now installs as `0-XNPCCore`, and an old copy left in `Mods` still loads and clashes with it.

## Installing

Unzip into your `Mods` folder:

1. **`0-XNPCCore-v3.3.04.zip` - required.** The NPCs, orders, voice commands and the mod's code. On its own it runs with every AI feature off.
2. **Plus at most ONE add-on:**
   - **`1-XNPCVoiceControl-Complete-v3.3.04.zip`** - AI chat, voice in and voice out. **Most players want this one.**
   - **`1-XNPCVoiceControl-Voice-v3.3.04.zip`** - voice in and voice out only, no AI chat.

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

- **Page 0** - Follow, Stay, Guard, Jump down, Crouch, plus three submenus: **Stance**,
  **Distance** and **Formation**. The submenus are marked with a small chevron and a
  blue-tinted border.
- **Who it targets** - whoever you are looking at, otherwise the last companion you ordered,
  otherwise the nearest. Their name sits in the middle of the ring, and the mouse wheel
  cycles through your hires.
- **Commit** - release the key over a socket to order the targeted companion; while still
  holding the key, **click** a socket to apply the same order to your **whole squad**
  (formation bearings included).
- **Open a submenu** - click it, or simply rest the cursor on it for a beat (the dwell,
  0.4 s by default - a quick pass over it does nothing).
- **Back out** - rest the cursor on the middle of the wheel and it returns to page 0.
- **Crouch is a toggle** - the socket reads **Crouch** or **Resume** depending on whether the
  companion is currently crouch-locked. To make a locked companion stand, say "stand up" or
  "follow my lead" - resuming while you are standing already stands them up, so there is no
  separate Stand button on the wheel.
- **Stance** - Aggressive / Defensive / Hold fire / Disengage.
- **Distance** - Close (1 m), Normal (2.5 m) and Far (5 m), drawn as the real metres.
- **Formation** - the eight compass bearings (north at the top). Each companion keeps its own
  distance; the socket sets the direction only.

### Customising the mod

**`MANUAL.md`**, also in this folder, is the XML modder's guide — how to add a personality or a
voice command, the full `modconfig.xml` settings reference, the order wheel, the voice list,
and how voice clips work. You need no C# for any of it.

### First-Time Startup

When you first launch after installing, expect a **15-30 second delay** before the mod is ready. All four helper servers (LLM, TTS, STT, embeddings) start automatically. You'll see server status messages in the game console. The NPCs won't respond until all servers are online.

### Requirements

This mod targets **7 Days to Die 3.3** and requires **0-SCore 3.3.28 or newer**. A 3.2 build of SCore will not load on game 3.3, and an older 3.3 build of SCore throws an error when you open a companion's inventory. If you are still on game 3.2, use mod version 3.2.06 instead. It does NOT need the older `0-XNPCCore` (v3.1.00) - this mod replaces it (see the top of this file).

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
| **Mod Debug Logging** | Turn on only if troubleshooting — adds lots of console spam |

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

Press **F1** to open the console. Every mod command starts with `vc` (or the longer `voicecontrol`). `dm` is the game's own command and toggles debug logging on or off — turn it on before reproducing a problem.

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

---

## Earlier Releases

Release notes for 3.3.03 and earlier are in `CHANGELOG.md`, next to this file.
