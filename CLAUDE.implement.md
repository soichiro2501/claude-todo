# 実装ガイドライン

コード実装時にClaudeが従うべきルールを定義します。

---

## 基本方針

- 要求された変更のみを行う。過剰なリファクタリング・機能追加は禁止
- 既存のコードスタイルと一貫性を保つ
- 実装前に関連ファイルを必ず読む
- セキュリティリスク（XSS, CSRF, SQLインジェクション等）を導入しない

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

### Nuxt固有
- データフェッチには `useFetch` / `useAsyncData` を使う（クライアント側の `fetch` は直接使わない）
- SEOメタは `useSeoMeta` / `useHead` で設定する
- 環境変数は `useRuntimeConfig` で参照する（`process.env` は直接使わない）
- ミドルウェアは `app/middleware/` に配置し、`defineNuxtRouteMiddleware` を使う

### エラーハンドリング
- サーバーエラーは `createError` でラップして返す
- クライアントでは `useError` / `clearError` を活用する
- try/catchを使う場合、catchブロックで握りつぶさない

---

## TypeScript 実装ルール

### 型定義
- グローバルな型は `app/types/` に配置する
- API レスポンスの型は `app/types/api/` に配置する
- Zodなどのバリデーションライブラリを使う場合、スキーマから型を推論する

### 非同期処理
- `async/await` を使う（`.then().catch()` チェーンは避ける）
- `Promise.all` で並列実行できる処理はまとめる

---

## パフォーマンス

- 重い処理は `useLazyFetch` / `lazy: true` で遅延ロードを検討する
- コンポーネントの動的インポートは `defineAsyncComponent` を使う
- `v-for` には必ず `:key` を指定する（インデックスは最終手段）
- 不要な `watchEffect` / `watch` は作らない
