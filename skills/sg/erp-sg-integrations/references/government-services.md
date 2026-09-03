# Singapore government-service integrations

Official interfaces, authentication requirements, filing formats, and submission channels can change. Verify them with the relevant authority and the entity's filing provider at implementation time.

Official starting points:

- [ACRA: Filing financial statements in XBRL format](https://www.acra.gov.sg/manage/companies/legal-requirements-common-offences/filing-financial-statements-in-xbrl-format/)
- [IRAS: Digital services](https://www.iras.gov.sg/digital-services/)

## Separate generation from submission

Treat these as distinct stages:

1. Close and reconcile the d6e ledger.
2. Generate the filing dataset or file using the approved accounting mapping and statutory-parameter versions.
3. Validate the artifact against the applicable schema or business rules.
4. Obtain preparer and approver sign-off.
5. Submit through an officially supported channel or approved filing provider.
6. Store the acknowledgement, receipt, filing reference, and final artifact digest.

Do not claim that d6e can submit directly to ACRA or IRAS unless the target connector and end-to-end acceptance have been verified. Manual portal submission remains distinct from automated integration.

## SQL records

Track at least:

- entity, UEN, tax or filing account identifier, filing type, and period
- ledger-close ID, extraction time, mapping version, and statutory-parameter versions
- generated storage file ID, MIME type, schema or taxonomy version, and digest
- validation result, preparer, approver, and approval time
- submission channel, provider, submitted by, submitted at, and request ID
- authority filing reference, acknowledgement, accepted or rejected status, and receipt file ID
- superseded filing ID and correction reason

Keep identity-provider credentials and signing secrets outside ERP SQL tables. Log references and outcomes without storing reusable authentication material.

## Controls

- Default to no submission without explicit authority and an approved artifact.
- Prevent changes to the submitted artifact without creating a new version and approval.
- Keep rejection and validation messages associated with the exact artifact digest.
- Model amendments and resubmissions as new filing versions linked to the original.
- Do not advance a filing to accepted based only on client-side upload completion or an HTTP success.
- Retain the source snapshot and drill-down keys required by `erp-sg-accounting`.

## Verification

Distinguish local schema validation, provider sandbox acceptance, production transmission, authority acknowledgement, and accounting reconciliation. If any stage is unavailable, report it as unverified instead of inferring success from an earlier stage.
