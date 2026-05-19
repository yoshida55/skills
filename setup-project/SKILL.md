# setup-project スキル

## 背景・目的
RESTARTさんからのスキルチェック課題（Webサイト全8ページのコーディング）で使うスキル。
家でも作業したいが、家での作業タイムスタンプをRESTARTさんに見せたくないため、
リモートを2つ（origin=自分用、restart=RESTART提出用）登録して使い分ける運用をしている。
このスキルはその初回セットアップを自動で行う。

## 概要
RESTARTさんの案件の初回セットアップを最初から全部行う。
clone・exclude設定・リモートの名前変更・自分のGitHub追加・ブランチ作成を自動で行う。
**再実行も可能**（既にセットアップ済みのフォルダで実行すると、足りない設定だけ補完する）。

## 実行手順

### Step 1: 4つ確認する
以下を順番にユーザーに聞く。

1. 「RESTARTさんのリポジトリURLを教えてください」
2. 「自分のGitHubのリポジトリURLを教えてください」
3. 「自分のブランチ名を教えてください（例：kenzo）」
4. 「保存先フォルダのパスを教えてください（例：D:\02_作業\プロジェクト名）」

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

### Step 3: cloneする（新規の場合のみ）
```
git clone （RESTARTさんのURL） "（保存先フォルダのパス）"
```
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

### Step 5: 既にtrackedされているCLAUDE.md / sankou/ をuntrackする
セットアップ前に誤ってコミットされている可能性があるため確認：
```
git ls-files | grep -E "^(CLAUDE\.md|sankou/|\.claude/)"
```

該当ファイルがあれば自動でuntrackする：
```
git rm --cached （該当ファイル）
```

### Step 6: restartのデフォルトブランチ名を取得＆保存
```
git remote show origin | grep "HEAD branch"
```
または
```
git symbolic-ref refs/remotes/origin/HEAD | sed 's@^refs/remotes/origin/@@'
```
※ この時点ではまだ「origin = RESTARTさんのURL」状態

取得したブランチ名（通常 `main`、稀に `master`）を **`.git/restart-branch` に保存** する：
```
git symbolic-ref refs/remotes/origin/HEAD | sed 's@^refs/remotes/origin/@@' > .git/restart-branch
```

これにより以降のスキル（work-start, work-end）が自動的にこの値を読み込み、master/main を自動判定できる。
**手動で書き換える必要なし。**

確認：
```
cat .git/restart-branch
```
→ `main` または `master` が表示されればOK。

### Step 7: originをrestartに名前変更
cloneすると自動で「origin = RESTARTさんのURL」が登録されている。
これをrestartという名前に変える。
```
git remote rename origin restart
```

既存リポジトリで既にrestartが存在する場合はスキップ。

### Step 8: 自分のGitHubをoriginとして追加
```
git remote add origin （入力されたURL）
```

既に origin が存在する場合は `git remote set-url origin （URL）` で上書き。

### Step 9: 確認
```
git remote -v
```
以下のようになっていればOK：
```
origin  = 自分のGitHubのURL
restart = RESTARTさんのURL
```

### Step 10: 自分のブランチを作成して切り替え
既に該当ブランチがあるかチェック：
```
git branch --list （入力されたブランチ名）
```

ない場合のみ作成：
```
git checkout -b （入力されたブランチ名）
```
ある場合は切り替えのみ：
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
以下を伝える：

```
✅ セットアップ完了！

【基本情報】
- restart のデフォルトブランチ: （Step 6で取得した名前。例: main）
- 自分のブランチ: （入力されたブランチ名。例: kenzo）

【毎日の流れ】
- 作業開始時（家）: /sync-home-start
- 作業終了時（家）: /sync-home-end
- 出社時（会社）: /work-start
- 退社時（会社）: /work-end

【別PCで作業する場合の注意】
別のPCで初めて作業する場合も /setup-project を実行してください。
.git/info/exclude はPCごとに設定が必要です。設定しないと CLAUDE.md 等が
RESTARTさんに送られてしまいます。

【restart のデフォルトブランチが master だった場合】
各スキルは main を前提に書かれています。master の場合は実行時に
「restart のメインブランチは master です」と伝えてください。
```
