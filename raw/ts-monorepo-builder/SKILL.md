---
name: ts-monorepo-builder
description: Turborepoベストプラクティスに従いTypeScriptモノレポ構造をスキャフォールド・保守する。apps/、packages/、共有設定、turbo.jsonを正しい依存関係で生成。pnpm/npmワークスペース対応。
---

# TypeScript Monorepo Builder

The keywords "MUST", "NEVER", "SHOULD", and "MAY" in this document are to be interpreted as described in RFC 2119.

## Overview

Builds and extends TypeScript monorepos following Turborepo + workspace (pnpm / npm) best practices.

## Trigger

This skill applies when:

- Setting up a new monorepo from scratch
- Adding a new app (`apps/`) or package (`packages/`) to an existing monorepo
- Creating shared configuration packages (TypeScript / ESLint)
- Configuring `turbo.json` task pipelines
- Organizing inter-package dependencies

## Procedure

### 1. Detect Package Manager

MUST detect the package manager from the current repository:

```
1. pnpm-lock.yaml exists → pnpm
2. package-lock.json exists → npm
3. Neither exists → ask the user (AskUserQuestion)
```

Use the detected package manager consistently for all commands and configuration throughout the session.

### 2. Directory Structure

MUST follow the `apps/` + `packages/` separation model:

```
monorepo/
├── apps/           # Deployable applications (leaf nodes in package graph)
│   ├── web/        # Next.js / Vite frontend
│   ├── api/        # Hono / Express backend
│   └── ...
├── packages/       # Shared libraries & config (consumed by other packages)
│   ├── ui/         # Shared UI components
│   ├── shared/     # Shared types & utilities
│   ├── database/   # DB schema + client
│   ├── typescript-config/  # Shared tsconfig
│   └── eslint-config/      # Shared ESLint config
├── turbo.json
├── pnpm-workspace.yaml  # pnpm only
└── package.json
```

#### Placement Rules

- **`apps/`**: Deployable artifacts (Next.js, Hono, Express, CLI, etc.). These are leaf nodes in the package graph. They are never installed by other packages.
- **`packages/`**: Shared code that cannot be deployed on its own. UI components, DB clients, type definitions, configuration files.

NEVER place deployable applications in `packages/`. Web apps and API servers MUST go in `apps/`.

### 3. Workspace Configuration

#### For pnpm

MUST create `pnpm-workspace.yaml`:

```yaml
packages:
  - 'apps/*'
  - 'packages/*'
```

#### For npm

MUST add `workspaces` to root `package.json`:

```json
{
  "workspaces": [
    "apps/*",
    "packages/*"
  ]
}
```

### 4. Package Naming Convention

MUST use scoped package names with `@<org>/` prefix:

```json
{
  "name": "@<org>/<package-name>",
  "version": "0.0.0",
  "private": true
}
```

- Detect `<org>` from existing packages in the repository. If no existing scope is found, ask the user.
- All internal packages MUST have `"private": true`.
- Pin version to `"0.0.0"` (referenced via workspace protocol).

### 5. `exports` Field

MUST define `exports` in each package's `package.json` to explicitly declare entry points:

```json
{
  "exports": {
    ".": "./src/index.ts",
    "./types": "./src/types/index.ts",
    "./utils": "./src/utils/index.ts"
  }
}
```

Prefer `exports` over `main`. This enables IDE auto-completion and conditional exports (CJS/ESM).

### 6. Internal Package Compilation Strategy

Default to the **Just-in-Time (JIT)** pattern:

- Export TypeScript source directly; the consumer's bundler compiles it
- Zero build step, minimal configuration
- Best suited for Turbopack / webpack / Vite consumers

```json
{
  "name": "@myapp/ui",
  "exports": {
    "./button": "./src/button.tsx",
    "./card": "./src/card.tsx"
  }
}
```

Switch to the **Compiled** pattern (`tsc` → `dist/`) only when the user explicitly requests it or when non-bundler tools need to consume the package.

