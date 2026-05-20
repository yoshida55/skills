# setup-project スキル

## 背景・目的
RESTARTさんからのスキルチェック課題（Webサイト全8ページのコーディング）で使うスキル。
家でも作業したいが、家での作業タイムスタンプをRESTARTさんに見せたくないため、
リモートを2つ（origin=自分用、restart=RESTART提出用）登録して使い分ける運用をしている。
このスキルはその初回セットアップを自動で行う。

## 概要
RESTARTさんの案件の初回セットアップを最初から全部行う。
clone・exclude設定・リモート設定・自分のブランチ作成・pre-push hook 設置を自動で行う。

**会社PCと家PCの両方に対応**：
- 会社PC: restart から clone（公式リポジトリが起点）
- 家PC: 自分の origin から clone（会社で push 済の最新作業を取得）

**再実行も可能**（既にセットアップ済みのフォルダで実行すると、足りない設定だけ補完する）。

## 実行手順

### Step 1: 環境確認 + 4つ確認する

まず環境を確認する：
```
このセットアップは「会社PC」と「家PC」のどちらですか？(c=会社 / h=家)
```

**c (会社PC) を選んだ場合** → 以下の順で聞く：
1. 「RESTARTさんのリポジトリURLを教えてください（cloneの起点）」
2. 「自分のGitHubのリポジトリURLを教えてください（origin として追加）」
3. 「自分のブランチ名を教えてください（例：kenzo）」
4. 「保存先フォルダのパスを教えてください（例：D:\02_作業\プロジェクト名）」

**h (家PC) を選んだ場合** → 以下の順で聞く：
1. 「自分のGitHubのリポジトリURLを教えてください（cloneの起点、会社で work-end 済の前提）」
2. 「RESTARTさんのリポジトリURLを教えてください（restart として追加）」
3. 「自分のブランチ名を教えてください（例：kenzo）」
4. 「保存先フォルダのパスを教えてください」

⚠ **家PCの前提条件**：会社で最低1回 /work-end して origin に push 済みであること。
未pushの場合は origin が空なので clone できない。先に会社で /setup-project + /work-end を実行してください。

入力された環境（c/h）と4つの回答を以降で参照する。
**環境フラグ `$ENV`**（"c" or "h"）として記憶する。

### Step 2: フォルダ存在チェック
保存先フォルダが既に存在するか確認する：
```
ls "（保存先フォルダのパス）"
```

**ケースA：フォルダが存在しない** → 通常のセットアップ（Step 3へ）

**ケースB：フォルダが存在する** → 既存リポジトリの可能性
- `.git` フォルダがあるか確認
- ある場合：「既にcloneされたフォルダのようです。不足設定だけ補完しますか？(Y/n)」
- ない場合：「フォルダは存在しますが.gitがありません。中身を確認してください」と止める

### Step 3: clone する（新規の場合のみ・環境別）

**会社PC (c) の場合**：
```
git clone （RESTARTさんのURL） "（保存先フォルダのパス）"
```

**家PC (h) の場合**：
```
git clone （自分のGitHubのURL） "（保存先フォルダのパス）"
```

⚠ 家PCで「remote: Repository is empty.」のエラーが出た場合：
→ 会社で /work-end が実行されておらず origin がまだ空です。
→ 会社で先に /setup-project + /work-end を実行してから家でセットアップしてください。

cloneが完了したら保存先フォルダに移動して以降の作業を続ける。

既存フォルダの場合はStep 3をスキップして以降に進む。

### Step 4: .git/info/excludeに除外設定を追加（cloneの直後・最優先）
⚠ ここを先にやることで、以降の作業で git add . をしても CLAUDE.md 等が追跡されない。
.gitignoreではなく.git/info/excludeに書く（RESTARTさんに見えないローカル専用設定）。

まず現在の exclude を確認する：
```
cat .git/info/exclude
```

以下のパターンを「未設定のものだけ」追記する：

**必須（個人情報・戦略漏洩防止）**：
```
sankou/
CLAUDE.md
.claude/
```

**推奨（一般的な雑多ファイル）**：
```
.vscode/
.DS_Store
Thumbs.db
*.bak
*.swp
*.log
.manual_commit_only
```
※ `.manual_commit_only` は commit_all.ps1 等の一括コミットスクリプトに「触るな」と伝えるマーカーファイル。restartには絶対送ってはいけないのでexcludeに入れる。

