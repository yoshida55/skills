# 🚨 実行チェックリスト（TodoWrite で必ず出す・絶対厳守）

**/memo-all 実行時、作業開始前に以下を TodoWrite で出すこと。**

```
1. PC判定（自宅 or 会社）※1会話1回だけ
2. mcp__auto-memo__memo_read_context（index_path指定）← 1回で log + config + index全件を取得
3. 重複チェック（インデックス全件を Claude が判断）← 必ず実施
4. mcp__auto-memo__memo_write_all（highlight_line含む）← 1回で全完結
```

## ⚡ 最速フロー（3ステップ / 2回目以降も同じ）

```
Step1: TodoWrite
Step2: Bash（PC判定）← 1会話1回のみ
Step3: mcp__auto-memo__memo_read_context（log_path + config_path + index_path を渡す）
        → log_tail + config（last_memo_line）+ index（全件）を1回で取得
Step4: 重複チェック（← 必ず実施・絶対に飛ばさない）
Step4.5: mis_log の今日のセクションとインデックスを照合して漏れチェック
         → mis_log にあるのに index にないトピックがあれば 01_memo.md にも追加する
         （ツール呼び出し不要・Claude が頭の中で照合するだけ）
Step5: mcp__auto-memo__memo_write_all（highlight_line を渡してhighlight内蔵）
```

## ⚡⚡ 同一会話内の2回目以降はさらに速くなる

- **Step2（PC判定）はスキップ**：1会話に1回だけでOK
- **Step3（memo_read_context）もスキップ可**：同一会話内ならインデックスは記憶済み
- 重複チェックは記憶したインデックスをそのまま使う
- **memo_write_all だけ実行**すれば完結

```
通常（1回目）: TodoWrite → PC判定 → memo_read_context → 重複チェック → memo_write_all
2回目以降:     TodoWrite → 重複チェック（記憶済み）→ memo_write_all だけ ← 最速
```

> ⚠ 会話をまたぐとインデックスの記憶はリセットされる → 次の会話では必ず Step3 から実行すること

- **Read ツール不要**（indexも memo_read_context が取得）
- **curl 不要**（highlight は memo_write_all に内蔵）
- **書き込みは MCP が一括処理**（Edit × 3 + Write × 1 が mcp 呼び出し1回に集約）
- ❌ **Edit ツールで直接書き込み禁止** → 必ず `memo_write_all`（MCP）を使うこと

## 📌 last_config.json の場所と使い方

- 自宅PC: `C:\Users\sensh\.claude\skills\deliver\last_config.json`
- 会社PC: `C:\Users\guest04\.claude\skills\deliver\last_config.json`

```json
{
  "last_submit_dir": "...",
  "last_memo_line": 19541
}
```

- Step3 で `last_memo_line` を取得した後、**必ず `wc -l` で実際の行数を確認する**
- config の値と実際の行数が異なる場合は **実際の行数を優先する**（上書き防止）
- 追記完了後、実際に追記した最終行番号で `last_memo_line` を更新して Write

```bash
# 実際の行数確認（会社PC例）
wc -l "C:/Users/guest04/Desktop/高橋研三/03_knowledge/01_memo.md"
```

---

# インデックスファイル（高速重複チェック）

## 📋 01_memo_index.txt とは

`01_memo.md` に書いた全エントリのキーワードを1行ずつまとめた小さなファイル。

- 自宅PC: `D:\50_knowledge\01_memo_index.txt`
- 会社PC: `C:\Users\guest04\Desktop\高橋研三\03_knowledge\01_memo_index.txt`

## 重複チェックの手順（絶対厳守）

memo_read_context で取得した `index` の全行を必ず確認する。

### ✅ 重複と判断する条件（どれか1つでも当てはまれば既存エントリを更新）
- 同じ関数名・プロパティ名・クラス名が含まれている
- 同じ概念・同じ操作（例：「アーカイブ」「ループ」「カスタム投稿」など）が含まれている
- 同じジャンル・シリーズ（例：「ギャグ」「テスト」「天気」など）の続きにあたる
- タイトルが違っても**内容が同じ知識**を説明している

### ❌ 「キーワードが1語も被らなければ新規」は禁止
- 「ギャグその２」は「ギャグ」と同じシリーズ → 既存エントリに追記すべき
- 「archive-works.php」と「archive.php」は別ファイルでも概念が近い → 既存エントリを更新

### 判断フロー
```
インデックス全件を見る
  ↓
似た行がある → 既存エントリを更新（Grep → memo.md の該当箇所を Edit）
似た行がない → 新規追記（memo_write_all）
```

## インデックスへの追記フォーマット（1行）

```
キーワード1 キーワード2 キーワード3 | タグ（HTML / WordPress / CSS など）
```

例：
```
margin-top: auto 直接のflex子 入れ子 親もflexにする | HTML CSS
wp_footer() JS body直前 wp_head() | WordPress
```

---

# 出力先（3ファイルに書き込み）

## 📘 詳細メモ（説明・仕組み）
存在する方に追記:
- 会社PC: `C:\Users\guest04\Desktop\高橋研三\03_knowledge\01_memo.md`
- 自宅PC: `D:\50_knowledge\01_memo.md`

## 📒 気づき・ミス（1行メモ）
存在する方に追記:
- 会社PC: `C:\Users\guest04\Desktop\高橋研三\03_knowledge\00_mis_log.md`
- 自宅PC: `D:\50_knowledge\00_mis_log.md`

