---
name: pnpm
description: "pnpm package manager conventions for Nuxt v4 projects. Use when installing packages, managing workspaces, writing lockfile-related CI steps, or troubleshooting dependency resolution in this stack."
---

# pnpm — Package Manager Conventions

The default package manager for all bigin web projects is **pnpm**. Do not use npm or yarn unless the user explicitly requests it.

---

## Installation & Setup

```bash
# Install pnpm globally (if not present)
npm install -g pnpm

# Verify
pnpm --version
```

### New project

```bash
pnpm create nuxt my-app
cd my-app
pnpm install
```

### Existing project

```bash
pnpm install
```

---

## Common Commands

| Action | Command |
|--------|---------|
| Install all deps | `pnpm install` |
| Add dependency | `pnpm add <pkg>` |
| Add dev dependency | `pnpm add -D <pkg>` |
| Remove package | `pnpm remove <pkg>` |
| Run script | `pnpm dev` / `pnpm build` / `pnpm preview` |
| Update packages | `pnpm update` |
| Update one package | `pnpm update <pkg>` |
| List outdated | `pnpm outdated` |
| Audit | `pnpm audit` |
| Dedupe lockfile | `pnpm dedupe` |

---

## Standard Stack Installation

For a Nuxt v4 fullstack or SPA project:

```bash
# Core
pnpm add @nuxt/ui @pinia/nuxt @vueuse/nuxt

# Pinia Colada (async data)
pnpm add @pinia/colada

# Dev tools
pnpm add -D @nuxt/devtools typescript vue-tsc

# Cloudflare (fullstack MVP only)
pnpm add -D wrangler nitro-cloudflare-dev
```

---

## package.json Conventions

Always define a `packageManager` field to pin the pnpm version:

```json
{
  "packageManager": "pnpm@9.x.x",
  "scripts": {
    "dev": "nuxt dev",
    "build": "nuxt build",
    "preview": "nuxt preview",
    "typecheck": "vue-tsc --noEmit",
    "deploy": "wrangler pages deploy .output/public",
    "cf-dev": "nitro-cloudflare-dev"
  }
}
```

---

## .npmrc

Add a `.npmrc` at the project root to enforce hoisting behavior:

```ini
shamefully-hoist=true
strict-peer-dependencies=false
```

`shamefully-hoist=true` is required for some Nuxt modules that expect flat `node_modules`. Include this file in version control.

---

## Lockfile

- Always commit `pnpm-lock.yaml`
- Never commit `package-lock.json` or `yarn.lock`
- In CI, use `pnpm install --frozen-lockfile` to prevent accidental updates

---

## CI/CD (GitHub Actions)

```yaml
- uses: pnpm/action-setup@v4
  with:
    version: 9

- uses: actions/setup-node@v4
  with:
    node-version: 20
    cache: 'pnpm'

- run: pnpm install --frozen-lockfile
- run: pnpm build
```

---

## Workspace (Monorepo)

If the project grows into a monorepo:

```yaml
# pnpm-workspace.yaml
packages:
  - 'apps/*'
  - 'packages/*'
```

```bash
# Run command in specific workspace
pnpm --filter my-app build

# Add dep to specific workspace
pnpm --filter my-app add <pkg>
```

---

## Troubleshooting

| Problem | Fix |
|---------|-----|
| Module not found after install | Try `pnpm install --shamefully-hoist` or add `.npmrc` |
| Peer dependency warnings | Add `strict-peer-dependencies=false` to `.npmrc` |
| CI fails with lockfile mismatch | Run `pnpm install` locally, commit updated `pnpm-lock.yaml` |
| Nuxt module resolution issues | Ensure `shamefully-hoist=true` is in `.npmrc` |