コマンド例：
```
echo "sankou/" >> .git/info/exclude
echo "CLAUDE.md" >> .git/info/exclude
echo ".claude/" >> .git/info/exclude
echo ".vscode/" >> .git/info/exclude
echo ".DS_Store" >> .git/info/exclude
echo "Thumbs.db" >> .git/info/exclude
echo "*.bak" >> .git/info/exclude
echo "*.swp" >> .git/info/exclude
echo "*.log" >> .git/info/exclude
echo ".manual_commit_only" >> .git/info/exclude
```

※ .gitignoreには書かない（restartに届いて隠していることがバレるため）

最後に **`.manual_commit_only` マーカーファイルを設置** する：
```
touch .manual_commit_only
```

⚠ このマーカーは `D:\Programs\commit_all.ps1` が「このフォルダは自動コミット対象外」と判断するための目印。
万一スクリプトの $directFolders / $parentFolders にこのフォルダが追加されても、
マーカーがある限り **自動コミットからスキップされる**（二重保険）。

⚠ 既に `.manual_commit_only` が存在する場合は touch しても無害（既存ファイルのタイムスタンプ更新のみ）。

⚠ このファイル自体は exclude に登録済なので、restart には絶対に送られない。

### Step 5: 既にtrackedされているCLAUDE.md / sankou/ をuntrackする
セットアップ前に誤ってコミットされている可能性があるため確認：
```
git ls-files | grep -E "^(CLAUDE\.md|sankou/|\.claude/)"
```

該当ファイルがあれば自動でuntrackする：
```
git rm --cached （該当ファイル）
```

### Step 6: リモートの整理（環境別）

clone直後の origin は環境によって意味が違う：
- 会社PC: origin = RESTARTさんのURL（rename が必要）
- 家PC: origin = 自分のGitHub（そのまま使う）

**会社PC (c) の場合**：

origin を restart に名前変更：
```
git remote rename origin restart
```
既存リポジトリで既にrestartが存在する場合はスキップ。

自分のGitHubを origin として追加：
```
git remote add origin （入力された自分のGitHub URL）
```
既に origin が存在する場合は `git remote set-url origin （URL）` で上書き。

**家PC (h) の場合**：

origin は既に自分のGitHubを指しているのでそのまま。
restart を追加するだけ：
```
git remote add restart （入力されたRESTART URL）
```
既に restart が存在する場合は `git remote set-url restart （URL）` で上書き。

restart のブランチ情報を取り込むため fetch：
```
git fetch restart
```

⚠ fetch しないと次の Step 7 で `refs/remotes/restart/HEAD` が見つからないので必須。

### Step 7: restartのデフォルトブランチを取得＆保存（共通）

リモート整理後、`refs/remotes/restart/HEAD` から restart のデフォルトブランチを取得し、
`.git/restart-branch` に保存する：
```
git symbolic-ref refs/remotes/restart/HEAD | sed 's@^refs/remotes/restart/@@' > .git/restart-branch
```

これにより以降のスキル（work-start, work-end）が自動的にこの値を読み込み、master/main を自動判定できる。
**手動で書き換える必要なし。**

⚠ 取得に失敗した場合のフォールバック：
```
git remote show restart | grep "HEAD branch" | awk '{print $NF}' > .git/restart-branch
```

それでも失敗する場合は手動で：
```
echo "main" > .git/restart-branch   # または master
```

確認：
```
cat .git/restart-branch
```
→ `main` または `master` が表示されればOK。

### Step 8: pre-push hook を設置（restart/main 誤push防止）
TortoiseGit 等の GUI 経由で誤って restart の main/master ブランチに push する事故を
構造的にブロックするため、`.git/hooks/pre-push` を設置する。

スキルディレクトリにテンプレが入っているので、それをコピーする。
**CRLF→LF 変換を必ず行う**（Windows の autocrlf でCRLFになってると bash がエラーになるため）：

```
# テンプレ位置（環境ごとに HOME が違うので展開する）
TEMPLATE="$HOME/.claude/skills/setup-project/pre-push-template"
HOOK=".git/hooks/pre-push"

# 既存フックがある場合の処理
if [ -f "$HOOK" ]; then
    echo "既に pre-push フックが存在します。上書きしますか？(Y/n)"
    # ユーザー確認 → Yなら上書き、Nならスキップ
fi

# CRLF を除去しながらコピー（autocrlf 対策）
tr -d '\r' < "$TEMPLATE" > "$HOOK"
chmod +x "$HOOK"
```

