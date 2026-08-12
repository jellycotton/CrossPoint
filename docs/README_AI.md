# CrossPoint Protocol for AI Agents

このドキュメントは、**CrossPoint** ツールを使用するユーザーと協調してコーディングを行うAIエージェント（あなた）のための仕様書です。
ユーザーは画面上の座標や領域を特定し、それをJSON形式であなたに伝えます。あなたはそれを解釈し、適切なUIコード（Tkinter, HTML/CSS等）を生成する必要があります。

## CrossPoint Pro で追加される機能（参考）
- **Candidates JSON の可視化**: CrossPoint Pro では、AIが提案した候補点（`rx`/`ry`）をユーザーが点として可視化できるワークフローが追加されます。

この仕様書（CrossPoint / 無印）では、AIは **State JSON**（`rect`/`cross`）を唯一の入力として扱い、提案は **State JSON**（または実装コード）で返してください。

## 基本概念

- **Plate**: 画面上の作業エリア。JSONでは `rect`（`x`,`y`,`w`,`h`）で表されます。
- **Sub-Plate**: Plateの中に作る追加の四角形エリア（1枚）。JSONでは `subrect`（`x`,`y`,`w`,`h`）で表されます（任意）。
- **Cross**: Plate内の注目点。JSONでは `cross`（`rx`,`ry`,`x`,`y`）で表されます。
- **単位**: `unit` は通常 `px`。`x`,`y`,`w`,`h` は小数になることもあります（DPI/描画都合）。
- **相対座標**: `rx`,`ry` は $0.0$〜$1.0$（Plate内）。実装側でクランプされるため、AI側も範囲内を前提にしてください。

## 重要: JSONはスナップショット（唯一の根拠）
CrossPointは「座標と領域をJSONで共有する」ためのツールです。あなた（AI）は、**その瞬間にユーザーPC上で動いているCrossPointの見た目やカーソル位置**を参照して推測してはいけません。

- ユーザーが渡すJSONは**スナップショット**であり、**唯一の正**です。
- 「JSONのcrossの位置」は、**数値として与えられた座標/比率**を意味します。
- 「いま画面に表示されているCrossPointのクロス位置」や「現在のウィンドウ配置」から推測して補正しないでください。
- 迷ったら、追加情報を要求するのではなく、まずJSONの値をそのまま使った案を提示してください。

補足: CrossPoint側の見た目設定（例: Overlayの線の太さ）はローカル設定（`settings.json` 等）で保持されることがありますが、これは State JSON の仕様とは無関係です。AIはUI見た目ではなく、ユーザーが貼り付けた State/Candidates JSON の数値のみを根拠にしてください。

### 推奨の受け取り方（AI側の内規）
ユーザーから「JSONのクロスの位置に○○を置いて」と来たら、次のように解釈してください:

- **“このJSONの `rect` / `cross` の値を唯一の入力としてUI配置を決める”**
- 実装は相対座標（`rx/ry`）優先。必要に応じて `rect.w/h` を初期サイズの参考にする。

### ユーザー向けの定型文（誤解防止）
ユーザーがAIに渡すメッセージに、次を添えると誤解が激減します。

"""
以下のJSONはCrossPointが生成したスナップショットです。あなたはCrossPointの実画面を参照せず、このJSONの数値（rect/cross）だけを根拠に配置してください。
"""

UI座標は **「基準となる四角形 (Rect)」** と **「その中の相対位置 (Cross)」** で表現されます。

*   **Rect (Plate)**: 画面上の絶対座標における作業エリア。ウィンドウそのものや、特定のフレームを指します。
*   **Cross (Point)**: Rect内における注目点へのポインタ。`0.0`〜`1.0` の相対座標で管理されます。

## JSONフォーマット

CrossPointで扱うJSONは主に **State JSON** です。

- **State JSON (ユーザー -> AI / AI -> ユーザー)**: Plate（rect）と Cross（cross）を含むスナップショット。

