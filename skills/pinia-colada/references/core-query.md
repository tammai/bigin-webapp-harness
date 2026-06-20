# Core Query API

## useQuery(options)

```ts
const {
  state,             // ComputedRef<DataState> — discriminated union { status, data, error }
  data,              // ShallowRef<TData | undefined>
  error,             // ShallowRef<TError | null>
  status,            // ShallowRef<'pending' | 'success' | 'error'>
  asyncStatus,       // ComputedRef<'idle' | 'loading'>
  isPending,         // ComputedRef<boolean>
  isLoading,         // ShallowRef<boolean>
  isPlaceholderData, // ComputedRef<boolean>
  refresh,           // () => Promise<DataState>  — only fetches if stale
  refetch,           // () => Promise<DataState>  — always fetches
} = useQuery(options)
```

### Options

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `key` | `MaybeRefOrGetter<EntryKey>` | required | Array of serializable values |
| `query` | `(ctx: { signal: AbortSignal }) => Promise<TData>` | required | Fetcher function |
| `staleTime` | `number` | `5000` | ms before data is considered stale |
| `gcTime` | `number \| false` | `300_000` | ms before unused entry is garbage-collected |
| `enabled` | `MaybeRefOrGetter<boolean>` | `true` | Set to ref/getter to pause query |
| `refetchOnMount` | `MaybeRefOrGetter<boolean \| 'always'>` | `true` | |
| `refetchOnWindowFocus` | `MaybeRefOrGetter<boolean \| 'always'>` | `true` | |
| `refetchOnReconnect` | `MaybeRefOrGetter<boolean \| 'always'>` | `true` | |
| `initialData` | `() => TDataInitial` | — | Pre-populates cache, sets status to `'success'` |
| `initialDataUpdatedAt` | `number \| (() => number)` | `Date.now()` | Staleness of `initialData` |
| `placeholderData` | `TData \| ((prev, entry) => TData)` | — | Shown while loading, doesn't mutate cache |
| `meta` | `MaybeRefOrGetter<QueryMeta>` | — | Arbitrary serializable metadata for plugins |
| `ssrCatchError` | `boolean` | `false` | Catch errors during SSR instead of throwing |

### Basic example

```vue
<script setup lang="ts">
import { useQuery } from '@pinia/colada'

const { data: todos, isPending, error } = useQuery({
  key: ['todos'],
  query: () => fetch('/api/todos').then(r => r.json()),
})
</script>

<template>
  <div v-if="isPending">Loading…</div>
  <div v-else-if="error">{{ error.message }}</div>
  <ul v-else>
    <li v-for="todo in todos" :key="todo.id">{{ todo.text }}</li>
  </ul>
</template>
```

### Dynamic keys (reactive params)

```ts
const route = useRoute()

const { data: contact } = useQuery({
  // getter function — re-runs when route.params.id changes
  key: () => ['contacts', route.params.id],
  query: ({ signal }) => fetchContact(route.params.id, { signal }),
})
```

### Conditional / dependent queries

```ts
const { data: user } = useQuery({
  key: ['user', userId],
  query: () => fetchUser(userId.value),
  enabled: () => !!userId.value,   // pauses until userId is defined
})
```

---

## defineQuery(optionsOrSetup) — reusable global queries

All components calling `defineQuery` share one reactive instance. Use it when multiple
components mount simultaneously with the same key, or when the query needs shared local state.

```ts
// Simple form — static options
export const useTodoList = defineQuery({
  key: ['todos'],
  query: () => fetch('/api/todos').then(r => r.json()),
})

// Setup function form — dynamic, shared local state
export const useFilteredTodos = defineQuery(() => {
  const search = ref('')
  const { data, ...rest } = useQuery({
    key: () => ['todos', { search: search.value }],
    query: () => fetch(`/api/todos?filter=${search.value}`).then(r => r.json()),
  })
  return { ...rest, todoList: data, search }
})

// In any component
const { todoList, search } = useFilteredTodos()
```

**Nuxt caveat:** Inside `defineQuery`, import `useRoute` from `vue-router`, not Nuxt's
auto-import — otherwise the query over-triggers or is undefined server-side.

**SSR caveat:** `ref()` state inside `defineQuery` is NOT serialized. For SSR-safe shared
state, put refs in a Pinia store or Nuxt's `useState()`.

---

## defineQueryOptions(optionsOrFn) — typed key factories

Does NOT create a composable. Returns a typed options object whose `.key` carries the `TData`
type, enabling fully type-safe cache reads/writes.

```ts
// Static
export const documentListQuery = defineQueryOptions({
  key: ['documents', 'list'],
  query: () => getDocumentList(),
})

// Dynamic (function overload)
export const documentByIdQuery = defineQueryOptions((id: string) => ({
  key: ['documents', id],
  query: () => getDocumentById(id),
}))

// In component — pass function result wrapped in getter for reactivity
const { data } = useQuery(() => documentByIdQuery(route.params.id))

// Type-safe cache access elsewhere
queryCache.getQueryData(documentByIdQuery('abc').key)          // typed as Document
queryCache.setQueryData(documentByIdQuery('abc').key, newDoc)  // type-checked
```

Prefer `defineQueryOptions` over bare key arrays for any shared key so cache operations
remain type-safe as the data shape evolves.
