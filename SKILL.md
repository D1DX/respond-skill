---
name: respond
description: Respond.io unofficial internal API (app.respond.io) for operations missing in the public API, including snippets, closing note categories, custom field update/delete, and Respond AI settings. Auto-triggers on Respond internal API, app.respond.io endpoints, snippet import, AI Assist/Prompt updates, and closing-note/category automation tasks.
disable-model-invocation: false
user-invokable: true
argument-hint: "task description"
---

# Respond Unofficial API Skill

Use this skill when `api.respond.io/v2` or MCP do not support the required operation and the web app (`app.respond.io`) does.

Last validated 2026-03-05.

## 1. When to Use

- Public Respond API endpoint is missing (for example snippet CRUD, custom field delete, closing note category CRUD)
- MCP toolset does not expose write operations you need
- You can authenticate interactively with a browser session (Cognito JWT)
- You need to update Respond AI settings (persona/prompts), which are only available on internal endpoints

Use public API first when possible.

## 2. Authentication Model (Internal API)

Internal API base:

```text
https://app.respond.io
```

Required headers:

```text
Authorization: Bearer {COGNITO_JWT}
Content-Type: application/json
botid: {WORKSPACE_ID}
orgid: {ORG_ID}
x-requested-with: XMLHttpRequest
origin: https://app.respond.io
```

How to capture JWT:

1. Open `https://app.respond.io`
2. DevTools -> Network -> filter `Fetch/XHR`
3. Open any request and copy the `Authorization: Bearer ...` value

Notes:

- JWT is short-lived; refresh by reloading app/respond session.

## 3. Endpoint Groups (High Value)

### Custom Fields

| Endpoint | Method | Purpose | Status |
|---|---|---|---|
| `/workspace/custom-field` | GET | List custom fields | Tested |
| `/workspace/custom-field` | PUT | Update field metadata/display name | Tested |
| `/workspace/custom-field/{id}` | DELETE | Delete custom field | Tested |

Critical payload rule for `PUT /workspace/custom-field`:

- `type` is required in the body; omitting it fails validation.
- Response shape is `data.items[]` (not `data[]`).

### Snippets

| Endpoint | Method | Purpose | Status |
|---|---|---|---|
| `/workspace/snippet/list` | POST | List snippets | Tested |
| `/workspace/snippet` | POST | Create snippet | Tested |
| `/workspace/snippet/{id}` | PUT | Update snippet | Discovered |
| `/workspace/snippet/{id}` | DELETE | Delete snippet | Discovered |

Critical payload rules for `POST /workspace/snippet`:

- Supported keys: `name`, `message`, `topics`, `uid`, `attachments`
- `uid` must be ASCII alphanumeric
- Unknown keys can return `ValidationError: Invalid property`
- `/workspace/snippet/list` response currently returns `data.total=24` and `data.items.length=23` in CBRN. Treat `items` as source of truth.

### Closing Note Categories

| Endpoint | Method | Purpose | Status |
|---|---|---|---|
| `/conversation/notes/categories` | POST | List categories | Tested |
| `/conversation/notes/category/add` | POST | Add category | Tested |
| `/conversation/notes/category/edit` | POST | Edit category | Tested |
| `/conversation/notes/category/delete` | POST | Delete category | Tested |

Critical request rule for `POST /conversation/notes/categories`:

- `pagination` is required; missing it returns `"Pagination is not defined"`.

### Respond AI (Assist + Prompts)

| Endpoint | Method | Purpose | Status |
|---|---|---|---|
| `/integration/respond-ai/settings` | GET | Get AI prompt settings | Tested |
| `/integration/respond-ai/settings` | PUT | Update AI prompt settings | Tested |
| `/integration/respond-ai/knowledge-base/settings` | GET | Get AI Assist settings | Tested |
| `/integration/respond-ai/knowledge-base/settings` | PUT | Update AI Assist settings | Tested |

Critical rules for `PUT /integration/respond-ai/settings`:

- Payload must include all 4 default prompts, even if `active=false`.
- Omitting defaults returns: `400 {"status":"error","message":"All default prompts must be present."}`.
- Keep custom `nameKey` stable (ASCII) when changing display name text.

### Tags (Internal List)

| Endpoint | Method | Purpose | Status |
|---|---|---|---|
| `/workspace/tags/list` | POST | List tags | Tested |

Critical request rule for `POST /workspace/tags/list`:

- Use `pagination.sortBy` as a string and `sortDesc` as boolean.
- Array format can return `ValidationError: must be string`.

### Useful Read-Only Endpoints

