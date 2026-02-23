# CLAUDE.md

このファイルはClaudeがこのリポジトリで作業する際のガイドラインです。
役割別の詳細は各ファイルを参照してください。

@CLAUDE.implement.md
@CLAUDE.test.md
@CLAUDE.review.md

---

## プロジェクト概要

- **フレームワーク**: Nuxt 4
- **言語**: TypeScript（strict mode）
- **スタイル**: Tailwind CSS v4 + shadcn/ui
- **パッケージマネージャ**: pnpm

---

## 共通ルール

### コミュニケーション
- 日本語で回答する
- コード・ファイル名・技術用語は英語のまま使用する

### コードスタイル
- インデント: スペース2つ
- 文字列: シングルクォート優先
- セミコロン: なし（Nuxt/Viteの慣習に従う）
- 行末の空白: 禁止

### TypeScript
- `strict: true` を前提とする
- `any` は原則禁止。使用する場合はコメントで理由を明記
- 型推論できる場合は明示的な型注釈を省略してよい
- `interface` より `type` を優先（拡張が必要な場合は `interface`）

### Nuxt 4の慣習
- `app/` ディレクトリ構成（Nuxt 4のデフォルト）を使用
- Composablesは `app/composables/` に配置
- ページは `app/pages/` に配置
- コンポーネントは `app/components/` に配置
- サーバーコードは `server/` に配置

### ファイル命名規則
- コンポーネント: PascalCase（例: `UserCard.vue`）
- Composables: camelCase、`use` プレフィックス（例: `useAuth.ts`）
- ページ: kebab-case（例: `user-profile.vue`）
- ユーティリティ: camelCase（例: `formatDate.ts`）

### Git
- コミットメッセージは日本語でも英語でもよい
- PRを作成するまでコミットを求めない
