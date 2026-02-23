# 実装ガイドライン

コード実装時にClaudeが従うべきルールを定義します。

---

## 基本方針

- 要求された変更のみを行う。過剰なリファクタリング・機能追加は禁止
- 既存のコードスタイルと一貫性を保つ
- 実装前に関連ファイルを必ず読む
- セキュリティリスク（XSS, CSRF, SQLインジェクション等）を導入しない

---

## 設計原則

### YAGNI / KISS
- YAGNI（You Aren't Gonna Need It）: 現時点で不要な機能・抽象化を作らない
- KISS（Keep It Simple, Stupid）: シンプルな実装を優先し、不必要な複雑さを持ち込まない

### DRY（Don't Repeat Yourself）
- 同じロジックを複数箇所に書かない
- 重複するロジックは Composables やユーティリティ関数に切り出す
- 重複する定数はファイルに集約する

---

## Vue / Nuxt 実装ルール

### Composition API
- `<script setup lang="ts">` を使用する（Options APIは使わない）
- `defineProps` / `defineEmits` に型を明示する

```vue
<script setup lang="ts">
const props = defineProps<{
  userId: string
  isActive?: boolean
}>()

const emit = defineEmits<{
  update: [value: string]
  close: []
}>()
</script>
```

### Reactivity
- プリミティブ値には `ref()`、オブジェクト・配列には `reactive()` を使う
- `ref` の値にアクセスする際は `.value` を忘れない
- `computed` は副作用を持たせない

### Composables
- 単一責任の原則に従い、1つのComposableに1つの関心事をまとめる
- 戻り値はオブジェクト形式（分割代入しやすくする）

```ts
// Good
export function useCounter() {
  const count = ref(0)
  const increment = () => count.value++
  return { count, increment }
}
```

### コンポーネント設計
- Props は読み取り専用。コンポーネント内で直接変更しない（単方向データフロー: props down, events up）
- ロジックは Composables に切り出し、テンプレートは表示のみに集中させる

### Nuxt固有
- データフェッチには `useFetch` / `useAsyncData` を使う（クライアント側の `fetch` は直接使わない）
- SEOメタは `useSeoMeta` / `useHead` で設定する
- 環境変数は `useRuntimeConfig` で参照する（`process.env` は直接使わない）
- ミドルウェアは `app/middleware/` に配置し、`defineNuxtRouteMiddleware` を使う

### SSR安全性
- `document` / `window` へのアクセスは `onMounted` か `import.meta.client` で保護する
- SSR安全なグローバル状態には `useState()` を使う（`ref()` をモジュールスコープに置かない）
- 内部遷移には `<NuxtLink>` を使う（`<a>` タグは外部リンクのみ）

### エラーハンドリング
- サーバーエラーは `createError` でラップして返す
- クライアントでは `useError` / `clearError` を活用する
- try/catchを使う場合、catchブロックで握りつぶさない

---

## セキュリティ（実装時）

- `v-html` の使用は原則禁止（XSSリスク）。使用する場合はコメントで理由を明記し、入力値を必ずサニタイズする
- `localStorage` / `sessionStorage` にトークン等の機密情報を保存しない

---

## TypeScript 実装ルール

### 型定義
- グローバルな型は `app/types/` に配置する
- API レスポンスの型は `app/types/api/` に配置する
- Zodなどのバリデーションライブラリを使う場合、スキーマから型を推論する
- Non-null assertion（`!`）は原則禁止。`?.` オプショナルチェーンか明示的な `null` チェックで代替する
- 型キャスト（`as`）を使う場合はコメントで理由を明記する

### 非同期処理
- `async/await` を使う（`.then().catch()` チェーンは避ける）
- `Promise.all` で並列実行できる処理はまとめる

---

## パフォーマンス

- 重い処理は `useLazyFetch` / `lazy: true` で遅延ロードを検討する
- コンポーネントの動的インポートは `defineAsyncComponent` を使う
- `v-for` には必ず `:key` を指定する（インデックスは最終手段）
- 不要な `watchEffect` / `watch` は作らない

---

## Tailwind CSS v4 + shadcn/ui 実装ルール

### Tailwind CSS v4
- テーマのカスタマイズはCSSファイル内の `@theme` ディレクティブで行う（`tailwind.config.ts` は使わない）
- カスタムカラーや変数は `app/assets/css/main.css` の `@theme` ブロックに定義する
- ユーティリティクラスは公式推奨順（レイアウト → ボックスモデル → タイポグラフィ → 見た目）で並べる

### shadcn/ui
- コンポーネントは `app/components/ui/` に配置する（shadcnのデフォルトに合わせる）
- コンポーネントの追加は `pnpm dlx shadcn@latest add <component>` を使う
- カスタマイズはコンポーネントファイルを直接編集する（ラッパーを作らない）
- クラスの結合には `cn()` ユーティリティ（`clsx` + `tailwind-merge`）を使う

```ts
import { cn } from '@/lib/utils'

// Good
const classes = cn('base-class', isActive && 'active-class', props.class)
```
