---
status: accepted
---

# One install per lender, with the same code for every lender

The form serves several lenders, and they set things up differently. For example, one needs 1 ID from each applicant and another needs 2. Each lender gets its own Saturn install, and the form's code is the same in every install. Everything that differs between lenders lives in that lender's Saturn records: `LoanType.minimum_identifications` for the number of IDs, `LiabilityType.revolving_rate` for revolving rates, which loan categories and loan types are active, and SystemConfiguration for the name, logo, and colours.

## Considered options

- **Several lenders sharing one install.** Rejected, because every record would need a link to its lender so applicants never see another lender's loan types.
- **Per-lender constants in the code.** Chosen at first, then rejected. It meant a different copy of the code for each lender, and a code change for every lender setting.
- **A new Saturn resource for lender settings.** Not needed. The settings that differ fit on `LoanType` and `LiabilityType`.

## Consequences

A lender changes its own settings in Saturn without touching the code, and a fix to the form ships the same way to every lender. A new kind of per-lender difference needs a new field on a Saturn resource, not a constant. The 3% revolving fallback and the Grenadian tax and NIS rules stay in the code, the same for everyone, while every lender is in Grenada.
