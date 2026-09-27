---
name: news-link
description: ニュースサイトを巡回し、全記事をカテゴリ別に要約・アフィリエイトリンク生成してNotionとreportフォルダに保存する。
---

# news_link コマンド

`.claude/skills/news_link.md` に定義された手順に従い、以下を実行する：

1. 対象ニュースサイト（8サイト）を巡回して記事リンクを収集する
2. 各記事を取得・カテゴリ判定・要約する（スキップなし・全記事対象）
3. 記事内容から商品購買につながるキーワードを3〜5個抽出し、アフィリエイト検索URLを生成する
4. Notion の `Claude Contents > news_link` に保存する（なければ自動作成）
5. Notion の `SNS投稿管理` データベースで、記事内容に関連する投稿テーマページ（10ページ）にニュース要約とアフィリエイトリンクを追記する
6. `report/` フォルダに Markdown ファイルとして保存する

引数にURLが指定された場合はそのURLのみを処理する。

## 対象サイト

- <https://newspicks.com/>
- <https://news.ycombinator.com/>
- <https://news.yahoo.co.jp/>
- <https://www.itmedia.co.jp/news/>
- <https://www.newsweekjapan.jp/>
- <https://www.designnews.com/>
- <https://dev.classmethod.jp/>
- <https://news.google.com/home?hl=ja&gl=JP&ceid=JP:ja>
