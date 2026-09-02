---
name: trisotech-find-and-route-user-tasks
description: >-
  Search the human tasks waiting across every Trisotech execution environment, service and
  version, narrowing by environment, group, artifact or version, with paging and sorting. Use
  when an agent must find work assigned to a person or group and route or report on it.
api: Trisotech Digital Enterprise Suite Public API
base_url: https://{instance}.trisotech.com/publicapi
scopes: [bpmn_x, cmmn_x]
operations:
  - searchUserTasks3
  - searchUserTasks4
  - searchUserTasks5
  - searchUserTasks6
  - searchUserTasks7
  - loginGet
  - executionrepositoryGet
generated: '2026-09-02'
method: generated
source: https://cloud.trisotech.com/help/des/system-integration/rest-api-documentation.html
---

# Find and route Trisotech user tasks

BPMN user tasks and CMMN human tasks belong to a running service instance, but the people who
work them think in terms of "what is assigned to me", not "which service". The UserTasksSearch
group exists for exactly that, and it narrows through five nested scopes.

## The five scopes

| Path | operationId | Scope |
|---|---|---|
| `GET /tasks` | `searchUserTasks3` | Every task across every environment and service |
| `GET /tasks/{env}` | `searchUserTasks4` | One execution environment |
| `GET /tasks/{env}/{group}` | `searchUserTasks5` | One group within it |
| `GET /tasks/{env}/{group}/{artifact}` | `searchUserTasks6` | One service |
| `GET /tasks/{env}/{group}/{artifact}/{version}` | `searchUserTasks7` | One version of one service |

The `{env}/{group}/{artifact}/{version}` segments are the same Maven coordinates the deployment
operations use.

## Paging and sorting

These operations accept the API's standard collection parameters:

- `page` — the page index, **starting at 0**.
- `pageSize` — defaults to **10**. Set it explicitly; the default is small.
- `sortBy` — a predefined field.
- `sortOrder` — `ascending` (default) or `descending`.

Page until a short page comes back. There is no cursor and no `Link` header.

## Steps

1. `GET /login` (`loginGet`, "Retrieve the information of the current logged in user.") to
   confirm which identity the token resolves to — task visibility follows group membership.
2. `GET /executionrepository` (`executionrepositoryGet`) to enumerate environments if you intend
   to narrow.
3. Call the narrowest scope that answers the question, with an explicit `pageSize`.

## Routing and reassignment

Reassignment does **not** live on the Public API. It is on the per-service Automation API,
under "Manage service instance resume points": a resume point referencing a user task can be
**delegated** to other performers by a current performer, or **reassigned** by an admin. The
payload's `unassign` flag decides whether new performers are appended to the existing set
(`false`) or replace it (`true`); leaving users and groups empty offers the task to anyone on
the instance. See
https://cloud.trisotech.com/help/service-library/service-rest-api-endpoint.html.

## Caveats

- Group membership has been a real source of defects here: release 13.0.11 and 12.14.4 both
  record "User task search did not properly consider group membership." Verify results against
  a known task before trusting an empty set.
- Reassignment is a write with no idempotency key. Do not replay it after a timeout; re-read the
  instance's resume points instead.
