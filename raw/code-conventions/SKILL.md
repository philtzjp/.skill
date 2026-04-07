---
name: code-conventions
description: 命名規則、フォーマット、エラーハンドリング、型の整理、アーキテクチャ制約をすべてのコード変更に強制する。常時有効。
---

# Code Conventions

The keywords "MUST", "NEVER", "SHOULD", and "MAY" in this document are to be interpreted as described in RFC 2119.

## Naming & Formatting

1. MUST follow these conventions: `snake_case` for variables, `camelCase` for functions, `PascalCase` for types, `CONSTANT_CASE` for env vars, 4-space indent, no unnecessary semicolons, prefer double quotes
2. MUST use descriptive names even if verbose (NG: `const handle = () => {}`)
3. SHOULD objectify variable names to normalize them into single words (e.g., `worksName` -> `works.name`)

## Error Handling & Values

1. NEVER use fallback values in constants (NG: `web_url: process.env.WEB_URL || 'http://localhost:3000'`)
2. NEVER use fallback values in functions; MUST return an error on failure

## File Organization

1. MUST define all error messages in a single file
2. MUST define all log messages in a single file
3. MUST define all types in files within a dedicated directory

## Architecture

1. MUST use a modular monolith architecture
2. IF backward compatibility is deemed necessary -> MUST confirm with the user before proceeding

## Environment Variables

1. NEVER use prefixes such as `NUXT_` or `NUXT_PUBLIC_` and `VITE_` for environment variable names

## Directives

1. NEVER write or format inline code within directives as multi-line (it causes errors and won't work)
