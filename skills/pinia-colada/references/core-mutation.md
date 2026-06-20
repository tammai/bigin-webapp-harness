# Mutations API

## useMutation(options)

```ts
const {
  state,       // ComputedRef<DataState>
  data,        // ShallowRef<TData | undefined>
  error,       // ShallowRef<TError | null>
  status,      // ShallowRef<DataStateStatus>
  asyncStatus, // ShallowRef<'idle' | 'loading'>
  isLoading,   // ComputedRef<boolean>
  variables,   // ShallowRef<TVars | undefined> — last vars passed to mutate/mutateAsync
  mutate,      // (vars: TVars) => void           — swallows errors
  mutateAsync, // (vars: TVars) => Promise<TData> — rethrows errors
  reset,       // () => void — clears error + data back to pending
} = useMutation(options)
```

### Options

| Option | Type | Description |
|--------|------|-------------|
| `mutation` | `(vars: TVars, ctx: TContext) => Promise<TData>` | required |
| `key` | `EntryKey \| ((vars) => EntryKey)` | Optional, for cache lookup |
| `gcTime` | `number \| false` | Default: `60_000` (1 min) |
| `onMutate` | `(vars, ctx) => TContext \| void` | Runs before mutation; return value becomes `context` |
| `onSuccess` | `(data, vars, context) => unknown` | Runs on success |
| `onError` | `(error, vars, context) => unknown` | Runs on error |
| `onSettled` | `(data, error, vars, context) => unknown` | Always runs after mutation |
| `meta` | `MutationMeta` | Arbitrary metadata for plugins |

### Basic example

```vue
<script setup lang="ts">
import { useMutation, useQueryCache } from '@pinia/colada'

const queryCache = useQueryCache()

const { mutate: createTodo, isLoading, error } = useMutation({
  mutation: (text: string) =>
    fetch('/api/todos', {
      method: 'POST',
      body: JSON.stringify({ text }),
    }).then(r => r.json()),
  onSettled() {
    // invalidate after success OR error
    queryCache.invalidateQueries({ key: ['todos'] })
  },
})
</script>

<template>
  <button :disabled="isLoading" @click="createTodo('Buy milk')">Add</button>
  <p v-if="error">{{ error.message }}</p>
</template>
```

### mutate vs mutateAsync

```ts
// mutate — fire and forget, errors surface via `error` ref only
mutate(vars)

// mutateAsync — use when caller needs to await or catch
try {
  await mutateAsync(vars)
  router.push('/success')
} catch (e) {
  // handle inline
}
```

### Lifecycle hooks order

`onMutate` → mutation runs → `onSuccess` OR `onError` → `onSettled`

Use `onMutate` to capture rollback data for optimistic updates (see [patterns.md](patterns.md)).

---

## defineMutation(optionsOrSetup) — reusable global mutations

Mirrors `defineQuery`. All callers share one instance.

```ts
// Simple form
export const useCreateTodo = defineMutation({
  mutation: (text: string) =>
    fetch('/api/todos', { method: 'POST', body: JSON.stringify({ text }) }).then(r => r.json()),
})

// Setup function form — exposes custom shared state
export const useCreateTodo = defineMutation(() => {
  const todoText = ref('')
  const { mutate, ...rest } = useMutation({
    mutation: () => createTodo(todoText.value),
  })
  return { ...rest, createTodo: mutate, todoText }
})

// In component
const { createTodo, todoText, isLoading } = useCreateTodo()
```

### Always pass vars explicitly

```ts
// ✅ good — explicit, testable
mutation: (id: string) => deleteTodo(id)
mutate(todo.id)

// ❌ avoid — closes over component state, breaks defineMutation sharing
mutation: () => deleteTodo(todo.id)
mutate()
```