| Endpoint | Method | Purpose |
|---|---|---|
| `/workspace/users` | GET | Workspace users |
| `/workspace/teams` | GET | Teams |
| `/workspace/lifecycles/` | GET | Lifecycle stages |
| `/workspace/file/list` | POST | Uploaded files |
| `/integration/respond-ai/settings` | GET | AI Prompt settings |
| `/integration/respond-ai/knowledge-base/settings` | GET | AI Assist settings |

## 9. Full Endpoint Reference

Path base: `https://app.respond.io`. Status: `Tested` = verified by replay; `Discovered` = observed in app traffic, not replayed.

### Configuration and Workspace

| Endpoint | Method | Purpose | Status | Notes |
|---|---|---|---|---|
| `/workspace/custom-field` | GET | List custom fields | Tested | Response shape: `data.items[]` |
| `/workspace/custom-field` | PUT | Update custom field | Tested | `type` required in payload |
| `/workspace/custom-field/{id}` | DELETE | Delete custom field | Tested | No public API equivalent |
| `/workspace/custom-fields/ordering` | GET | Field ordering metadata | Discovered | |
| `/workspace/tags/list` | POST | List tags | Tested | `pagination.sortBy` must be string |
| `/workspace/file/list` | POST | List uploaded files | Discovered | |
| `/workspace/users` | GET | List workspace users | Discovered | |
| `/workspace/users/status` | GET | User online/offline status | Discovered | |
| `/workspace/teams` | GET | List teams | Discovered | |
| `/workspace/spaces/{id}` | GET | Workspace details | Discovered | |
| `/workspace/lifecycles/` | GET | Lifecycle stages | Discovered | |

### Snippets

| Endpoint | Method | Purpose | Status | Notes |
|---|---|---|---|---|
| `/workspace/snippet/list` | POST | List snippets | Tested | In CBRN: `data.total` may not match `data.items.length` |
| `/workspace/snippet` | POST | Create snippet | Tested | Keys: `name,message,topics,uid,attachments` |
| `/workspace/snippet/{id}` | PUT | Update snippet | Discovered | |
| `/workspace/snippet/{id}` | DELETE | Delete snippet | Discovered | |

### Integrations and AI

| Endpoint | Method | Purpose | Status | Notes |
|---|---|---|---|---|
| `/integration/channels` | GET | List channels | Tested | |
| `/integration/respond-ai/settings` | GET | AI prompt settings | Tested | |
| `/integration/respond-ai/settings` | PUT | Update AI prompt settings | Tested | Must include all default prompts |
| `/integration/respond-ai/knowledge-base/settings` | GET | AI Assist settings | Tested | |
| `/integration/respond-ai/knowledge-base/settings` | PUT | Update AI Assist settings | Tested | Controls persona + source toggles |

### Closing Notes

| Endpoint | Method | Purpose | Status | Notes |
|---|---|---|---|---|
| `/conversation/notes/categories` | POST | List categories | Tested | Requires `pagination` in body |
| `/conversation/notes/setting` | GET | Note settings | Discovered | |
| `/conversation/notes/category/add` | POST | Create category | Tested | |
| `/conversation/notes/category/edit` | POST | Edit category | Tested | |
| `/conversation/notes/category/delete` | POST | Delete category | Tested | |

### Other Internal APIs Seen in UI Traffic

| Endpoint | Method | Purpose | Status |
|---|---|---|---|
| `/api/v2/conversation/list` | POST | Inbox conversation list | Discovered |
| `/api/v2/contact/import/history` | POST | Contact import history | Discovered |
| `/api/v2/workflows/count` | GET | Workflow count | Discovered |
| `/auth/user` | GET | Current user info | Discovered |
| `/auth/user/organizations` | GET | User organizations | Discovered |
| `/auth/user/spaces` | GET | User workspaces | Discovered |
| `/auth/user/activity` | POST | Activity heartbeat | Discovered |
| `/billing/organization/plan` | GET | Billing plan | Discovered |
| `/billing/organization/usage` | GET | Billing usage | Discovered |
| `/billing/bsp/list-waba-balance` | POST | WABA balances | Discovered |
| `/billing/bsp/disabled-wabas` | GET | Disabled WABAs | Discovered |
| `/api/organization/{orgId}` | GET | Organization details | Discovered |
| `/api/organization/{orgId}/segment/fallback` | GET | Segment fallback | Discovered |
| `/api/notification/unread-count` | GET | Notification count | Discovered |

## 5. Curl Templates

Export once:

```bash
BASE_URL="https://app.respond.io"
JWT="REPLACE_ME"
BOTID="REPLACE_WORKSPACE_ID"
ORGID="REPLACE_ORG_ID"
```

