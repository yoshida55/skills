# work-start スキル

## 背景・目的
RESTARTさんのスキルチェック課題で使うスキル。
リモートが2つある（origin=自分用、restart=RESTART提出用）。
朝はまずrestartのメインブランチからチームの最新を取り込み、次にoriginから家の作業を取り込む。
競合が起きた場合は両方の変更を手動でマージしてからコミットする。

## 概要
会社での作業開始時（朝）に実行する。
restartとoriginの最新を取り込む。

## 実行手順

### Step 1: restartのデフォルトブランチ名を取得
```
git remote show restart | grep "HEAD branch"
```
通常は `main`、稀に `master` の場合がある。取得した名前を以降のpullで使う。

取得に失敗した場合は `main` を仮定する。

### Step 2: restartに該当ブランチがあるか確認
```
git ls-remote --heads restart （取得したブランチ名）
```

**結果が空の場合**（初回・restartが空）：
→ 「restartはまだ空です。Step 3はスキップしてStep 4へ進みます。」と伝える。

**結果がある場合**：Step 3へ。

### Step 3: restartの最新を取り込む
```
git pull restart （取得したブランチ名）
```

⚠ 競合が発生した場合（CONFLICT メッセージ）：
以下を伝える：
```
⚠ チームの変更と自分の変更が衝突しました。
解決手順：
1. 競合しているファイルを開く（<<<<<<< が入っている箇所）
2. どちらを残すか判断して編集（記号も全部消す）
3. 完了したら以下を実行：
   git add （ファイル名）
   git commit -m "merge: 競合解決"

わからない場合は私に「競合解決して」と言ってください。
解決後、Step 4に進みます。
```

### Step 4: originにmainブランチがあるか確認
```
git ls-remote --heads origin main
```

**結果が空の場合**（初回・originがまだ空）：
→ 「originはまだ空です。pullをスキップします。」と伝えてStep 6へ。

**結果がある場合**：Step 5へ。

### Step 5: originから最新を取り込む（家の作業を取り込む）
```
git pull origin main
```

⚠ 競合が発生した場合：Step 3と同じ手順で解決する。

### Step 6: 完了報告
「最新を取り込みました。作業を開始してください！」と伝える。

## 注意
- Step 3 → Step 5 の順番厳守（逆だと競合しやすくなる）
- 競合解決時は両方の変更を残すか、どちらかを採用するか判断が必要
