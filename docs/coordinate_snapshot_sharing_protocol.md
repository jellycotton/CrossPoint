# Coordinate Snapshot Sharing Protocol

Author: Jelly (じぇりー)  
Project: CrossPoint  
First published: 2026-01-30  
Repository: https://github.com/jellycotton/CrossPoint.git

This document describes a coordinate snapshot sharing method
for humanAI UI collaboration.

---

# Coordinate Snapshot Sharing Protocol for HumanAI UI Collaboration

## 1. Technical Field

This technique relates to a **method for sharing and interpreting coordinate information**
used when a user and an AI agent collaboratively decide UI layout and screen composition.

In particular, it targets a **coordinate snapshot sharing method** that can convey the user's layout intent to an AI without loss even across:

- different display resolutions
- different DPI scales
- different UI frameworks
- different execution environments

---

## 2. Background and Problems

Conventionally, instructions to an AI for UI placement relied mainly on:

- ambiguous natural language (e.g., a bit more to the right)
- specifying absolute coordinates in pixels
- environment-dependent screen size and DPI information

However, these methods have the following problems:

1. **Environment dependency**  
   The same absolute coordinates can produce very different appearances, occupancy, and positional relationships across resolutions and DPI settings.

2. **Loss of intent**  
   The essential user intentrelative placement within which regionis not preserved as numbers.

3. **AI inference/correction interference**  
   If the AI tries to infer or correct based on the real screen/environment, unintended transforms and endless adjustment loops can occur, breaking the layout.

---

## 3. Overview of the Solution

To solve these issues, this method adopts **Coordinate Snapshot Sharing**.

Key points:

- Define a rectangular region used as the basis for UI placement (Plate)
- Express focus points / placement intent inside the Plate as **relative coordinates**
- Fix and share them as a **JSON snapshot**
- The AI interprets and implements based only on this snapshot as the single source of truth

This enables sharing the user's layout intent itself, independent of environment differences.

---

## 4. Required Components (Mandatory)

### 4.1 Reference Rectangle (Plate)

A rectangle representing the working area on screen, including:

- position (x, y)
- size (w, h)

This rectangle functions as the **reference space** for interpreting layout intent.

### 4.2 Relative Focus Point (Cross)

A focus point / placement intent inside the Plate expressed as **normalized relative coordinates (rx, ry) in the range 0.01.0**.

This relative coordinate expresses the positional relationship inside the Plate itself and is independent of:

- resolution
- DPI
- UI framework

### 4.3 Coordinate Snapshot (State JSON)

A JSON data snapshot that includes the Plate and Cross, fixed as the state at a moment in time.

This JSON is treated as:

- the **single source of truth** at that moment
- not subject to inference or correction
- the reference value for subsequent processing

---

## 5. Optional Elements (Extended Info)

In addition to mandatory elements, the snapshot may include:

- additional rectangles (Sub-Plate / Sub-Rect)
- monitor size information
- environment info such as DPI / UI scaling

However, these are restricted to:

- diagnostics / explanation / troubleshooting only  
- **must not be used to correct layout intent**

---

## 6. Interpretation Rules (Important)

The following rules apply:

1. Relative coordinates (rx, ry) represent the true intent
2. Absolute coordinates (x, y, w, h) are the container and reference values
3. Even if there are contradictions, the AI must not apply its own corrections
4. The AI must not infer real screen/cursor/environment state

This prevents layout breakage caused by goodwill corrections.

---

## 7. Technical Effects

This method provides:

- preserved layout intent across different resolutions
- avoidance of breakage due to DPI scaling differences
- consistent coordinate interpretation between user and AI
- improved reproducibility of UI implementation

In particular, it avoids the traditional problem of:
**numbers are correct, but the meaning is broken.**

---

## 8. Essence of the Invention

The essence is fixing and sharing **the meaning (intent) of coordinates**
as a snapshot, rather than merely sharing numeric coordinates.

This differs from:

- a simple UI placement tool
- a simple relative layout
- a simple JSON format

It is a structure that fixes the user's intent in a way the AI cannot misinterpret.

---

## 9. Summary

This coordinate snapshot sharing method removes uncertainty caused by:

- environment differences
- inference
- correction

and shares layout intent directly as numbers.

It is not dependent on a particular UI library, OS, or AI model and can be broadly applied.

---

# 座標スナップショット共有方式（原文）

# Coordinate Snapshot Sharing Protocol

Author: Jelly (じぇりー)  
Project: CrossPoint  
First published: 2026-01-30  
Repository: https://github.com/jellycotton/CrossPoint.git

This document describes a coordinate snapshot sharing method
for humanAI UI collaboration.


