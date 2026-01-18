# CrossPoint

- English version: [README_EN.md](README_EN.md)

**CrossPoint** は、AIに「このへん」「このくらいのサイズ」といった画面領域を **座標スナップショット（JSON）** として正確に共有するためのツールです。

このZIPは **EXE配布版** です（Pythonインストール不要）。

## 基本コンセプト（Plate / Cross / JSON）
- **Plate**: 作業領域となる四角形（基準の枠）
- **Cross**: Plate内の注目点（相対位置）
- **JSON**: 状態を数値化したデータ。これをコピー＆ペーストして正確な位置を共有します。

ポイントは **「見た目」ではなく「JSONの数値」が唯一の根拠になる** ことです。

## 何ができるか
- 画面上に Plate（作業領域）を定義
- Cross を動かして注目点を指定
- 現在の状態を **State JSON** としてコピーし、AIに送る

## クイックスタート（起動 〜 JSONコピー）
1. ZIPを任意のフォルダに展開
2. `CrossPoint.exe` を実行
3. **Set Plate** → 画面を **ドラッグ** してPlate四角形を描画
<img width="3837" height="2169" alt="スクリーンショット (311)" src="https://github.com/user-attachments/assets/31272527-353f-42a5-9be3-21a22e42c390" />
<img width="3835" height="2169" alt="スクリーンショット (312)" src="https://github.com/user-attachments/assets/46796783-6988-4612-815e-75504a913417" />
<img width="3836" height="2169" alt="スクリーンショット (314)" src="https://github.com/user-attachments/assets/a0f58337-2c2e-4e2a-8712-36d30369f331" />
	- 作業中のダイアログなどに合わせて囲むと、クロスによる座標指示がしやすくなります。
	- AIに「このサイズで作って」と指示するのにも使えます。
	- 操作中はオーバーレイモード（画面が暗転）になります。右クリックまたは `Esc` で解除できます。
5. **Move Cross** → クリック/ドラッグで Cross を配置
<img width="3835" height="2169" alt="スクリーンショット (315)" src="https://github.com/user-attachments/assets/79a95f3a-d257-458b-9d0f-4edf9ad8a6a3" />
7. **Copy State JSON** → JSONをコピーしてAIのチャットに貼り付け

## ボタン機能一覧
- **Set Plate**: Plate（基準枠）を作成します。
- **Square**: Plateを正方形にします。
- **□ Center**: Plateを画面中央に移動します。
- **Move Cross**: Cross移動モードにします。
- **+ Center**: CrossをPlateの中心に戻します。
- **Line thickness**: オーバーレイの線の太さを調整します（次回起動時も保持）。
- **Copy State JSON**: 現在の状態（Plate + Cross）をJSONとしてコピーします。
	- 実行環境の DPI/スケール情報（`env`）も任意で含まれます（例: `ui_scale=1.25`）。

## ステータスバー
画面下部のステータスバーには、現在のモードや直近のアクション結果が表示されます。
- 例: `Ready` / `Mode: SET_PLATE` / `State copied!`

## DPI / スケール（env）
State JSON には DPI/スケール情報（`env`）が含まれることがあります。
ただし **rect（Plate四角形）は常に「クライアント領域の論理px」** として扱い、`env` は観測用メタ情報です（rectの補正には使いません）。
もしAIが出力したレイアウトが、意図したサイズや位置とズレている場合、`env.ui_scale`（例: 1.25 / 1.5）と Windowsのディスプレイ設定（拡大縮小）を確認してください。

## 設定の保存場所
EXE版の設定はユーザープロファイル配下に保存されます。

- CrossPoint: `%APPDATA%\CrossPoint`
- CrossPoint Pro: `%APPDATA%\CrossPoint Pro`

※ `AppData` は隠しフォルダです。
- `Win + R` → `%APPDATA%\CrossPoint` → Enter で開けます。

もし `settings.json` を削除しても古い設定が戻ってくる場合、別の版から自動移行されている可能性があります。その場合は両方のフォルダを確認してください。

