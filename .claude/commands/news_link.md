---
name: news-link
description: 指定ニュースURLからビジネス・社会情勢記事を取得・要約し、関連キーワードのアフィリエイト検索URLを生成してNotionとreportフォルダに保存する。
---

# news_link スキル

## 概要

ニュース記事のURLを受け取り、以下を自動実行する：

1. 記事を取得し、ビジネス・社会情勢に関連するものをピックアップ
2. 日本語で要約（200〜400字）
3. 記事から3〜5個のキーワードを抽出し、アフィリエイト検索URLを生成
4. Notion データベースに保存
5. `report/` フォルダに Markdown ファイルとして保存

---

## 使い方

```text
/news-link          # 全サイトから取得
/news-link <URL>    # 特定URLのみ（デフォルトリストを上書き）
```

---

## 対象ニュースサイト（固定）

以下のサイトを毎回巡回する：

| # | サイト | URL |
| --- | --- | --- |
| 1 | NewsPicks | <https://newspicks.com/> |
| 2 | Hacker News | <https://news.ycombinator.com/> |
| 3 | Yahoo!ニュース | <https://news.yahoo.co.jp/> |
| 4 | ITmedia NEWS | <https://www.itmedia.co.jp/news/> |
| 5 | Newsweek日本版 | <https://www.newsweekjapan.jp/> |
| 6 | Design News | <https://www.designnews.com/> |
| 7 | DevelopersIO | <https://dev.classmethod.jp/> |
| 8 | Google ニュース（日本） | <https://news.google.com/home?hl=ja&gl=JP&ceid=JP:ja> |

引数でURLを指定した場合はそのURLのみを処理し、上記リストは使用しない。

---

## 実行手順（Claude がこの順で実行する）

### Step 1: 記事を取得する

`WebFetch` ツールで各サイトのトップページを取得し、記事リンクを収集する。
各サイトから最大**5件**の記事リンクを抽出し、それぞれの記事本文を取得する。

取得できない場合（ペイウォール・タイムアウトなど）は、その旨をレポートに記録してスキップ。

### Step 2: ビジネス・社会情勢フィルタリング

以下のカテゴリに該当する内容を「対象記事」と判定する：

- 経済・金融・市場・為替・株価
- 企業動向・M&A・決算・スタートアップ
- 政策・規制・法改正・行政
- AI・テクノロジー・DX・サイバーセキュリティ
- 社会問題・人口・労働・医療・環境
- 国際情勢・貿易・地政学リスク

上記に**まったく関係しない**（スポーツ・芸能・グルメなど）は除外し、除外理由を記録する。

### Step 3: 要約の生成

対象記事ごとに以下の形式で要約する：

```text
【タイトル】記事タイトル
【カテゴリ】経済 / テクノロジー / 政策 / 国際 / 社会 / 企業 （1つ選択）
【要約】200〜400字の日本語要約。
　　　  ・何が起きたか（Fact）
　　　  ・なぜ重要か（Why it matters）
　　　  ・今後の注目点（What to watch）
【出典URL】元URL
【取得日時】YYYY-MM-DD HH:mm
```

### Step 4: キーワード抽出 & アフィリエイト検索URL生成

要約から**ビジネス・商品購買に結びつきやすいキーワード**を3〜5個抽出する。

抽出基準：

- 具体的な商品・サービス名（例：「生成AI」「太陽光パネル」「EV」）
- 課題解決に関連するツール・書籍カテゴリ（例：「リスキリング」→ オンライン講座・書籍）
- 投資・節約・副業など金融行動を促すテーマ

各キーワードについて以下のアフィリエイト検索URLを生成する：

```text
楽天  : https://affiliate.rakuten.co.jp/search?sitem={keyword}&l-id=af_header_cta_search
Amazon: https://www.amazon.co.jp/s?k={keyword}&crid=3LMLYI7X0CA62&sprefix={keyword}%2Caps%2C273&ref=nb_sb_ss_saint-jp-refocus-candidate_9_1
A8.net: https://media-console.a8.net/program/search/keyword?keywords={keyword}&pageNo=1&pageSize=20&sortKey=NORMAL
```

キーワードはURLエンコードして埋め込む（スペース→`+`、日本語は `%XX` 形式）。

### Step 5: Notion への保存

保存先は Notion ワークスペース内の **`Claude Contents` > `news_link`** とする。

#### 5-1: 保存先ページの確認・作成

