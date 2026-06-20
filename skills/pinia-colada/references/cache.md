# Query Cache API

## useQueryCache()

Available in `setup()`, Pinia stores, Nuxt plugins, and router guards.
**Not** available in global scope or plain functions called outside a Vue context.

```ts
const queryCache = useQueryCache()
```

---

## Invalidation

```ts
// Invalidate by key prefix — marks stale + refetches all active matching entries
queryCache.invalidateQueries({ key: ['todos'] })

// Exact key only
queryCache.invalidateQueries({ key: ['todos', 1], exact: true })

// Invalidate all queries
queryCache.invalidateQueries()

// Include inactive entries (not mounted anywhere)
queryCache.invalidateQueries({ key: ['todos'] }, 'all')

// Await invalidation (keeps mutation asyncStatus 'loading' until refetch completes)
await queryCache.invalidateQueries({ key: ['todos'] })
```

---

## Reading & Writing Cache Data

```ts
// Read a single entry's data (undefined if not cached)
const todos = queryCache.getQueryData<Todo[]>(['todos'])
const todo  = queryCache.getQueryData<Todo>(['todos', 1])

// Write a single entry (creates entry if missing)
queryCache.setQueryData(['todos', 1], newTodo)
queryCache.setQueryData(['todos', 1], prev => ({ ...prev, name: 'Updated' }))

// Write to all matching entries (useful after paginated list mutations)
queryCache.setQueriesData<Contact[]>(
  { key: ['contacts', 'list'] },
  (list = []) => [...list, newContact],
)
```

---

## Other Cache Operations

```ts
// Cancel in-flight fetches (e.g. before optimistic update)
queryCache.cancelQueries({ key: ['contact', id] })

// Read raw cache entries (snapshot, no mutation)
queryCache.get(['todos', 1])           // exact entry or undefined
queryCache.getEntries({
  key: ['todos'],
  stale: true,
  active: true,
  status: 'success',
})
```

---

## Entry Filters

| Filter | Values | Description |
|--------|--------|-------------|
| `key` | `EntryKey` | Prefix match (hierarchical) |
| `exact` | `boolean` | Exact match only, no children |
| `active` | `boolean \| null` | `true` = has active subscribers |
| `stale` | `boolean \| null` | `true` = past staleTime |
| `status` | `'pending' \| 'success' \| 'error' \| null` | |
| `predicate` | `(entry) => boolean` | Custom filter |

---

## Precise Entry Actions

```ts
const entry = queryCache.get(['todos', 1])
if (entry) {
  queryCache.invalidate(entry)     // mark stale + refetch if active
  queryCache.fetch(entry)          // force fetch
  queryCache.refresh(entry)        // fetch only if stale
  queryCache.cancel(entry)         // abort in-flight request
  queryCache.remove(entry)         // remove from cache entirely
  queryCache.setEntryState(entry, newState)
}
```

---

## Key Factories

Centralise key definitions to prevent typos and enable hierarchical invalidation.

```ts
export const todoKeys = {
  root:     ['todos'] as const,
  list:     (filter?: string) => ['todos', 'list', { filter }] as const,
  detail:   (id: number)      => ['todos', id] as const,
}

// Usage
const { data } = useQuery({ key: todoKeys.detail(1), query: () => fetchTodo(1) })

// Invalidate all todos (root + list + details)
queryCache.invalidateQueries({ key: todoKeys.root })

// Invalidate only list queries
queryCache.invalidateQueries({ key: todoKeys.list() })
```

### Key serialisation rules

- Array order matters: `['a', 'b'] ≠ ['b', 'a']`
- Object key order is normalised: `{ a:1, b:2 } == { b:2, a:1 }`
- `undefined` in objects is stripped; `null` is kept
- Type matters: `'2' ≠ 2`
- Nest related keys to leverage prefix invalidation
