# auto-clean スキル

> ## ⚡ 絶対最初にやること（これを忘れたら全部やり直し）
> **スキル読み込み直後、作業の1行目は必ず `TodoWrite` でStep 0のチェックリストを出す**
> ❌ TodoWrite なしで Grep や Read を始める → 禁止
> ✅ TodoWrite → その後に検索・作業 → 正しい順序

提出前のHTML/CSSクリーンアップを実行するスキル

## 使い方
```
/auto-clean
```

---

## 🎯 目的

提出前にコードを検査し、開発用の痕跡やミスを検出・修正する。
**修正は実行せず、検出結果を一覧で報告する。ユーザーが確認後に修正を指示する。**

---

## 🚀 Step 0. TodoWriteでチェックリスト表示（絶対厳守・スキル開始直後に必ず実行）

```
TodoWrite([
  { content: "└コメント検出（CSS・HTML）",                       status: "in_progress", activeForm: "└コメント検出中" },
  { content: "デバッグ用コード検出",                              status: "pending",     activeForm: "デバッグコード検出中" },
  { content: "AI生成文の検出",                                    status: "pending",     activeForm: "AI生成文検出中" },
  { content: "共通化候補の検出",                                  status: "pending",     activeForm: "共通化候補検出中" },
  { content: "命名ミスの検出",                                    status: "pending",     activeForm: "命名ミス検出中" },
  { content: "参考フォルダの検出",                                status: "pending",     activeForm: "参考フォルダ検出中" },
  { content: "CSS↔HTMLコメント整合性チェック",                   status: "pending",     activeForm: "コメント整合性チェック中" },
  { content: "その他（TODO/空ルール/!important等）検出",          status: "pending",     activeForm: "その他検出中" },
  { content: "不必要なCSSプロパティの検出",                       status: "pending",     activeForm: "不必要なCSSプロパティ検出中" },
  { content: "CSSプロパティ並び順チェック",                        status: "pending",     activeForm: "プロパティ並び順チェック中" },
  { content: "br前後の改行スペース検出",                          status: "pending",     activeForm: "br改行スペース検出中" },
  { content: "コメント誤字・表記ミス検出",                        status: "pending",     activeForm: "コメント誤字検出中" },
  { content: "結果一覧をユーザーに報告",                          status: "pending",     activeForm: "結果報告中" }
])
```

各ステップ完了ごとに `completed` に更新すること。

---

## 📋 検出項目

### 1. `└` コメントの削除
auto-skeleton で作成段階用に入れた `└` コメントを検出。

**CSS**:
```css
/* └ サムネイル（左） */   ← 検出対象
/* └ └ 詳細文 */          ← 検出対象
/* お知らせカラム */       ← 残す
```

**HTML**:
```html
<!-- └ テキスト側（右） -->      ← 検出対象
<!-- └ └ カテゴリ+日付 -->       ← 検出対象
<!-- お知らせエリア -->           ← 残す
```

**判定**: コメント内に `└` を含むものすべて

---

### 2. デバッグ用コードの検出
コメントやプロパティに以下の文字列が含まれていたら報告。

**検出キーワード**:
- コメント内: `debug` `test` `tmp` `temp` `仮` `確認用` `あとで`
- CSSプロパティ: `border: .*solid red` `border: .*solid blue` `background: red` `background: blue` `outline: .*red`
- HTML: `console.log` (script内)

**報告形式**:
```
⚠ デバッグ痕跡:
- work.css:650 → border: 0.5rem solid red
- work.css:656 → border: 0.5rem solid red
- index.html:45 → console.log(...)
```

---

### 3. AI生成文の検出
AIが生成しがちな不自然な表現をコメントやテキストから検出。

**検出パターン**:
- 「以下の通り」「下記の通り」
- 「適切に」「正しく」「適宜」
- 「～を実現します」「～を提供します」
- 過剰な丁寧表現

**報告のみ**。修正はユーザーが判断する。

---

### 4. 共通化の提案
同じスタイルが複数セクションで重複している場合に報告。

**例**:
```
💡 共通化候補:
- .news_item と .blog_item → 同じ display:flex 構造
  → 共通クラス化を検討？
```

**報告のみ**。共通化するかはユーザーが判断する。

---

