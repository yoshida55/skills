---
name: mockup
description: スクショからCSS設計情報付きSVG設計図を生成する
---

# mockup スキル

スクリーンショットを受け取り、CSS配置情報をアノテーションしたSVG設計図を生成する。

---

## 実行フロー

### 1. スクショ確認
スクショが渡されていない場合は必ず先に要求する:
「スクリーンショットを渡してください」

スクショが渡されたら → 分析してSVG生成へ

### 2. ファイル番号の決定
プロジェクトルートの `mockup_*.svg` を確認して次の番号を提案する:

```
既存ファイル確認 → mockup_01_concept.svg, mockup_02_case.svg があれば
→ 「今回は mockup_03_[セクション名].svg にします」と伝えて生成
```

- 既存なし → `mockup_01_[セクション名].svg`
- ユーザーが番号を指定した場合はそちらを優先

### 3. レイアウト分析
スクショから以下を読み取る:
- セクション全体の背景色・サイズ感
- 要素の配置方法（絶対配置か通常フローか）
- 横並びの仕組み（flex か grid か）
- 重なりやはみ出しがあるか（overflow: hidden が必要か）
- 使うHTMLタグ（section / article / button / img など）

### 4. SVG生成
出力ファイル: `mockup_[連番]_[セクション名].svg`（プロジェクトのルートに配置）

---

## レイアウト設計方針（シンプル優先）

### 装飾写真はすべて absolute
- 背景に散りばめる写真・装飾画像 → `position: absolute`
- section に `position: relative` を設定して基点にする
- **grid / flex で写真を並べない**（複雑になるため）

### 中央テキストは通常フロー
- 見出し・本文・Conceptラベルなど → 通常フロー（position なし）
- padding でレイアウトを調整

### flex / grid を使う場面
- カードの横並び → `flex`
- ナビゲーション・タグ群 → `flex flex-wrap`
- 格子状に均等に並ぶ要素 → `grid`

---

## SVG書き方ルール

### サイズ
```
width="1120" height="[内容に合わせる]"
viewBox="0 0 560 [内容の半分]"
```
2倍スケールが基本。内容が長ければ高さを増やす。

### XMLルール（必須）
- コメント内に `--` を使わない（XMLエラーになる）
  - ❌ `<!-- ---- 上部 ---- -->`
  - ✅ `<!-- 上部エリア -->`
- `<` `>` は `&lt;` `&gt;` でエスケープ

---

## 色コード（凡例と統一）

| 色 | 用途 |
|---|---|
| `#ff8800` | relative（基点になる要素） |
| `#0055ff` | absolute（絶対配置） |
| `#007722` | flex / grid（レイアウト） |
| `#cc7700` | overflow: hidden |
| `#ff4444` | fixed（画面固定） |

全て `font-weight="bold"` で表示。

---

## 凡例バー（必ず冒頭に入れる）

```svg
<rect x="0" y="0" width="560" height="28" fill="#111"/>
<rect x="8" y="8" width="12" height="12" fill="none" stroke="#ff8800" stroke-width="2"/>
<text x="24" y="19" fill="#ff8800" font-weight="bold">relative</text>
<rect x="110" y="8" width="12" height="12" fill="none" stroke="#0055ff" stroke-width="2"/>
<text x="126" y="19" fill="#0055ff" font-weight="bold">absolute</text>
<rect x="220" y="8" width="12" height="12" fill="none" stroke="#007722" stroke-width="2"/>
<text x="236" y="19" fill="#007722" font-weight="bold">flex / grid</text>
<rect x="330" y="8" width="12" height="12" fill="none" stroke="#cc7700" stroke-dasharray="3" stroke-width="2"/>
<text x="346" y="19" fill="#cc7700" font-weight="bold">overflow: hidden</text>
<rect x="460" y="8" width="12" height="12" fill="none" stroke="#ff4444" stroke-width="2"/>
<text x="476" y="19" fill="#ff4444" font-weight="bold">fixed</text>
```

---

## アノテーションの書き方

### 枠線の意味
- **実線**（stroke-dasharray なし）→ 通常の要素
- **破線**（stroke-dasharray="4"）→ レイアウトコンテナ（flex/gridの親）
- **太い破線**（stroke-dasharray="5"）→ overflow: hidden の範囲

### テキスト注釈パターン
```
親要素:    section [position: relative]
絶対配置:  img [absolute]  top: 0  left: 0
flex:      flex  justify-content: space-between
grid:      grid  grid-template-columns: repeat(2, 1fr)
タグ名:    h2  /  p  /  button  /  span  ← タグ名
```

---

## 注意事項

- クラス名は書かない（後でユーザーが決める）
- 「コピペできるHTML」にしない（設計図であること）
- 判断が必要な箇所は「？」をつけて注記する（例: `img [absolute?]`）
- 一番下に余計な説明ブロックを追加しない（SVG内に必要事項は収める）
- 複雑にしない: シンプルな設計で表現できるなら grid/flex より absolute を優先
