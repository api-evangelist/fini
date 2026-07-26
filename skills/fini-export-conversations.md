---
name: Export Fini conversations for QA and analytics
description: List agents, page through conversations in a time window, fetch full conversation detail, and optionally clean up test conversations.
api: openapi/fini-openapi.yml
operations: [listAgents, listConversations, getConversation, bulkDeleteConversations]
---

# Export Fini conversations for QA and analytics

Pull conversation data out of Fini for analytics, QA review, or downstream processing. This is an export-only, read-first flow.

## Auth
`Authorization: Bearer fini_...` with the `read` scope for listing/fetching. Deleting requires the `write` scope.

## Steps
1. **Map agents** — `GET /bots/public` (`listAgents`) to resolve agent names to `botId` values.
2. **List conversations** — `GET /hc-interactions/public` (`listConversations`). Set `since`/`until` (Unix epoch ms; window <= 90 days, `since` strictly before `until`), `limit` (1-100), optional `agentId`, `source`, and `channel` filters. Only conversations Fini has touched are returned, newest first.
3. **Paginate** — pass the returned `nextCursor`/`prevCursor` back as `cursor` with `direction`. Note the inverted mapping: `next` moves to OLDER conversations.
4. **Fetch detail** — `GET /hc-interactions/{id}/public` (`getConversation`) for the full record of any conversation ID.
5. **(Optional) clean up test data** — `DELETE /hc-interactions/public` (`bulkDeleteConversations`) with 1-50 conversation UUIDs per request.

## Conventions
- A window where `since == until`, or wider than 90 days, returns `400 Bad Request`.
- Cursor pagination + error envelope details: `conventions/fini-conventions.yml`, `errors/fini-problem-types.yml`.
