---
name: Ingest sources and publish knowledge to a Fini agent
description: Ingest web/document sources into Fini, generate knowledge, publish an article, and assign the knowledge folder to an agent.
api: openapi/fini-openapi.yml
operations: [ingestSources, checkKnowledgeJobs, listSources, generateKnowledge, createArticle, publishArticleDraft, getKnowledgeFolders, assignKnowledgeToAgents]
---

# Ingest sources and publish knowledge to a Fini agent

Load Fini's Knowledge Atlas from your own content, then make it usable by a support agent.

## Auth
All calls use `Authorization: Bearer fini_...` (see `authentication/fini-authentication.yml`). This flow mutates, so the key needs the `write` scope. A key without it returns `403 {"error":"Forbidden"}`.

## Steps
1. **Ingest sources** — `POST /documents/public` (`ingestSources`) with your web links or existing source IDs. The call returns immediately; processing is async.
2. **Poll** — `POST /knowledge/public/jobs/status` (`checkKnowledgeJobs`) or `GET /documents/public` (`listSources`) until ingestion completes. Sources sort by `updatedAt` desc.
3. **Generate knowledge** — `POST /knowledge/public` (`generateKnowledge`) from candidate text, optionally linked to a source.
4. **Create the article** — `POST /hc-articles/public` (`createArticle`) as a live article or draft.
5. **Publish** if drafted — `POST /hc-articles/{id}/publish/public` (`publishArticleDraft`).
6. **Assign to an agent** — read the tree with `GET /hc-folders/public` (`getKnowledgeFolders`), then `POST /hc-bot-folder-junctions/public` (`assignKnowledgeToAgents`) to attach the folder to the target agent (bot).

## Conventions
- Ingestion/generation are asynchronous — always poll rather than assuming synchronous completion.
- Pagination is cursor-based (`cursor` + `direction`; `next` = older). See `conventions/fini-conventions.yml`.
- Errors use `{statusCode, message, error}` — not RFC 9457. See `errors/fini-problem-types.yml`.
