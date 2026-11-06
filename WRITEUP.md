\# Acme Health — CGE-P GRC Engineering Capstone Write-up



\## 1. Primary Compliance Framework



The selected primary framework is \*\*Cybersecurity Maturity Model Certification (CMMC) Level 2\*\*, using NIST SP 800-171 Revision 2 control mappings for this capstone.



Acme Health is a fictional 50-person telehealth organization pursuing a federal pilot that may involve Controlled Unclassified Information (CUI). CMMC Level 2 was selected because its security requirements provide a clear basis for assessing encryption, least privilege, boundary protection, and security monitoring.



CMMC Level 2 was also selected because its requirements can be mapped directly to the application's identified security gaps, Terraform configurations, Rego policies, and automated compliance evidence.



HIPAA and SOC 2 remain secondary considerations given Acme Health's healthcare operations and enterprise assurance objectives. This capstone prioritizes one framework to demonstrate focused implementation and traceability rather than attempting to satisfy all three simultaneously.



The project does not claim certification or full compliance with CMMC Level 2, HIPAA, or SOC 2.



\## 2. Security Gap Remediation and Control Coverage



The starter application contained eight intentional security gaps documented in `GAPS.md`. The remediation approach combines Terraform security hardening, automated policy enforcement, and OSCAL documentation.



| Gap | CMMC Level 2 practice | Security weakness | Terraform implementation | Automated policy |

|---|---|---|---|---|

| GAP-01 | SC.L2-3.13.11 | S3 lacks customer-managed KMS encryption | SSE-KMS using `aws\_kms\_key.grc` | `gap01\_s3\_encryption.rego` |

| GAP-02 | SC.L2-3.13.11 | DynamoDB lacks customer-managed KMS encryption | Customer-managed KMS encryption | `gap02\_dynamodb\_encryption.rego` |

| GAP-03 | SC.L2-3.13.8 | S3 does not explicitly enforce TLS | Bucket policy denies insecure transport | `gap03\_s3\_tls.rego` |

| GAP-04 | MP.L2-3.8.9 | S3 uploads bucket versioning disabled | Enabled S3 versioning | `gap04\_s3\_versioning.rego` |

| GAP-05 | SC.L2-3.13.1 | Lambda is not attached to the existing VPC | Private subnets, security group, and VPC endpoints | `gap05\_lambda\_vpc.rego` |

| GAP-06 | SI.L2-3.14.6 | Missing Lambda operational safeguards | SQS dead-letter queue and X-Ray tracing added; reserved concurrency not implemented | Not covered |

| GAP-07 | AC.L2-3.1.5 | Excessive Lambda permissions | IAM actions and resource scope restricted | Not covered |

| GAP-08 | AU.L2-3.3.1 | API Gateway monitoring and traffic protections incomplete | Access logging and throttling configured; WAF not implemented | Not covered |



Five substantive Rego policies were implemented to detect the reintroduction of GAP-01 through GAP-05. Each policy includes framework metadata, control identifiers, severity, remediation guidance, and passing and failing tests.



\*\*Control implementation approach:\*\* Terraform establishes the secure configuration, while Rego evaluates subsequent Terraform plans to prevent known weaknesses from being reintroduced.



The five policies map to four distinct CMMC practices because GAP-01 and GAP-02 both support SC.L2-3.13.11.



These mappings represent supporting technical safeguards, not complete practice satisfaction. Customer-managed KMS encryption alone does not demonstrate FIPS-validated cryptography compliance, and S3 versioning alone does not satisfy the confidentiality-of-backup-CUI requirement under MP.L2-3.8.9.



\## 3. Architecture and Design Decisions



\*\*AWS Region — us-east-1\*\*



The project uses `us-east-1` for the application and security infrastructure, maintaining consistency with the existing AWS sandbox while avoiding unnecessary regional complexity.



\*\*Infrastructure — Terraform and Existing VPC\*\*



