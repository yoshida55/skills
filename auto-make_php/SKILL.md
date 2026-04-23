# auto-make_php スキル

WordPressテーマのPHPファイルを作成するときの標準フロー。

## 対象ファイルの種類

| ファイル | 説明 |
|---|---|
| `page-{スラッグ}.php` | 固定ページ用テンプレート |
| `archive.php` / `archive-{スラッグ}.php` | 投稿一覧ページ |
| `single.php` / `single-{スラッグ}.php` | 投稿詳細ページ |
| `category-{スラッグ}.php` | カテゴリーアーカイブ |

---

## 実行時の最初のアクション

会話からページ名・HTMLパスが読み取れる場合は質問をスキップして即作業に入る。
読み取れない場合のみ以下を聞く：

1. **「どのページのPHPファイルを作りますか？（例：works / contact / news）」**
2. **「元のHTMLファイルのパスを教えてください」**

---

## 標準フロー（固定ページの場合）

AIが担当する作業：

```
① HTMLのセクション部分をコピーして page-{name}.php を作成
   - get_header() を先頭に追加
   - get_footer() を末尾に追加
   - 画像パス・テキスト等の変換はユーザーが行うのでそのままにする

② css/page-{name}.css を新規作成（空ファイル）
   - 例：page-works.php → css/page-works.css
   - フォルダ確認不要・そのまま Write で作成する

③ functions.php に wp_enqueue_style() を追記
   - 必ず先に Read してから Edit する（Read なしで Edit するとエラーになる）
```

ユーザーが担当する作業：
- 画像パスを `get_theme_file_uri()` に変換
- その他PHPへの変換作業

```
④ 管理画面で固定ページを作成
   - 固定ページ → 新規追加
   - スラッグをPHPファイル名と一致させる（例: page-about.php → スラッグ: about）
   - 公開する
```

## newsなど投稿アーカイブの場合

④ の「固定ページ作成」の代わりに：
- 管理画面 → 投稿 → 新規追加 で記事を登録する
- `archive.php` または `category-news.php` がテンプレートとして使われる

---

## ⚠ 注意点

- PHPファイルを作っただけではページは存在しない → 必ず管理画面でコンテンツを作る
- スラッグはPHPファイル名と完全に一致させること
- デバッグ用の `<?php echo basename(__FILE__); ?>` は本番前に必ず削除する
