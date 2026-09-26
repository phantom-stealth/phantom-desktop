# Phantom SQLite Database Schema Specification

Phantom uses SQLite with Write-Ahead Logging (WAL) as its core transactional and analytics data store.

## Configuration & Pragmas

```sql
PRAGMA journal_mode = WAL;
PRAGMA synchronous = NORMAL;
PRAGMA cache_size = -64000;       -- 64 MB in-memory cache
PRAGMA temp_store = MEMORY;
PRAGMA mmap_size = 268435456;     -- 256 MB memory-mapped I/O
PRAGMA foreign_keys = ON;
```

## Schema Entities

### 1. `tenants`
Stores SaaS organizations and subscription metadata.
- `id` (TEXT, PK): Unique UUID
- `name` (TEXT): Organization name
- `email` (TEXT, UNIQUE): Owner email
- `plan` (TEXT): `solo` | `team` | `agency` | `enterprise`
- `status` (TEXT): `trial` | `active` | `suspended` | `cancelled`
- `stripe_customer_id` (TEXT)
- `stripe_subscription_id` (TEXT)
- `api_key_hash` (TEXT)
- `branding` (TEXT): White-label config JSON
- `trial_ends_at` (INTEGER): Unix timestamp
- `created_at` (INTEGER), `updated_at` (INTEGER)

### 2. `personas`
Automated social and dating personas.
- `id` (TEXT, PK)
- `tenant_id` (TEXT, FK → tenants.id ON DELETE CASCADE)
- `name` (TEXT)
- `status` (TEXT): `idle` | `active` | `warming` | `flagged`
- `identity_type` (TEXT)
- `platform_targets` (TEXT): JSON array of platforms
- `geo_target` (TEXT)
- `proxy_id` (TEXT)
- `identity_data` (TEXT, ENCRYPTED): AES-256-GCM encrypted profile attributes
- `cookie_data` (TEXT, ENCRYPTED): AES-256-GCM encrypted session cookies
- `swipes_today`, `matches_today`, `messages_today` (INTEGER)
- `ban_health` (INTEGER, 0-100)

### 3. `matches`
Lead and match pipeline tracking.
- `id` (TEXT, PK)
- `tenant_id` (TEXT, FK → tenants.id)
- `persona_id` (TEXT, FK → personas.id)
- `platform` (TEXT)
- `their_name` (TEXT), `their_age` (INTEGER)
- `status` (TEXT), `stage` (TEXT)
- `interest_score` (INTEGER), `ghosting_risk` (INTEGER)
- `external_platform` (TEXT)
- `external_contact` (TEXT, ENCRYPTED)
- `matched_at` (INTEGER), `last_message_at` (INTEGER)

### 4. `messages`
Conversation transcript records.
- `id` (TEXT, PK)
- `match_id` (TEXT, FK → matches.id)
- `tenant_id` (TEXT, FK → tenants.id)
- `direction` (TEXT): `inbound` | `outbound` | `them` | `us`
- `content` (TEXT, ENCRYPTED): AES-256-GCM encrypted message body
- `message_type` (TEXT)
- `ai_generated` (INTEGER)
- `strategy_used` (TEXT)
- `sent_at` (INTEGER)

### 5. `usage_events`
High-speed metering events for billing and funnels.
- `id` (TEXT, PK)
- `tenant_id` (TEXT, FK → tenants.id)
- `resource` (TEXT): `swipe` | `match` | `message` | `video` | `identity` | `escalation` | `date_set`
- `quantity` (INTEGER)
- `platform` (TEXT), `persona_id` (TEXT)
- `occurred_at` (INTEGER)

### 6. `events_log`
Global audit log and SSE streaming buffer.
- `id` (TEXT, PK)
- `tenant_id` (TEXT, FK → tenants.id)
- `event_type` (TEXT)
- `platform` (TEXT), `persona_id` (TEXT)
- `payload` (TEXT)
- `occurred_at` (INTEGER)

### 7. `proxies`
Proxy rotation pool with health scoring.
- `id` (TEXT, PK), `tenant_id` (TEXT, FK)
- `host`, `port`, `protocol`
- `credentials` (TEXT, ENCRYPTED)
- `country_code`, `status`, `latency_ms`, `last_tested_at`

### 8. `api_keys`
Hashed tenant API access keys.
- `id` (TEXT, PK), `tenant_id` (TEXT, FK)
- `name` (TEXT)
- `key_hash` (TEXT, UNIQUE): SHA-256
- `key_prefix` (TEXT): First 8 characters
- `last_used_at`, `created_at`, `revoked_at`

### 9. `webhooks` & `webhook_deliveries`
Outbound event subscriptions and delivery tracking.
- `webhooks`: `url`, `secret` (ENCRYPTED), `events`, `status`, `failure_count`
- `webhook_deliveries`: `webhook_id`, `event_type`, `payload`, `status_code`, `attempt`, `duration_ms`

### 10. `email_queue`
Transactional email queue with exponential retry backoff.
- `to_email`, `template`, `context`, `status`, `attempts`, `scheduled_at`, `sent_at`, `error`

### 11. `sessions`
Database-backed session token store.
- `tenant_id`, `token_hash` (UNIQUE SHA-256), `is_admin`, `last_active`, `expires_at`
