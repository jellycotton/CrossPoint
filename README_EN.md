# CrossPoint

- 日本語: [README.md](README.md)

**CrossPoint** helps you share “this area / around here” with an AI by turning a screen region into a precise **coordinate snapshot (JSON)**.

This ZIP is the **EXE distribution** (no Python install required).

## Core Concepts (Plate / Cross / JSON)
- **Plate**: A rectangle that defines your working area.
- **Cross**: A focus point inside the Plate (stored as relative coordinates).
- **JSON**: A numeric snapshot you can copy/paste to share exact positions.

The key is: **the numbers in the JSON are the single source of truth** (do not rely on what the overlay “looks like”).

## What You Can Do (Free)
- Define a Plate on the screen
- Move the Cross to pick a target point
- Copy the current state as **State JSON** and send it to the AI

## Quick Start (Launch → Copy JSON)
1. Extract the ZIP to any folder
2. Run `CrossPoint.exe`
	- On first run, Windows SmartScreen may appear. Click `More info` → `Run anyway`.
3. **Set Plate** → **drag** to draw the Plate rectangle.
	- In the screenshots, the overlay opacity is set to 100%.
	  In actual use, you can still see your desktop/apps through the overlay while you operate.
	- You can place it over an in-progress dialog and match it exactly to make coordinate-based instructions easier.
	- You can also use it to tell the AI “make it this size”.
	- While editing Plate/Cross, CrossPoint enters **overlay mode** (the screen dims and other apps cannot be interacted with). Right-click or `Esc` to exit.
4. **Move Cross** → click/drag to place the Cross
5. **Copy State JSON** → paste it into your AI chat

## Button Quick Reference
- **Set Plate**: Create the Plate (main rectangle).
- **Square**: Make the Plate a perfect square.
- **□ Center**: Move the Plate to the center of the screen.
- **Move Cross**: Switch to Cross move mode.
- **+ Center**: Reset the Cross to the center of the Plate.
- **Line thickness**: Adjust overlay line thickness (saved for next launch).
- **Line opacity**: Adjust overlay opacity (how transparent the overlay is; saved for next launch).
- **Copy State JSON**: Copy the current state (Plate + Cross) as JSON.
	- Includes DPI/scaling metadata as an optional `env` block (e.g., `ui_scale=1.25`).

## Status Bar
The status bar at the bottom shows the current mode and recent action results.

- Examples: `Ready` / `Mode: SET_PLATE` / `State copied!`

## DPI / Scaling (`env`)
State JSON may include DPI/scaling metadata in an optional `env` block.
However, **`rect` is always treated as the client/content-area logical pixels**, and `env` is observation-only metadata (do not use it to “correct” rect).
If an AI-generated layout ends up offset from the intended size/position, check `env.ui_scale` (e.g., 1.25 / 1.5) and your Windows Display Scale (Settings → System → Display → Scale).

## Where Settings Are Saved
The EXE version stores settings (and presets in Pro) under your user profile:

- CrossPoint: `%APPDATA%\CrossPoint`
- CrossPoint Pro: `%APPDATA%\CrossPoint Pro`

Note: `AppData` is hidden by default on Windows, so you may not be able to browse to it normally.

- Easiest: Press `Win + R` → type `%APPDATA%\CrossPoint` (or `%APPDATA%\CrossPoint Pro`) → Enter
- Or: Paste the path into the File Explorer address bar and press Enter

If you delete `settings.json` but old values “come back”, it may be auto-migrating from another edition’s legacy settings.
In that case, check both `%APPDATA%\CrossPoint` and `%APPDATA%\CrossPoint Pro`, and delete/rename `settings.json`, then restart.

## Troubleshooting
- Windows SmartScreen / Defender may warn on first run.
- If the window seems missing, check the taskbar or use Alt+Tab.
- If it crashes right after launch, check `err.txt` / `startup.log`.
- If AI output does not match the intended size/position, check the State JSON `env` (DPI/scaling).

## AI Protocol Doc (Important)
To prevent AI agents from making “helpful” but incorrect DPI/scale corrections, this ZIP includes an AI-facing protocol/spec:

- `README_AI.md` (Japanese)
- `README_AI_EN.md` (English)

When you ask an AI to place UI elements using CrossPoint JSON, tell it to follow this protocol (e.g., treat `rect` as logical pixels as-is; prefer `cross.rx/ry`).

---

## Useful Peripheral Tools (tools/)

The following tools are provided in the `tools/` directory to help diagnose DPI/scaling-related misalignment.

### 1. DPI/Rect Diagnosis Tool (`tools/CrossPointRectDiagnosis.exe`)
**Purpose**: Diagnose whether CrossPoint follows the canonical rule (**rect = client-area logical px**). This tool does not save or apply correction coefficients.

**Usage**:
1. Run `tools/CrossPointRectDiagnosis.exe`.
2. Click **"Show Target"** to display a red rectangle of known physical size.
3. Open `CrossPoint` and use **"Set Plate"** to capture EXCLUSIVELY the red area of the target window (exclude title bars).
4. Copy the State JSON from CrossPoint.
5. In the tool, click **"Paste & Analyze"**.
6. The tool prints scale ratios (ideal is ~1.0).

### 2. Dummy Window Generator (`tools/CrossPointDummyJsonWindow.exe`)
**Purpose**: Verify if a State JSON correctly maps to physical screen coordinates by spawning a dummy window at the exact location specified in the JSON.

**Why use this?**
- To verify if the AI-generated JSON aligns with the intended UI elements on your screen.
- It uses `rect` **as-is (no ui_scale division)** and best-effort aligns the **client-area origin**.

**Usage**:
1. Run `tools/CrossPointDummyJsonWindow.exe`.
2. Paste any **State JSON** (e.g., generated by AI or copied from CrossPoint) into the text box.
3. Click **"Spawn Window"**.
4. A gray window will appear.
   - Its **Client Area** (inner size) will match the `rect` in the JSON.
   - A **Red Cross** will mark the `cross` position.
   - A **Blue Button** will mark the `subrect` position (if present).

**For AI Agents**:
When debugging coordinate issues, ask the user to:
1. Copy the JSON you generated.
2. Paste it into this tool.
3. Report back if the window aligns effectively with the target UI.

Advanced:
- `tools/CrossPointDebugWindowRects.exe` is a debug utility to inspect Win32/Tk outer/client measurements. It does not emit a canonical State JSON.

---

## Pro (Booth)
Pro (paid) is available here:
- Booth: https://jellycotton.booth.pm/items/7886153

Pro can also display layout proposals received from an AI as JSON (e.g., window size, button positions) directly in CrossPoint, without implementing or running any code.

Note: This GitHub (Free) does not include Pro.

## License / Usage Terms
### Development / Copyright
Jelly

Bluesky: https://bsky.app/profile/jellycotton.bsky.social

### License
Copyright (c) 2026 Jelly

### Usage Terms
This software is provided for personal use.
Redistribution and commercial use are not permitted.
For any other use, please contact the author.

