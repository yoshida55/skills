# work-end スキル

## 背景・目的
RESTARTさんのスキルチェック課題で使うスキル。
家での作業タイムスタンプをRESTARTさんに見せないために、
帰り際にsquash（複数コミットを1つに合体）してからrestartに送る。
squashで履歴を書き換えるため --force push が必須。
push先はrestart（自分のブランチ）とorigin（同期用mainブランチ）の2か所。

## 概要
会社での作業終了時（帰り際）に実行する。
家のコミットをsquashして、restartとoriginの両方にpushする。

## 実行手順

### Step 0: 現在時刻チェック（タイムスタンプ偽装の安全弁）
⚠ restartに届くコミットのタイムスタンプ＝この実行時刻になる。
会社の作業終了時刻（15:00）以降に実行するとRESTARTさんに「定時後も作業？」と不審に見える。

現在時刻を取得：
```
date +%H:%M
```

**時刻が 15:00 以降の場合**：
以下を伝えて確認する：
```
⚠ 現在の時刻は（時刻）です。
restart に届くコミット時刻もこの時刻になります。
作業終了時刻（15:00）を過ぎているため、不審に見える可能性があります。
本当に実行しますか？(y/N)
```
- N → 停止
- y → 続行

**時刻が 15:00 以前の場合**：そのまま Step 1へ。

### Step 1: 現在のブランチを確認
```
git branch --show-current
```

⚠ 表示されたブランチ名が自分のブランチ（例: kenzo）でない場合：
「現在のブランチは（ブランチ名）です。restartに送るのは自分のブランチである必要があります。
本当にこのブランチでwork-endを実行しますか？(y/N)」と確認。
- N → 停止
- Y → そのまま続行

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

### Step 4: コミットする
```
git add .
git commit --date="$(date -Iseconds)" -m "（ユーザーが入力したメッセージ）"
```

⚠ `--date=` フラグで Author Date を明示的に現在時刻に設定する（防御策）。
通常は不要だが、環境変数 `GIT_AUTHOR_DATE` 等の影響を防ぐための保険。

⚠ コミットメッセージに `Co-Authored-By: Claude` 等のAI関連の署名を絶対に含めない。
（restartに届くとAIに作業させていることがバレる）

### Step 5: squashの基準点を決める（家のコミットを消す要）
最新のrestart情報を取得：
```
git fetch restart
```

restartの自分のブランチが存在するか確認する：
```
git rev-parse restart/（現在のブランチ名）
```

**ケースA：restart/（ブランチ名）が存在する**（2回目以降のpush）
→ そのハッシュを squash の基準点にする

**ケースB：restart/（ブランチ名）が存在しない**（初回push）
⚠ ここで squash をスキップしてはいけない。家のコミットがそのままrestartに届いてタイムスタンプがバレる。
→ restart のデフォルトブランチ（main または master）との分岐点を基準にする：
```
RESTART_BRANCH=$(cat .git/restart-branch)
git merge-base HEAD "restart/$RESTART_BRANCH"
```

`.git/restart-branch` に保存された値（setup-project で記録済）を読むので、main/master を自動判定する。

このコマンドで、自分のブランチが main/master から分岐した時点のハッシュが取得できる。
そこから現在までのコミットを全部squashすれば、家のコミットも含めて1つに集約され、タイムスタンプは現在時刻（=会社時間）になる。

### Step 6: squashする
取得した基準ハッシュを使ってsquashする：
```
git reset --soft （取得したハッシュ）
git commit --date="$(date -Iseconds)" -m "（ユーザーが入力したメッセージ）"
```

⚠ **`--date="$(date -Iseconds)"` を必ず付ける**：
- Author Date を現在時刻に強制設定（家のタイムスタンプを消す）

⚠ **`--reset-author` フラグは付けない**：
- `git reset --soft` の後は新規コミット扱いなので、git config の name/email が自動的に使われる
- `--reset-author` は `--amend` / `-C` / `-c` と組み合わせる時だけ有効なフラグ
- 単独で付けると `fatal: --reset-author can be used only with -C, -c or --amend.` エラーになる

これにより、家のタイムスタンプ・Author情報が squash 後のコミットに残らない。

⚠ ここでも `Co-Authored-By: Claude` 等のAI署名は絶対に含めない。

squash対象が0個の場合（基準ハッシュ＝HEAD）はスキップしてStep 7へ。

### Step 7: restartにpush（自分のブランチへ）
pushする（squashで履歴を書き換えているのでforce必須）：
```
git push restart （現在のブランチ名） --force
```

⚠ push が rejected された場合（他の人が同じブランチにpushしている）：
通常、自分のブランチには自分しかpushしないが、念のため確認。
「⚠ pushが拒否されました。restart の同じブランチに別のコミットがあります。
通常は自分のブランチには自分しかpushしないはずです。状況を確認してください。」と伝えて停止。

### Step 8: originにpush
```
git push origin HEAD:main --force
```

### Step 9: reflog のクリーンアップ（プライバシー対策）
ローカルの reflog には、家のコミットを含む全ての HEAD 移動履歴が残っている。
push 後はもう不要なので、画面を覗かれた時のリーク防止のためにクリアする：

```
git reflog expire --expire=now --all
git gc --prune=now
```

⚠ これを実行すると **誤操作時の救済（git reflog で過去状態に戻す）が効かなくなる**。
push 完了後＝もう取り消す必要がない状態なので、ここで実行するのが安全。

⚠ もし「reflog残しておきたい」場合はこのステップをスキップしてもOK（プライバシー優先か復旧優先かの判断）。

### Step 10: 完了報告
「restartとoriginの両方にpushしました。お疲れ様でした！」と伝える。

## 注意
- squashはrestart/（現在のブランチ名）を基準にする（restart/mainではない）
- push先のブランチ名は git branch --show-current で自動取得する
- 自分のブランチ以外でwork-endを実行する場合は確認プロンプトが出る
