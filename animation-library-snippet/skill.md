# animation-library-snippet

**Description**: アニメーションライブラリのデモスニペットを生成（index.html管理+4デモHTML）

**Arguments**: `<library_name> <animation_type>`

**Examples**:
- `/animation-library-snippet swiper フェード切替` → フェード切替スライダー
- `/animation-library-snippet gsap スクロール連動` → スクロール連動アニメーション
- `/animation-library-snippet anime パララックス` → パララックスエフェクト

---

## 🎯 このスキルの目的

アニメーションライブラリ（Swiper, GSAP, anime.js, AOS, Lottie等）の学習用デモを統一フォーマットで生成。

### 生成物
```
images/
  ├── index.html (更新)
  ├── <library>_demo1_<type>_simple.html
  ├── <library>_demo2_<pattern1>.html
  ├── <library>_demo3_<pattern2>.html
  └── <library>_demo4_<pattern3>.html

その他/00_サンプルソース/
  └── ★<library>_<animation_type>.md
```

---

## 📋 必須ルール（絶対厳守）

### A. ファイル名規則
❌ `swiper_basic_to_advanced.md`
✅ `swiper_フェード切替.md` （ライブラリ英語＋タイプ日本語）

#### ファイル名例
| ライブラリ | タイプ | ファイル名 |
|---|---|---|
| swiper | フェード切替 | `swiper_フェード切替.md` |
| swiper | サムネイル連動 | `swiper_サムネイル連動.md` |
| gsap | スクロール連動 | `gsap_スクロール連動.md` |
| anime | パララックス | `anime_パララックス.md` |
| aos | フェードイン表示 | `aos_フェードイン表示.md` |

### B. デモ構成（常に4つ）
1. **シンプル版（必須）** - ライブラリの基本機能のみ
2. **パターン1** - よく使う実装例
3. **パターン2** - よく使う実装例
4. **パターン3** - 多機能応用例

### C. index.html 管理（最重要）

**`C:\Users\guest04\Desktop\高橋研三\03_knowledge\images\index.html` がマスター**

#### 絶対厳守ルール
- ✅ **全ライブラリのデモがここから見える**（Swiper, AOS, GSAP等すべて）
- ✅ タイトル: 「🎨 アニメーションライブラリ デモ一覧」（特定ライブラリ名禁止）
- ✅ 新規ライブラリ追加時: index.htmlに新規カード追加
- ✅ 各ライブラリのシンプル版は緑枠強調
- ✅ ファイルは分けるが、index.htmlから全て見える

#### index.html構造
```
🎨 アニメーションライブラリ デモ一覧
├── Swiper.js セクション（①シンプル版 ②③④パターン）
├── AOS.js セクション（①シンプル版 ②③④パターン）
├── GSAP セクション（①シンプル版 ②③④パターン）
└── ... 他のライブラリ
```

#### 初回ライブラリ追加時
- タイトルが「Swiper.js デモ一覧」等になっている場合 → 「アニメーションライブラリ デモ一覧」に変更

#### 2回目以降
- カードを追加するのみ（タイトル変更不要）

### D. クライアント対応
❌ デモHTML内に説明パネル配置
✅ 説明は index.html のカードのみ

### E. パス指定
- MD内リンク: `../../images/` （相対パス）
- HTML: 同階層ファイル名のみ

---

## 🛠 実装手順

### 1️⃣ ユーザーから情報収集
```
📋 必要情報
- ライブラリ名: swiper, gsap, anime, aos, lottie 等（英語）
- アニメーションタイプ: フェード切替, スクロール連動 等（日本語）
- CDN URL: 最新版を確認
- 基本機能: 何をデモするか（3つ程度）
```

### 2️⃣ ファイル生成

#### ① MDファイル作成

**重要**: HTML/CSS/JavaScriptを**分けて記載**（実務でコピペしやすい）

