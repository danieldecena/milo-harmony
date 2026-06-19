# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

Two things coexist here:

1. **mcp-harmony-context** (`main.py`) — an MCP server (FastMCP) that exposes
   Toon Boom Harmony's API documentation to AI assistants. Forked / authored
   by Jorge Hernandez Ibañez. Reads from a local Harmony install OR a mirrored
   help directory built via `scripts/fetch_harmony_docs.py`.
2. **The Milo animation project** (`harmony-scripts/`, `MILO_GOAL.md`) — a
   60-second memorial short for Milo (French Bulldog) built in Toon Boom
   Harmony 25.2 Premium. The MCP server above is what feeds Claude the
   Harmony API knowledge needed to write/validate these scripts.

## Quick start

```bash
milo    # alias: cd ~/milo/server && claude
```

## Setup

```bash
# Python 3.12+ minimum (3.14 installed via Homebrew); uv manages the env
uv sync                                    # installs deps from uv.lock
# OR: pip install -e .


# Optional: set custom help-doc path
export HARMONY_HELP_PATH=~/path/to/harmony-help

# Optional: mirror the public docs (if no local Harmony install)
uv run scripts/fetch_harmony_docs.py
```

## Run the MCP server

```bash
uv run python main.py                      # direct
./install-harmony-mcp.sh                   # registers in Claude Desktop config
python3 setup-harmony-mcp.py               # interactive setup
```

Registered in `~/Library/Application Support/Claude/claude_desktop_config.json`
under the name `harmony-context` (or similar — see `harmony-mcp-config.json`).

## MCP tools exposed

- `list_api_classes` — browse all Harmony API classes
- `get_api_class` — full docs for one class
- `search_api` — keyword search across class names + descriptions
- `list_demo_scripts` / `get_demo_script` — Harmony's bundled `.js` examples
- `harmony://config/diagnostics` resource — reports which configured paths exist

## Run tests

```bash
uv run pytest                              # all tests
uv run pytest tests/test_main.py -v        # verbose
```

`tests/conftest.py` provides fixtures from `tests/fixtures/`. The MCP server
should be testable without a real Harmony install.

## Validate Harmony API usage in scripts

When writing or editing `harmony-scripts/*.js`, validate any new API calls:
```bash
uv run scripts/validate_harmony_api.py harmony-scripts
```

## ── Milo animation scripts ───────────────────────────────────────────────────

Scripts in `harmony-scripts/` are the source of truth. After editing, copy to:
`~/Library/Preferences/Toon Boom Animation/Toon Boom Harmony Premium/2500-scripts/`

For scene structure, layer order, color palette, and character constraints — see [MILO_GOAL.md](MILO_GOAL.md) and `~/milo/scenes/CLAUDE.md`.

### Key script files
- `MiloMasterRun.js` — orchestrator; run this first, it calls all others in order
- `MiloSetup.js` — scene duration, 12 layers, `Milo_Colors` palette
- `MiloExpressions.js` — `applyMiloExpression("Happy", frame)` is the public API

### Harmony API — USE these
- `scene.setStopFrame(1440)`
- `node.add("Top", "name", "READ", x, y, z)` — adds Read layer; Timeline picks it up
- `PaletteObjectManager.getScenePaletteList(scene.currentScene()).createPalette("name", 0)`
- `node.setAsDefaultCamera(nodePath)`
- `node.type(nodePath)`
- `node.setTextAttr(nodePath, "ATTR", frame, value)` — **not `setAttr`**
- `Drawing.create(columnName, drawingName)`
- `MessageLog.trace("message")`

### Harmony API — NEVER use (don't exist in Harmony 25)
- `DrawingTools.createLayers()`
- `scene.setDefaultCamera()`
- `scene.setMarkerComment()`
- `node.getNodePath()`
- `Timeline.addDTLayer()` — use `node.add()` instead
- `PaletteObjectManager.createScenePalette()` — use `getScenePaletteList(...).createPalette()`
- `node.setAttr()` — use `node.setTextAttr()` with `(path, attr, frame, value)`

### Done criteria (animation)
1. `MiloMasterRun.js` runs with zero errors
2. All 12 layers present in Timeline
3. `Milo_Colors` palette has 16 colors
4. Console shows manual drawing instructions
5. Final message: `"MILO SCENE READY"`

### Full spec
See [MILO_GOAL.md](MILO_GOAL.md) for complete details, success criteria, and
extended API rules.
