# compliant-gcs-bucket

A Terraform module that provisions a hardened Google Cloud Storage bucket with
the security baseline built into the module itself, not left to the consumer.

## Controls enforced

- **SC-12** — The module establishes and owns its own KMS keyring and crypto
  key, rather than relying on Google-managed encryption keys.
- **SC-13 / SC-28** — Data at rest is encrypted using that customer-managed
  key (CMEK), which rotates automatically every 90 days.
- **AC-3** — Uniform bucket-level access and public access prevention are
  enforced, so the bucket can never be made publicly accessible and access
  can't fall back to legacy per-object ACLs.
- **AU-11** — A retention policy is applied to every bucket the module
  creates, with production environments required to retain objects for at
  least 365 days.
- **CM-6** — Required compliance labels (`project`, `environment`,
  `managed_by`, `compliance_scope`) are merged onto every bucket and cannot
  be overridden or removed by the consumer.

## Usage

Consumers call this module and supply only business settings (project,
environment, retention period, bucket suffix). The security controls above
are hardcoded in the module body and are not exposed as configurable inputs.