### 5. 命名ミスの検出
auto-class の命名規則に違反しているものを検出。

**検出対象**:
- ハイフン `-` 使用 → `_` アンダースコアであるべき
- キャメルケース `newsItem` → `news_item` であるべき
- 省略形 `_btn` `_desc` `_nav` → `_button` `_description` （`img` `nav` は例外OK）
- 4単語以上のクラス名
- スペルミス（明らかなもの）

**報告形式**:
```
❌ 命名ミス:
- news-item → news_item（ハイフン→アンダースコア）
- newsBtn → news_button（キャメルケース+省略）
- accoridon_item → accordion_item（スペルミス）
```

---

### 6. 参考フォルダの削除
提出時に不要な参考用フォルダ（`img/mockup/` 等）の存在を検出し、削除を提案する。
- **報告のみ**。削除はユーザー確認後に実行する。

---

### 7. コメントの整合性チェック
CSS↔HTMLのコメントが一致しているか確認する。
- flex/grid親の `【】` コメントがCSS・HTML両方にあるか
- `└` の親子関係がCSS・HTMLで一致しているか
- コメントの文言がCSS・HTMLで同じか（片方だけ修正してズレていないか）

---

### 8. その他の検出
- `TODO` `FIXME` `HACK` コメント → 報告
- 空ルールセット `{ }` → スケルトン消し忘れの可能性を報告
- コメントアウトされたCSS/HTML → 削除忘れを報告
- `!important` → 本当に必要か確認を促す

---

### 9. 不必要なCSSプロパティの検出
効果がない・重複している・上書きされているプロパティを検出し報告する。

**検出対象**:
- **同一ルールセット内の重複プロパティ**: 同じプロパティが2回書かれている
- **上書きされて無効なプロパティ**: 後のルールで完全に上書きされている
- **効果のないプロパティ**: 例）`display: block` な要素に `vertical-align`、`inline` 要素に `width/height`
- **flexbox不要プロパティ**: `float` が `display: flex` と共存している等
- **0値の単位**: `0px` `0rem` → `0` で十分

**報告形式**:
```
⚠ 不必要なCSSプロパティ:
- work.css:123 → .news_item に float: left（display:flexと併用で無効）
- work.css:456 → .case_img に vertical-align: middle（block要素に無効）
- work.css:789 → margin: 0px → margin: 0 で十分
```

**報告のみ**。削除はユーザーが確認後に指示する。

---

### 10. CSSプロパティ並び順チェック
各ルールセット内のプロパティが以下の順番になっているか確認する。

**正しい並び順**:
```
① サイズ・位置     width / height / position / top / left / right / bottom / z-index
② 余白             padding / margin
③ 見た目           background / color / border / border-radius / opacity / box-shadow
④ 文字             font-size / font-weight / line-height / text-align / text-decoration
⑤ レイアウト       display / flex-direction / justify-content / align-items / gap / grid-template-columns
```

**検出対象**: 上の順番に対して明らかに逆転しているもの

**例（NG）**:
```css
.news_item {
  display: flex;        ← ⑤ レイアウト（先に書かれている）
  width: 100%;          ← ① サイズ（後に書かれている → 逆転）
  gap: 1rem;
}
```

**例（OK）**:
```css
.news_item {
  width: 100%;          ← ① サイズ
  padding: 2rem;        ← ② 余白
  background: white;    ← ③ 見た目
  font-size: 1.4rem;    ← ④ 文字
  display: flex;        ← ⑤ レイアウト
  gap: 1rem;
}
```

**報告形式**:
```
⚠ プロパティ並び順:
- work.css:965 → .footer_sns_link: display:flex（⑤）が width（①）より上に記述
```

**報告のみ**。並び替えはユーザーが確認後に指示する。

---

### 11. `<br>`前後の改行スペース検出
HTMLソースで `<br>` や `<br />` の前後に改行（ソースコードの改行）があると、`<br>`を削除した際に半角スペースが残る問題を検出する。

**原因**: HTMLではソースコードの改行は半角スペース1つに変換される仕様。`<br>`があるときは改行で見えないが、取ると「余計なスペース」が出る。

