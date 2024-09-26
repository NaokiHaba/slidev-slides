---
theme: seriph
background: https://source.unsplash.com/collection/94734566/1920x1080
class: text-center
highlighter: shiki
lineNumbers: false
info: |
  ## Vue 3.5: パフォーマンスと開発体験を革新する包括的アップデート
  開発者向けプレゼンテーションスライド
drawings:
  persist: false
transition: slide-left
title: Vue 3.5 アップデート概要
mdc: true
---

# Vue 3.5

パフォーマンスと開発体験を革新する包括的アップデート

<div class="abs-br m-6 flex gap-2">
  <a href="https://blog.vuejs.org/posts/vue-3-5" target="_blank" alt="Official Blog"
    class="text-xl slidev-icon-btn opacity-50 !border-none !hover:text-white">
    <carbon-blog />
  </a>
  <a href="https://github.com/vuejs/core" target="_blank" alt="GitHub"
    class="text-xl slidev-icon-btn opacity-50 !border-none !hover:text-white">
    <carbon-logo-github />
  </a>
</div>

---

# Vue 3.5 の主な新機能と改善点

- **Reactive Props Destructure**
- **新しい API の導入**
  - `useTemplateRef()`, `onWatcherCleanup()`, `useHost()`, `useShadowRoot()`
- **Deferred Teleport**
- **Reactivity System の大幅な改善**
- **SSRの改善** (Lazy Hydration, `useId()`, `data-allow-mismatch`)
- **Custom Elements の機能強化**

これらの機能により、開発効率とアプリケーションのパフォーマンスが大幅に向上します。

---

# Reactive Props Destructure

Vue 3.5 以前：

```vue
<script setup lang="ts">
import { computed } from 'vue'

const props = withDefaults(defineProps<{
  count?: number
}>(), {
  count: 0
})

const double = computed(() => props.count * 2)
</script>

<template>
  <div>Count: {{ props.count }}</div>
  <div>Double: {{ double }}</div>
</template>
```

---

# Reactive Props Destructure (続き)

Vue 3.5 以降：

```vue
<script setup lang="ts">
import { computed } from 'vue'

const { count = 0 } = defineProps<{
  count?: number
}>()

const double = computed(() => count * 2)
</script>

<template>
  <div>Count: {{ count }}</div>
  <div>Double: {{ double }}</div>
</template>
```

この新機能により、コードがより簡潔になり、可読性が向上します。
プロパティの分割代入とデフォルト値の設定が一箇所で行えるようになりました。