## トラブルシューティング
- Windows SmartScreen / Defender が初回起動時に警告を出す場合があります。
- ウィンドウが見当たらない場合、タスクバーや Alt+Tab を確認してください。
- 起動直後にクラッシュする場合、`err.txt` や `startup.log` を確認してください。
- AIの出力がズレる場合、State JSON の `env`（DPI/スケール）を確認してください。

## AIに渡す「仕様書」（重要）
AIが勝手にDPI補正などをして座標が狂うのを防ぐため、AIエージェント向けの仕様書を同梱しています。

- `README_AI.md`（日本語）
- `README_AI_EN.md`（English）

AIに作業を依頼する際は、必要に応じてこのドキュメントのルール（例: rectは論理pxで補正しない、crossはrx/ry優先）に従うよう明示してください。

---

## 便利な周辺ツール (tools/)

DPI/スケーリング起因の座標ズレ問題を調査・解決するために、以下の2つのツールを `tools/` フォルダに同梱しています。

### 1. DPI/Rect Diagnosis Tool (`tools/CrossPointRectDiagnosis.exe`)
**目的**: CrossPointが **rect=クライアント領域の論理px** という規約どおりに動いているかを診断すること（補正係数を保存・適用するツールではありません）。

**使い方**:
1. `tools/CrossPointRectDiagnosis.exe` を実行します。
2. **"Show Target"** を押し、既知のサイズの赤い四角形ウィンドウを表示させます。
3. `CrossPoint` を起動し、**"Set Plate"** で「ターゲットの赤枠」を**ピッタリ**囲んでください（タイトルバーを含めず、中身だけを囲むのがポイントです）。
4. CrossPointから State JSON をコピーします。
5. ツール側の **"Paste & Analyze"** を押します。
6. 期待値（Target）とCapturedの比（scale）が表示されます。理想は 1.0 付近です。

### 2. Dummy Window Generator (`tools/CrossPointDummyJsonWindow.exe`)
**目的**: State JSON で指定された座標が、実際に画面上の意図した位置に出るかを検証すること。

**なぜこれが必要？**
- AIが生成したJSONが、自分の想定するUI要素と合致しているかを確認するため。
- このツールは **rectをそのまま（除算なし）** 使ってウィンドウを生成し、必要に応じて **クライアント原点のズレ（タイトルバー等）** を best-effort で補正します。

**使い方**:
1. `tools/CrossPointDummyJsonWindow.exe` を実行します。
2. テキストボックスに任意の **State JSON**（DeepSeek等が生成したもの、あるいはCrossPointからコピーしたもの）を貼り付けます。
3. **"Spawn Window"** を押します。
4. グレーのウィンドウが表示されます。
   - ウィンドウの**中身（クライアント領域）**が、JSONの `rect` と一致します。
   - **赤い十字** が、`cross` の位置を示します。
   - （もしあれば）**青いボタン** が、`subrect` の位置を示します。

**AIエージェントの方へ**:
座標ズレが疑われる場合、ユーザーにこう依頼してください：
1. 「私が生成したJSONをコピーしてください」
2. 「`tools/CrossPointDummyJsonWindow.exe` に貼り付けて表示してみてください」
3. 「ウィンドウが作りたいUIの位置と合っているか教えてください」

補足（上級者向け）:
- `tools/CrossPointDebugWindowRects.exe` は Win32/Tk の outer/client を観測する **デバッグ用** です（State JSON そのものではありません）。

---

## Pro版（Booth）
Pro版（有料）の販売ページはこちら：
Booth: [https://jellycotton.booth.pm/items/7886153]

※ このGitHub（無料版）には Pro版は同梱されません。

---

## ライセンス / 利用条件
### 開発・著作
じぇりー (Jelly)

Bluesky: https://bsky.app/profile/jellycotton.bsky.social
### ライセンス
Copyright (c) 2026 Jelly

### 利用条件
本ソフトウェアは個人利用を目的として提供されています。
再配布および商用利用は許可していません。
その他の利用については作者にお問い合わせください。