### 1. State (ユーザー -> AI)
ユーザーが「ここにボタンを置きたい」といった意図を示すデータです。

#### 任意: env（DPI/スケール情報）
環境差（Windowsの表示倍率、Tk/CTkのスケーリング等）によるズレの切り分けをしやすくするため、State/Candidates には任意で `env` を含めることがあります。

- `env.dpi_ppi`: Tkが報告する 1インチあたりのピクセル数（例: 96 / 120 / 144）
- `env.ui_scale`: `dpi_ppi / 96.0`（Windows 100% を 1.0 とした倍率）
- `env.tk_scaling`: Tk の `tk scaling` 値

AIは `env` を必須扱いにせず、存在する場合にのみ「ズレ要因の推定」や「診断（scaleが1.0からズレている理由の説明）」に利用してください。

```json
{
  "version": "1.0",
  "basis": "plate",
  "unit": "px",
  "env": {
    "dpi_ppi": 120.0,
    "ui_scale": 1.25,
    "tk_scaling": 1.25
  },
  "rect": {
    "x": 1168,
    "y": 392,
    "w": 1602,
    "h": 1074
  },
  "cross": {
    "rx": 0.5,
    "ry": 0.5,
    "x": 1969.0,
    "y": 929.0
  },
  "monitor": {
    "w": 3840,
    "h": 2160
  }
}
```
*   **重要**: `rect` は「画面内のPlate位置とサイズ」です（`x`,`y`,`w`,`h`）。
*   **重要**: 相対配置は `cross.rx`,`cross.ry` を優先して解釈してください（0.0〜1.0）。
*   `cross.x`,`cross.y` はスナップショット上の絶対座標（参考値）です。`rect` + `rx/ry` から再計算できるので、矛盾があっても補正せず、まず `rx/ry` を優先してください。
*   `subrect` は任意です。存在する場合、**画面座標の四角形**としてサブプレート領域を表します（メインPlate外にあることもあります: 例: 別位置に出るダイアログ）。
*   `monitor` は任意です（存在すれば画面サイズ）。
*   `version`/`basis`/`unit` や未知フィールドは将来拡張の可能性があります。AIは必要フィールド（`rect`,`cross` または `candidates`）以外で失敗しない実装・提案を優先してください。

#### basis について
現行の `basis` は次の文字列です。

- `plate`: rect/cross の基準がPlate
- `screen`: 画面基準（将来/例外用途）

Candidates JSON の `basis` は、次も使われます。

- `subplate`: `candidates[].rx/ry` の基準がサブプレート（`subrect`）

#### 代替ワークフロー（独立領域として扱う）
サブプレートを「メインとは独立した領域」として作業したい場合、AIは `basis: "subplate"` を使わずに、
**State JSON の `rect` 自体をサブプレート相当の四角形として返す**（= rectをサブ側の領域として扱う）方針でも構いません。

- 長所: AI側の計算は常に `rect` 基準で統一できる
- 注意: メインPlateとの関係（同時表示/比較）を1つのJSONで表す用途には向きません

