---
name: auto-class
description: HTMLタグにクラス名を自動付与
triggers:
  - auto-class
  - クラス名
  - クラス付与
---

# auto-class スキル

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

### 1. ユーザーがHTMLタグのみ記述
```html
<div>
  <h2>質問タイトル</h2>
  <p>回答内容</p>
</div>
```

### 2. AIが全タグにクラス名付与
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
