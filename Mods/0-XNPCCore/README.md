# NPC SDCS Characters

NPCs for 7 Days to Die built on the game's own modular character system — the one the player
character uses. They get archetypes, hair, armour and clothing sets from XML, instead of needing a
hand-authored rig per character.

**Status: design complete, implementation not started.** Target game version **3.2**.

## Why

The game already has a modular character pipeline: a shared base rig, meshes stitched on per slot,
driven by an archetype defined in XML. It ships with the player, and every piece of it below the
entry point turns out to be entity-agnostic — the mesh builder takes an `EntityAlive`, and
`equipment` is a field on `EntityAlive` rather than on the player.

What stops a non-player using it is a small number of places that reach for an `EntityPlayer`. This
mod replaces those, and nothing else.

The alternative — authoring a rig per character and hand-fitting weapons into its hands in Unity —
is what this avoids. See [docs/NPC-CHARACTER-OPTIONS.md](docs/NPC-CHARACTER-OPTIONS.md) for the
comparison, including why reusing bandit rigs was rejected.

## What it does not do

- **It replaces nothing.** Existing NPC characters keep working, untouched. This is an additional
  character option, not a migration.
- **No new animation states.** It uses a copy of the stock third-person player controller, so
  base-game weapon hold types, carry types and item animation data keep working. Patrol is movement
  code, not an animator state.
- **No new avatar or skeleton.** Every character instantiates the same stock base rig.

## Layout

| Path | What |
|---|---|
| `Scripts/` | the mod. Compiled into `NPCSdcsCharacters.dll`. |
| `Config/` | entity classes and archetype definitions, patched in by xpath. |
| `docs/` | the design spec, the approach comparison, and the weapon-placement reference. |

## Build

Requires 7 Days to Die installed. The path is auto-detected; override with
`-p:GamePath="E:\Your\Path\7 Days To Die"` or the `SDTD_GAMEPATH` environment variable.

```bash
dotnet build NPCSdcsCharacters.csproj -c Release
```

Deploy into the live `Mods` folder — **the game must be closed** or the DLL copy silently skips:

```bash
dotnet build NPCSdcsCharacters.csproj -c Release -p:DeployToGame=true
```

## Requirements

7 Days to Die 3.2, SCore, and NPCCore. Loads after all three.
