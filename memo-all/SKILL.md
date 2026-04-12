# 🚨 実行チェックリスト（TodoWrite で必ず出す・絶対厳守）

**/memo-all 実行時、作業開始前に以下を TodoWrite で出すこと。**

```
1. PC判定（自宅 or 会社）※1会話1回だけ
2. Read（01_memo_index.txt）+ Read（00_mis_log.md末尾） ← 2つ同時
3. Edit（01_memo.md）+ Edit（00_mis_log.md）+ Edit（01_memo_index.txt） ← 3つ同時
4. 行番号報告 + VS Codeハイライト（01_memo.md のみ）
```

## ⚡ 最速フロー（4ステップ / 2回目以降は3ステップ）

```
Step1: TodoWrite
Step2: Bash（PC判定）← 1会話1回のみ
Step3: [並列] Read（01_memo_index.txt）+ Read末尾（00_mis_log.md）
Step4: [並列] Edit（01_memo.md末尾）+ Edit（00_mis_log.md）+ Edit（01_memo_index.txt末尾）
```

- **重複チェックは 01_memo_index.txt だけ読めばOK**（19,000行のGrep不要）
- 依存関係のないツール呼び出しは必ず同時実行する

---

# インデックスファイル（高速重複チェック）

## 📋 01_memo_index.txt とは

`01_memo.md` に書いた全エントリのキーワードを1行ずつまとめた小さなファイル。

- 自宅PC: `D:\50_knowledge\01_memo_index.txt`
- 会社PC: `C:\Users\guest04\Desktop\高橋研三\03_knowledge\01_memo_index.txt`

## 重複チェックの手順

1. `Read` で `01_memo_index.txt` を読み込む（数百行・一瞬で読める）
2. 今から書こうとしているトピックと**キーワードが似ている行**がないか確認する
3. 似ている行があれば → 既存エントリを更新（01_memo.md の該当箇所を Grep → Edit）
4. なければ → 末尾に追記（01_memo.md + 01_memo_index.txt 両方に追加）

## インデックスへの追記フォーマット（1行）

```
キーワード1 キーワード2 キーワード3 | タグ（HTML / WordPress / CSS など）
```

例：
```
margin-top: auto 直接のflex子 入れ子 親もflexにする | HTML CSS
wp_footer() JS body直前 wp_head() | WordPress
```

---

# 出力先（3ファイルに書き込み）

## 📘 詳細メモ（説明・仕組み）
存在する方に追記:
- 会社PC: `C:\Users\guest04\Desktop\高橋研三\03_knowledge\01_memo.md`
- 自宅PC: `D:\50_knowledge\01_memo.md`

## 📒 気づき・ミス（1行メモ）
存在する方に追記:
- 会社PC: `C:\Users\guest04\Desktop\高橋研三\03_knowledge\00_mis_log.md`
- 自宅PC: `D:\50_knowledge\00_mis_log.md`

## 📋 インデックス（重複チェック用）
- 会社PC: `C:\Users\guest04\Desktop\高橋研三\03_knowledge\01_memo_index.txt`
- 自宅PC: `D:\50_knowledge\01_memo_index.txt`

判定方法: Bash で `test -f` を実行し、存在する方を使用

> ⚡ **チェックは1会話に1回だけ**

---

# 書き込みルール

## 📘 01_memo.md（詳細メモ）
→ **memo-format スキルと同じフォーマット**で書く

- タイトルは「読んだだけで答えがわかる」レベル
- 【日付】【結論】【具体例】【補足】の構成
- カテゴリタグ（`WordPress` / `HTML`）を末尾に付ける
- 重複チェックは **01_memo_index.txt を Read するだけ**（01_memo.md 本体の Grep 不要）

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