Terraform manages the application and security baseline. The existing VPC was reused rather than creating another network, and Lambda was configured to use private subnets, a designated security group, and VPC endpoints.



This approach reduced infrastructure complexity while demonstrating how to secure an inherited application.



\*\*Encryption — Customer-Managed AWS KMS\*\*



A customer-managed AWS KMS key with automatic rotation supports S3 uploads and DynamoDB encryption. This provides greater administrative control over cryptographic keys than relying exclusively on AWS-owned encryption.



\*\*Audit Logging — AWS CloudTrail\*\*



A multi-region CloudTrail trail with log-file validation records AWS management activity in a dedicated logging bucket, supporting auditability, security investigations, and detection of changes to delivered log files.



\*\*Evidence Storage — S3 Object Lock\*\*



An encrypted, versioned evidence bucket uses \*\*Object Lock in GOVERNANCE mode with 30-day retention\*\*.



GOVERNANCE mode was selected to balance evidence retention with administrative flexibility in a temporary AWS sandbox. Unlike COMPLIANCE mode, it permits appropriately privileged users to bypass retention. A production implementation would require stronger separation of duties and consideration of COMPLIANCE mode.



\*\*Account Architecture — Single AWS Account\*\*



A single AWS sandbox account was selected to reduce implementation complexity within the capstone timeline. A separate evidence account would provide stronger administrative isolation in production.



\*\*Deployment Controls — GitHub Actions and AWS OIDC\*\*



GitHub Actions uses AWS OpenID Connect (OIDC) authentication to assume designated IAM roles without storing long-lived AWS credentials.



Pull-request validation runs Terraform planning and Conftest security checks. After changes are merged into `main`, the deployment workflow repeats validation, performs the permitted Terraform apply operation, signs the resulting evidence, and uploads it to S3.



Automated validation was chosen to make security enforcement repeatable without introducing unnecessary manual approval steps.



The current implementation uses separate validation and deployment workflows and restricts deployment to no-change plans. These limitations remain to be addressed to fully satisfy the Capstone Brief's intended integrated deployment flow.



\## 4. Automated Policy Enforcement



OPA/Rego and Conftest evaluate Terraform plan JSON against five CMMC-mapped security policies.



The policy suite passed \*\*23 of 23 OPA unit tests\*\* during development.



Two pull requests demonstrate the security gate.



\*\*Successful PR — Compliant Security Baseline\*\*



