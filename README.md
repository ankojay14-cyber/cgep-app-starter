# Acme Health — CGE-P GRC Engineering Capstone

This repository demonstrates the application of GRC Engineering to Acme Health's Patient Intake API, using **CMMC Level 2** as the primary compliance framework.

The project integrates Terraform security hardening, OPA/Rego policies, GitHub Actions validation and deployment, cryptographically signed evidence, S3 Object Lock retention, and OSCAL documentation.

## Project Deliverables

| Layer | Implementation |
|---|---|
| Terraform | Customer-managed KMS, S3 hardening, VPC configuration, CloudTrail, and evidence vault |
| Policy as Code | Five CMMC-mapped Rego policies with passing and failing tests |
| GitHub Actions | Terraform planning, Conftest validation, deployment, Cosign signing, and S3 evidence upload |
| OSCAL | Validated component definition and CMMC control profile |

## Capstone Documentation

- [WRITEUP.md](WRITEUP.md) — Framework selection, control coverage, design decisions, evidence, and limitations
- [GAPS.md](GAPS.md) — Original application security gaps
- [OSCAL Component](oscal/components/acme-health.json)
- [OSCAL Profile](oscal/profiles/cge-p-minimum.json)

## Policy Enforcement Evidence

- [PR #1 — Successful Security Baseline](https://github.com/ankojay14-cyber/cgep-app-starter/pull/1)
- [PR #2 — Blocked Security Violation](https://github.com/ankojay14-cyber/cgep-app-starter/pull/2)

The OPA test suite passed 23 of 23 tests during development.

## Signed Evidence

[Successful GitHub Actions Evidence Run](https://github.com/ankojay14-cyber/cgep-app-starter/actions/runs/42122881135)

- **AWS Region:** `us-east-1`
- **Evidence bucket:** `acme-health-intake-evidence-896bdc57`
- **Workflow run:** `42122881135`
- **Retention:** S3 Object Lock GOVERNANCE, 30 days

Evidence prefix:

`ci-evidence/27f2a19ed65d1983eb038f25621852ea5fabff32/42122881135-1/`

## Grader Verification

**1. Validate policy tests**

```bash
opa test ./policies -v
```

**2. Validate OSCAL documents**

```bash
trestle validate -f oscal/components/acme-health.json
trestle validate -f oscal/profiles/cge-p-minimum.json
```

**3. Verify signed evidence**

Using authorized AWS credentials, retrieve the three evidence objects from the S3 prefix above:

- `evidence-bundle.tar.gz`
- `evidence-bundle.tar.gz.sha256`
- `evidence-bundle.sigstore.json`

Recompute the SHA-256 checksum, verify the Cosign signature and Sigstore transparency record, and inspect the S3 Object Lock retention settings.

The signed bundle was successfully verified during GitHub Actions run `42122881135`. Independent verification is a final submission check.

## Known Limitations

The existing repository uses separate validation and deployment workflows, and its deployment safeguard currently permits only no-change Terraform plans. These limitations, together with incomplete GAP-06 and GAP-08 remediation, are documented in [WRITEUP.md](WRITEUP.md).

The project demonstrates supporting CMMC-mapped technical controls and does not claim CMMC certification or complete regulatory compliance.