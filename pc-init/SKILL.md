# pc-init スキル

## 背景・目的
新しいPC（自宅・会社・買い替え等）にRESTART案件の作業環境を整える時に、
PC全体の準備状態を1回でチェック＆セットアップするスキル。
プロジェクト単位の /setup-project とは責任を分離し、PC側のチェックはここに集約する。

## 概要
新PCで初回作業前に1回だけ実行する。以下を順番にチェックし、不足があれば対話で補完する：
- git のインストール
- git config (user.name / user.email) の設定
- gh CLI のインストール
- gh CLI の認証
- ~/.claude/skills/ の clone 状態
- 完了マーカーファイルの設置

**再実行も可能**（毎回チェックして既に通ってる項目はスキップ）。

## 実行手順

### Step 1: 環境確認の冒頭メッセージ
以下を伝える：
```
🖥️ PC初期チェックを開始します。
新しいPC、または久しぶりに使うPCで実行することを想定しています。

以下を順番にチェックします：
1. git のインストール
2. git config（コミット時の名前・メール）
3. gh CLI（GitHub操作）
4. gh CLI の認証
5. ~/.claude/skills/ のスキル群
6. 完了マーカーの設置
```

### Step 2: git のインストール確認
```
git --version
```

**コマンドが見つからない場合**：
```
❌ git がインストールされていません。

【インストール方法】
1. https://git-scm.com/download/win から Git for Windows をダウンロード
2. インストーラーを実行（基本デフォルト設定でOK）
3. インストール完了後、ターミナル再起動してこのスキルを再実行してください

止めます。
```
→ 停止。

**インストール済の場合**：
```
✅ git バージョン: （表示されたバージョン）
```

### Step 3: git config の確認
```
git config --global user.name
git config --global user.email
```

**両方が設定済の場合**：
```
✅ git config 設定済
   名前:   （表示された名前）
   メール: （表示されたメール）

⚠ 他のPC（家/会社）と完全に一致してますか？
他PCで別の値だと、restart にコミットが届いた時に Author が分かれて
RESTARTさんに「別人？」と疑われる可能性があります。

一致してれば「OK」、違ってたら「修正したい」と言ってください。
```

- 「OK」→ Step 4 へ
- 「修正したい」→ 以下を実行：
  ```
  git config --global user.name "（修正値）"
  git config --global user.email "（修正値）"
  ```

**未設定の場合**：
```
⚠ git config が未設定です。

以下を確認させてください（他PCと完全一致させること）：
- 名前: 
- メール: 
```
ユーザーから受け取った値で設定：
```
git config --global user.name "（入力値）"
git config --global user.email "（入力値）"
```

### Step 4: gh CLI のインストール確認
```
gh --version
```

**コマンドが見つからない場合**：
```
⚠ gh CLI が未インストールです。

これがあると /setup-project Step 12 で origin の公開設定を
自動チェック・自動Private化できるので、入れることを強く推奨します。

【インストール方法（Windows）】
winget install GitHub.cli

【インストール方法（macOS）】
brew install gh

インストール後、ターミナル再起動してこのスキルを再実行してください。

スキップして続行しますか？(y=スキップ / n=停止して入れる)
```

- y → Step 5 をスキップして Step 6 へ（gh 認証もスキップ）
- n → 停止

**インストール済の場合**：
```
✅ gh CLI バージョン: （表示されたバージョン）
```

### Step 5: gh CLI の認証確認
```
gh auth status
```

**認証済の場合**：
```
✅ gh CLI 認証済
   アカウント: （表示されたアカウント名）
   スコープ:   （表示されたスコープ）
```

⚠ スコープに `repo` が含まれているか確認。なければ `gh auth refresh -s repo` を案内。

**未認証の場合**：
```
⚠ gh CLI が認証されていません。

これから認証します。ブラウザが開くので GitHub にログインしてください：
```

```
gh auth login
```

対話プロンプトの選択肢：
- Where do you use GitHub? → GitHub.com
- Preferred protocol → HTTPS
- Authenticate Git with your GitHub credentials? → Yes
- How would you like to authenticate? → Login with a web browser

完了後、再度 `gh auth status` で確認。

### Step 6: ~/.claude/skills/ のスキル群を確認
```
ls "$HOME/.claude/skills"
```

**フォルダが存在し、複数スキルがある場合**：
```
✅ スキル群が存在します。

含まれるスキル：
（ls の結果）
```

⚠ skills フォルダが git で管理されてるか確認（推奨）：
```
cd "$HOME/.claude/skills" && git status 2>&1
```

git 管理されてない場合：
```
⚠ skills フォルダが git で管理されていません。
新PC間で同期したい場合は、GitHubでskillsリポジトリを作って push しておくと
別PCで `git clone` だけで同期できます。
（今回は対応不要、続行します）
```

**フォルダが無い、またはほぼ空の場合**：
```
❌ ~/.claude/skills/ にスキル群がありません。

【対処】
1. GitHub上のスキルリポジトリのURLを教えてください
2. clone します：
   cd "$HOME/.claude" && git clone （URL） skills

完了したらこのスキルを再実行してください。
```
→ 停止。

### Step 7: 完了マーカーの設置
PC初期化が完了したことを記録する：
```
date -Iseconds > "$HOME/.claude/.pc-init-done"
```

⚠ このファイルは将来 /setup-project などから「PC初期化済か」を判定する際に使える。
（今は単に記録目的）

### Step 8: 完了報告
```
✅ PC初期化完了！

【セットアップされた環境】
- git:                  （バージョン）
- git config user.name: （値）
- git config user.email: （値）
- gh CLI:               （バージョン or 未インストール）
- gh 認証:              （アカウント or 未認証）
- スキル群:             （個数）件

【次にやること】
プロジェクトごとに以下を1回実行：
  /setup-project

【他PCとの同期】
家・会社 両方のPCでこのスキルを実行し、git config を同じ値にすること。
これだけで「別人扱い」事故を100%防げます。

【再実行】
何か環境を変えた時は、また /pc-init を実行すればチェック＆補完されます。
```

## 注意
- このスキルは「PC1台につき1回」が基本（再実行は無害）
- /setup-project の前に1回必ず通すと安全
- 各チェックは独立して動くので、途中で失敗しても致命傷にはならない
- skills フォルダの clone URL はユーザー固有なので、初回はユーザーに聞く