## 📋 インデックス（重複チェック用）
- 会社PC: `C:\Users\guest04\Desktop\高橋研三\03_knowledge\01_memo_index.txt`
- 自宅PC: `D:\50_knowledge\01_memo_index.txt`

判定方法: Bash で `test -f` を実行し、存在する方を使用

> ⚡ **チェックは1会話に1回だけ**

---

# 書き込みルール

## 📘 01_memo.md（詳細メモ）
→ **memo-format スキルと同じフォーマット**で書く

- タイトルは「読んだだけで答えがわかる」レベル
- 【日付】【結論】【具体例】【補足】の構成
- カテゴリタグ（`WordPress` / `HTML`）を末尾に付ける
- 重複チェックは **01_memo_index.txt を Read するだけ**（01_memo.md 本体の Grep 不要）

### ⚠ 具体例コードのルール（絶対厳守）
- **最小コードでも「実際に動くもの」を書く**
- `add_action` のラップが必要なら省かない
- `wp_enqueue_style` の第1引数など省略すると動かなくなる部分は必ず入れる
- 「わかりやすさのために省略」は禁止 → 動かないコードはメモとして有害

### ✅ 正しいタイトル・日付フォーマット（厳守）

```markdown
## 📌 タイトル（内容がわかるレベル） タグ（WordPress / HTML など）

【日付】YYYY-MM-DD
```

❌ 禁止：`## 【2026-04-13】タイトル`（日付をタイトルに入れない）

### 📝 タイトルの書き方ルール

**「このタイトルだけ読んで、答えがわかる」レベルまで書く。**
- 2〜3行になってもOK
- 「何が・どうなる・どうすれば解決するか」まで含める

**✅ 良い例：**
```
get_categories() はデフォルトで空カテゴリーを非表示にする →
array('hide_empty' => false) を渡すと全カテゴリーを表示できる WordPress
```
```
WP_Query でカテゴリ絞り込みはループの前（クエリ）で行う →
ループ内でif絞りするとページネーションがズレる・wp_reset_postdata()必須 WordPress
```
```
margin-top: auto は直接の flex 子要素でないと効かない →
入れ子になっている場合は親も flex にするか、要素を外に出す HTML CSS
```
```
カスタムタクソノミーとは → カスタム投稿タイプ専用の分類機能。
イメージとしては、➀デフォルトの「カテゴリー・タグ」は通常投稿専用。
➁カスタム投稿タイプには独自の分類（タクソノミー）を別途作る必要がある。
➂CPT UI の「タクソノミーを追加」から作成できる。 WordPress
```

**❌ 悪い例（短すぎて意味不明）：**
```
get_categories について
WP_Query の使い方
margin-top: auto
```

### 🎯 タイトルの粒度ルール（絶対厳守）

**「これだけ読めば思い出せる」レベルまで書く。素人でも読んで理解できる粒度。**

- 流れが大事なものは ➀➁➂ で手順を書く（その時と場合による・強制ではない）
- 「何と何の違い」が大事なものは比較で書く
- 2〜4行になってもOK・短くまとめようとしない
- 専門用語だけで終わらせない（「何がどうなるか」まで書く）

## 📒 00_mis_log.md（気づき・ミス）
→ **memo_days スキルと同じフォーマット**で書く

- **プロジェクト固有の値を使わない**（絶対厳守）
  - ❌ `company_info はフィールド名 → グループ名はPHPに出てこない`
  - ✅ `SCF 繰り返しフィールド → PHPで使うのはフィールド名。グループ名は管理画面の整理用でPHPに出てこない`
  - 後から読んでも「なんのこっちゃ」にならないよう、**一般的な言葉で概念を説明する**

- **特定サイトのクラス名を書かない**（絶対厳守）
  - ❌ `.home_page .header を transparent にする`（→ .home_page は特定サイト固有のクラス名）
  - ✅ `ヒーロー上のヘッダーを background: transparent にして画像と一体化させる`
  - クラス名ではなく「何をしているか」を一般的な言葉で書く

- **CSS値は必ずプロパティ名とセットで書く**（絶対厳守）
  - ❌ `transparent で初期は透明に`
  - ✅ `background: transparent で初期は透明に`
  - ❌ `none にする`
  - ✅ `border: none にする` / `backdrop-filter: none にする`

- **1行にこだわらない** → 後から読んでわかる量を書く
  - なぜそうなるかが重要な場合は理由も入れる
  - 1行で書けるなら1行でOK。でも複数行になっても問題なし

- **追記前に必ず空行を1行入れる**（前の内容と混ざらないように）
- 単純なミスは1行: `- 気づき・ミスの内容 → 正しい書き方・考え方`
- **難しいもの・理由が大事なものは複数行になってOK**（1行にまとめようとしない）
- 手順が必要なもの（導入フロー・複数ステップ）はフロー形式:
  ```
  - ○○ 導入フロー
    1. 手順1
    2. 手順2
  ```

### 日付セクションの書き込みルール（重要）
- Read でファイルを読み込み、今日の日付セクション `## YYYY-MM-DD` があるか確認する
- あれば → 内容だけ末尾に追記（Edit）
- なければ → 日付セクションを作ってから追記（Edit）
- **同じ日の2回目以降は `## 日付` を書かない**

---

# 追記後の報告

- 行番号・ハイライトの報告は不要

---

name: memo-all
description: 詳細メモ（01_memo.md）と気づき・ミスメモ（00_mis_log.md）の両方に同時書き込みする
triggers:
  - /memo-all
  - 両方にメモ
  - まとめてメモ
