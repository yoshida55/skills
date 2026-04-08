---
name: deliver
description: 現在のプロジェクトを納品用にクリーンコピー→圧縮→指定フォルダへ移動する
---

# deliver スキル

> ## ⚡ 絶対最初にやること（これを忘れたら全部やり直し）
> **スキル読み込み直後、作業の1行目は必ず `TodoWrite` でStep 0のチェックリストを出す**
> ❌ TodoWrite なしで作業開始 → 禁止
> ✅ TodoWrite → その後に作業開始 → 正しい順序

プロジェクトフォルダを汚さずに、納品用のクリーンなZIPを作成して指定フォルダへ移動するスキル。

---

## 🚀 Step 0. TodoWriteでチェックリスト表示（スキル開始直後に必ず実行）

```
TodoWrite([
  { content: "ユーザーに出力ファイル名・提出先フォルダを確認",   status: "in_progress" },
  { content: "プロジェクト内の削除候補フォルダをスキャン",       status: "pending" },
  { content: "削除リストをユーザーに確認（承認待ち）",           status: "pending" },
  { content: "一時フォルダへコピー",                             status: "pending" },
  { content: "不要ファイル・フォルダを一時フォルダから削除",     status: "pending" },
  { content: "ZIP圧縮",                                         status: "pending" },
  { content: "提出先フォルダへ移動",                             status: "pending" },
  { content: "一時フォルダを削除",                               status: "pending" },
  { content: "完了報告",                                         status: "pending" }
])
```

---

## 📋 Step 1. ユーザーへの確認（実行前に必ず聞く）

### 前回の提出先を確認する

スキルフォルダ内の `last_config.json` を Read で読み込み、前回の提出先を取得する。
- ファイルが存在すれば → 提出先のデフォルトとして表示
- 存在しなければ → ユーザーに入力してもらう

### 質問内容

```
📦 納品パッケージを作成します。以下を教えてください：

① 出力名（例: 85_自社ホームページ_提出）
   → ZIP名・ZIP内フォルダ名の両方がこの名前になります
   → 「[①].zip」の中に「[①]/」フォルダが入った構成になります

② 提出先フォルダのパス
   → 前回: D:\02_作業\00_提出物（そのままでよければ Enter）
   → 初回は入力してください
```

### 実行後に保存する

完了後、以下を `C:\Users\sensh\.claude\skills\deliver\last_config.json` に Write で保存する：

```json
{
  "last_submit_dir": "[ユーザーが指定した提出先パス]"
}
```

---

## 🔍 Step 2. 削除候補のスキャン

### 必ず削除するもの（確認不要）

| 対象 | 理由 |
|------|------|
| `.git/` | GitHubのバージョン管理フォルダ |
| `.github/` | GitHub設定フォルダ |
| `.vscode/` | VS Code設定フォルダ |
| `.claude/` | Claude Code設定フォルダ |
| `node_modules/` | パッケージ（重い・不要） |
| `.DS_Store` | Mac用の不要ファイル |
| `Thumbs.db` | Windows用の不要ファイル |
| `*.log` | ログファイル |

### スキャンして提案するもの（ユーザー確認後に削除）

Glob でプロジェクト直下・2階層目までを確認し、**AIが判断して**削除候補を提示する。
キーワードに頼りすぎず、「納品物として不要そうか」を総合的に判断すること。

**判断の観点**:
- 名前から「参考・下書き・バックアップ・開発用」とわかるもの
  - 例: `sankō/` `参考img/` `ref/` `bkup/` `old_index.html` `draft/`
- 中身が納品に不要と思われるもの
  - 例: `img/mockup/`（画像の中にある設計用フォルダ）
  - 例: `_old/`（フォルダ名に old が入っている）
  - 例: `*.psd` `*.ai` `*.xd` `*.fig`（デザインデータ）
- 明らかに開発中のメモ・スクラッチファイル
  - 例: `memo.txt` `todo.md` `test.html`

**⚠ 判断に迷う場合**: 削除せず「❓ 確認」扱いにする。勝手に決めない。

**報告形式**:
```
🗂 削除候補（確認をお願いします）:

必ず削除:
  ✅ .git/           → バージョン管理フォルダ
  ✅ .vscode/        → VS Code設定
  ✅ .claude/        → Claude Code設定

削除してよいですか？（番号で残したいものを指定 / 「はい」で全削除）:
  ❓ 1. img/参考/       → 参考画像フォルダと判断
  ❓ 2. bkup/           → バックアップフォルダと判断
  ❓ 3. design.psd      → デザインデータと判断
```

---

## ⚙️ Step 3. コピー・削除・圧縮の実行

ユーザーが確認後、Bash で以下を実行する。

### ① 一時フォルダへコピー（コピー先フォルダ名をリネーム）

```bash
DEST_NAME="[ユーザー指定の出力名]"
PROJECT_DIR="[現在のプロジェクトパス]"   # ← 元フォルダ。絶対に触らない

# コピー先は「プロジェクトと同階層」に「DEST_NAME」という名前で作る
# 元のプロジェクトフォルダ名は変えない。コピー先だけ新しい名前になる
TMP_DIR="${PROJECT_DIR}/../_deliver_tmp_work/${DEST_NAME}"

# robocopy でコピー（元フォルダは読み取りのみ。変更ゼロ）
robocopy "$PROJECT_DIR" "$TMP_DIR" /E /XD ".git" ".github" ".vscode" ".claude" "node_modules" /XF "*.log" "Thumbs.db" ".DS_Store"
```

### ② スキャン結果の追加削除

```bash
# ユーザーが承認した削除候補を削除
rm -rf "$TMP_DIR/[削除対象1]"
rm -rf "$TMP_DIR/[削除対象2]"
```

### ③ ZIP圧縮

```bash
SUBMIT_DIR="[提出先フォルダ]"

# $TMP_DIR（= .../DEST_NAME/）を圧縮
# → ZIP内は「DEST_NAME/」フォルダになる（ZIP名とフォルダ名が一致）
powershell -Command "Compress-Archive -Path '$TMP_DIR' -DestinationPath '$SUBMIT_DIR/${DEST_NAME}.zip' -Force"
```

### ④ 一時作業フォルダをまとめて削除

```bash
# _deliver_tmp_work/ ごと削除（DEST_NAME フォルダも含めて一掃）
rm -rf "${PROJECT_DIR}/../_deliver_tmp_work"
```

---

## 🎯 完了報告フォーマット

```
✅ 納品パッケージ完成！

📦 ファイル名: 85_自社ホームページ_提出.zip
📁 場所: D:\02_作業\00_提出物\
🗂 削除したもの: .git / .vscode / .claude / img/参考/（5件）
📏 サイズ: 〇〇 MB

元のプロジェクトフォルダは変更していません。
```

---

## ⚠️ 注意事項

1. **元フォルダは絶対に触らない** → コピー先（一時フォルダ）だけ操作する
2. **一時フォルダの場所** → プロジェクトと同階層に `_deliver_tmp_[名前]` で作成
3. **圧縮後に一時フォルダを必ず削除** → ゴミが残らないようにする
4. **提出先フォルダが存在しない場合** → ユーザーに確認してから `mkdir` する
5. **同名ZIPが既に存在する場合** → 上書きしてよいか確認する
