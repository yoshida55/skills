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

完了後、以下を `C:\Users\guest04\.claude\skills\deliver\last_config.json` に Write で保存する：

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

### ⚠️ 隠しフォルダのスキャン方法（必須）

**`ls -la | grep` は隠しフォルダを取りこぼすため禁止。必ず PowerShell で確認する：**

```powershell
powershell -Command "Get-ChildItem -Path '[プロジェクトパス]' -Force | Select-Object Name, Attributes"
```

これで `.git` / `.vscode` / `.claude` 等の隠しフォルダも確実に検出できる。

### スキャンして提案するもの（ユーザー確認後に削除）

PowerShell `Get-ChildItem -Force` の結果をもとに確認し、**AIが判断して**削除候補を提示する。
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

### ⚠️ 日本語パスの扱い（必読・絶対厳守）

**PowerShell に日本語パスを渡す唯一の確実な方法 = `-EncodedCommand`**

```bash
# PowerShellコマンドを UTF-16LE → base64 に変換して渡す
CMD='実行したいPowerShellコマンド（日本語パスを含む）'
ENC=$(printf '%s' "$CMD" | iconv -f UTF-8 -t UTF-16LE | base64 -w 0)
powershell -EncodedCommand "$ENC"
```

**❌ 以下は日本語パスで失敗するため禁止：**
- `powershell -Command "..."` → 文字化けで失敗
- `powershell -File script.ps1` → ファイルの文字コード不一致で失敗
- `robocopy` → 文字化けで失敗
- `zip` / `python3` → このPC環境に存在しない
- `tar` → Windowsドライブ文字（`c:`）を認識できない

### ① 一時フォルダへコピー（bashのcpを使う）

**bashのcpは日本語パスをそのまま扱えるため、コピーにはcpを使う：**

```bash
SRC="[プロジェクトパス（スラッシュ区切り）]"
DST="[親フォルダ]/_deliver_tmp_work/[出力名]"
EXCLUDE=(".git" ".vscode" ".claude" "node_modules" "[ユーザー承認済み削除候補]")

mkdir -p "$DST"

# サブフォルダをコピー（除外リストを除く）
for item in "$SRC"/*/; do
  name=$(basename "$item")
  skip=false
  for ex in "${EXCLUDE[@]}"; do [ "$name" = "$ex" ] && skip=true && break; done
  $skip && echo "除外: $name" || (cp -r "$item" "$DST/" && echo "コピー: $name")
done

# ルート直下のファイルをコピー
for ext in html css js; do
  for f in "$SRC"/*.$ext; do
    [ -f "$f" ] && cp "$f" "$DST/" && echo "コピー: $(basename $f)"
  done
done
echo "COPY_DONE"
```

### ② ZIP圧縮（-EncodedCommand を使う）

```bash
CMD='Compress-Archive -Path "[一時フォルダのWindowsパス]" -DestinationPath "[提出先のWindowsパス]\[出力名].zip" -Force; Write-Output "ZIP_DONE"'
ENC=$(printf '%s' "$CMD" | iconv -f UTF-8 -t UTF-16LE | base64 -w 0)
powershell -EncodedCommand "$ENC"
```

出力に `ZIP_DONE` が含まれていれば成功。

### ③ 一時作業フォルダをまとめて削除

```bash
# bashのrm -rfは日本語パスに対応している
rm -rf "[親フォルダ（スラッシュ区切り）]/_deliver_tmp_work"
echo "CLEANUP_DONE"
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