確認：
```
ls -la .git/hooks/pre-push
head -3 .git/hooks/pre-push
```
→ ファイルが存在し、先頭が `#!/bin/sh` で始まっていればOK。

⚠ このフックは `.git/hooks/` 配下なのでリモートには **絶対に送られない**（restartに見えない）。
⚠ origin への push は素通り。restart の自分ブランチへの push も素通り。
⚠ restart/main(master) 宛の push **のみ**ブロックする。

### Step 9: 確認
```
git remote -v
```
以下のようになっていればOK：
```
origin  = 自分のGitHubのURL
restart = RESTARTさんのURL
```

### Step 10: 自分のブランチを作成して切り替え（環境別の補足）

ローカルにブランチが既にあるかチェック：
```
git branch --list （入力されたブランチ名）
```

**ローカルに無い場合**：

【会社PC (c) の場合】
restart からの clone 直後なので、main から自分のブランチを作成：
```
git checkout -b （入力されたブランチ名）
```

【家PC (h) の場合】
origin/main は既に最新の作業を含むので、そこから自分のブランチを作成：
```
git checkout -b （入力されたブランチ名）
```
あるいは restart/(ブランチ名) が既に存在するなら、そこから直接：
```
git checkout -B （入力されたブランチ名） restart/（入力されたブランチ名）
```
（fetch 済なのでこのコマンドは使える。前回の会社作業の最終commit hashで揃う）

**ローカルに既にある場合**：切り替えのみ：
```
git checkout （入力されたブランチ名）
```

### Step 11: git config の確認（コミット時の名前・メール）
コミットには Author名・メールが記録される。RESTARTさんに見える情報なので確認する。

```
git config user.name
git config user.email
```

以下をユーザーに伝える：
```
【現在のgit config】
名前: （表示された名前）
メール: （表示されたメール）

これがRESTARTさんに見えるコミット作者情報です。
- 個人メール（gmail等）が見えても問題ないか？
- 名前はRESTARTさんが想定している名前と一致しているか？
- 別PC（会社/家）と同じ name/email になっているか？（違うとAuthor情報が分かれてバレる原因）

問題があれば以下で変更できます：
git config user.name "（名前）"
git config user.email "（メール）"
```

### Step 12: origin の公開設定 注意喚起
origin（自分のGitHub）が **public** だと、家での作業履歴が誰でも見える状態になる。
RESTARTさんが偶然見つけたら一発でバレる。

以下を伝える：
```
⚠ 重要：origin の GitHub リポジトリは必ず private にしてください。
  URL: （originのURL）
  確認方法: ブラウザで上記URLを開き、Settings > General > Danger Zone で
           "Change repository visibility" が Private になっているか確認。
  
  Public のままだと、squashで隠した家の作業履歴が origin 側に残るため、
  RESTARTさんがURLを推測すれば閲覧できてしまいます。
```

### Step 13: 完了報告
以下を伝える（環境に応じてメッセージを微調整）：

```
✅ セットアップ完了！（環境: 会社PC / 家PC）

【基本情報】
- restart のデフォルトブランチ: （Step 7で .git/restart-branch に保存済。例: main）
- 自分のブランチ: （入力されたブランチ名。例: kenzo）
- pre-push hook: 設置済（restart/main 誤push 防止）

【毎日の流れ】
- 作業開始時（家）: /sync-home-start
- 作業終了時（家）: /sync-home-end
- 出社時（会社）: /work-start
- 退社時（会社）: /work-end

【別PCで作業する場合の注意】
別のPCで初めて作業する場合も /setup-project を実行してください。
そのとき「会社PC or 家PC?」を正しく選んでください：
- 会社PC: restart からclone
- 家PC: 自分のorigin からclone（会社で push 済の前提）

.git/info/exclude / .git/hooks/pre-push / .git/restart-branch は
すべてPCごとに設定が必要です（.git/配下なので同期されないため）。

【restart のデフォルトブランチが master の場合】
.git/restart-branch に自動保存されているので、各スキルが自動判定します。
手動対応は不要です。
```
