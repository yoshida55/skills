# sync-home-start スキル

## 背景・目的
RESTARTさんのスキルチェック課題で使うスキル。
リモートが2つある運用（origin=自分用、restart=RESTART提出用）。
家では origin にだけ push・pull する。restart には送らない。
会社の帰りに work-end でsquashしてからrestartに送ることで、家での作業タイムスタンプを隠す。

## 概要
自宅での作業開始時に実行する。
originから最新を取り込んで作業を開始できる状態にする。

## 実行手順

### Step 1: originにmainブランチが存在するか確認
```
git ls-remote --heads origin main
```

**結果が空の場合**（初回・originがまだ空）：
→ 「originはまだ空です。pullをスキップして作業を開始してください。」と伝えて終了。

**結果がある場合**：Step 2へ。

### Step 2: ローカルの状態をチェック（force-push検知）

会社の /work-end は origin/main を **squash + force-push** するため、
通常の git pull だと家のローカル履歴とズレてconflictが起きる。
それを防ぐため、家側は常に origin/main の鏡（ミラー）として扱う。

まず origin の最新情報を取得：
```
git fetch origin
```

ローカルが origin/main より進んでいないか確認：
```
git rev-list origin/main..HEAD --count
```

**結果が 0 の場合**：ローカルは origin の追随中 → Step 3 へ。

**結果が 1 以上の場合**：⚠ 想定外の状態
家でコミットしたまま /sync-home-end を忘れている可能性がある。

以下を伝える：
```
⚠ ローカルに origin/main にないコミットが（N個）あります。
これは /sync-home-end の実行忘れの可能性があります。

確認してください：
  git log origin/main..HEAD --oneline

このコミットを保護したい場合：
  → 先に /sync-home-end を実行してから再度 /sync-home-start

このコミットを捨てて origin の状態に合わせたい場合：
  → 「reset実行」と言ってください
```

### Step 3: 未コミットの変更をチェック
```
git status --short
```

**結果が空の場合**：Step 4 へ。

**変更がある場合**：自動で stash して退避する：
```
git stash push -m "sync-home-start-auto-stash"
```
「⚠ 未コミットの変更を一時退避しました。同期後に自動で戻します。」と伝える。

### Step 4: origin/main に強制同期（rebase相当だが安全）
```
git reset --hard origin/main
```

⚠ これは家を「origin の鏡」にする操作。会社で squash されたコミットがあっても
家のローカルが必ず最新の origin と一致するため、conflict が発生しない。

家での前回の作業内容は **既に squash されて origin/main に統合済み** なので、
ローカルの古い履歴を捨てても情報は失われない。

### Step 5: stash があれば戻す
Step 3 で stash した場合のみ：
```
git stash list | grep -q "sync-home-start-auto-stash" && git stash pop
```

⚠ stash pop で競合が発生した場合：
```
⚠ 退避した修正と最新の origin が衝突しました。
解決手順：
1. 競合しているファイル（<<<<<<< が入っている）を開く
2. どちらを残すか判断して編集（記号も全部消す）
3. git add （ファイル名）
4. git stash drop  ← 古いstashを削除

わからない場合は「競合解決して」と言ってください。
```

### Step 6: 結果判定

**成功した場合**：
「最新を取り込みました。作業を開始してください！」と伝える。

**Step 4 の reset でエラーが出た場合**：
エラー内容を表示して原因を説明する。
