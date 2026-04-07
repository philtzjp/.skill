---
name: operational-rules
description: データモデル、環境変数、ツール、バージョニング、アーキテクチャドキュメントの運用手順を強制する。開発作業中は常時有効。
---

# Operational Rules

The keywords "MUST", "NEVER", "SHOULD", and "MAY" in this document are to be interpreted as described in RFC 2119.

## Data Models & Documentation

1. MUST record all data models in `llm/models.yaml`; IF implementation changes -> MUST update this file
2. IF environment variables change -> MUST update `.env.example`
3. IF a service version change is deemed necessary -> MUST update `VERSION` based on Semantic Versioning and create `llm/version/${version}.md`
4. IF architecture changes -> MUST update the Mermaid diagram in `./llm/ARCHITECTURE.md`

## Tooling

1. IF bulk find-and-replace is preferable -> SHOULD write a `.js` script inside `temp/`, execute it, then delete the script
2. MUST introduce Biome & ESLint Vue and run format commands as appropriate
3. NEVER run `biome check --fix --unsafe` on `.vue` files (Biome cannot analyze Vue template scope, causing false positives like `_` prefix renaming)

## Development Server

1. NEVER run `npm run dev` in the background; MUST prompt the user to run it

## Language

1. MUST always respond in Japanese
