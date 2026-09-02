---
name: trisotech-deploy-a-service
description: >-
  Publish a BPMN workflow, CMMN case or DMN decision model from a Trisotech modeling place
  into an execution environment, then deploy it as a callable service. Use when an agent must
  promote a model to a running Automation API endpoint.
api: Trisotech Digital Enterprise Suite Public API
base_url: https://{instance}.trisotech.com/publicapi
scopes: [repo_r, mvn_r, mvn_w]
operations:
  - repositoryGet
  - repositorycontentGet
  - executionrepositoryGet
  - executionrepositoryartifactPost
  - deploymentsBpmnEnvironmentGroupIdArtifactIdVersionPost
  - deploymentsDmnEnvironmentGroupIdArtifactIdVersionPost
  - deploymentsCmmnEnvironmentGroupIdArtifactIdVersionPost
  - executionrepositoryartifactDelete
  - deploymentsBpmnEnvironmentGroupIdArtifactIdVersionDelete
generated: '2026-09-02'
method: generated
source: https://cloud.trisotech.com/help/des/system-integration/rest-api-documentation.html
---

# Deploy a Trisotech model as a running service

Every operationId below was read from Trisotech's published API reference. Nothing here is
invented; where the docs do not state something, this skill says so.

## Before you start

- Send `Authorization: Bearer <token>` on every call and **always** send
  `Accept: application/json`. Without an Accept header the API returns XML, which the provider
  marks deprecated.
- Your Client App must hold `repo_r` to read models, and `mvn_r` / `mvn_w` to read and write
  execution environments.
- Deployed services are addressed by Maven coordinates: `{environment}/{groupId}/{artifactId}/{version}`.

## Steps

1. **Find the modeling place.** `GET /repository` (`repositoryGet`) — "List all the places that
   you have access to." Read `data[].id`; that UUID is the place identifier used everywhere else,
   and is also the suffix of the place's RDF graph URI.
2. **Locate the model.** `GET /repositorycontent` (`repositorycontentGet`) for a directory
   listing, or `GET /repositorysearch` (`repositorysearchGet`) to "Search a place (or all places)
   for a file or folder matching the search query".
3. **Choose the target environment.** `GET /executionrepository` (`executionrepositoryGet`) —
   "List all the execution environments that you have access to."
4. **Publish the artifact.** `POST /executionrepositoryartifact`
   (`executionrepositoryartifactPost`) puts the built service into the execution environment.
5. **Deploy it as a service**, picking the operation that matches the model language:
   - BPMN — `POST /deployments/bpmn/{environment}/{groupId}/{artifactId}/{version}`
     (`deploymentsBpmnEnvironmentGroupIdArtifactIdVersionPost`, "Deploys process model(s) as service.")
   - DMN — `POST /deployments/dmn/{environment}/{groupId}/{artifactId}/{version}`
   - CMMN — `POST /deployments/cmmn/{environment}/{groupId}/{artifactId}/{version}`
6. **Verify before you announce success.** Re-read the artifact with
   `GET /executionrepositoryartifact` (`executionrepositoryartifactGet`) rather than trusting the
   deploy response alone.

## Reversal

Each deployment operation has a matching `DELETE` on the same path
(`deploymentsBpmnEnvironmentGroupIdArtifactIdVersionDelete` and its CMMN/DMN siblings), and
`executionrepositoryartifactDelete` removes the published artifact. **No time window is published
for either**, so treat the rollback as available but unwarranted — confirm it against your own
instance before you rely on it.

## Failure handling

- **Do not blind-retry a deploy.** Trisotech publishes no idempotency key and no
  request-deduplication window. A timed-out `POST` may or may not have deployed; re-read with
  `executionrepositoryartifactGet` before retrying.
- **Read `error[0].code`, not the HTTP status.** A missing or expired token returns **HTTP 500**
  with `{"error":[{"code":"RequiresLogin",...}]}`, not a 401. Codes you may see on this flow
  include `NoRight`, `NoRepository`, `Unlicensed`, `NoSubscription` and `InvalidParameters`.
- Capture the `request-tracking-identifier` response header (a UUID) for any support request.
- There are no rate-limit headers. Bound your own concurrency.
