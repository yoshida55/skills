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

### Step 2: origin の最新情報を取得
```
git fetch origin
```

### Step 3: 今日 /work-end が実行されたかチェック（やり忘れ検知）

origin/main の最新コミット日付を確認して、今日 /work-end が実行されたか推定する：
```
last_commit_date=$(git log -1 --format=%cd --date=short origin/main 2>/dev/null)
today=$(date +%Y-%m-%d)
echo "origin/main 最新: $last_commit_date / 今日: $today"
```

**今日と一致する場合**：
→ 今日 /work-end 済み（または家で /sync-home-end 済み）。Step 4 へ進む。

**一致しない場合**：⚠ /work-end のやり忘れの可能性
以下を伝える：
```
⚠ origin/main の最新コミットは（$last_commit_date）です。
今日（$today）の /work-end がまだ実行されていない可能性があります。

このまま家で作業すると、以下のリスクがあります：
  - 家作業の起点が「会社の今日の作業」を含まない古い状態になる
  - 翌朝 /work-start で会社local とマージ時に競合が増える
  - 会社local の今日分が古く見えて、家修正と整合性がズレる

選択肢：
  (a) このまま続行          → 今までの origin の状態を元に家作業
  (b) 中止して会社に確認     → 翌日会社で /work-end してから家作業
  
続行しますか？(y=続行 / N=中止)
```
- N → 停止。「明日会社で /work-end を実行してから家作業してください」と伝える
- y → Step 4 へ進む

⚠ 例外：今日が休日／在宅勤務で会社に行ってない場合は、当然 /work-end は無いので
このチェックは false positive となる。その場合は y で続行してOK。

### Step 4: ローカルの状態をチェック（force-push検知）

会社の /work-end は origin/main を **squash + force-push** するため、
通常の git pull だと家のローカル履歴とズレてconflictが起きる。
それを防ぐため、家側は常に origin/main の鏡（ミラー）として扱う。

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

### Step 5: 未コミットの変更をチェック
```
git status --short
```

**結果が空の場合**：Step 6 へ。

**変更がある場合**：自動で stash して退避する：
```
git stash push -m "sync-home-start-auto-stash"
```
「⚠ 未コミットの変更を一時退避しました。同期後に自動で戻します。」と伝える。

### Step 6: origin/main に強制同期（rebase相当だが安全）
```
git reset --hard origin/main
```

⚠ これは家を「origin の鏡」にする操作。会社で squash されたコミットがあっても
家のローカルが必ず最新の origin と一致するため、conflict が発生しない。

家での前回の作業内容は **既に squash されて origin/main に統合済み** なので、
ローカルの古い履歴を捨てても情報は失われない。

### Step 7: stash があれば戻す
Step 5 で stash した場合のみ：
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

### Step 8: 結果判定

**成功した場合**：
「最新を取り込みました。作業を開始してください！」と伝える。

**Step 6 の reset でエラーが出た場合**：
エラー内容を表示して原因を説明する。
