# Compliance Policy Library — CGE-P Lab 3.3

Rego policies for Open Policy Agent (OPA) that evaluate a Terraform plan
and deny it if it violates a NIST 800-53 control. Every deny message names
the resource and the control so a developer can fix the violation without
a GRC ticket.

## Policies

| File | Control | Severity | What it checks | Remediation |
|---|---|---|---|---|
| `sc28_encryption.rego` | SC-28 | High | Every `google_storage_bucket` must have a customer-managed encryption key (CMEK) | Add `encryption { default_kms_key_name = ... }` referencing a `google_kms_crypto_key` |
| `ac3_no_public.rego` | AC-3 | Critical | Buckets must have `uniform_bucket_level_access = true` and `public_access_prevention = "enforced"`; firewall rules must not open ports 22 or 3389 to `0.0.0.0/0` | Lock down bucket access settings; narrow `source_ranges` or remove the firewall rule |
| `cm6_required_tags.rego` | CM-6 | Medium | Every taggable resource must carry all four required labels: `project`, `environment`, `managed_by`, `compliance_scope` | Add the missing labels to the resource |

## Usage

Run the full test suite:
```bash
opa test -v policies/
```

Evaluate a policy against a Terraform plan:
```bash
opa eval -d policies -i <path/to/plan.json> data.compliance.sc28.deny --format=pretty
opa eval -d policies -i <path/to/plan.json> data.compliance.ac3.deny  --format=pretty
opa eval -d policies -i <path/to/plan.json> data.compliance.cm6.deny  --format=pretty
```

An empty result (`[]`) means compliant. Any message in the output is a
violation that must be resolved before the plan is applied.

## Tests

Unit tests live in `tests/` alongside each policy. Each test file covers
two cases: compliant input stays silent, non-compliant input produces a
deny message carrying the correct control ID.

## Framework

NIST SP 800-53 Rev 5

## Which file targets which cloud

| Control | GCP (Lab 3.3) | AWS (Lab 3.4) |
|---|---|---|
| SC-28 Encryption at Rest | `sc28_encryption.rego` | `sc28_encryption_aws.rego` |
| AC-3 Access Enforcement | `ac3_no_public.rego` | `ac3_no_public_aws.rego` |
| CM-6 Configuration Settings | `cm6_required_tags.rego` | `cm6_required_tags_aws.rego` |

Control IDs are portable across clouds, but resource types are not. A GCP rule run against an AWS plan passes with zero coverage, so each cloud gets its own variant under the same control ID. `scripts/policy-gate.sh` runs the AWS namespaces only.