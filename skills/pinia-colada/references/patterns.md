# Patterns, SSR, TypeScript & Plugins

## Optimistic Updates

Pattern: update cache in `onMutate`, rollback in `onError`, invalidate in `onSettled`.

```ts
const queryCache = useQueryCache()

const { mutate: updateContact } = useMutation({
  mutation: patchContact,

  onMutate(contactInfo) {
    // 1. Cancel in-flight requests to avoid overwrite race
    queryCache.cancelQueries({ key: ['contact', contactInfo.id] })
    // 2. Snapshot current value for rollback
    const old = queryCache.getQueryData<Contact>(['contact', contactInfo.id])
    // 3. Apply optimistic update
    queryCache.setQueryData(['contact', contactInfo.id], { ...old, ...contactInfo })
    return { old }
  },

  onError(_err, _vars, { old }) {
    // Rollback on failure
    if (old) queryCache.setQueryData(['contact', old.id], old)
  },

  onSettled(_data, _err, vars) {
    // Always re-sync from server
    queryCache.invalidateQueries({ key: ['contact', vars.id] })
  },
})
```

---

## SSR

Pinia Colada handles SSR via `onServerPrefetch` automatically — no need to manually `await`
queries in components.

### Nuxt (recommended)

```ts
// nuxt.config.ts
modules: ['@pinia/colada-nuxt']
// optionally: colada.options.ts at project root for global settings
```

No additional configuration needed. `useQuery` works the same on server and client.

### Manual SSR (non-Nuxt)

```ts
// server.ts
import { serializeQueryCache } from '@pinia/colada'

const pinia = createPinia()
const app = createApp(App)
app.use(pinia).use(PiniaColada)

await renderToString(app)  // triggers onServerPrefetch in components

const serialized = serializeQueryCache(pinia)
// Embed in HTML: window.__PINIA_COLADA__ = JSON.stringify(serialized)
```

```ts
// client.ts
import { hydrateQueryCache } from '@pinia/colada'
hydrateQueryCache(pinia, JSON.parse(window.__PINIA_COLADA__))
```

### Server-only / client-only queries

```ts
// Only run on client (skips SSR fetch)
const { data } = useQuery({
  key: ['client-only'],
  query: () => fetchSomething(),
  enabled: () => typeof document !== 'undefined',
})
```

### Disable GC on server (prevents dangling timers)

```ts
app.use(PiniaColada, {
  plugins: [PiniaColadaSSRNoGc],
})
```

---

## TypeScript

### Custom global error type

```ts
// global.d.ts or vite-env.d.ts
import '@pinia/colada'
declare module '@pinia/colada' {
  interface TypesConfig {
    defaultError: MyApiError
  }
}
```

### Extend useQuery options via plugin

```ts
declare module '@pinia/colada' {
  interface UseQueryOptions {
    myCustomOption?: boolean
  }
}
```

### Extend global mutation context

```ts
declare module '@pinia/colada' {
  interface UseMutationGlobalContext {
    router: Router
  }
}
```

---

## Official Plugins

| Plugin | Package | Purpose |
|--------|---------|---------|
| `PiniaColadaQueryHooksPlugin` | `@pinia/colada` | Global `onSuccess/onError/onSettled` hooks for all queries |
| `PiniaColadaSSRNoGc` | `@pinia/colada` | Disable GC timers on server |
| `PiniaColadaAutoRefetch` | `@pinia/colada-plugin-auto-refetch` | Refetch on interval |
| `PiniaColadaRetry` | `@pinia/colada-plugin-retry` | Auto-retry with exponential backoff |
| `PiniaColadaDelay` | `@pinia/colada-plugin-delay` | Debounce loading state (avoids loading flicker) |
| Cache persistence | `@pinia/colada-plugin-cache-persistence` | Persist cache to localStorage / IndexedDB |

### Global query hooks

```ts
app.use(PiniaColada, {
  plugins: [
    PiniaColadaQueryHooksPlugin({
      onError(error, entry) {
        toast.error(error.message)
      },
    }),
  ],
})
```

### Retry plugin

```ts
import { PiniaColadaRetry } from '@pinia/colada-plugin-retry'

app.use(PiniaColada, {
  plugins: [PiniaColadaRetry({ retry: 3, delay: 1000 })],
})
```

### Auto-refetch plugin

```ts
import { PiniaColadaAutoRefetch } from '@pinia/colada-plugin-auto-refetch'

// Per-query option after installing plugin
const { data } = useQuery({
  key: ['live-data'],
  query: fetchLiveData,
  refetchInterval: 5000,  // added by plugin
})
```

---

## Non-throwing APIs (fetch, axios)

`fetch` doesn't throw on 4xx/5xx — throw explicitly so Pinia Colada can set `status: 'error'`:

```ts
query: async () => {
  const res = await fetch('/api/todos')
  if (!res.ok) throw new Error(`HTTP ${res.status}`)
  return res.json()
}
```

---

## placeholderData — keep previous data during navigation

```ts
import { useQuery } from '@pinia/colada'

const { data, isPlaceholderData } = useQuery({
  key: () => ['todos', page.value],
  query: () => fetchTodos(page.value),
  // show previous page while next page loads
  placeholderData: prev => prev,
})
```

`isPlaceholderData` is `true` while showing previous data — use it to style a "loading" overlay
without replacing the visible content.
