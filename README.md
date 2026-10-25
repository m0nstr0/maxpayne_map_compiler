# MaxED Tools 1.0.0

**Compile, recover and light Max Payne 1 maps — and generate AI navigation networks.**

MaxED Tools is a Windows toolset for working with Max Payne level files. It
includes a graphical application and two command-line utilities. Use it to
compile an editable map for the game, recover an editable level from an LDB,
bake lightmaps, or generate an AI network for an existing compiled map.

**Author:** Bolotaev Sergey Borisovich aka **m0nstr0**  
**Website:** [bolotaev.com](https://bolotaev.com)  
**GitHub:** [github.com/m0nstr0](https://github.com/m0nstr0)

## Features

| Operation | Input → output | What it does |
|---|---|---|
| Compile a level | `.lvl` → `.ldb` | Builds a playable map, including geometry, collision BSP, rooms, portals, dynamic objects, animations and scripts. Packs existing lightmaps and can generate an AI network. |
| Recover a level | `.ldb` → `.lvl` | Recovers the available geometry, materials, lightmaps, objects, animations and scripts for editing in MaxED. Creates a recovery report. |
| Bake lightmaps | `.lvl` → `.lvl` | Calculates lighting and writes the result into a separate LVL. |
| Generate AI from LDB | `.ldb` → `.ai` | Generates a navigation network directly from a compiled map, including MaxED exports, without recompiling or modifying the LDB. |

The GUI supports English and Russian, drag and drop, command previews, progress
and stage estimates, cancellation, saved settings and complete execution logs.
English is the default language.

## Requirements

- **64-bit Windows.**
- **.NET Framework 4.8** for the GUI. The console utilities can be used separately.
- Your own **Max Payne 1 / MaxFX files**. Game assets are not included.
- The game's **`materials.txt`** for AI generation and lightmap baking.
  Keep the appropriate textures and other game assets available when working
  with your maps.

The tools target Max Payne 1 LVL and LDB formats, including LDB version 32.
Max Payne 2 maps are not supported. MaxED is not required to run the utilities;
it is used to edit recovered LVL files.

## Download and setup

Download the Windows package from this repository's **Releases** page and
extract it to a folder containing:

```text
MaxEdTools.exe    Graphical application
lvl2ldb.exe      Level compiler, lightmap baker and AI generator
ldb2lvl.exe      LDB-to-LVL recovery utility
README.md        This guide
```

Keep the three EXEs together. No installation is required. This repository
distributes ready-to-run binaries; source code is not included.

## Using the GUI

1. Run **MaxEdTools.exe**. If the console utilities are elsewhere, select their
   folder with **Tools folder**.
2. Choose a mode: **LVL → LDB**, **LDB → LVL**, **Bake lightmaps**, or **AI from LDB**.
3. Select the input file and output path. You can also drag a map into the window.
4. For AI or lighting, select the game's `materials.txt`, or leave its field
   empty to use automatic discovery.
5. Set the options and click the action button at the bottom. Follow progress
   and messages in the output log.

Use the **Language / Язык** selector to switch languages. Hover over an option
for a description. **Auto** restores the suggested output name.

For AI generation from an existing LDB, choose the fourth mode and save an `.ai`
file with the same base name as the map. For example, `MyMap.ldb` uses
`MyMap.ai`. The LDB remains unchanged.

Lightmap baking produces a new LVL. Compile that LVL in the first mode to create
the final LDB. When packing lightmaps for large maps, **1024 × 1024, By room**
is a useful starting point.

**Shortcuts:** Ctrl+O selects a map, Ctrl+Enter starts, Esc stops,
Ctrl+1–4 switches modes, and F1 opens help.

## Command-line examples

Open Command Prompt in the folder containing the EXEs. Quote paths that contain
spaces. In PowerShell, prefix the executable with `.\`, for example `./lvl2ldb.exe`.

Both console utilities print their version when launched. Use
`lvl2ldb.exe --version` or `ldb2lvl.exe --version` to display it without running
a conversion.

### Compile LVL to LDB

```bat
lvl2ldb.exe "MyMap.lvl" "MyMap.ldb" --lightmap-size 1024 --lightmap-group room
```

Generate AI during compilation:

```bat
lvl2ldb.exe "MyMap.lvl" "MyMap.ldb" --ai --materials "D:\MaxPayne\data\database\materials.txt"
```

The default AI output is `MyMap.ai`. Use `--ai-output "OtherName.ai"` to choose
a different path.

### Generate AI for an existing LDB

```bat
lvl2ldb.exe --ai-from-ldb "MyMap.ldb" "MyMap.ai" --materials "D:\MaxPayne\data\database\materials.txt"
```

This uses the compiled map's geometry, collision data, portals and AI settings.
It does not rebuild the map or repack its lightmaps.

### Recover an editable LVL

```bat
ldb2lvl.exe "MyMap.ldb" "MyMap_recovered.lvl"
```

By default, the utility also creates:

- `MyMap_recovered.lvl.recovery.json` — recovery warnings and information about
  data that could not be reconstructed.
- `MyMap_recovered.lvl.recovery.ldb` — an unchanged copy of the input LDB.

Use `--no-lightmaps` to skip lightmap recovery and `--no-source-copy` to omit
the companion LDB copy. The JSON report is still created.

### Bake lightmaps

```bat
lvl2ldb.exe --bake "MyMap.lvl" "MyMap_lit.lvl" --passes 2 --hemicube 16 --density 1 --threads 0 --ambient 0 --materials "D:\MaxPayne\data\database\materials.txt"
```

### Options

| Option | Values / behavior |
|---|---|
| `--lightmap-size` | `256` (default), `512`, `1024`, or `0` to disable packing. |
| `--lightmap-group` | `polygon` (default, global packing), `room`, or `material`. |
| `--ai` | Generate AI while compiling LVL to LDB. |
| `--ai-output` | Set a custom AI output path; also enables AI generation during compilation. |
| `--ai-from-ldb` | Separate mode: `--ai-from-ldb input.ldb output.ai`. |
| `--materials` | Path to the game's material-category definitions. Automatic discovery is used when omitted. |
| `--passes` | Baking passes: `1–64`, default `2`. |
| `--hemicube` | Even value `2–256`, default `16`. Higher values require more calculation time. |
| `--density` | Nonnegative lightmap density, default `1`. `0` reuses valid saved mapping. |
| `--threads` | `0–256`, default `0` for automatic selection; the baker uses up to 32 workers. |
| `--ambient` | Nonnegative ambient-light contribution, default `0`. |
| `--no-lightmaps` | Skip lightmap recovery in `ldb2lvl.exe`. |
| `--no-source-copy` | Omit the companion source LDB in `ldb2lvl.exe`. |

Baking options apply to `--bake`; packing options apply to LVL compilation.
Standalone AI generation accepts `--materials`. Use a decimal point for
numeric CLI values. The GUI also accepts a decimal comma.

## Recovery and compatibility notes

LDB is a compiled format. Recovery preserves as much available data as possible,
but cannot recreate every detail of the original editor project. The recovered
LVL may have different mesh organization and tessellation. Read the generated
recovery report and retain the original LDB.

Recovered LVLs are intended for MaxED. To hide editor helper boundaries, disable
**View → Display non-exporting meshes and NoDraw-polygons** in MaxED.

Generated AI files have been confirmed to work in the game. The game's own
`GM_InitAndCalcAI` can crash on some original maps as well; the standalone
AI-from-LDB mode provides a way to generate the network outside the game.

Use different input and output paths and keep backups of your maps. The GUI
asks before overwriting existing output files. The console utilities overwrite
the output paths you specify. Stopping a process can leave partial output files.

## Logs and troubleshooting

- GUI logs: `%LOCALAPPDATA%\MaxEdTools\logs\`.
- GUI settings: `%LOCALAPPDATA%\MaxEdTools\settings.xml`.
- If materials cannot be found, explicitly select the game's `materials.txt`.
- If a console utility fails, read its error message or the full GUI log.
- When reporting an issue, include the operation, command/options, full log,
  map format and whether the input came from MaxED or another tool.

## Author and copyright

Copyright © **2026 Bolotaev Sergey Borisovich aka m0nstr0**.

[bolotaev.com](https://bolotaev.com) · [github.com/m0nstr0](https://github.com/m0nstr0)
