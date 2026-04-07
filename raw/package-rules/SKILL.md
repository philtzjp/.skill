---
name: package-rules
description: パッケージインストール方法とフレームワーク/ライブラリ選定規則を強制する。パッケージのインストール、フレームワーク選択、依存関係追加時に発動。
---

# Package Rules

The keywords "MUST", "NEVER", "SHOULD", and "MAY" in this document are to be interpreted as described in RFC 2119.

## Installation

1. MUST install packages via `npm install`; NEVER write directly into `package.json`

## Framework & Library Selection

| Condition | Rule |
|-----------|------|
| Using Nuxt | MUST use `latest` version |
| Using Firebase | MUST use `firebase-admin`; NEVER use client packages |
| Processing dates | MUST use `date-fns` |
| AI-related features | SHOULD prefer Vercel AI SDK |
| `Slack`, `Discord`, `Microsoft Teams`, `GitHub`, `Telegram`, `Linear` integrations | MUST use Vercel Labs Chat SDK |
| Using `unkey` | MUST use the v2 SDK |