### List custom fields

```bash
curl -s "$BASE_URL/workspace/custom-field" \
  -H "Authorization: Bearer $JWT" \
  -H "botid: $BOTID" \
  -H "orgid: $ORGID" \
  -H "x-requested-with: XMLHttpRequest" \
  -H "origin: https://app.respond.io"
```

### Update custom field display name

```bash
curl -s -X PUT "$BASE_URL/workspace/custom-field" \
  -H "Authorization: Bearer $JWT" \
  -H "Content-Type: application/json" \
  -H "botid: $BOTID" \
  -H "orgid: $ORGID" \
  -H "x-requested-with: XMLHttpRequest" \
  -H "origin: https://app.respond.io" \
  -d '{
    "id": 12345,
    "name": "Students Bio",
    "slug": "custom_field_slug",
    "description": "Description of the custom field",
    "type": "text",
    "listValues": [],
    "passViaContext": false
  }'
```

### Create snippet

```bash
curl -s -X POST "$BASE_URL/workspace/snippet" \
  -H "Authorization: Bearer $JWT" \
  -H "Content-Type: application/json" \
  -H "botid: $BOTID" \
  -H "orgid: $ORGID" \
  -H "x-requested-with: XMLHttpRequest" \
  -H "origin: https://app.respond.io" \
  -d '{
    "name": "Opening",
    "uid": "opening",
    "message": "Hello {{contact.first_name}}!",
    "topics": ["Opening & Routing"],
    "attachments": []
  }'
```

### Replace closing note categories

```bash
# Delete category
curl -s -X POST "$BASE_URL/conversation/notes/category/delete" \
  -H "Authorization: Bearer $JWT" \
  -H "Content-Type: application/json" \
  -H "botid: $BOTID" \
  -H "orgid: $ORGID" \
  -H "x-requested-with: XMLHttpRequest" \
  -H "origin: https://app.respond.io" \
  -d '{"category":"General Inquiry"}'

# Add category
curl -s -X POST "$BASE_URL/conversation/notes/category/add" \
  -H "Authorization: Bearer $JWT" \
  -H "Content-Type: application/json" \
  -H "botid: $BOTID" \
  -H "orgid: $ORGID" \
  -H "x-requested-with: XMLHttpRequest" \
  -H "origin: https://app.respond.io" \
  -d '{"category":"General","description":"General follow-up"}'
```

### Update AI prompt pack (safe pattern)

```bash
# 1) GET current settings
CURRENT=$(curl -s "$BASE_URL/integration/respond-ai/settings" \
  -H "Authorization: Bearer $JWT" \
  -H "botid: $BOTID" \
  -H "orgid: $ORGID")

# 2) Edit .data.prompts only, preserving all defaults
# 3) PUT back full object {token,prompts}
```

## 6. Failure Patterns and Fixes

- `403 ValidationError: Snippet ID is required`
  - Cause: wrong body shape for snippet endpoint.
  - Fix: use exact keys `name,message,topics,uid,attachments`.

- `403 ValidationError: Invalid property`
  - Cause: unsupported key in payload.
  - Fix: remove extra keys and replay exact browser payload.

- `403` that looks like auth but is data validation
  - Cause: invalid payload values (for example null phone in certain channel payloads).
  - Fix: validate body fields before request.

- `400 Pagination is not defined`
  - Cause: missing `pagination` in notes categories list request.
  - Fix: send `pagination` object.

- `400 All default prompts must be present.`
  - Cause: PUT AI prompts payload omitted default prompts.
  - Fix: include all defaults in `prompts[]` even if inactive.

- `ValidationError: must be string` on tags list
  - Cause: wrong pagination shape (`sortBy` as array).
  - Fix: set `sortBy` to string and `sortDesc` to boolean.

- JWT expired / unauthorized
  - Cause: Cognito session expired.
  - Fix: capture fresh bearer token from browser.

## 7. Safe Operating Rules

- Prefer public API or MCP for production automation.
- Use internal API for one-off admin/config tasks.
- Keep requests paced (about 1-2s between writes). Snippet writes can be rate-limited.
- Never commit JWT tokens, HAR files, or copied cookies.
- Re-verify endpoint + payload from DevTools before bulk write operations.

## 8. Execution Checklist

1. GET current state and save a local snapshot before any PUT/POST bulk update.
2. Apply minimal changes (avoid changing unrelated keys returned by GET).
3. Re-read endpoint after write and verify exact values.
4. Update project docs with absolute date and workspace/org IDs.
5. Log known API quirks encountered during the run.
