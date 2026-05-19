# sync-home-end スキル

## 背景・目的
RESTARTさんのスキルチェック課題で使うスキル。
家では origin にだけ push する（restartには送らない）。
翌朝会社で work-end を実行するとsquashされ、家の作業タイムスタンプが消えてからrestartに届く。

## 概要
自宅での作業終了時に実行する。
変更をコミットしてoriginにpushする。

## 実行手順

### Step 0: origin との差分チェック（sync-home-start スキップ救済）
朝にsync-home-startを忘れて修正してしまった場合の自動救済処理。

```
git fetch origin
git rev-list HEAD..origin/main --count
```

**結果が 0 の場合**：ローカルは最新 → Step 1へ。

**結果が 1 以上の場合**：originが進んでいる（sync-home-start忘れの可能性）

以下を伝える：
```
⚠ originが（N個）進んでいます。
sync-home-start を忘れていませんか？
先に同期してから続行します。
```

未コミットの変更があるか確認：
```
git status --short
```

**未コミットの変更がない場合**：
```
git pull origin main --rebase
```

**未コミットの変更がある場合**（修正してから sync-home-end を実行したケース）：
```
git stash                            # 修正を一時退避
git pull origin main --rebase        # originと同期
git stash pop                        # 修正を戻す
```

⚠ stash pop で競合が発生した場合：
```
⚠ 退避した修正と origin の最新が衝突しました。
解決手順：
1. 競合しているファイルを開く（<<<<<<< が入っている箇所）
2. どちらを残すか判断して編集
3. 完了したら以下を実行：
   git add （ファイル名）

その後、もう一度 /sync-home-end を実行してください。
わからない場合は「競合解決して」と言ってください。
```

### Step 1: 現在のブランチを確認
```
git branch --show-current
```

⚠ 表示されたブランチ名が自分のブランチ（例: kenzo）でない場合：
「現在のブランチは（ブランチ名）です。自分のブランチに切り替えてから実行してください。」と伝えて停止。

### Step 2: 変更ファイルをリストアップしてユーザーに確認する
以下を実行して変更ファイルを表示する（手動stageを含めて完全網羅）：
```
git status --short
```

凡例：
- ` M` = 変更（未ステージ）
- `M ` = 変更（ステージ済み）
- `A ` = 新規追加（ステージ済み）
- ` D` = 削除
- `??` = 未追跡（新規ファイル）

表示されたファイルを一覧でユーザーに見せて
「以下のファイルをコミットします。問題ありませんか？」と確認する。

変更がない場合は「変更がありません。作業は完了していますか？」と伝えて終了する。

⚠ 安全チェック：以下のファイルが含まれていたら自動でuntrackしてから続行する。
- CLAUDE.md
- sankou/ 配下のファイル（sankou/で始まるもの）
- .claude/ 配下のファイル

対処：
「⚠ RESTARTさんに見せてはいけないファイルが含まれています：（ファイル名）
追跡を解除（untrack）してからコミットします。」
と伝えて、以下を実行する：
```
git rm --cached （ファイル名）
```
複数ある場合はすべて実行してからStep 3に進む。

### Step 3: コミットメッセージをユーザーに確認する
「コミットメッセージを入力してください（例：ヘッダーHTML作成）」と聞く。

### Step 4: コミットしてpush
```
git add .
git commit -m "（ユーザーが入力したメッセージ）"
git push origin HEAD:main
```

⚠ コミットメッセージに `Co-Authored-By: Claude` 等のAI関連の署名を絶対に含めない。
（このコミットは家のものとしてorigin止まりだが、翌朝のwork-endでsquashされてrestartに届く可能性があるため）

### Step 5: 結果判定

**成功した場合**：
「originにpushしました。会社でwork-startを実行して続きを作業してください！」と伝える。

**push が rejected された場合**（リモートが先に進んでいる）：
以下を伝える：
```
⚠ originのmainが先に進んでいます。先にpullしてください：
git pull origin main --rebase

その後、もう一度 /sync-home-end を実行してください。
```

**その他のエラー**：エラー内容を表示して原因を説明する。

## 注意
- push先はorigin mainのみ（restartには送らない）
- restartへのpushはwork-endで行う
