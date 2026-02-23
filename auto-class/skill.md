---
name: auto-class
description: HTMLタグにクラス名を自動付与
triggers:
  - auto-class
  - クラス名
  - クラス付与
---

# auto-class スキル

> ## ⚡ 絶対最初にやること（これを忘れたら全部やり直し）
> **スキル読み込み直後、作業の1行目は必ず `TodoWrite` でStep 0のチェックリストを出す**
> ❌ TodoWrite なしで Read や Edit を始める → 禁止
> ✅ TodoWrite → その後に作業 → 正しい順序

HTMLタグにクラス名を自動付与するスキル

## 使い方
```
/auto-class
```

## 📐 命名規則（絶対厳守）

### 基本ルール
```
基本: [section]_[element]          (2単語推奨)
必要時: [section]_[element]_[detail] (3単語OK)

例：
✅ header_title (2単語 - シンプルな場合)
✅ header_item_title (3単語 - 階層が深い場合)
❌ header_item_title_active (4単語 - 長すぎ)
```

### 使い分け
- **2単語推奨**: 基本はシンプルに2単語
- **3単語OK**: 階層が深い・区別が必要な場合のみ
- **4単語以上禁止**: 複雑すぎる場合は構造を見直す

### 接続ルール
- ✅ `_` (アンダースコア) のみ使用
- ❌ `-` (ハイフン) 禁止
- ❌ キャメルケース禁止

### 省略禁止
- ❌ `accordion_desc` → ✅ `accordion_description`
- ❌ `accordion_btn` → ✅ `accordion_button`
- ✅ 例外: `img` `nav` のみOK

### 番号禁止
- ❌ `accordion_item1` `accordion_item2`
- ✅ `accordion_item` を複数回使う

### セクション外枠の命名
- ✅ `_area` を使用（`<section>` `<header>` `<footer>` の外枠）
- ❌ `_container` は使わない
- 例: `header_area` `news_area` `footer_area`

---

## 🎯 実行フロー

### Step 0. TodoWriteでチェックリスト表示（絶対厳守・スキル開始直後に必ず実行）

以下のTodoWriteを**最初に必ず実行**し、チャット画面にチェックリストを表示する:

```
TodoWrite([
  { content: "既存HTMLをgrepして共通クラス確認",                   status: "in_progress", activeForm: "既存クラスをgrep確認中" },
  { content: "スクショの文言を全部読み取る（読めない箇所はユーザーに確認）", status: "pending", activeForm: "スクショ文言を読み取り中" },
  { content: "スクショと既存テキストの差分確認（文言・電話番号等）", status: "pending",     activeForm: "スクショとテキスト差分確認中" },
  { content: "セマンティックタグチェック（address/article/time等）", status: "pending",     activeForm: "セマンティックタグチェック中" },
  { content: "全タグにクラス名付与",                               status: "pending",     activeForm: "クラス名付与中" },
  { content: "構造・命名ミスの指摘",                               status: "pending",     activeForm: "構造・命名ミス確認中" },
  { content: "不足タグの確認・ユーザーへの追加提案",               status: "pending",     activeForm: "不足タグ確認中" }
])
```

各ステップ完了ごとに `completed` に更新すること。

- ⚠ **スクショが提供された場合、スクショが正式な情報源**
- ⚠ **読み取れない箇所は推測せず、必ずユーザーに質問する**
- ⚠ **共通クラスが見つかったら、新規クラスを作らず使い回す**

### Step 1. ユーザーがHTMLタグのみ記述
```html
<div>
  <h2>質問タイトル</h2>
  <p>回答内容</p>
</div>
```

### Step 2. AIが全タグにクラス名付与
```html
<div class="accordion_area">
  <div class="accordion_item">
    <h2 class="accordion_title">質問タイトル</h2>
    <p class="accordion_description">回答内容</p>
  </div>
</div>
```

---

## ⚠️ 確認が必要な場合

以下の場合は**必ずユーザーに確認**:

### ケース1: ネスト深すぎ (3階層以上)
```html
<div><div><div><div>深い</div></div></div></div>
```
→ 「4階層あるけど、どう命名する？」

### ケース2: 文法的におかしい構造
```html
<button><h1>見出し</h1></button>
```
→ 「ボタン内に見出しは変だけどOK？」

### ケース3: 役割が不明
```html
<div><!-- 複雑な役割 --></div>
```
→ 「これは `accordion_wrapper` でいい？」

### ケース5: セマンティックタグが不適切
コンテンツの意味に合ったHTMLタグが使われていない場合は指摘する（修正はしない）:

```html
<!-- ❌ 住所を <p> で書いている -->
<p>住所：熊本県...</p>
→ 「住所は <address> タグが適切です」

<!-- ❌ ブログ記事を <div> で書いている -->
<div class="blog_item">...</div>
→ 「記事コンテンツは <article> タグが適切です」

<!-- ❌ 日付を <p> で書いている -->
<p>2025.11.11</p>
→ 「日付は <time datetime="2025-11-11"> タグが適切です」
```

**セマンティックチェック一覧**

| コンテンツの種類 | 適切なタグ | よくあるNG |
|----------------|-----------|-----------|
| 住所 | `<address>` | `<p>` `<div>` |
| 記事・ブログ | `<article>` | `<div>` |
| 日付・時刻 | `<time datetime="...">` | `<p>` `<span>` |
| ナビゲーション | `<nav>` | `<div>` `<ul>` |
| メインコンテンツ | `<main>` | `<div>` |
| サイドバー | `<aside>` | `<div>` |
| 図・キャプション | `<figure>` + `<figcaption>` | `<div>` + `<p>` |
| リスト | `<ul>` / `<ol>` + `<li>` | `<div>` の羅列 |
| 定義リスト | `<dl>` + `<dt>` + `<dd>` | `<p>` の羅列 |
| 太字強調（意味あり） | `<strong>` | `<b>` |
| 斜体強調（意味あり） | `<em>` | `<i>` |
| ボタン | `<button>` | `<div>` `<a>` |

