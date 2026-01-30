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

## Quick Start Guide
1. Extract the ZIP to any folder
2. Run `CrossPoint.exe`
	- On first run, Windows SmartScreen may appear. Click `More info` → `Run anyway`.
3. **Set Plate** → **drag** to draw the Plate rectangle.
	- You can place it over an in-progress dialog and match it exactly to make coordinate-based instructions easier.
	- You can also use it to tell the AI “make it this size”.
	- While editing Plate/Cross, CrossPoint enters **overlay mode** (the screen dims and other apps cannot be interacted with). Right-click or `Esc` to exit.
4. **Move Cross** → click/drag to place the Cross
5. **Copy State JSON** → paste it into your AI chat
<img width="3835" height="2169" alt="0999スクリーンショット (399)" src="https://github.com/user-attachments/assets/6c056f5d-e0e1-43e1-a041-d641767bc477" />

Demo: Moving this button with the Standard Edition.

<img width="3840" height="2169" alt="888スクリーンショット (400)" src="https://github.com/user-attachments/assets/1d79e587-9ddf-46fa-b4ab-a2d30c33dd0e" />

Select "Set Plate" to enter overlay mode. Place the Plate (blue line) to perfectly enclose the dialog you want to adjust.

<img width="3836" height="2169" alt="888スクリーンショット (401)" src="https://github.com/user-attachments/assets/79c68d62-6ecb-4b06-b5a6-e44fb37ee24e" />

Click "Move Cross", then place the Cross (red mark) where you want to position the button.

<img width="3836" height="2169" alt="888スクリーンショット (402)" src="https://github.com/user-attachments/assets/34046f6a-812e-4449-b0a7-b569bf6356a1" />

Paste the JSON copied via the panel's Copy button into the AI chat and send your request. (e.g., "Move the OK button to the Cross position.")

<img width="3836" height="2169" alt="888スクリーンショット (403)" src="https://github.com/user-attachments/assets/b6b173ba-97d4-404c-a34b-45e420274498" />

Then...

<img width="3838" height="2169" alt="888スクリーンショット (404)" src="https://github.com/user-attachments/assets/3fb0c574-dadc-4c8b-9195-4397ba239de3" />

The AI updates the code, and the button moves to the exact position.

> **Note: When asking an AI, it is recommended to share the included `docs/README_AI_EN.md` first.**
> CrossPoint JSON is designed to be used as-is without correction.
> If this spec is not shared, the AI may perform DPI corrections, causing unintended misalignment.


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
- Temporarily setting the display scaling to 100% may make **troubleshooting easier**.

## AI Protocol Doc (Important)
To prevent AI agents from making “helpful” but incorrect DPI/scale corrections, this ZIP includes an AI-facing protocol/spec:

- `README_AI.md` (Japanese)
- `README_AI_EN.md` (English)

When you ask an AI to place UI elements using CrossPoint JSON, tell it to follow this protocol (e.g., treat `rect` as logical pixels as-is; prefer `cross.rx/ry`).

## Prior Art
- [docs/coordinate_snapshot_sharing_protocol.md](docs/coordinate_snapshot_sharing_protocol.md)

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
Pro (Supporter Edition) is available here:
- Booth: https://jellycotton.booth.pm/items/7886153

Pro can also display layout proposals received from an AI as JSON (e.g., window size, button positions) directly in CrossPoint, without implementing or running any code.

Note: This GitHub (Free) does not include Pro.

<img width="3836" height="2169" alt="000スクリーンショット (410)" src="https://github.com/user-attachments/assets/bef604c3-b0fb-4bd5-81f6-39c27b6a7e72" />

In the Supporter Edition (Pro), let's resize the button using a Sub-Plate.

<img width="3835" height="2169" alt="000スクリーンショット (412)" src="https://github.com/user-attachments/assets/797adea3-c81e-4ac3-9ef6-a6e91840b8f5" />

We will change the height while keeping the width the same.

<img width="3835" height="2169" alt="000スクリーンショット (413)" src="https://github.com/user-attachments/assets/4ff0456f-5f97-4f2b-b9b7-7e046a99c177" />

Adjust the Blue Plate to resize the dialog. Then, add a Green Sub-Plate to specify the button's new size and position.

<img width="3835" height="2169" alt="000スクリーンショット (415)" src="https://github.com/user-attachments/assets/95e83549-167a-4e14-810f-84fdcecbdad9" />

Sent the request to the AI... and another success!



## License / Usage Terms
### Development / Copyright
Jelly

Bluesky: https://bsky.app/profile/jellycotton.bsky.social

### License
Copyright (c) 2026 Jelly

### Disclaimer
THIS SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND. THE AUTHOR IS NOT LIABLE FOR ANY CLAIM, DAMAGES, OR OTHER LIABILITY ARISING FROM THE USE OF THIS SOFTWARE. USE AT YOUR OWN RISK.

### Usage Terms
- Redistribution or resale of the software itself is prohibited.
- You are free to use this tool in commercial projects (including business use and selling deliverables).

For any other use, please contact the author.

