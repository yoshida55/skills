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

### Step 3: 直近の /work-end からの経過日数チェック（やり忘れ検知）

origin/main の日付だけだと「家の /sync-home-end も同じ origin に push する」せいで
work-end やり忘れと区別できない。
そこで `restart/takahashi` を見る。**ここは /work-end だけが push する場所** なので、
最新コミット日時 = 最後に /work-end した時刻 が正確にわかる。

restart の最新情報を取得：
```
git fetch restart 2>/dev/null
```

経過時間を計算：
```
last_workend_ts=$(git log -1 --format='%ct' restart/takahashi 2>/dev/null)
last_workend_iso=$(git log -1 --format='%cd' --date=iso restart/takahashi 2>/dev/null)
now_ts=$(date +%s)
gap_seconds=$((now_ts - last_workend_ts))
gap_days=$((gap_seconds / 86400))
```

**restart/takahashi が取得失敗の場合**：
→ 「初回 setup-project 直後の可能性。スキップして Step 4 へ進みます。」

**gap_days が 1日以内の場合**（昨日の夜～今日の work-end）：
→ 通常運用とみなして静かに続行。短いメッセージのみ表示：
```
✅ 直近の /work-end: 約 $gap_days 日前（$last_workend_iso）。続行します。
```
→ Step 4 へ進む。

**gap_days が 2日以上の場合**：⚠ やり忘れの可能性ありとして警告：
```
⚠ 直近の /work-end から $gap_days 日経過しています。
   最後の /work-end: $last_workend_iso

この間に「会社で作業した日」がありますか？

判断の目安：
  ✅ 続行してOK：
     - 土日を挟んだだけで会社作業してない（金→土日→月の朝など）
     - 在宅勤務／休暇で会社に行っていない
     
  ⚠ 中止すべき：
     - この日付以降に会社で作業した日がある（やり忘れの可能性）

選択：
  (y) 続行     上記日付以降に会社作業してない、または問題ない
  (N) 中止     やり忘れ疑い → 次回会社で /work-end してから家作業

続行しますか？(y/N)
```

- N → 停止。「次回会社出社時に /work-end を実行してから家作業を再開してください」と伝える
- y → Step 4 へ進む

⚠ この方式の利点：
- 1日以内は毎日のルーティンと判断して静かに続行（鬱陶しくない）
- 2日以上空いた時だけ警告 → 通常週末や休暇では穏当な確認に
- 金曜やり忘れ→日曜sync の場合は最終 work-end が 3日以上前になるので確実に検知

### Step 4: ローカルの状態をチェック（force-push検知）

会社の /work-end は origin/main を **squash + force-push** するため、
通常の git pull だと家のローカル履歴とズレてconflictが起きる。
それを防ぐため、家側は常に origin/main の鏡（ミラー）として扱う。

ローカルが origin/main より進んでいないか確認：
```
git rev-list origin/main..HEAD --count
```

**結果が 0 の場合**：ローカルは origin の追随中 → Step 5 へ。

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