[Reactive Props Destructure RFC](https://github.com/vuejs/rfcs/discussions/502)

[詳細な説明記事](https://zenn.dev/comm_vue_nuxt/articles/reactive-props-destructure)

---

# 新しい API: useTemplateRef()

使用例：

```vue
<script setup>
import { useTemplateRef } from 'vue'

const inputRef = useTemplateRef('input')

// inputRef.value にアクセスして操作可能
</script>

<template>
  <input ref="input">
</template>
```

主な利点：
- 動的に変更される ID への `ref` バインディングがサポート
- より直感的な参照の取得方法
- TypeScript との相性が良い
- コンポーネントのライフサイクルと連動

---

# 新しい API: onWatcherCleanup()

使用例：

```javascript
import { watch, onWatcherCleanup } from 'vue'

watch(id, (newId) => {
  const controller = new AbortController()

  fetch(`/api/${newId}`, { signal: controller.signal }).then(() => {
    // コールバックロジック
  })

  onWatcherCleanup(() => {
    // 古いリクエストを中止
    controller.abort()
  })
})
```

`onWatcherCleanup()` を使用することで、ウォッチャー内でのリソースのクリーンアップが
より簡単かつ確実になります。これは特に非同期処理を含むウォッチャーで有用です。

---

# Deferred Teleport

使用例：

```vue
<Teleport defer to="#container">
  <ComplexComponent />
</Teleport>
<div id="container"></div>
```

#### 特徴：
- 同じ更新サイクル内の他のDOMコンテンツがレンダリングされるまで待機
- ターゲットコンテナを探して子要素をマウント
- `<Suspense>` 内での使用も可能

#### ユースケース：
- 動的に生成されるターゲット要素への転送
- パフォーマンス最適化
- 複雑なレイアウト構造の管理

Deferred Teleportにより、より柔軟なコンポーネント構造の管理が可能になります。

---

# Reactivity System の改善

主な改善点：

1. **アーキテクチャの刷新**
   - バージョンカウンティングの導入
   - 双方向リンクリスト構造の採用

2. **メモリ使用量の最適化**
   - 56%のメモリ削減を実現
   - 遅延サブスクリプションの導入

3. **コンピューテッドプロパティの改善**
   - 常に最新の値を保持
   - SSRでの最適化

これらの改善により、特に大規模アプリケーションでのパフォーマンスが大幅に向上します。

---

# Reactivity System の改善 (続き)

期待される効果：
- 大規模アプリケーションでのパフォーマンス向上
- メモリ使用量の大幅な削減
- より効率的な更新処理

具体的な改善例：
- 複雑なデータ構造を持つアプリケーションでの反応性の向上
- 多数のコンピューテッドプロパティを持つコンポーネントの処理速度向上
- SSRでのパフォーマンス改善、特に大規模なアプリケーションでの効果が顕著

[Reactivity System 改善 ](https://github.com/vuejs/core/pull/10397)

[Reactivity System 改善 PR](https://github.com/vuejs/core/pull/9511)

---

# SSRの改善: Lazy Hydration

Lazy Hydration 戦略：

1. **Hydrate on Idle** (アイドル時にハイドレーション)
2. **Hydrate on Visible** (表示時にハイドレーション)
3. **Hydrate on Media Query** (メディアクエリ一致時にハイドレーション)
4. **Hydrate on Interaction** (インタラクション時にハイドレーション)
5. **Custom Strategy** (カスタム戦略)

これらの戦略により、初期ページロードの最適化と、ユーザーエクスペリエンスの向上が可能になります。

---

# SSRの改善: Lazy Hydration (続き)

使用例：

```javascript
import { defineAsyncComponent, hydrateOnIdle, hydrateOnVisible } from 'vue'

const IdleComp = defineAsyncComponent({
  loader: () => import('./HeavyComponent.vue'),
  hydrate: hydrateOnIdle(5000) // 最大5秒待機
})

const VisibleComp = defineAsyncComponent({
  loader: () => import('./LazyLoadedComponent.vue'),
  hydrate: hydrateOnVisible({ rootMargin: '200px' })
})
```

これらの戦略を適切に使用することで、初期ページロード時間を短縮し、
必要なときにのみコンポーネントをハイドレーションすることができます。

[Lazy Hydration 詳細](https://vueschool.io/articles/vuejs-tutorials/lazy-hydration-and-server-components-in-nuxt-vue-js-3-performance/)

---

# SSRの改善: useId() と data-allow-mismatch

## useId()

サーバーサイドとクライアントサイドで一貫した一意のIDを生成するためのAPI。

使用例：
```javascript
import { useId } from 'vue'

const id = useId()
```

## data-allow-mismatch

SSRとクライアントサイドのハイドレーション間の不一致警告を抑制するための属性。

使用例：
```html
<span data-allow-mismatch>{{ data.toLocaleString() }}</span>
```

これらの機能により、SSRアプリケーションの開発がより柔軟かつ堅牢になります。

---

# Custom Elements の改良

主な改善点：

1. **アプリケーション設定の柔軟性向上**
   - `configureApp` オプションの導入

2. **新 API の追加**
   - `useHost()`, `useShadowRoot()`, `this.$host`

3. **`shadowRoot` オプションの追加**

4. **セキュリティ強化：nonce サポート**

これらの改善により、Vue.jsをより広範囲のシナリオで活用できるようになりました。

---

# Custom Elements の改良 (続き)

使用例：

```javascript
import MyElement from './MyElement.ce.vue'

defineCustomElement(MyElement, {
  shadowRoot: false,  // シャドウDOMを使用しない
  nonce: 'xxx',
  configureApp(app) {
    app.config.errorHandler = // エラーハンドラーの設定
  }
})
```

これらの新機能により、カスタム要素の作成と管理がより柔軟になり、
セキュリティも向上しています。特に、`shadowRoot` オプションにより、
シャドウDOMを使用しないカスタム要素の作成が可能になりました。

[Custom Elements の改良詳細](https://zenn.dev/comm_vue_nuxt/articles/improvements-to-custom-elements-in-vue3-5)

---

# まとめ

- Vue 3.5は、パフォーマンスと開発体験を大幅に向上
- 新機能の導入と既存機能の改善により、より柔軟なアプリケーション開発が可能に
- Reactivity Systemの改善により、大規模アプリケーションでの性能が向上
- SSRとLazy Hydrationの改善で、初期ロード時間を最適化
- Custom Elementsの強化で、より広範囲なシナリオでVueを活用可能に

これらの改善により、Vue.jsはより強力で効率的なフレームワークとなりました。
大規模アプリケーションの開発者や、パフォーマンスに敏感なプロジェクトに
特に大きな恩恵をもたらすでしょう。

Vue 3.5 を自身のプロジェクトに導入して、これらの新機能や改善点を体験してみましょう！

[Vue 3.5 リリースノート](https://blog.vuejs.org/posts/vue-3-5)

---
layout: center
class: text-center
---

# ありがとうございました！

[公式ブログ](https://blog.vuejs.org/posts/vue-3-5) · [GitHub リポジトリ](https://github.com/vuejs/core) · [Vue.js 公式ドキュメント](https://vuejs.org/)
