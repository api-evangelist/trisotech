---
name: trisotech-promote-models-with-governance
description: >-
  Move models between Trisotech modeling places through the change-request workflow — raise a
  promotion request, attach models, and follow it to approval or rejection. Use when an agent
  must ship a model change through governance rather than writing directly to production.
api: Trisotech Digital Enterprise Suite Public API
base_url: https://{instance}.trisotech.com/publicapi
scopes: [repo_r, repo_w]
operations:
  - repositoryGet
  - repositorysearchGet
  - changerequestPromoteModelsPost
  - changerequestPromoteModelsGet
  - changerequestPromoteModelsIdGet
  - changerequestPromoteModelsIdPut
  - changerequestPromoteModelsIdContentGet
  - changerequestPromoteModelsIdContentPut
  - changerequestPromoteModelsIdDelete
  - changerequestPromotePlacePost
  - repositorypromotePost
  - repositoryfileversionGet
  - repositoryfilerecoveryGet
  - repositoryfilerecoveryPost
generated: '2026-09-02'
method: generated
source: https://cloud.trisotech.com/help/des/system-integration/rest-api-documentation.html
---

# Promote Trisotech models through a change request

Trisotech separates *editing* a model from *promoting* it. Direct promotion exists
(`POST /repositorypromote`, `repositorypromotePost`), but the governed path is a change request,
and that is the path an agent should take by default.

## Steps

1. **Identify source and target places.** `GET /repository` (`repositoryGet`).
2. **Find the models.** `GET /repositorysearch` (`repositorysearchGet`).
3. **Raise the request.** `POST /changerequest/promote-models` (`changerequestPromoteModelsPost`).
4. **Attach or revise the model set.**
   `PUT /changerequest/promote-models/{id}/content` (`changerequestPromoteModelsIdContentPut`);
   read it back with `changerequestPromoteModelsIdContentGet`.
5. **Track it.** `GET /changerequest/promote-models` (`changerequestPromoteModelsGet`) for the
   queue, `GET /changerequest/promote-models/{id}` (`changerequestPromoteModelsIdGet`) for one.
6. **Update or decide.** `PUT /changerequest/promote-models/{id}`
   (`changerequestPromoteModelsIdPut`). The published models include
   `ChangeRequestDecisionRequest`, so approval and rejection are both first-class.

A parallel set of operations promotes a whole place rather than a model set —
`changerequestPromotePlacePost` and its `/changerequest/promote-place/{id}` siblings.

## Reversal

- Withdraw a pending request with `DELETE /changerequest/promote-models/{id}`
  (`changerequestPromoteModelsIdDelete`).
- A model that was overwritten can be rolled back through its version history —
  `GET /repositoryfileversion` (`repositoryfileversionGet`).
- A model that was deleted may be recoverable: list with `GET /repositoryfilerecovery`
  (`repositoryfilerecoveryGet`), restore with `POST /repositoryfilerecovery`
  (`repositoryfilerecoveryPost`). **Trisotech publishes no retention window for this** — do not
  promise a user that a deletion is undoable without checking the recovery list first.

## Failure handling

Expect `NoRight`, `NoRepository`, `InvalidRepository`, `DuplicateName` and `InvalidPath` in
`error[0].code`. Remember that an unauthenticated call returns HTTP 500, not 401.
