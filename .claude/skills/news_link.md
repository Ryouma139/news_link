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
/news-link <URL>
/news-link <URL1> <URL2> ...   # 複数URL対応
```

---

## 実行手順（Claude がこの順で実行する）

### Step 1: 記事を取得する

`WebFetch` ツールで各URLの記事本文を取得する。

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

各キーワードについて以下の検索URLを生成する：

```text
Amazon   : https://www.amazon.co.jp/s?k={keyword}&tag={AMAZON_AFFILIATE_TAG}
楽天市場 : https://search.rakuten.co.jp/search/mall/{keyword}/?R=1
Yahoo!   : https://shopping.yahoo.co.jp/search?p={keyword}
```

> `{AMAZON_AFFILIATE_TAG}` は環境変数 `AMAZON_TAG` があればそれを使用。
> 未設定の場合は `amazon-tag-placeholder` を挿入してレポートに注記する。

キーワードはURLエンコードして埋め込む（スペース→`+`）。

### Step 5: Notion への保存

`notion-create-pages` ツールを使用して、設定された Notion データベースにページを作成する。

**Notion データベースID の取得順序：**

1. 環境変数 `NOTION_NEWS_DB_ID` を確認
2. プロジェクトルートの `config.json` の `notion_db_id` を確認
3. いずれもない場合 → 保存をスキップし、レポートにその旨を記録

**Notion ページの構成：**

```text
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

| キーワード | Amazon | 楽天 | Yahoo! |
| --- | --- | --- | --- |
| {kw1} | [検索]({amazon_url1}) | [検索]({rakuten_url1}) | [検索]({yahoo_url1}) |
| {kw2} | [検索]({amazon_url2}) | [検索]({rakuten_url2}) | [検索]({yahoo_url2}) |

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

プロジェクトルートに以下の形式で `config.json` を置くと動作をカスタマイズできる：

```json
{
  "notion_db_id": "xxxx-xxxx-xxxx-xxxx",
  "amazon_tag": "your-affiliate-tag-22",
  "affiliate_platforms": ["amazon", "rakuten", "yahoo"],
  "report_dir": "report",
  "summary_max_chars": 400,
  "categories": ["経済", "テクノロジー", "政策", "国際", "社会", "企業"]
}
```

---

## エラーハンドリング

| 状況 | 対応 |
| --- | --- |
| URLが取得できない | スキップ・理由をレポートに記録 |
| ビジネス関連でない記事 | 除外・理由を記録 |
| Notion 未設定 | Notion保存スキップ・report保存は実行 |
| report/ フォルダ不在 | 自動作成して保存 |
| Amazon TAG 未設定 | プレースホルダーで出力・注記を追加 |