```markdown
# [Library] - [Animation Type]

## 📌 概要
[ライブラリの説明]

## 🎯 学習目標
- 基本機能A
- 基本機能B
- 基本機能C

## 📂 デモ構成

### ① シンプル版【まずはここから】

💡 **用途**: [用途の説明]

#### HTML
```html
<div class="container">
  <div class="box" data-aos="fade">
    <!-- HTML部分のみ -->
  </div>
</div>
```

#### CSS
```css
/* CSS部分のみ */
.container {
  max-width: 800px;
  margin: 0 auto;
}
.box {
  padding: 40px;
}
```

#### JavaScript
```javascript
// JavaScript部分のみ
AOS.init({
  duration: 800,
  once: false
});
```

### ② [パターン1名]
[同様にHTML/CSS/JavaScript分けて記載]

### ③ [パターン2名]
[同様にHTML/CSS/JavaScript分けて記載]

### ④ [パターン3名・多機能版]
[同様にHTML/CSS/JavaScript分けて記載]

## 🔗 デモファイル
- [シンプル版](../../images/<library>_demo1_<type>_simple.html)
- [パターン1](../../images/<library>_demo2_<pattern1>.html)
- [パターン2](../../images/<library>_demo3_<pattern2>.html)
- [パターン3](../../images/<library>_demo4_<pattern3>.html)

## 📖 公式リソース
- [公式サイト](URL)
- [API Documentation](URL)
- [GitHub](URL)
```

**なぜ分けるか**:
- ✅ 実務ではファイルを分けてコーディング
- ✅ 各パートを個別にコピペ可能
- ✅ 理解しやすい

#### ② HTML生成（4ファイル）

**注意**: デモHTMLファイルは**1ファイルにまとめる**（インライン）
- MDファイルとは異なり、すぐブラウザで確認できる形式

**共通構造**:
```html
<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8">
  <title>[Library] - [Demo Name]</title>
  <link rel="stylesheet" href="[CDN]">
  <style>
    /* CSS全部ここに記載（インライン） */
    /* ❌ 説明パネルCSS禁止 */
    /* ✅ デモ本体のみ */
  </style>
</head>
<body>
  <!-- 戻るボタン（固定） -->
  <div class="back-link">
    <a href="index.html">← デモ一覧に戻る</a>
  </div>

  <!-- ❌ 説明パネル禁止 -->
  <!-- ✅ デモ本体のみ -->

  <script src="[CDN]"></script>
  <script>
    // JavaScript全部ここに記載（インライン）
  </script>
</body>
</html>
```

**デモ1（シンプル版）の特徴**:
- ライブラリの機能のみ
- カスタムCSS最小限
- コメントで機能説明

**デモ2-4の特徴**:
- 実務で使うパターン
- 必要なカスタムあり
- デモ4は多機能版

#### ③ index.html 更新

**重要**: `images/index.html` は全ライブラリの統合マスター

##### 構造
```
🎨 アニメーションライブラリ デモ一覧
├── 目次（各セクションへのリンク）
├── Swiper.js セクション（見出し + 4カード）
├── AOS.js セクション（見出し + 4カード）
└── GSAP セクション（見出し + 4カード）
```

##### 初回実行時（Swiper等が既にある場合）
1. タイトルを「🎨 アニメーションライブラリ デモ一覧」に変更
2. **目次セクション追加**（container内、subtitle直下）
3. **既存カードをセクション分け**（`<h2 id="swiper">Swiper.js</h2>` + demo-grid）
4. **新規ライブラリセクション追加**（`<h2 id="aos">AOS.js</h2>` + demo-grid + 4カード）

##### 目次セクション追加
```html
<!-- 目次 -->
<div class="toc">
  <h2 class="toc-title">📚 ライブラリ一覧</h2>
  <div class="toc-links">
    <a href="#swiper" class="toc-link">Swiper.js</a>
    <a href="#aos" class="toc-link">AOS.js</a>
    <a href="#gsap" class="toc-link">GSAP</a>
  </div>
</div>
```