# Coordinate Snapshot Sharing Protocol for HumanAI UI Collaboration

（座標スナップショット共有方式）

## 1. 技術分野

本技術は、ユーザーとAIエージェントが協調してUI配置画面構成を決定する際に用いられる、
**座標情報の共有および解釈方式**に関する。

特に、
異なる表示解像度
異なるDPIスケール
異なるUIフレームワーク
異なる実行環境

を跨いだ状況においても、
ユーザーの配置意図を損なわずにAIへ伝達するための
**座標スナップショット共有方式**を対象とする。

---

## 2. 背景技術と課題

従来、UI配置をAIに指示する方法は主に以下に依存していた。

自然言語による曖昧な指示（「もう少し右」など）
絶対座標値（px）による指定
実行環境依存の画面サイズやDPI情報

しかし、これらの方法には次の問題があった。

1. **環境依存性**
   同一の絶対座標値であっても、
   解像度やDPIが異なる環境では、
   UIの見た目占有率位置関係が大きく変化する。

2. **意図の非保存性**
   「どの領域に対して、どの位置関係で配置したいか」という
   ユーザーの本質的な意図が数値として保存されない。

3. **AIによる推測補正の介入**
   AIが実画面や環境情報を推測補正しようとすると、
   意図しない変換や無限補正が発生し、
   結果として配置が破綻する。

---

## 3. 解決手段の概要

本技術では、上記課題を解決するために、
**座標スナップショット共有方式**を採用する。

本方式の要点は次の通りである。

UI配置の基準となる矩形領域（Plate）を定義する
その内部における注目点配置意図を **相対座標** として表現する
これらを **JSON形式のスナップショット** として固定し共有する
AIは、このスナップショットのみを唯一の根拠として解釈実装を行う

これにより、
環境差や表示条件の違いに依存せず、
ユーザーの配置意図そのものをAIと共有できる。

---

## 4. 構成要件（必須要素）

本方式は、少なくとも以下の構成要件を含む。

### 4.1 基準矩形（Plate）

画面上の作業領域を表す矩形であり、
以下の情報を含む。

位置（x, y）
サイズ（w, h）

この矩形は、
**配置意図を解釈するための基準空間**として機能する。

### 4.2 相対座標による注目点（Cross）

Plate 内における注目点または配置意図を、
**0.0〜1.0 の正規化された相対座標（rx, ry）**として表現する。

この相対座標は、

解像度
DPI
UIフレームワーク

に依存せず、
**Plate 内での位置関係そのもの**を表す。

### 4.3 座標スナップショット（State JSON）

上記の Plate および Cross を含む情報を、
**一時点の状態として固定した JSON データ**として生成共有する。

この JSON は、

その時点における唯一の正（Single Source of Truth）
推測や補正の対象ではない
後続処理における基準値

として扱われる。

---

## 5. 任意要素（拡張情報）

本方式は、必須要素に加えて、以下の任意要素を含んでもよい。

追加矩形領域（Sub-Plate / Sub-Rect）
モニタサイズ情報
DPI / UIスケール等の環境情報

ただし、これらの情報は、

**配置意図を補正するために使用してはならない**
診断説明トラブルシューティング用途に限定される

---

## 6. 解釈規則（重要）

本方式において、解釈には次の規則が適用される。

1. **相対座標（rx, ry）が意図の正である**
2. 絶対座標（x, y, w, h）は器および参考値である
3. 矛盾がある場合でも、AIは独自補正を行ってはならない
4. AIは実画面カーソル位置環境状態を推測してはならない

これにより、
「AIの善意による補正」が引き起こす破綻を防ぐ。

---

## 7. 技術的効果

本方式により、次の効果が得られる。

異なる解像度間で配置意図が保持される
DPIスケール差による破綻が回避される
ユーザーとAIの間で座標解釈が一致する
UI実装の再現性が向上する

特に、
**「数値は正しいが意味が壊れる」**
という従来の問題を回避できる。

---

## 8. 発明の本質

本技術の本質は、

**座標値そのものではなく、
座標の意味（意図）をスナップショットとして共有する点**

にある。

これは、

単なるUI配置ツール
単なる相対レイアウト
単なるJSONフォーマット

とは異なり、
**人間の意図を、AIが誤解しない形で固定する構造**である。

---

## 9. まとめ

本座標スナップショット共有方式は、
ユーザーとAIが協調してUI構成を行う際に、

環境差
推測
補正

といった不確定要素を排除し、
**意図を数値として直接共有する**ための技術である。

本方式は、
特定のUIライブラリ、OS、AIモデルに依存せず、
広範な応用が可能である。

---
