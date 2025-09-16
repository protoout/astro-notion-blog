# AGENTS notes for astro-notion-blog

このリポジトリで作業するエージェント/開発者向けの運用メモです。リポジトリ全体に適用されます（root スコープ）。

## 変更履歴（重要な決定）

- 2025-09-16
  - ブログのルートを `/posts` から `/posts2` に変更。
  - ルーティング：`src/pages/posts` ディレクトリを `src/pages/posts2` に移行。
  - ハードコードされたパスを撤廃し、リンクはヘルパーに集約。
    - 変更した主なファイル:
      - `src/lib/blog-helpers.ts`: `getPostLink`, `getTagLink`, `getPageLink` を `/posts2` ベースに変更。
      - `src/pages/posts2/page/[page].astro`: `Layout` の `path` を `getPageLink(...)` を使う形へ。
      - `src/pages/posts2/tag/[tag].astro`: `Layout` の `path` を `getTagLink(tag)` に変更。
      - `src/pages/posts2/tag/[tag]/page/[page].astro`: `Layout` の `path` を `getPageLink(..., tag)` に変更。
  - Cloudflare Workers/Pages デプロイ設定を導入：
    - `wrangler.toml` を追加し、以下を設定。
      - `name = "protout-blog"`
      - `compatibility_date = "2025-09-16"`
      - `[assets] directory = "./dist"`
    - これによりデプロイコマンドは `npx wrangler deploy` で動作。

## コーディング規約（このプロジェクト特有）

- リンク生成は必ずヘルパーを使用すること。
  - `src/lib/blog-helpers.ts`: `getPostLink`, `getTagLink`, `getPageLink`, `getNavLink`, `getStaticFilePath`
  - テンプレート内で `"/posts2/..."` のような生文字列は直書きしない。
- `Layout` コンポーネントの `path` プロップには、ヘルパーから得たパス（`getPostLink`/`getTagLink`/`getPageLink`）を渡す。
- RSS/外部リンクは `src/pages/feed.ts` が `getPostLink` を利用している。ルート変更時はヘルパーだけ更新すれば RSS も追従する。
- サブディレクトリ配信（`BASE_PATH`）に対応するため、パス結合は `pathJoin(BASE_PATH, ...)` を使う実装に統一済み。BASE_PATH を無視する直結パスを追加しない。

## ルートを再度変更したい場合の手順

1. `src/pages` 配下のディレクトリ名を希望のルート名に変更（例: `posts2` → `blog`）。
2. `src/lib/blog-helpers.ts` の以下を新ルートに合わせて変更。
   - `getPostLink(slug)`
   - `getTagLink(tag)`
   - `getPageLink(page, tag)`
3. `rg "/posts\b|/posts/|/posts2\b|/posts2/" src` でハードコード漏れがないか確認。

## ローカル開発

- 必須環境変数（`.env` もしくはシェル環境）
  - `NOTION_API_SECRET`
  - `DATABASE_ID`
- 実行
  - `npm install`
  - `npm run dev`（http://localhost:4321）

## デプロイ（Cloudflare）

- Pages を静的ホスティングで使う場合（Workers 統合なし）
  - ビルドコマンド: `npm run build`
  - 出力ディレクトリ: `dist`
  - デプロイコマンド: 空で OK（Pages が `dist` を配信）

- Workers 統合でデプロイする場合（本リポジトリの現状）
  - ルートにある `wrangler.toml` を使用。
  - デプロイコマンド: `npx wrangler deploy`
  - 非本番用: `npx wrangler versions upload`
  - 必須: `compatibility_date` が指定されていること（本ファイルに定義済み）。
  - `assets.directory = ./dist` のため、`npm run build` 後に `dist` が必要。

- 環境変数の設定（重要）
  - Notion API 関連 (`NOTION_API_SECRET`, `DATABASE_ID`) は **Cloudflare Pages のプロジェクト設定（Production/Preview）** に追加する。Workers のみのシークレット設定では Pages のビルドには渡らない。

## トラブルシュート

- `@notionhq/client warn: API token is invalid.`
  - Pages 側の環境変数に `NOTION_API_SECRET` と `DATABASE_ID` が設定されているか。貼り付け時の余分な空白に注意。Notion Integration を対象 DB に共有しているか確認。
- Wrangler で `compatibility_date` が必要というエラー
  - `wrangler.toml` に `compatibility_date` を追加済み。手元でコマンドに付与する場合は `--compatibility-date=YYYY-MM-DD`。
- `--assets` が必要というエラー
  - `wrangler.toml` の `[assets] directory = "./dist"` で解消。未設定の場合は `npx wrangler deploy --assets=dist` を使用。

## チェックリスト

- [ ] 新規ページ/コンポーネントでリンク直書きしていないか（ヘルパー使用）。
- [ ] `rg "/posts\b|/posts/|/posts2\b|/posts2/" src` で意図しないハードコードがない。
- [ ] `npm run build` が通る。
- [ ] Cloudflare Pages の環境変数が Production/Preview 両方に入っている。