**CSS追加**:
```css
.toc {
  background: rgba(255,255,255,0.15);
  padding: 30px;
  border-radius: 15px;
  margin-bottom: 50px;
  text-align: center;
}
.toc-title {
  color: white;
  font-size: 24px;
  margin-bottom: 20px;
}
.toc-links {
  display: flex;
  gap: 20px;
  justify-content: center;
  flex-wrap: wrap;
}
.toc-link {
  background: white;
  color: #667eea;
  padding: 12px 24px;
  border-radius: 8px;
  text-decoration: none;
  font-weight: bold;
  transition: transform 0.3s;
}
.toc-link:hover {
  transform: translateY(-3px);
}
.section-title {
  color: white;
  font-size: 36px;
  text-align: center;
  margin: 60px 0 30px;
  text-shadow: 2px 2px 4px rgba(0,0,0,0.3);
  border-bottom: 3px solid rgba(255,255,255,0.3);
  padding-bottom: 15px;
}
```

##### 新規ライブラリセクション追加（必ず4つ）

**重要**: 1ライブラリにつき**セクション見出し + 4つのカード**を追加

```html
<!-- ライブラリセクション -->
<h2 id="<library>" class="section-title">[Library Name]</h2>

<div class="demo-grid">
  <!-- 4つのカード -->
</div>
```

```html
<!-- [Library Name] セクション -->
<h2 id="<library>" class="section-title">[Library Name]</h2>

<div class="demo-grid">
  <!-- ① シンプル版（緑枠） -->
  <a href="<library>_demo1_<type>_simple.html" class="demo-card" style="border: 3px solid #4AE290;">
  <div class="demo-icon">✅</div>
  <h2 class="demo-title">① シンプル版【まずはここから】</h2>
  <p class="demo-description">
    [ライブラリ名]の機能のみを使った最小限のコード。
    <strong>これが[ライブラリ名]本来の姿</strong>です
  </p>
  <div class="demo-usage" style="background: #E8F5E9;">
    <div class="demo-usage-title" style="color: #4AE290;">💡 おすすめポイント</div>
    <div class="demo-usage-text">
      ✓ [ライブラリ名]の機能のみ<br>
      ✓ 最小限のコード<br>
      ✓ カスタマイズしやすい<br>
      ✓ 学習に最適
    </div>
  </div>
  <div class="demo-button" style="background: linear-gradient(135deg, #4AE290 0%, #4BC0C8 100%);">シンプル版を見る →</div>
</a>

<!-- ② パターン1 -->
<a href="<library>_demo2_<pattern1>.html" class="demo-card">
  <div class="demo-icon">🥇</div>
  <h2 class="demo-title">② [パターン1名]</h2>
  <p class="demo-description">
    [パターン1の説明]
  </p>
  <div class="demo-usage">
    <div class="demo-usage-title">📋 主な用途</div>
    <div class="demo-usage-text">
      ✓ 用途1<br>
      ✓ 用途2<br>
      ✓ 用途3
    </div>
  </div>
  <div class="demo-button">デモを見る →</div>
</a>

<!-- ③ パターン2 -->
<a href="<library>_demo3_<pattern2>.html" class="demo-card">
  <div class="demo-icon">🥈</div>
  <h2 class="demo-title">③ [パターン2名]</h2>
  <p class="demo-description">
    [パターン2の説明]
  </p>
  <div class="demo-usage">
    <div class="demo-usage-title">📋 主な用途</div>
    <div class="demo-usage-text">
      ✓ 用途1<br>
      ✓ 用途2<br>
      ✓ 用途3
    </div>
  </div>
  <div class="demo-button">デモを見る →</div>
</a>

<!-- ④ 多機能版 -->
<a href="<library>_demo4_<pattern3>.html" class="demo-card">
  <div class="demo-icon">🥉</div>
  <h2 class="demo-title">④ [パターン3名]【多機能版】</h2>
  <p class="demo-description">
    [パターン3の説明 - 多機能応用例]
  </p>
  <div class="demo-usage">
    <div class="demo-usage-title">⚠ 含まれる機能</div>
    <div class="demo-usage-text">
      ✅ [ライブラリ名]機能（機能A、機能B等）<br>
      ❌ カスタムCSS（追加機能等）<br>
      💡 多機能で見た目は派手だが複雑
    </div>
  </div>
  <div class="demo-button">多機能版を見る →</div>
  </a>
</div>
<!-- セクション終了 -->
```