\[PR #1 — CMMC Policy-as-Code Controls and Secure AWS Baseline](https://github.com/ankojay14-cyber/cgep-app-starter/pull/1)



The Terraform baseline and automated policy checks passed validation, and the pull request was merged into `main`.



\*\*Blocked PR — Intentional GAP-04 Violation\*\*



\[PR #2 — Verify CMMC GAP-04 Policy Blocks Suspended S3 Versioning](https://github.com/ankojay14-cyber/cgep-app-starter/pull/2)



The S3 uploads bucket versioning configuration was intentionally changed from `Enabled` to `Suspended`.



Conftest detected the violation and failed the required GitHub Actions check. The insecure pull request remained unmerged.



These demonstrations confirm that the policy gate allows compliant configurations to pass while detecting and blocking an intentionally insecure change.



\## 5. Signed and Retained Compliance Evidence



Following successful deployment validation, GitHub Actions generates an evidence archive containing:



\- `evidence-manifest.json`

\- `terraform-plan-summary.json`

\- `conftest-results.txt`



The workflow packages these artifacts into `evidence-bundle.tar.gz`, calculates its SHA-256 checksum, signs the archive using keyless Cosign, and verifies the signature.



The archive, checksum, and Sigstore bundle are uploaded to the S3 evidence vault with Object Lock retention.



\### Verified Deployment Evidence



\[GitHub Actions — Successful Signed Evidence Deployment](https://github.com/ankojay14-cyber/cgep-app-starter/actions/runs/42122881135)



| Evidence attribute | Verified value |

|---|---|

| GitHub Actions run ID | `42122881135` |

| Merge commit | `27f2a19ed65d1983eb038f25621852ea5fabff32` |

| Evidence bucket | `acme-health-intake-evidence-896bdc57` |

| Object Lock mode | GOVERNANCE |

| Retention through | December 5, 2026 |

| SHA-256 verification | Passed in CI |

| Cosign verification | Passed in CI |

| Evidence upload | Successful |



\*\*Evidence S3 prefix:\*\*



`ci-evidence/27f2a19ed65d1983eb038f25621852ea5fabff32/42122881135-1/`



GitHub Actions verified the archive checksum and Cosign signature, and AWS S3 confirmed retention settings for all three evidence objects.



These artifacts provide traceability between the source commit, Terraform plan, policy validation, deployment run, and signed evidence.



Independent verification of the stored objects remains a final submission check and requires authorized AWS access.



\## 6. OSCAL Documentation



The capstone includes:



\- `oscal/components/acme-health.json`

\- `oscal/profiles/cge-p-minimum.json`



The component definition documents the implemented AWS safeguards, identifies corresponding Terraform resources, and links to the signed evidence archive.



The profile selects the four distinct CMMC practices represented by the five automated policies.



Both OSCAL documents passed \*\*Compliance Trestle schema validation\*\*.



The documents reference a community-maintained CMMC Level 2 OSCAL catalog based on NIST SP 800-171 Revision 2, rather than an official DoD-published catalog.



These OSCAL artifacts provide machine-readable control-to-implementation traceability without claiming complete CMMC practice satisfaction.



\## 7. Trade-offs, Remaining Limitations, and Next Sprint



The project prioritizes automated control enforcement and verifiable evidence over unnecessary infrastructure expansion.



\### Remaining Limitations



\- \*\*Lambda safeguards:\*\* AWS sandbox concurrency quotas prevented reserved-concurrency configuration. The dead-letter queue and X-Ray tracing were implemented, leaving GAP-06 partially remediated.

\- \*\*API Gateway protection:\*\* Direct AWS WAF association is not supported for the starter's HTTP API configuration. Access logging and throttling were implemented, leaving GAP-08 partially remediated.

\- \*\*Policy coverage:\*\* Five Rego policies enforce GAP-01 through GAP-05; GAP-06 through GAP-08 do not have dedicated automated policy checks.

\- \*\*Evidence retention:\*\* GOVERNANCE mode permits authorized bypass, and evidence is stored in the same AWS account as the application.

\- \*\*Deployment integration:\*\* The deployment workflow successfully applied a no-change Terraform plan. A new infrastructure change deployed through CI has not been demonstrated, and the project currently uses two workflows instead of the single five-stage workflow specified in the brief.

\- \*\*Compliance scope:\*\* Technical safeguards do not establish full CMMC Level 2 compliance or satisfy all assessment objectives.

\- \*\*Independent verification:\*\* CI-based signing and checksum checks passed, but independent retrieval and verification of the stored evidence remain to be completed.



\### What I Would Do With Another Sprint



I would prioritize consolidating the deployment process into the required \*\*Plan → Policy Check → Apply → Sign → Upload\*\* workflow and demonstrating successful deployment of a compliant infrastructure change.



I would then implement reproducible evidence verification covering SHA-256, Cosign and the Sigstore transparency record, and S3 Object Lock retention.



Additional improvements would include expanding Rego coverage to GAP-06 through GAP-08, strengthening evidence-account isolation, and testing additional CMMC assessment objectives.



These improvements would strengthen the system's audit defensibility without expanding infrastructure beyond the project's needs.



\## 8. Conclusion



This capstone demonstrates an integrated GRC Engineering approach combining Terraform security hardening, automated CMMC-mapped policy checks, protected pull requests, GitHub Actions deployment, cryptographic evidence signing, S3 Object Lock retention, and OSCAL documentation.



The result is a working technical foundation for continuous compliance evidence generation and review, with remaining limitations explicitly documented.