以下の順で保存先を確認し、存在しない場合は作成する：

1. `notion-search` で `Claude Contents` ページを検索する
   - 見つかった場合 → そのページIDを `parent_id` として記録
   - 見つからない場合 → `notion-create-pages` でワークスペースルートに `Claude Contents` ページを作成し、そのIDを記録

2. `Claude Contents` の子ページから `news_link` を検索する
   - 見つかった場合 → そのページIDを `news_link_parent_id` として記録
   - 見つからない場合 → `notion-create-pages` で `Claude Contents` 配下に `news_link` ページを作成し、IDを記録

3. 取得した `news_link_parent_id` を `config.json` の `notion_db_id` に自動書き込みして次回以降はスキップできるようにする

#### 5-2: 記事ページの作成

`notion-create-pages` で `news_link` ページ配下に記事ページを作成する。

```text
保存先    : Claude Contents / news_link / {記事タイトル}
タイトル  : 記事タイトル（最大100文字）
プロパティ:
  - カテゴリ (select)  : Step3で判定したカテゴリ
  - 取得日 (date)      : 実行日（YYYY-MM-DD）
  - 出典URL (url)      : 元記事URL
  - 評価 (select)      : "未読" （デフォルト）
本文      :
  ## 要約
  [Step3の要約本文]

  ## アフィリエイトキーワード
  [キーワード一覧とリンク表]
```

### Step 6: report フォルダへの保存

プロジェクトルート（スキルが呼ばれたディレクトリ）の `report/` フォルダに
Markdown ファイルを作成する。

**ファイル名：** `YYYYMMDD_HHMMSS_{連番}.md`（同一実行の複数記事は連番で区別）

**ファイル内容テンプレート：**

```markdown
# {記事タイトル}

- **カテゴリ**: {カテゴリ}
- **取得日時**: {YYYY-MM-DD HH:mm}
- **出典**: {URL}
- **Notion**: {NotionページURL（取得できた場合）}

---

## 要約

{200〜400字の要約}

---

## 関連キーワード & アフィリエイトリンク

| キーワード | 楽天 | Amazon | A8.net |
| --- | --- | --- | --- |
| {kw1} | [検索]({rakuten_url1}) | [検索]({amazon_url1}) | [検索]({a8_url1}) |
| {kw2} | [検索]({rakuten_url2}) | [検索]({amazon_url2}) | [検索]({a8_url2}) |

---

*生成: claude news-link スキル / {実行日時}*
```

---

## 実行後の出力（ターミナル）

処理が終わったら以下の形式でサマリを表示する：

```text
処理完了: {n} 件中 {m} 件を保存

[1] {タイトル（短縮）}
    カテゴリ: {カテゴリ}
    キーワード: {kw1}, {kw2}, {kw3}
    report/{ファイル名}
    Notion: {ページURL or "スキップ（DB未設定）"}

[2] ...

スキップ: {スキップ件数} 件
  - {URL}: {理由}
```

---

## 設定（config.json）

`config.example.json` をコピーして `config.json` を作成し、各値を設定する。
`config.json` は `.gitignore` で除外されるため GitHub に公開されない。

```bash
cp config.example.json config.json
```

読み込み優先順：`config.json` → 環境変数 → デフォルト値

| キー | 説明 | 必須 |
| --- | --- | --- |
| `notion_db_id` | Notion データベースID | Notion保存時のみ |
| `affiliate_ids.amazon_tag` | Amazon アフィリエイトタグ | 任意 |
| `affiliate_ids.rakuten_affiliate_id` | 楽天アフィリエイトID | 任意 |
| `affiliate_ids.a8net_media_id` | A8.net メディアID | 任意 |
| `report_dir` | レポート保存先フォルダ | 任意（デフォルト: `report`） |
| `summary_max_chars` | 要約の最大文字数 | 任意（デフォルト: 400） |
| `max_articles_per_site` | 1サイトあたりの最大取得記事数 | 任意（デフォルト: 5） |

---

## エラーハンドリング

| 状況 | 対応 |
| --- | --- |
| URLが取得できない | スキップ・理由をレポートに記録 |
| ビジネス関連でない記事 | 除外・理由を記録 |
| `Claude Contents` が存在しない | 自動作成してから保存 |
| `news_link` ページが存在しない | 自動作成してから保存 |
| Notion MCP が未接続 | Notion保存スキップ・report保存は実行 |
| report/ フォルダ不在 | 自動作成して保存 |