### 7. Shared TypeScript Configuration

MUST create `packages/typescript-config/` with base + purpose-specific configs:

```
packages/typescript-config/
├── package.json
├── base.json       # Shared strict settings
├── nextjs.json     # Next.js apps
├── library.json    # Library packages
└── node.json       # Node.js backends (if needed)
```

#### base.json

```json
{
  "compilerOptions": {
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "moduleResolution": "bundler",
    "target": "ES2022",
    "lib": ["ES2022"],
    "resolveJsonModule": true,
    "isolatedModules": true
  }
}
```

Each package extends the shared config:

```json
{
  "extends": "@<org>/typescript-config/base.json",
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

NEVER place a shared tsconfig at the repository root. Each package MUST have its own `tsconfig.json` that extends the shared config.

### 8. Shared ESLint Configuration

SHOULD create `packages/eslint-config/` with purpose-specific configs (flat config format):

```
packages/eslint-config/
├── package.json
├── base.js         # Common rules
├── web.js          # React frontend
└── node.js         # Node.js backend
```

### 9. `turbo.json` Task Pipeline

MUST create `turbo.json` with dependency-aware task pipeline:

```json
{
  "$schema": "https://turbo.build/schema.json",
  "tasks": {
    "build": {
      "dependsOn": ["^build"],
      "outputs": [".next/**", "dist/**"]
    },
    "dev": {
      "cache": false,
      "persistent": true
    },
    "lint": {
      "dependsOn": []
    },
    "type-check": {
      "dependsOn": ["^build"],
      "outputs": []
    },
    "test": {
      "dependsOn": ["^build"],
      "outputs": []
    }
  }
}
```

- `^build` means "run dependent packages' build first"
- `dev` is uncached and persistent
- `lint` has no dependencies (can run in parallel)

### 10. Inter-Package Dependencies

Use workspace protocol for internal dependencies:

#### For pnpm

```json
{
  "dependencies": {
    "@<org>/shared": "workspace:*",
    "@<org>/ui": "workspace:*"
  }
}
```

#### For npm

```json
{
  "dependencies": {
    "@<org>/shared": "*",
    "@<org>/ui": "*"
  }
}
```

After adding dependencies, MUST run the install command:
- pnpm: `pnpm install`
- npm: `npm install`

### 11. Frontend + Backend Type Sharing

Types shared between `apps/web` and `apps/api` MUST live in `packages/shared`.

With the JIT pattern:

```json
{
  "name": "@<org>/shared",
  "exports": {
    ".": "./src/index.ts",
    "./types": "./src/types/index.ts"
  }
}
```

Switch to the Compiled pattern with conditional exports only when both ESM and CJS are required.

## Checklist for Adding a Package

When adding a new package, verify:

1. Placed in the correct directory (`apps/` vs `packages/`)
2. `package.json` has scoped name, `private: true`, and `exports`
3. `tsconfig.json` extends the shared config
4. Consumer `package.json` files have workspace references
5. Ran `pnpm install` / `npm install`

## Package Extraction Criteria

Decide whether to extract a new package using these 3 axes:

1. **Used by 2+ apps** — the clearest signal for extraction
2. **Platform-independent** — code tied to `"use client"` or framework-specific APIs is harder to extract
3. **Change frequency vs. blast radius** — high-frequency, wide-impact code benefits from type-boundary protection via packaging

SHOULD start small (3–5 packages) and extract new packages only when the need arises.

NEVER over-split: a team of 3–5 with 25 packages is an anti-pattern.

## Prohibitions

- NEVER use a package manager other than the one detected in step 1
- NEVER place deployable apps in `packages/`
- NEVER place shared libraries in `apps/`
- NEVER create nested packages (e.g., `apps/group/app-a` with `apps/group/package.json`)
- NEVER omit `"private": true` on internal packages
- NEVER use `main` field without also setting `exports`
- NEVER install Turborepo globally — use `npx turbo` or add as root devDependency
