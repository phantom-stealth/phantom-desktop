# Phantom REST API Reference

The Phantom API provides comprehensive programmatic control over browser profiles, AI dating fleets, synthetic identities, social automation, and multi-tenant SaaS features.

Interactive Swagger UI documentation is available locally at:
`http://localhost:7842/api/docs`
OpenAPI 3.1.0 specification:
`http://localhost:7842/api/docs/openapi.json`

## Authentication

All protected endpoints require either a Session Bearer token or an API Key:
```http
Authorization: Bearer phantom_your_session_token_here
```
or
```http
x-api-key: phantom_your_api_key_here
```

Superadmin endpoints require:
```http
x-phantom-admin-key: your_superadmin_secret_key
```

## Key Endpoint Groups

### 1. Personas & Fleet Management
- `GET /api/personas` — List all personas for tenant
- `POST /api/personas` — Create new automated persona
- `GET /api/personas/:id` — Retrieve persona details and ban health
- `PATCH /api/personas/:id` — Update persona platform targets or geo targets
- `DELETE /api/personas/:id` — Delete persona and cascade associated data

### 2. Dating Intelligence & Matches
- `GET /api/dating/matches` — List matches with filtering by status and stage
- `GET /api/dating/matches/:id/messages` — Retrieve decrypted conversation transcript
- `POST /api/dating/messages` — Append or send AI-generated reply
- `GET /api/dating/fleet` — Global fleet execution status and active sessions

### 3. Analytics & Conversion Funnels
- `GET /api/analytics/summary` — Daily swipes, matches, messages, and percentage trends
- `GET /api/analytics/funnel` — 6-stage conversion funnel with stage-to-stage percentages
- `GET /api/analytics/platforms` — Performance breakdown across platforms
- `GET /api/analytics/activity` — 365-day activity heatmap dataset

### 4. Billing & Metering
- `GET /api/billing/usage` — Current monthly metered usage against plan quota
- `POST /api/saas/checkout` — Generate Stripe checkout URL for upgrades
- `POST /api/saas/billing-portal` — Generate Stripe customer self-service portal URL
- `POST /api/saas/webhook` — Stripe HMAC-signed webhook receiver

### 5. Outbound Webhooks
- `GET /api/webhooks` — List active webhook endpoints
- `POST /api/webhooks` — Register webhook URL with events subscription
- `GET /api/webhooks/:id` — Get delivery history and status codes
- `DELETE /api/webhooks/:id` — Unregister webhook

### 6. Superadmin Operations
- `GET /api/admin/tenants` — Global tenant list and MRR metrics
- `PATCH /api/admin/tenants/:id/plan` — Override tenant plan tier
- `PATCH /api/admin/tenants/:id/status` — Suspend/activate tenant and revoke sessions
- `POST /api/admin/tenants/:id/impersonate` — Generate 15-minute scoped impersonation token
- `GET /api/admin/revenue` — Real-time MRR, ARR, and cohort analytics
