# NPCCore

Survivors and bandits for 7 Days to Die **3.4**, built on the game's own modular character system (the one the player character
uses): every NPC gets archetypes, hair, armour and clothing from XML. Hire survivors, give them orders by the order wheel, the
hold-E ring, the talk menu or your voice, and let them follow, fight, loot, harvest, craft, guard and repair your base.

**Requires 0-SCore 3.4.14 or newer.** On a server: the server and every player, same versions.

## Start here
- **`Config/ReadMeFirst.md`** - installing, the quick start, the settings menu, common changes, and what is new in this release.
- **`Config/MANUAL.md`** - every feature and every setting, in detail.
- **`Config/CHANGELOG.md`** - all release notes, newest first.
- **`Config/voice-command-card.html`** - the voice commands on one page (open it in a browser).
- **`Config/UAI-MODDING.md`** and **`Config/POI-BUILDERS.md`** - for modders and POI builders.

## The folder
| Path | What |
|---|---|
| `NPCSdcsCharacters.dll`, `1-XNPCVoiceControl.dll` | the mod's code (characters and behaviour; orders, voice and chat) |
| `Config/` | the XML the game loads, `Settings.xml` (gameplay numbers) and `modconfig.xml` (voice and chat), and the docs above |
| `Resources/`, `Textures/`, `UIAtlases/`, `Prefabs/` | models, animations, icons and blocks |

The optional voice add-on (`1-XNPCVoiceControl-Complete` or `-Voice`, installed beside this folder) holds only the speech and chat
models and their helper servers; it does nothing without this folder.

## Credits
- Emote, dance and pose animations by Mumpfy and LittleRedSonja, used with their permission.
- The heal roll animation by Mumpfy.
- Crafting, looting, gathering, reading and gesture animations from licensed asset packs: Crafting Mecanim Animation Pack
  (Explosive LLC), Ultimate Animation Collection (PROTOFACTOR), MoCap Central sample animations, and Human Animations (Kevin Iglesias).

## License
See `LICENSE`.
