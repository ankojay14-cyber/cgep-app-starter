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

[Successful Integrated CMMC GRC Gate Run](https://github.com/ankojay14-cyber/cgep-app-starter/actions/runs/42153405575)

- **AWS Region:** `us-east-1`
- **Evidence bucket:** `acme-health-intake-evidence-896bdc57`
- **Workflow run:** `42153405575`
- **Retention:** S3 Object Lock GOVERNANCE, 30 days

Evidence prefix:

ci-evidence/97eff0fa2d3fa5e28b9cf5724050509b4430a58c/42153405575-1/

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

The evidence archive was independently downloaded from S3. SHA-256 checksum verification and Cosign signature verification both passed. S3 Object Lock GOVERNANCE retention was independently confirmed for all three evidence objects through December 6, 2026.

## Known Limitations

The integrated .github/workflows/grc-gate.yml performs pull-request policy validation and executes Plan → Policy Check → Apply → Sign → Upload on pushes to main. The workflow successfully applied a no-change Terraform plan, but deployment of a new infrastructure change through CI has not yet been demonstrated. GAP-06 and GAP-08 remain partially remediated, as documented in WRITEUP.md.

The project demonstrates supporting CMMC-mapped technical controls and does not claim CMMC certification or complete regulatory compliance.