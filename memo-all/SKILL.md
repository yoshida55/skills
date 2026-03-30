# ⚡ バックグラウンド実行（常時・確認不要・絶対厳守）

> メモの書き込みは**常にバックグラウンドで実行する**。
> ユーザーへの確認は不要。有無を言わさずバックグラウンドで走らせる。

- **Bash ツールの `run_in_background: true`** を使ってメモ書き込みを実行（Agentツール禁止：権限がサブエージェントに引き継がれないため）
- メインはメモ書き込みを待たずに、すぐ次の説明・作業へ進む
- 確認文は出さない・待たない・聞かない

---

# 🚨 実行チェックリスト（TodoWrite で必ず出す・絶対厳守）

**/memo-all 実行時、作業開始前に以下を TodoWrite で出すこと。**

```
1. PC判定（自宅 or 会社）
2. 重複チェック（Grep）
3. ✅ Bash ツール（run_in_background: true）で両ファイルへ書き込み実行
4. 行番号報告 + VS Codeハイライト（01_memo.md のみ）
```

---

# 出力先（2ファイルに同時書き込み）

## 📘 詳細メモ（説明・仕組み）
存在する方に追記:
- 会社PC: `C:\Users\guest04\Desktop\高橋研三\03_knowledge\01_memo.md`
- 自宅PC: `D:\50_knowledge\01_memo.md`

## 📒 気づき・ミス（1行メモ）
存在する方に追記:
- 会社PC: `C:\Users\guest04\Desktop\高橋研三\03_knowledge\00_mis_log.md`
- 自宅PC: `D:\50_knowledge\00_mis_log.md`

判定方法: Bash で `test -f` を実行し、存在する方を使用

> ⚡ **チェックは1会話に1回だけ**

---

# 書き込みルール

## 📘 01_memo.md（詳細メモ）
→ **memo-format スキルと同じフォーマット**で書く

- タイトルは「読んだだけで答えがわかる」レベル
- 【日付】【結論】【具体例】【補足】の構成
- カテゴリタグ（`WordPress` / `HTML`）を末尾に付ける
- 重複チェック必須（あれば既存を更新・なければ末尾追記）

## 📒 00_mis_log.md（気づき・ミス）
→ **memo_days スキルと同じフォーマット**で書く

- 1行フォーマット: `- 気づき・ミスの内容 → 正しい書き方・考え方`
- 重複チェック必須（既に同じ内容があれば追記しない）

### 日付セクションの書き込みルール（重要）
```bash
# 今日の日付セクションがあるか確認
grep -q "## $(date +%Y-%m-%d)" "D:/50_knowledge/00_mis_log.md"
if [ $? -ne 0 ]; then
  # ❌ なければ → 日付セクションを新規作成してから追記
  printf "\n## $(date +%Y-%m-%d)\n\n- 内容\n" >> "D:/50_knowledge/00_mis_log.md"
else
  # ✅ あれば → 日付セクションなしで内容だけ追記
  printf "- 内容\n" >> "D:/50_knowledge/00_mis_log.md"
fi
```
- **同じ日の2回目以降は `## 日付` を書かない**

---

# 追記後の報告

- `01_memo.md` の追記行番号を報告する
- VS Codeハイライトを実行する（自宅PC）:
  ```bash
  curl -s -X POST http://127.0.0.1:3848/highlight-line -H "Content-Type: application/json" --data-binary "{\"filePath\": \"D:/50_knowledge/01_memo.md\", \"lineNumber\": 追記開始行番号}"
  ```
- curlが失敗してもエラーは無視してよい

---

name: memo-all
description: 詳細メモ（01_memo.md）と気づき・ミスメモ（00_mis_log.md）の両方に同時書き込みする
triggers:
  - /memo-all
  - 両方にメモ
  - まとめてメモ