## 出力時の注意（AI）
- JSONは **コードブロック（```json）** でそのまま貼れる形にしてください。


## コーディング指針
1.  **相対座標の活用**: ユーザーから `rx: 0.9` (右端付近) と指示された場合、ハードコーディングせず `relx=0.9` (Tkinter) や `left: 90%` (CSS) のように、親コンテナに対する相対位置での実装を優先してください。
2.  **Rect情報の利用**: `rect.w`, `rect.h` は、ウィンドウの初期サイズ(`geometry`)やアスペクト比の参考にしてください。

---

## ウィンドウサイズ実装のベストプラクティス (Tkinter/High-DPI)

ユーザーから「ウィンドウサイズをJSONの `rect` に合わせて」と指示された場合、特にHigh-DPI環境（Windows 150%スケールなど）でのトラブルを避けるために以下のルールを守ってください。

### 3. High-DPI環境での安全な実装指針
アプリの信頼性を高めるためには、OSの拡大率を正しく認識する **「DPI-Aware（可能ならPer-Monitor V2）」** での実装を推奨します。

*   **推奨**: DPI-Aware有効。`1px` = `1論理ピクセル` としてOSのAPIが正しくマッピングします。
*   **妥協案**: どうしても実装が複雑になる場合、DPI-Awareを無効（Unaware）にしてOSの拡大（DWM Scaling）に任せる手もありますが、座標の定義がぼやける可能性があります。

どちらの場合も、JSONの `rect` は **「論理ピクセル」** としてそのまま適用してください。AI側で `rect.w * 1.5` や `rect.w / 1.5` のような手動計算をする必要はありません。

### 4. 過剰な自動補正をしない
「本当に物理ピクセルが合っているか？」を確認するために、ウィンドウ生成後にスクリーンショット判定やAPI計測を行ってリサイズし直すような**自己補正ロジックは実装しないでください**。
環境ごとの誤差で無限ループや縮小過多に陥る原因になります。「単純に論理サイズの数値を設定する」のが最も安全で互換性の高い実装です。

### 5. ライブラリ固有の自動スケーリングに注意（重要）
使用するGUIライブラリによっては、OSの設定とは別に**独自の自動拡大/縮小機能（Auto-Scaling）**がデフォルトで有効になっている場合があります。
（例: `CustomTkinter` はデフォルトでスケーリングを行い、指定した `width=100` を `150` などに勝手に変換して描画することがあります）

- **AIへの指示**: ライブラリを選定する際は、そのライブラリが「数値を勝手に加工しないか」を確認し、必要であれば**自動スケーリングを無効化（1.0に固定）**するか、**標準ライブラリ（tkinter等）**を使用してください。
- ユーザーに提供するコードは、JSONの数値を **論理ピクセル（DIP）** として正しく反映できる構成にしてください。

---
このプロトコルを守ることで、ユーザーの「あそこ」や「この辺」という曖昧な指示を正確なコードに変換できます。

---

### 6. 【開発者向け】ズレが出る典型例とトラブルシューティング（100%の意味）

CrossPointの **State JSON** は、Windowsの表示倍率が 125% / 150% であっても **「論理ピクセル（DIP）」として正しい値** です。
したがって、AIは `env.ui_scale` を使って `rect.w * 1.5` や `rect.w / 1.5` のような手動補正をしてはいけません。値は常に JSONの数値をそのまま使うのが**適切**です。

それでも実装結果がズレる場合、原因はほぼ「実装側が別の座標系で解釈している」ことです。よくある例は次です。

**例1: アプリのDPI Awarenessが違う**
（DPI-aware / Unaware / System-aware の混在）
同じ `geometry(1600x900)` を指定しても、OS側のDPI仮想化（DWM Scaling）が介入し、見た目が巨大化・縮小・位置ズレを起こすことがあります。**これはJSONの誤りではありません。**

**例2: GUIライブラリが独自にAuto-Scalingしている**
（例: `CustomTkinter` など）
`width=100` を内部で `150` のように変換して描画し、JSONの意図と一致しなくなることがあります。必要ならAuto-Scalingを無効化（1.0固定）するか、標準tkinterを使ってください。

**例3: クライアント領域と外枠（タイトルバー込み）を混同している**
JSONの `rect.w / h` を「ウィンドウ外枠サイズ」と誤解すると、余白分だけズレます。JSONの `rect` は **“Plate（作業領域）”** として解釈し、実装側でも同じ対象（クライアント領域）に合わせてください。

**例4: マルチモニタ（混在DPI）で取得したJSONを、別モニタ条件で再現している**
同じPCでも、表示倍率やDPIが異なるモニタへ移動すると座標の意味が変わったように見えることがあります（補正で解決しようとすると悪化しやすい）。

#### 100%（Scale=1.0）の意味

**100%は「正しさの条件」ではありません。**
100%はあくまで **トラブルシューティングを単純化するための基準点** です。

100%環境では、論理と物理の差が最小になり、比較・検証が単純になります。
「ズレた」と感じたときに、同じJSONを **一時的に100%でも再現して差分を観察する** と、原因が「DPI awareness」「ライブラリのAuto-Scaling」「クライアント領域の取り違え」などに絞り込みやすくなります。

> **重要**: 125%/150%で生成されたJSONも常に正しい論理値です。
> 「150%で作ったから 1.5で割る／掛ける」と考える**必要はありません**。

## ライセンス
Copyright (c) 2026 じぇりー (Jelly)

## 免責事項
本ソフトウェアは「現状のまま（AS IS）」提供され、明示または黙示を問わず、いかなる保証も行いません。
作者は、本ソフトウェアの使用または使用不能に起因するいかなる損害（請求、損害、その他の責任）についても責任を負いません。
ご利用は自己責任でお願いします。

## 利用条件
本ソフトウェアは、個人利用・業務利用・商用利用を問わず利用できます。
ただし、本ソフトウェア自体またはその複製物を、無償・有償を問わず第三者へ再配布または転売することは禁止します。
その他の利用については作者にお問い合わせください。

---

# CrossPoint Protocol for AI Agents (English)

This document is a specification for AI agents (you) who collaborate with a user using the **CrossPoint** tool.
The user identifies screen regions/points and shares them with you as JSON. You must interpret that JSON and generate appropriate UI code (Tkinter, HTML/CSS, etc.).

## What CrossPoint Pro Adds (FYI)
- **Candidates visualization**: CrossPoint Pro adds a workflow where users can visualize AI-proposed points (`rx`/`ry`) on screen.

In this document (CrossPoint / standard), treat **State JSON** (`rect`/`cross`) as the only input, and return proposals as **State JSON** (or direct implementation code).

## Core Concepts

- **Plate**: The working area on the screen. In JSON, this is `rect` (`x`, `y`, `w`, `h`).
- **Sub-Plate**: An additional rectangular area (one). In JSON, this is `subrect` (`x`, `y`, `w`, `h`) (optional).
- **Cross**: A focus point inside the Plate. In JSON, this is `cross` (`rx`, `ry`, `x`, `y`).
- **Units**: `unit` is usually `px`. `x`, `y`, `w`, `h` may be floats (DPI/rendering).
- **Relative coordinates**: `rx`, `ry` are $0.0$ to $1.0$ (inside the Plate). The app clamps values; the AI should also assume the valid range.

## Important: JSON is a Snapshot (Single Source of Truth)
CrossPoint exists to share **coordinates and regions via JSON**. As an AI, you must not “correct” or infer from what CrossPoint looks like on the user's screen.

- The JSON the user provides is a **snapshot** and the **only truth**.
- The `cross` position means the **numeric coordinates/ratios** given in the JSON.
- Do not try to infer/correct based on the current CrossPoint overlay, cursor position, window placement, etc.
- If you are unsure, do not immediately ask for more info—first propose an implementation based strictly on the JSON values.

Note: CrossPoint's local UI appearance settings (e.g., overlay line thickness saved in `settings.json`) are unrelated to the State/Candidates JSON specification. Only use the numeric values in State/Candidates JSON.

### Recommended Interpretation Rule (AI internal policy)
When the user says “place X at the JSON cross”, interpret it as:

- **Use `rect` / `cross` values in the JSON as the only input for layout.**
- Prefer relative coordinates (`rx`/`ry`). Use `rect.w`/`rect.h` as a hint for initial size if needed.

### Suggested User Message Snippet (to prevent misunderstandings)
Encourage the user to include this (or similar) text when sending JSON:

```
The following JSON is a snapshot generated by CrossPoint.
Do not rely on CrossPoint's on-screen appearance.
Use only these numeric values (rect/cross) as the basis for layout.
```

UI coordinates are expressed as a **reference rectangle (Plate/Rect)** plus a **relative focus point (Cross)**.

## JSON Formats

CrossPoint uses two main JSON formats:

- **State JSON (User → AI / AI → User)**: A snapshot containing Plate (`rect`) and Cross (`cross`).

### 1) State (User → AI)
This is the snapshot that expresses intent like “I want the button around here”.

#### Optional: `env` (DPI / scaling metadata)
To help diagnose and reduce layout mismatches caused by environment differences (Windows display scaling, Tk/CTk scaling, etc.), State/Candidates JSON may optionally include an `env` block.

- `env.dpi_ppi`: pixels-per-inch reported by Tk (e.g., 96 / 120 / 144)
- `env.ui_scale`: `dpi_ppi / 96.0` (Windows 100% == 1.0)
- `env.tk_scaling`: Tk's `tk scaling` value

AI should treat `env` as optional (never required). When present, use it only as a hint to estimate scaling-related drift and to consider correction factors.

```json
{
	"version": "1.0",
	"basis": "plate",
	"unit": "px",
	"env": {
		"dpi_ppi": 120.0,
		"ui_scale": 1.25,
		"tk_scaling": 1.25
	},
	"rect": {
		"x": 1168,
		"y": 392,
		"w": 1602,
		"h": 1074
	},
	"cross": {
		"rx": 0.5,
		"ry": 0.5,
		"x": 1969.0,
		"y": 929.0
	},
	"monitor": {
		"w": 3840,
		"h": 2160
	}
}
```

- **Important**: `rect` is the Plate position and size on the screen.
- **Important**: For layout, prefer `cross.rx` / `cross.ry` ($0.0$–$1.0$).
- `cross.x` / `cross.y` are absolute coordinates in the snapshot (reference). They can be recomputed from `rect` + `rx/ry`. If there is any inconsistency, **do not “fix” it**—prefer `rx/ry`.
- `subrect` is optional. If present, it is an **absolute screen rectangle** representing the Sub-Plate (may be outside the main Plate, e.g., a dialog that appears elsewhere).
- `monitor` is optional.
- Unknown fields may appear in future versions. Prefer implementations that are tolerant of extra fields.

#### About `basis`
Current `basis` values:

- `plate`: `rect`/`cross` are Plate-based
- `screen`: screen-based (future/exceptional use)

Candidates JSON may also use:

- `subplate`: `candidates[].rx/ry` are relative to the Sub-Plate (`subrect`)

#### Alternative Workflow (treat Sub-Plate as an independent area)
If you want to treat Sub-Plate as an independent work area, you may avoid `basis: "subplate"` and instead:

- Return a State JSON where `rect` itself represents the Sub-Plate area (i.e., treat the Sub-Plate rectangle as the main Plate).

Pros: calculations stay consistently `rect`-based.
Cons: not ideal if you need to represent both main Plate and Sub-Plate together in one JSON.

## Output Notes (AI)
- Return JSON in a fenced code block tagged as `json`.


## Coding Guidelines
1. **Use relative coordinates**: If the user says `rx: 0.9` (near the right edge), prefer relative placement such as `relx=0.9` (Tkinter) or `left: 90%` (CSS), rather than hard-coding pixels.
2. **Use `rect` size as a hint**: `rect.w` / `rect.h` can guide initial window size (`geometry`) or aspect ratio.

---

### 5. Typical Alignment Issues and Troubleshooting (Meaning of 100%) [Developer Guide]

CrossPoint **State JSON** values are **correct Logical Pixels (DIP)** even if generated at 125% or 150% Windows display scaling.
Therefore, the AI must NOT manually correct values like `rect.w * 1.5` or `rect.w / 1.5` using `env.ui_scale`. Using the JSON values **as-is** is always appropriate.

If the implementation still results in misalignment, the cause is almost always "the implementation side interpreting the coordinates in a different system". Common examples include:

**Case 1: Mismatched DPI Awareness**
(Mixing DPI-aware / Unaware / System-aware)
Even with the same `geometry(1600x900)`, OS-side DPI virtualization (DWM Scaling) may intervene, causing the window to appear huge, shrunk, or offset. **This is not an error in the JSON.**

**Case 2: GUI Library Auto-Scaling**
(e.g., `CustomTkinter`)
Some libraries convert `width=100` to `150` internally for drawing, conflicting with the JSON's intent. If necessary, disable Auto-Scaling (fix to 1.0) or use standard Tkinter.

**Case 3: Confusing Client Area with Window Frame**
Misinterpreting JSON `rect.w / h` as "Window Outer Size" causes offsets equal to the border/title bar size. JSON `rect` should be interpreted as **"Plate (Working Area)"**, and the implementation should match the same target (Client Area).

**Case 4: Reproducing on different monitors (Mixed DPI)**
Moving to a monitor with different scaling or DPI may make the coordinates seem incorrect (trying to fix this with manual calculation usually makes it worse).

#### The Meaning of 100% (Scale=1.0)

**"100%" is NOT a condition for correctness.**
It is merely a **baseline to simplify troubleshooting**.

In a 100% environment, the difference between logical and physical pixels is minimized, making comparison and verification simple.
If you feel "it's misaligned", **temporarily reproducing it at 100% and observing the difference** helps narrow down the cause to "DPI awareness", "Library Auto-Scaling", "Client Area confusion", etc.

> **Important**: State JSON generated at 125% or 150% is also valid logical data.
> There is **no need to think** "It was made at 150%, so I must divide/multiply by 1.5".

### 3. Safe Implementation Guidelines for High-DPI Environments
 To ensure reliability, it is recommended to implement the app as **"DPI-Aware (Per-Monitor V2 if possible)"** so it correctly recognizes OS scaling.

*   **Recommended**: DPI-Aware enabled. The OS API maps `1px` to `1 logical pixel` correctly.
*   **Compromise**: If implementation is too complex, you can disable DPI Awareness (Unaware) and rely on OS bitmap scaling (DWM Scaling), but coordinate definition may become blurry.

In either case, apply the JSON `rect` as **"Logical Pixels"** directly. The AI does not need to manually calculate `rect.w * 1.5` or `rect.w / 1.5`.

### 2. Base on "Client Area"
The `rect` provided by the user typically refers to the "visible content area."
Implement `geometry(WxH)` as the size of the **Client Area**, not the entire window.

- There is no need to calculate and subtract standard window frame sizes (Non-client area) like title bars.
- By setting the specified `rect.w` / `rect.h` directly in `geometry`, you ensure the user's expected "content space" is preserved.

### 3. Avoid Over-Correction
**Do NOT implement self-correction loops** (e.g., measuring the window after creation and resizing it again) to verify physical pixel accuracy.
Scaling discrepancies across environments can cause infinite loops or excessive shrinking. "Simply setting the logical size value" is the safest and most compatible approach.

---
Following this protocol helps convert vague human instructions (“around here”) into precise, reproducible UI code.

---

## License / Usage Terms
### Development / Copyright
Jelly

X(Twitter): https://x.com/dialoguevolts

note: https://note.com/sane_swift272

Bluesky: https://bsky.app/profile/jellycotton.bsky.social

### License
Copyright (c) 2026 Jelly

### Disclaimer
THIS SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND. THE AUTHOR IS NOT LIABLE FOR ANY CLAIM, DAMAGES, OR OTHER LIABILITY ARISING FROM THE USE OF THIS SOFTWARE. USE AT YOUR OWN RISK.

### Usage Terms
This software may be used for personal, business, and commercial purposes alike.
However, redistributing or reselling the software itself or copies of it to third parties, whether free of charge or for a fee, is prohibited.

For any other use, please contact the author.

## Utility Tools for DPI & Debugging

Two independent tools are provided in the `tools/` directory to help diagnose and solve DPI/Scaling issues.

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