**注意**:
- ✅ **4つのカードを個別に追加**（1つにまとめない）
- ✅ 既存のカード（Swiper等）は削除しない
- ✅ demo-grid内に追加することで、グリッド表示で並ぶ
- ✅ シンプル版（demo1）のみ緑枠（`style="border: 3px solid #4AE290;"`）
- ✅ demo2, 3, 4は通常の枠（borderスタイル指定なし）

---

## 💡 実装例

### 例1: Swiper フェード切替スライダー
```bash
/animation-library-snippet swiper フェード切替
```

**生成ファイル**:
- `★swiper_フェード切替.md`
- `swiper_demo1_fade_simple.html` （シンプル版）
- `swiper_demo2_thumbnail.html` （サムネイル連動）
- `swiper_demo3_autoplay.html` （自動再生）
- `swiper_demo4_fade_advanced.html` （多機能版）

### 例2: GSAP スクロールアニメーション
```bash
/animation-library-snippet gsap スクロール連動
```

**生成ファイル**:
- `★gsap_スクロール連動.md`
- `gsap_demo1_scroll_simple.html` （基本スクロール）
- `gsap_demo2_parallax.html` （パララックス）
- `gsap_demo3_pin.html` （ピン固定）
- `gsap_demo4_timeline.html` （タイムライン制御）

### 例3: anime.js パララックスエフェクト
```bash
/animation-library-snippet anime パララックス
```

**生成ファイル**:
- `★anime_パララックス.md`
- `anime_demo1_parallax_simple.html` （基本パララックス）
- `anime_demo2_stagger.html` （段階表示）
- `anime_demo3_svg.html` （SVGアニメ）
- `anime_demo4_timeline.html` （複合アニメ）

---

## ⚠ 注意事項

### やってはいけないこと
❌ デモHTML内に `.info-panel` 等の説明要素
❌ 絶対パス使用
❌ 汎用的な名前（basic_to_advanced等）
❌ 5ファイル以上のデモ作成

### 必ずやること
✅ アニメーションタイプを表す分かりやすい日本語ファイル名
✅ シンプル版を必ず最初に配置
✅ 説明は index.html のみ
✅ 相対パス使用
✅ 戻るボタン配置（全デモ）

---

## 📝 チェックリスト

生成完了後、以下を確認：

```
□ index.htmlのタイトルが「🎨 アニメーションライブラリ デモ一覧」
□ index.htmlに新規カード追加済み（既存カード削除してない）
□ index.htmlから全デモが見える
□ ファイル名がアニメーションタイプを表している
□ MDファイルが相対パスを使用
□ デモHTMLに説明パネルなし
□ シンプル版が最初（緑枠）
□ 4デモすべてに戻るボタンあり
□ CDNが最新版
□ 動作確認済み
```

---

## 🎓 使用上の注意

1. **index.html確認（最重要）**:
   - パス: `C:\Users\guest04\Desktop\高橋研三\03_knowledge\images\index.html`
   - タイトルが「Swiper.js デモ一覧」等になっている場合 → 「🎨 アニメーションライブラリ デモ一覧」に変更
   - 既存のカード（Swiper等）は削除せず、新規カードを追加

2. **CDN確認**: 必ず最新版のCDN URLを公式サイトで確認

3. **ファイル名**: ユーザーと相談してアニメーションタイプを決定

4. **デモ内容**: ライブラリの特徴が分かる3パターンを選定

5. **全ライブラリ統合管理**:
   - Swiper, AOS, GSAP等すべて同じindex.htmlで管理
   - ファイルは分けるが、index.htmlから全て見える

---

**実行パス**: `C:\Users\guest04\Desktop\高橋研三\03_knowledge\`

**Note**: ライブラリごとに特徴的なデモを作成し、学習効率を最大化すること
