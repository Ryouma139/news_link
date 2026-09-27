# news_link

ニュースサイトを巡回してビジネス・社会情勢記事をピックアップ・要約し、
アフィリエイトリンクを生成して Notion と `report/` フォルダに保存する Claude Code スキル。

## セットアップ

1. `config.example.json` を `config.json` にコピーして値を設定する
2. Notion MCP が必要な場合は `claude mcp` で接続する
3. `/news-link` を実行する

```bash
cp config.example.json config.json
# config.json を編集して notion_db_id などを設定
```

## 設定ファイル

| ファイル | 用途 | Git管理 |
| --- | --- | --- |
| `config.json` | 個人設定・アフィリエイトID（クラウド実行で読むためコミットする） | 含む |
| `config.example.json` | 設定テンプレート | 含む |
| `report/` | 生成されたレポート | 除外（.gitignore） |

## 対象ニュースサイト

- NewsPicks
- Hacker News
- Yahoo!ニュース
- ITmedia NEWS
- Newsweek日本版
- Design News
- DevelopersIO
- Google ニュース（日本）

## アフィリエイトプラットフォーム

- 楽天アフィリエイト
- Amazon
- A8.net

## スキル

- `.claude/skills/news_link.md` — メインスキル定義
