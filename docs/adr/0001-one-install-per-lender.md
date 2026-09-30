# One install per lender, with lender differences kept in the code

Status: proposed

The form serves several lenders, and they set things up differently (for example one needs 1 ID and another needs 2). Each lender gets its own Saturn install, and anything that differs between lenders is a constant at the top of the main form, changed in the code when the form is set up for that lender. Logo and colours are the exception, since they already come from each install's SystemConfiguration.

## Considered options

- **Several lenders sharing one install.** Rejected: every record would need a link to its lender, so applicants never see another lender's loan types.
- **A setting on `LoanType` (for example `minimum_identifications`).** Rejected in favour of a code constant, because it keeps the lender's setup with the developer and adds no Saturn fields. (Reason to be confirmed by the developer.)
- **A new Saturn resource for lender settings.** Rejected for the same reason.

## Consequences

Each lender runs a copy of the code that differs only in the constants. A change to those constants means editing and republishing the main form for that lender, so lender staff can't change them themselves.
