---
name: google-analytics
description: Google Analyticsの同意シグナルとCookieハンドリング規則を強制する。GAトラッキング、Cookie同意、広告関連アナリティクスの実装時に発動。
---

# Google Analytics

The keywords "MUST", "NEVER", "SHOULD", and "MAY" in this document are to be interpreted as described in RFC 2119.

## Consent Rules

1. MUST ask about cookie usage and send a consent signal upon agreement
2. `analytics_storage` MUST always be `granted` (basic analytics are always enabled)
3. `ad_storage`, `ad_user_data`, `ad_personalization` SHOULD be `granted` by default
4. IF user selects "Do not consent" -> MUST update advertising-related values to `denied`
