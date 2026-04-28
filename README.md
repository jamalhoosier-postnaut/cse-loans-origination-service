# Loan Origination API — Postman CSE Exercise (Service 2)

This repo onboards the **Loan Origination API** into Postman using the same CSE-maintained GitHub Actions pattern as Payments, proving the approach generalizes across services with different real-world characteristics.

**Postman project/workspace name:** `cse-loans-origination-service-v1`

---

## What this workflow produces (end-to-end)

After a successful run you will have:

**In Postman**
- Workspace created/reused: `cse-loans-origination-service-v1`
- OpenAPI spec uploaded to **Spec Hub**
- Generated collections:
  - Baseline
  - Smoke
  - Contract
- Environments created/updated: `prod`, `stage`
- Operational assets:
  - Mock server
  - Smoke monitor

**In GitHub**
- Exported collections committed under `postman/collections/`
- `.postman/resources.yaml` mapping file committed (IDs + metadata)

---

## How the workflow works (decisions)

### Actions chain (explicit)
1) **postman-bootstrap-action**  
Creates/reuses workspace, uploads spec, generates suites, creates environments.

2) **postman-repo-sync-action**  
Exports suites, writes `.postman/resources.yaml`, creates mock + monitor, commits updates.

I kept the chain explicit so the platform team can own it and so I can clearly explain rerun behavior and scaling.

### Public repo spec fetch
Spec is fetched directly via `raw.githubusercontent.com` (public repo), eliminating private-repo fetching issues.

### CI workflow generation disabled
`generate-ci-workflow: false` keeps CI control with the platform team.

---

## Universal vs per-service changes

### Universal
- Inputs: spec + env targets + auth/runtime variable contract
- Pipeline: Bootstrap → Repo Sync
- Outputs: Spec Hub + Baseline/Smoke/Contract + envs + exports + mock/monitor

### Changes per service (consulting view)
Loan Origination represents a service that may require:
- internal-only access (ECS/ALB behind private networking)
- mTLS requirements

In those cases, **contract tests run from in-network CI runners** and certs are injected securely via secret management. The Postman pattern stays the same; execution location and credential material differ.

---

## What the customer platform team must provide

- Real `prod`/`stage` URLs + routing/path rules
- JWT strategy and rotation plan (and mTLS cert distribution if required)
- Runner placement for internal-only services (VPC/self-hosted)
- GitLab CI template for the GitLab-only team (Postman CLI + JUnit + merge gates)
- Governance: ownership, naming, lifecycle/archiving, spec versioning policy

---

## Run instructions

### 1) Required secrets
Repo → Settings → Secrets and variables → Actions → Repository secrets:
- `POSTMAN_API_KEY`
- `POSTMAN_ACCESS_TOKEN`
- `GH_PAT`

### 2) Run the workflow
GitHub → Actions → **Postman - Onboard Loans API** → Run workflow

---

## Validation

**In GitHub**
- `postman/collections/` populated
- `.postman/resources.yaml` exists
- commit like: `chore: sync Postman artifacts and metadata`

**In Postman**
Workspace `cse-loans-origination-service-v1` contains:
- Spec in Spec Hub
- Baseline/Smoke/Contract collections
- `prod` and `stage` environments
- Mock server + smoke monitor

---

## Trade-offs
- Contract tests are only as strong as the spec; govern drift with review + CI gates.
- Internal/mTLS services often can’t use cloud monitors; run suites on in-network runners.
- PAT is used here due to org constraints; customers should use managed service account tokens with rotation.

---

## AI usage disclosure
AI accelerated YAML/README drafting and debugging. Outputs were validated by successful runs and confirming Postman assets + committed exports.
