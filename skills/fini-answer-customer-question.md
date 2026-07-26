---
name: Answer a customer question with a Fini agent
description: Resolve which agent to use, then submit a message event into Fini's agent loop and read back the events it produces.
api: openapi/fini-openapi.yml
operations: [listAgents, generateAnswer, getConversation]
---

# Answer a customer question with a Fini agent

Drive the Fini Agent Loop: submit a customer message and get the agent's response events.

## Auth
`Authorization: Bearer fini_...` with the `write` scope (Generate Answer is a mutating route).

## Steps
1. **Pick the agent** — `GET /bots/public` (`listAgents`) to get the `botId` for the workspace agent you want to answer with.
2. **Generate an answer** — `POST /hc-interactions/events/public` (`generateAnswer`): send a message event into Fini for that agent. The response returns the public events created for that submission (the agent's reply and any actions).
3. **(Optional) fetch the conversation** — `GET /hc-interactions/{id}/public` (`getConversation`) to read the full conversation record afterward.

## Conventions
- Fini answers autonomously via RAGless retrieval + Agentic Actions; guardrails and escalation are configured per agent (rules and prompts).
- Errors use `{statusCode, message, error}`; a key without `write` scope returns `403`. See `errors/fini-problem-types.yml`.
