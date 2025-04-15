# Hugo Project

## 初めてのセットアップ

textlintを導入する。

```bash
npm install
```

## 基本的な使い方

### 新しい記事を作成する

以下のコマンドを実行して、新しい記事ファイルを作成します。

```bash
hugo new posts/your-article-name.md
```

`posts` の部分は、コンテンツの構成に合わせて変更してください（例： `blog`, `docs` など）。

### 開発サーバーを起動する

以下のコマンドを実行すると、ローカルで開発サーバーが起動し、ブラウザでプレビューできます。

```bash
hugo server -D
```

`-D` オプションは、ドラフト（draft: true）の記事も表示するために必要です。
