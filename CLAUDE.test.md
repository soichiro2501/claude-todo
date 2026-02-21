# テストガイドライン

テストコード作成時にClaudeが従うべきルールを定義します。

---

## テスト構成

| 種別 | ツール | 配置場所 |
|------|--------|----------|
| ユニットテスト | Vitest | `tests/unit/` |
| コンポーネントテスト | Vitest + Vue Test Utils | `tests/components/` |
| E2Eテスト | Playwright | `tests/e2e/` |

---

## 基本方針

- テストは実装コードと同じディレクトリ構成を反映させる
- テストファイル名は対象ファイル名に `.spec.ts` / `.test.ts` を付ける
- 1テストケースは1つのことだけ検証する
- テストはお互いに独立させる（実行順序に依存しない）
- モックは必要最小限にとどめる

---

## Vitestルール

### 基本構造

```ts
import { describe, it, expect, beforeEach, vi } from 'vitest'

describe('対象モジュール名', () => {
  beforeEach(() => {
    // 各テスト前のセットアップ
  })

  it('〜したとき、〜になること', () => {
    // Arrange
    // Act
    // Assert
    expect(actual).toBe(expected)
  })
})
```

### テスト名の書き方
- 日本語で「〜したとき、〜になること」の形式で書く
- 何をテストしているか誰でも読んでわかるようにする

### Composablesのテスト

```ts
import { renderComposable } from '@test/utils' // プロジェクトで定義するユーティリティ

describe('useCounter', () => {
  it('初期値が0であること', () => {
    const { count } = renderComposable(() => useCounter())
    expect(count.value).toBe(0)
  })

  it('incrementを呼んだとき、countが1増えること', () => {
    const { count, increment } = renderComposable(() => useCounter())
    increment()
    expect(count.value).toBe(1)
  })
})
```

### モック

```ts
// APIモック例
vi.mock('#app', () => ({
  useFetch: vi.fn().mockResolvedValue({ data: ref({ id: 1 }), error: ref(null) })
}))

// 各テスト後にモックをリセット
afterEach(() => {
  vi.clearAllMocks()
})
```

---

## Vue Test Utils ルール

### コンポーネントテストの基本

```ts
import { mount } from '@vue/test-utils'
import MyComponent from '@/components/MyComponent.vue'

describe('MyComponent', () => {
  it('propsが正しく表示されること', () => {
    const wrapper = mount(MyComponent, {
      props: { title: 'テスト' }
    })
    expect(wrapper.text()).toContain('テスト')
  })

  it('ボタンクリック時にイベントが発火すること', async () => {
    const wrapper = mount(MyComponent)
    await wrapper.find('button').trigger('click')
    expect(wrapper.emitted('update')).toBeTruthy()
  })
})
```

### セレクタの優先順位
1. `data-testid` 属性（テスト専用セレクタ）
2. WAI-ARIAロール（`getByRole`）
3. テキスト内容
4. クラス名（最終手段）

```vue
<!-- 実装側で付与する -->
<button data-testid="submit-button">送信</button>
```

---

## Playwright (E2E) ルール

### 基本構造

```ts
import { test, expect } from '@playwright/test'

test.describe('ユーザー登録フロー', () => {
  test('正常系: フォームを入力して登録できること', async ({ page }) => {
    await page.goto('/register')
    await page.getByLabel('メールアドレス').fill('test@example.com')
    await page.getByRole('button', { name: '登録' }).click()
    await expect(page).toHaveURL('/dashboard')
  })
})
```

### E2Eテストの方針
- ユーザーの操作フローに沿ったテストを書く
- 内部実装の詳細には依存しない
- テストデータは `tests/e2e/fixtures/` に配置する
- テスト実行後にデータをクリーンアップする

---

## カバレッジ目標

- ユニット・コンポーネントテスト: ステートメントカバレッジ 80% 以上
- ビジネスロジックを含むComposables: 90% 以上
- E2Eは主要ユーザーフローを網羅する
