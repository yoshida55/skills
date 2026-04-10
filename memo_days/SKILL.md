---
name: memo_days
description: 今日のミスを00_mis_log.mdに1行で記録する
---

# memo_days スキル

今日ミスったこと・気づいたことを `00_mis_log.md` に追記するスキル。

## 使い方
```
/memo_days
```

---

## ⚡ バックグラウンド実行（常時・確認不要・絶対厳守）

- **書き込みは必ず `Agent ツール（run_in_background: true）` で実行する**
- ユーザーを待たせない・確認も不要・黙って実行してOK
- Agent に渡す指示には「ファイルパス・日付・追記内容」を全部含める

---

## 実行フロー

### Step 1: ファイルパスを確定する
- 会社PC: `C:\Users\guest04\Desktop\高橋研三\03_knowledge\00_mis_log.md`
- 自宅PC: `D:\50_knowledge\00_mis_log.md`

判定:
```bash
test -f "D:\50_knowledge\00_mis_log.md" && echo "home" || echo "company"
```

### Step 2: 今日の日付セクションがあるか確認してから書き込む（Agent バックグラウンドで実行）

**必ず `Agent（run_in_background: true）` に以下を渡して実行する：**

必ず以下のBashロジックで書き込む（日付の重複を防ぐ）：

```bash
TODAY=$(date +%Y-%m-%d)
FILE="D:/50_knowledge/00_mis_log.md"  # 自宅PC

if grep -q "## $TODAY" "$FILE"; then
  # ✅ 今日の日付セクションがある → 内容だけ追記
  printf -- "- 内容\n" >> "$FILE"
else
  # ❌ 今日の日付セクションがない → 日付セクションを作ってから追記
  printf "\n## %s\n\n- 内容\n" "$TODAY" >> "$FILE"
fi
```

フォーマット：
```
- 間違えた内容 → 正しい書き方・使い方
```

---

## 保存フォーマット

```markdown
## 2026-03-30

- get_the_category() はループ外不可 → get_categories() を使う
- ->name はスペースなし

- esc_html = テキスト用 / esc_url = URL用
- esc_attr = HTML属性の値に使う
```

グループが変わるときは **空行1行** を入れて見やすくする。

---

## ルール

- **単純なミスは1行** で書く: `- 間違い → 正解`
- **手順が必要なもの（導入フロー・複数ステップ）はフロー形式** で書く:
  ```
  - ○○ 導入フロー
    1. 手順1
    2. 手順2
    3. 手順3
  ```
- **正しい答えだけ** 書く（間違いの詳細は不要）
- 日付ごとに `##` で区切る
- 同じ日に複数追記するときは同じ `##` セクションに追加
- 同じ内容が既にある場合は追記しない（Grep で確認する）
- **ユーザーが手書きで追記した内容は絶対に消さない・上書きしない**（Write ツールで全体を書き直すときも既存の内容を必ず保持する）
