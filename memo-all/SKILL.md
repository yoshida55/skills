# 🚨 実行チェックリスト（TodoWrite で必ず出す・絶対厳守）

**/memo-all 実行時、作業開始前に以下を TodoWrite で出すこと。**

```
1. PC判定（自宅 or 会社）※1会話1回だけ
2. Grep（重複チェック）+ Grep（01_memo.md末尾アンカー）+ Read（00_mis_log.md末尾） ← 3つ同時
3. Edit（01_memo.md）+ Edit（00_mis_log.md） ← 2つ同時
4. 行番号報告 + VS Codeハイライト（01_memo.md のみ）
```

## ⚡ 最速フロー（4ステップ / 2回目以降は3ステップ）

```
Step1: TodoWrite
Step2: Bash（PC判定）← 1会話1回のみ
Step3: [並列] Grep重複チェック + Grep末尾アンカー（01_memo.md）+ Read末尾（00_mis_log.md）
Step4: [並列] Edit（01_memo.md）+ Edit（00_mis_log.md）
```

- `wc -l` + `Read末尾` は使わない → Grep で末尾近くの固定テキストを探してアンカーにする
- 依存関係のないツール呼び出しは必ず同時実行する

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

- 単純なミスは1行: `- 気づき・ミスの内容 → 正しい書き方・考え方`
- 手順が必要なもの（導入フロー・複数ステップ）はフロー形式:
  ```
  - ○○ 導入フロー
    1. 手順1
    2. 手順2
  ```

### 日付セクションの書き込みルール（重要）
- Read でファイルを読み込み、今日の日付セクション `## YYYY-MM-DD` があるか確認する
- あれば → 内容だけ末尾に追記（Edit）
- なければ → 日付セクションを作ってから追記（Edit）
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