指摘は **「〇〇は `<タグ名>` が適切です」** の1行で簡潔に。

---

### ケース6: 不足タグの検出・追加提案
HTMLを見て、よく忘れられる必須・推奨タグが抜けていたらユーザーに提案する（追加はしない）:

**チェックリスト**:
| タグ / 要素 | よくある見落とし |
|------------|----------------|
| `<footer>` 内の著作権表示 | `copyright` div が footer の外に出ている |
| `<main>` タグ | body直下のメインコンテンツが `<div>` になっている |
| `<nav>` タグ | ナビゲーションが `<div>` になっている |
| `lang` 属性 | `<html lang="ja">` が抜けている |
| `alt` 属性 | `<img>` に `alt` がない |
| `<title>` タグ | `<head>` 内に `<title>` がない |

**報告形式**:
```
💡 不足タグ・要素の提案:
- copyright が <footer> の外にあります。<footer> 内に移動しますか？
- <main> タグがありません。メインコンテンツを <main> で囲みますか？
```

指摘のみ。追加・移動はユーザーが「追加して」と言ってから実行する。

---

### ケース4: 構造とビジュアルイメージが合っていない
HTMLの構造が実際のレイアウト意図と噛み合っていないと判断した場合は指摘する:

```html
<!-- 例: 写真を全部wrapしている不要なdiv -->
<section>
  <div class="photo_wrapper">  ← これが必要か？
    <img /><img /><img />
  </div>
</section>
```
→ 「`photo_wrapper` のdivは不要かもしれません。sectionを `position: relative` にすれば直下のimgをabsoluteで配置できます」

指摘は1行で簡潔に。修正はしない（ユーザーが判断する）。

---

## 📋 よく使う要素名パターン

### 2単語（基本）
| 役割 | クラス名 |
|------|----------|
| 外枠 | `[section]_area` |
| 1項目 | `[section]_item` |
| 見出し | `[section]_title` / `[section]_header` |
| 本文 | `[section]_description` / `[section]_content` |
| ボタン | `[section]_button` |
| アイコン | `[section]_icon` |
| ラッパー | `[section]_wrapper` |

### 3単語（階層が深い場合）
| 役割 | クラス名 |
|------|----------|
| 項目内の見出し | `[section]_item_title` |
| 項目内の説明 | `[section]_item_description` |
| ヘッダー内のボタン | `[section]_header_button` |
| カード内の画像 | `[section]_card_img` |

---

## 🎨 状態管理例

```html
<!-- 通常 -->
<div class="accordion_item">...</div>

<!-- 開いてる状態 -->
<div class="accordion_item accordion_item_active">...</div>

<!-- 無効状態 -->
<div class="accordion_item accordion_item_disabled">...</div>
```

```css
/* 閉じてる */
.accordion_description {
  display: none;
}

/* 開いてる */
.accordion_item_active .accordion_description {
  display: block;
}
```

---

## 🔧 実装例

### セクションコメントの読み取り
```html
<!-- ===============================================
     アコーディオンメニュー
     =============================================== -->
```
→ セクション名 = `accordion`

### HTMLタグの入力
ユーザー入力:
```html
<div>
  <div>
    <h2>タイトル</h2>
    <button>+</button>
  </div>
  <p>説明文</p>
</div>
```

### AIの出力
```html
<div class="accordion_area">
  <div class="accordion_item">
    <div class="accordion_header">
      <h2 class="accordion_title">タイトル</h2>
      <button class="accordion_button">+</button>
    </div>
    <p class="accordion_description">説明文</p>
  </div>
</div>
```

---

## 🔄 上書きルール

- ✅ **既存クラス名は上書きOK**: サジェスト等で仮クラスが付いていたら正しい命名で上書き
- 例: `<section class="acc">` → `<section class="flex_area">`

---

## 🖼️ 画像ルール

- ✅ 画像タグは `<img>` を使用
- クラス名: `[section]_img`

---

## ✏️ 明らかな間違いの自動修正

- ✅ HTMLコメント: `/* */` → `<!-- -->`
- ✅ セクションコメント形式の統一
- ✅ 閉じタグ漏れの補完
- ⚠️ 構造的な問題 → ユーザー確認

---

## 注意事項

1. **セクションコメント必須**: クラス名の先頭は必ずコメントから判断
2. **全タグにクラス付与**: 漏れなく全てのタグに付ける
3. **2単語を優先**: 基本は2単語、階層が深い場合のみ3単語
4. **確認は簡潔に**: --ucモード維持（記号多用）
5. **勝手に構造変更しない**: タグ構造はユーザーの記述通り
6. **既存クラス上書きOK**: 仮クラス名は正しい命名で上書き
7. **明らかな文法ミスは自動修正**: コメント形式等
8. **ユーザーのコメントは残す**: `<!-- -->` コメントは削除せず、そのまま保持する
9. **構造の問題は指摘する**: レイアウト意図と構造が噛み合っていない箇所はユーザーに指摘する（修正はしない）
10. **共通化の提案**: 既存ソースを確認し、同じ構造のクラス名があれば共通化を提案する
11. **迷ったら聞く**: クラス名の判断に迷った場合はユーザーに確認する
12. **セマンティックチェック必須**: クラス付与と同時に、タグの意味的な適切さを確認する。問題があれば「〇〇は `<タグ名>` が適切です」と1行で指摘（修正はしない）