**検出対象**:
```html
<!-- NG: brの前後でソースが改行されている -->
<p>WEBからの<br />
お問い合わせ</p>

<p>WEBからの
<br />お問い合わせ</p>

<p>WEBからの
<br />
お問い合わせ</p>
```

```html
<!-- OK: 1行で書かれている -->
<p>WEBからの<br />お問い合わせ</p>
```

**判定**: `<br>` `<br />` `<br/>` の直前または直後にHTMLソースの改行があるもの

**報告形式**:
```
⚠ br前後の改行スペース:
- index.html:410 → <br />の後にソース改行あり（brを取るとスペースが残る）
- index.html:523 → <br>の前にソース改行あり
```

**報告のみ**。修正はユーザーが確認後に指示する。

---

### 12. コメントの誤字・表記ミス検出
HTML/CSS/JSのコメント内にある誤字・ひらがな表記・不自然な文章を検出する。

**検出対象**:
- **ひらがなで書かれたカタカナ語**: `いんでっくす` → `インデックス`、`ふっだー` → `フッター` 等
- **明らかな誤字**: `ヘッタ` → `ヘッダー`、`フッダー` → `フッター` 等
- **意味不明なコメント**: 開発中のメモが残っている等
- **英語スペルミス**: `/* hedder */` → `/* header */` 等

**報告形式**:
```
⚠ コメント誤字・表記ミス:
- script.js:26 → 「いんでっくす」→「インデックス」（ひらがな→カタカナ）
- work.css:15 → 「hedder」→「header」（スペルミス）
```

**報告のみ**。修正はユーザーが確認後に指示する。

---

## 🎯 出力フォーマット

全項目をチェックリストで表示し、漏れなく網羅すること。

```
## auto-clean 結果

### 1. `└` コメント削除
- [ ] CSS: ○件 / HTML: ○件

### 2. デバッグ用コード
- [ ] デバッグCSS（border:red等）: ○件
- [ ] console.log: ○件
- [ ] デバッグコメント（debug/test/仮 等）: ○件

### 3. AI生成文
- [ ] AI生成表現: ○件

### 4. 共通化の提案
- [ ] 共通化候補: ○件

### 5. 命名ミス
- [ ] ハイフン使用: ○件
- [ ] キャメルケース: ○件
- [ ] 省略形: ○件
- [ ] スペルミス: ○件

### 6. 参考フォルダ
- [ ] 不要フォルダ: ○件

### 7. コメント整合性
- [ ] CSS↔HTMLコメントのズレ: ○件

### 8. その他
- [ ] TODO/FIXME/HACK: ○件
- [ ] 空ルールセット: ○件
- [ ] コメントアウトされたコード: ○件
- [ ] !important: ○件

### 9. 不必要なCSSプロパティ
- [ ] 重複プロパティ: ○件
- [ ] 上書きされて無効なプロパティ: ○件
- [ ] 効果のないプロパティ（inline要素のwidth等）: ○件
- [ ] 0値の単位（0px→0）: ○件

### 10. CSSプロパティ並び順
- [ ] 順番が逆転しているルールセット: ○件

### 11. br前後の改行スペース
- [ ] brの前後にソース改行があるタグ: ○件

### 12. コメント誤字・表記ミス
- [ ] ひらがなカタカナ語: ○件
- [ ] スペルミス: ○件
- [ ] 意味不明コメント: ○件
```

---

## ⚠ 注意事項

1. **報告ファースト**: まず一覧を出す。勝手に修正しない
2. **ユーザー確認後に修正**: 「修正して」と言われたら実行
3. **判断が必要なものは提案のみ**: 共通化・AI文など
4. **`└` 削除時は空行も整理**: コメント削除後に空行が連続しないようにする
5. **Grepファースト（トークン節約）**: ファイルを丸ごと `Read` する前に、必ず `Grep` でパターン検索してから該当箇所のみ読む
   - ✅ `└` → Grep で `└` を検索
   - ✅ デバッグ → Grep で `border.*red` `console.log` を検索
   - ✅ `!important` → Grep で `!important` を検索
   - ✅ `<br>` 改行 → Grep で `<br` を検索し、前後行を確認
   - ✅ コメント誤字 → Grep でコメント（`/\*` `<!--` `//`）を検索し内容確認
   - ❌ 全文 Read してから目視で探す → 禁止
