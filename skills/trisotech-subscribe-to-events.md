---
name: trisotech-subscribe-to-events
description: >-
  Subscribe to Trisotech Digital Enterprise Suite events by creating an event emitter, filter
  the stream with a FEEL expression, and read the CloudEvents-shaped messages. Use when an
  agent needs to react to model, service, security or AI-agent activity in the suite.
api: Trisotech Digital Enterprise Suite Public API
base_url: https://{instance}.trisotech.com/publicapi
scopes: [emitter_r, emitter_w]
operations:
  - emittersGet
  - emittersDefinitionsGet
  - emittersDefinitionsPost
  - emittersDefinitionsIdGet
  - emittersDefinitionsIdPut
  - emittersDefinitionsIdEnablePost
  - emittersDefinitionsIdDisablePost
  - emittersDefinitionsIdDelete
  - emittersAuditIdGet
  - eventPost
  - identityGet
generated: '2026-09-02'
method: generated
source: https://cloud.trisotech.com/help/des/system-integration/asynchronous-events.html
---

# Subscribe to Trisotech events

Trisotech publishes **23 topics carrying 104 message types**, each with typed fields, as an
anonymous JSON catalog at `https://{instance}.trisotech.com/docs/messages-documentation.json`.
A copy is saved in this repo at `asyncapi/trisotech-events-catalog.json`. **Read the catalog
first** — it is the authoritative list of what you can subscribe to.

## Message shape

Messages are **CloudEvents 1.0.1** in JSON. Required attributes: `id`, `time` (RFC 3339),
`source` (defaults to the instance name), `type` (the message type name), `specversion`.
Optional: `data` (the type-specific payload), `subject`, `Ip`, `user`, `app` (the OAuth client
id, when the event came through a Client App).

## Steps

1. **Read the catalog** and pick your topics and message types.
   `GET /emitters` (`emittersGet`) lists the emitter types available on the instance.
2. **List what already exists** — `GET /emitters/definitions` (`emittersDefinitionsGet`) — so you
   do not create a duplicate subscription.
3. **Create the emitter.** `POST /emitters/definitions` (`emittersDefinitionsPost`,
   "Creates new event emitter definition.") with:
   - `type` — the integration strategy / target system.
   - `topics` — an **array of arrays**, each inner array a hierarchical topic path.
     Trisotech's own example: `[["service","production","finance"]]` subscribes to service
     events for the production environment from a group called finance.
   - `messages` — optional array of message type names. Omit to receive every type on the topic.
   - `filter` — optional **FEEL** expression evaluated against the message data. The message is
     emitted when it evaluates to true.
   - `identityRef` — optional identity used to publish; read candidates with `GET /identity`
     (`identityGet`).
4. **Enable it.** `POST /emitters/definitions/{id}/enable` (`emittersDefinitionsIdEnablePost`).
5. **Audit delivery.** `GET /emitters/audit/{id}` (`emittersAuditIdGet`) downloads the emitter's
   audit log for a date range.

## Publishing your own message

`POST /event` (`eventPost`, "Publish a message") puts a message on the bus.

## Reversal

`POST /emitters/definitions/{id}/disable` (`emittersDefinitionsIdDisablePost`) stops delivery
without destroying the definition — prefer it over `emittersDefinitionsIdDelete`, which is
permanent and has no published recovery path.

## What you cannot see

Each topic carries its own security constraint. Most administrative topics — `security`,
`user`, `group`, `client-app`, `aiagent`, `email`, `preference` — are **Administrator only**.
A non-admin token subscribing to them will simply receive nothing; check
`asyncapi/trisotech-events-catalog.json` for the per-topic constraint before you debug silence
as a delivery fault.

## Notable topics for an agent

- `aiagent` — `AIAgentToolInvoke` (toolInvoker, toolName, toolArguments, toolResults, product,
  usecase) and `AIAgentChat`. This is the audit trail of what an AI agent actually did.
- `service` (10 types) — instance started, node started/finished, events received and published.
- `security` (6 types) — login succeeded/failed, unauthorized, virus detected.
- `repository` (19 types) — the richest topic; model reads, comments, git pushes and pulls.
