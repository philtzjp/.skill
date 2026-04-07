---
name: api-design
description: OpenAPI準拠、RFC 9457エラー構造、バージョニング、パス規約、フレームワーク選定などのAPI設計標準を強制する。REST APIの設計・実装時に発動。
---

# API Design

The keywords "MUST", "NEVER", "SHOULD", and "MAY" in this document are to be interpreted as described in RFC 2119.

## Standards

1. MUST conform to OpenAPI
2. Error response structure MUST conform to RFC 9457
3. Health check endpoint structure MUST conform to [draft-inadarei-api-health-check](https://datatracker.ietf.org/doc/html/draft-inadarei-api-health-check)

## URL & Path

1. MUST use URL path versioning (e.g., `/api/v1/`)
2. SHOULD adopt paths as short as possible; IF unavoidable -> MUST use `kebab-case`
3. MUST use singular nouns (`/user` instead of `/users`)

## Authentication

1. MUST use Bearer authentication

## Linting

1. MUST lint the API using [Spectral](https://github.com/stoplightio/spectral)

## Framework

1. SHOULD use [Hono](https://hono.dev/) as the HTTP framework; IF building an MCP server -> MUST use Hono with `@hono/mcp`
