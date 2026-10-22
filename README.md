# commerce-catalogue

Owner: Catalogue team.

Public product read API. During application migration, the orders commerce core owns
inventory and its transactional persistence; this service reads it over HTTP. It does
not query another service's database and is not yet an independently owned inventory store.

Project guide: [DOC.md](https://github.com/Narx-Space-Prod/platform-infrastructure/blob/main/DOC.md).

## Current state

Application source is in `main.go` and `metrics.go`, with tests in `main_test.go`.
Missing or unavailable commerce configuration returns 503 instead of demo prices.
Readiness checks the upstream catalogue; liveness remains independent of that dependency.
AWS deployment and GitOps promotion are separate implementation milestones.
All changes after repository initialization go through feature branches and pull requests.
These repositories are public on GitHub Free. Protected `main` requires a CODEOWNER
approval, passing CI checks and resolved conversations; administrators follow the same rules.

## Application delivery contract

- Build and release this service independently; preserve existing commerce features during migration.
- Serve health endpoints at `/health/live` and `/health/ready` on an unprivileged port.
- Run as a non-root user with a read-only root filesystem; use `/tmp` for temporary data.
- Handle SIGTERM gracefully and report logs to stdout/stderr without secrets or payment data.
- Expose metrics; propagate request/event IDs across API and queue boundaries.
- Publish immutable images to ECR and promote the same digest between environments.

## Build and preview

```bash
docker build --target runtime -t narx-local/catalogue:dev .
```

The build checks Go formatting, runs vet and tests, then compiles the runtime binary.
The server listens on port 8081. Set `COMMERCE_URL` to the orders API, such as
`http://orders:8083`. Product routes are `/products` and `/products/{id}`.
Metrics are exposed at `/metrics`. SIGTERM drains active HTTP requests.

Use the integrated preview in `platform-gitops/local/compose.yaml` for all services.

Imported with its license from [boutique-catalogue, revision 27aa280](https://github.com/Local-Narx-Space/boutique-catalogue/tree/27aa28010a8f9552409d304ebb32f3db9dcf5ba9).

## Delivery

PRs run shared workflow/secret checks, then component Docker tests, a non-root runtime
check, a blocking HIGH/CRITICAL vulnerability scan and SBOM generation. Trusted main
can publish the exact tested image to ECR through OIDC and sign its digest, provenance
and SBOM. Publishing is held unless `DELIVERY_ENABLED=true`; no AWS resources are
created by this repository. GitOps verifies release records and promotes the same
digests through reviewed dev, QA and production PRs.

App credentials, AWS setup and local registry refresh are documented in the single
[project guide](https://github.com/Narx-Space-Prod/platform-infrastructure/blob/main/DOC.md).
