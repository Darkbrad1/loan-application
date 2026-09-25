# CLAUDE.md: Adaptive Loan Application (Saturn)

Context for AI assistants working on this project. Read this before making changes.

## How to work with the developer

These are the developer's standing preferences. Follow them in every session.

- **Explain at a beginner level.** Write the way you'd explain it to a first-year student: plain words, short sentences, everyday comparisons. Avoid jargon; when a technical term is needed, say what it means.
- **Ask before product decisions.** If a choice changes how the app behaves for the applicant (how a feature works, what happens in an edge case, what something is called), lay out the options, say which you'd pick and why, and let the developer choose. Don't decide quietly.
- **Technical choices are yours.** If a choice doesn't change what the user sees or experiences, decide it, then say what you picked.
- **Make the requested changes and push them to `main`.** Don't paste code into the chat; edit the files in the repo, commit, and push to the `main` branch. If pushing to `main` fails for any reason, tell the developer what went wrong.
- **Flag any Saturn resource changes.** When a change needs a new resource, a new or changed field (property) on a resource, a new dropdown option (`lookup_reference`), a new AttachmentGroup, or a workflow change, say so clearly in a "Changes to make in Saturn" list: which resource, which field, its type, and the options if it's a dropdown. The developer makes these by hand, so the code won't work until they're done.
- **Label every file clearly as NEW or UPDATED,** and say which files didn't change.
- **Keep this file up to date as we go.** Whenever a change affects anything described here (components, rules, data model, decisions, lessons learned, or status), update this file in the same session and push it with the code. When a decision is made, move it out of "Decisions still waiting" and record the outcome.

## What this is

An online loan application form for a Grenadian credit union (localStorage keys use the `gccu_` prefix). Applicants apply from anywhere on their own device, not at a kiosk. It runs inside **Saturn**, a low-code platform, as a set of Saturn dynamic components.

- **Framework:** Vue 3, **Options API** (`data`, `computed`, `methods`).
- **UI libraries:** Element Plus (`el-*` inputs, buttons, selects) and Vuetify icons (`v-icon` with `mdi-*` names).
- **Currency:** EC$ (Eastern Caribbean dollars). Default country: Grenada.
- **Shared tools:** none outside the main form. All helpers, rules, and settings live inside the main form (see below).
- **Server access:** Saturn's global `Resource` class, e.g. `new Resource(this, "Asset").get(id)`, `.list()`, `.create()`, `.update()`, `.delete()`, `.request()`.
- **Saturn documentation:** `Saturn_Dynamic_Component_Documentation.md` (uploaded by the developer).

## Components

|File|Role|
|---|---|
|`AdaptiveLoanApplcationFrom.vue`|The main form (parent), called the Adaptive Loan intake form. Owns all state, the wizard steps, validation, saving, restoring, deleting, document scopes, and uploads. Also holds every shared helper and setting (IDs, money, dates, validation rules, NIS and income tax). About 5,200 lines. (The file name's spelling is how it is in the repo.)|
|`AdaptiveLoanProductSection`|Step 1: loan category and product.|
|`AdaptiveLoanApplicantsSection.vue`|Applicants step: one tab per applicant, plus that applicant's documents.|
|`AdaptiveLoanApplicantEditor.vue`|One applicant: personal details, split address, NIS number, IDs (with scans), employment and income, estimated deductions, and consents.|
|`AdaptiveLoanRequestSection`|Amount, term, purpose, and auto, home, or business details.|
|`AdaptiveLoanAssetsSection.vue`|Assets with ownership splits and per-asset documents.|
|`AdaptiveLoanLiabilitiesSection.vue`|Liabilities with responsibility splits, credit limits for revolving credit, and documents.|
|`AdaptiveLoanExpensesSection.vue`|Expenses, filtered by loan category, with documents.|
|`AdaptiveLoanCollateralSection.vue`|Collateral: a declared asset or a third-party asset, plus insurance and documents.|
|`AdaptiveLoanAllocationEditor`|Percentage split editor (ownership and responsibility) that must total 100%.|
|`AdaptiveLoanDocumentRequirements.vue`|Reusable document upload cards for one owner (the application, an applicant, an ID, or an item).|
|`AdaptiveLoanReviewSection.vue`|Final review step. Currently thin: totals and counts only.|

`AdaptiveLoanDocumentsSection` was replaced by `AdaptiveLoanDocumentRequirements` and should be deleted from Saturn.

### Everything lives in the main form

The developer tried moving shared code into a Saturn composable (`useLoanIntake`) and **reverted it**. All shared code is back inside the main form, and that's the setup to work with.

- **Don't create composables or move code out of the main form.** Keep helpers, rules, and settings in the main form.
- The settings sit as constants at the top of the main form's `<script>`: `DEFAULT_REVOLVING_RATE`, `TRACKED_RESOURCES`, `MINIMUM_IDENTIFICATIONS`, `STATUTORY_DEDUCTIONS` (NIS and tax), and `DEFAULT_COUNTRY`, plus the `createEmpty…` factories for new rows.

### The section pattern

Section components keep a local `draft` (a deep copy of `modelValue`), edit it, and emit a fresh copy with `update:modelValue`. A deep watcher re-copies when the parent changes. Consequences:

- **A section's new rows must have the same shape as the main form's `createEmpty…` factory.** If a field the main form checks is missing from the screen, the applicant sees a warning they can't fix.
- **Never store `File` objects inside section data.** The JSON copy turns them into `{}`. Staged files live only in the parent's `documentState`.
- **The parent's objects can be replaced mid-operation** when the applicant types. After async saves, `syncSavedIds(scopeKey, saved)` copies server IDs onto the current object.
- Document events (`stage-file`, `remove-file`, `request-file-upload`, `file-rejected`) are passed straight up from sections to the parent.

## Wizard steps

Choose loan → Applicants → Your request → Assets → Liabilities → Expenses → Collateral (only when required) → Documents (only when the product has application-level documents) → Review.

- **Collateral is required** for auto and home loans, or when the product has `secured === true`. At least one collateral item is then required.
- **Leaving each data step** (parties, assets, liabilities, expenses, collateral) auto-saves the draft.

## Saving, restoring, and deleting

- **The draft is created on the server** the first time it's saved: leaving the Applicants step, or uploading a document. Nothing is created on page load.
- **Restoring uses the record ID,** kept in `localStorage` as `gccu_draft_app_id`, with the step in `gccu_draft_step`. The application number is **not** used for restoring and must never be put in localStorage or a resume link, because numbers are guessable.
- **Restore reads the application's own lists** (`liability_ids`, `expense_ids`), which are rewritten on every save, so removed items can't come back.
- **Deletion:** `persistedIds` remembers every server ID saved for the draft. On each save, `deleteStaleRecords()` deletes anything no longer in the form, in `TRACKED_RESOURCES` order (records that point at others go first). Party records are never deleted, since a Party can belong to other applications. Failed deletes are retried on the next save.
- **Removed identifications** are deleted per applicant using `saved_identification_ids`.
- **Removing an applicant** clears their asset ownerships, liability responsibilities, and expenses, so validation forces them to be reassigned.
- **Use `??`, not `||`,** when restoring numbers, or a value of 0 becomes empty.

## Document uploads

Requirements come from Saturn **AttachmentGroups** named `<kind>-<name>`. A group named `<kind>-all` applies to every item of that kind.

|Group label|Applies to|
|---|---|
|`application-<loan name>`|The application, for that product (its own Documents step)|
|`applicant-<loan name>`|Each applicant, for that product|
|`identification-<ID type>`|Each ID of that type (e.g. a passport scan)|
|`asset-<asset type>`|Assets of that type|
|`liability-<liability type>`|Liabilities of that type|
|`expense-<expense type>`|Expenses of that type|
|`collateral-<asset type>`|Collateral backed by that asset type|
|`collateral-third party`|Third-party collateral only (e.g. the owner's consent)|
|`applicant-all`|Every applicant. The mandatory NIS card goes here.|

- **Scope keys** are `application` or `<kind>:<client_key>`. Upload state is keyed `<scopeKey>::<attachmentTypeId>`.
- **Uploads save only the owning record** (plus what it depends on), not the whole draft. Uploads stay locked, with a message, until that item's required fields are filled in. Co-applicant uploads also need the primary applicant to be complete first.
- **Upload endpoint:** `POST /uploads/<Resource>/<ownerId>/documents` with multipart fields **`file`, `name`, `saturn_file_type`, `tags` (`"[]"`), and `meta_data` (`"{}"`)**. Missing `tags` or `meta_data` gives a 400 whose response is `{"status":"FAILURE","type":"single","message":{}}`, with no useful message.
- After uploading, the form confirms by re-reading the owner record, tags the Upload with `saturn_file_type`, and updates the owner's `documents` list.

## Data model (Saturn resources)

Only fields this form relies on are listed.

- **Application:** `status`, `loan_category`, `loan_type_id`, `loan_name`, amounts and terms, category-specific fields, `parties`/`application_parties`, `asset_ids`, `liability_ids`, `expense_ids`, `documents`, `submitted_at`, and `application_number` (**assigned by the submit workflow; the form never sends it**).
- **Party** (shared across applications): `kind` (`PERSON` or `ORGANIZATION`), names, `business_name`, `date_of_birth`, `marital_status`, `email`, `phone`, `address`, `parish`, `country`, `nis_number`, and `ids` (links to PartyIdentification). `identification_type` and `identification_number` were removed from Party.
- **PartyIdentification:** `party` (**required**), `identification_type`, `identification_number`, `issuing_country`, `issue_date`, `expiry_date`, `is_primary`, `documents`.
- **ApplicationParty:** `party`, `role`, employment fields, `gross_monthly_income`, `nis_deduction` and `income_tax_deduction` (**calculated, not entered**), consent fields, `documents`.
- **Asset / AssetOwnership:** ownership percentages link to Party.
- **Liability:** `liability_type` (link to LiabilityType), `credit_limit` and `assessed_payment` (revolving only), `documents`.
- **LiabilityResponsibility:** responsibility percentages link to ApplicationParty.
- **LiabilityType:** `name`, `code`, `is_revolving`, `revolving_rate` (e.g. 3 or 0.03), `is_active`, `sort_order`.
- **Expense:** `expense_type` (link to ExpenseType), `applicationpartiesid`, `amount`, `frequency`, `documents`.
- **ExpenseType:** `name`, `code`, `applies_to` (loan categories; empty means all), `group`, `user_selectable`, `is_active`, `sort_order`.
- **Collateral:** `application`, `ownership` (`applicant` or `third_party`), `asset_id`, `third_party_owner`, `third_party_relationship`, `category` (derived from the asset type, with no dropdown), `estimated_value`, the `insurance_*` fields, `status`, and `documents`.

Dropdown options come from each resource's property `lookup_reference`, loaded with `loadResourceProps()` and parsed by `options()`.

## Business rules

- **Revolving credit** (credit cards, overdrafts): the assessed repayment is `credit_limit × rate`. The rate is the type's `revolving_rate`, falling back to `DEFAULT_REVOLVING_RATE` (3%).
- **NIS and income tax are calculated** from gross monthly income and shown read-only. The settings are in `STATUTORY_DEDUCTIONS`, near the top of the main form:
    - **NIS:** 6.25% of income up to EC$5,200 (maximum EC$325). Self-employed pay 13.5%. Nothing for the unemployed, the retired, anyone under 16, or anyone at or over 65. 65 is used because pensionable age is being phased from 60 to 65 by birth year, and 65 never understates NIS.
    - **Income tax:** 0% on the first EC$3,000 a month, 10% on the next EC$2,000, and 30% above EC$5,000.
    - Worked examples that must hold: EC$4,000 gives EC$250 NIS, EC$100 tax, EC$3,650 net. EC$6,000 gives EC$325 NIS, EC$500 tax, EC$5,175 net.
- **IDs:** `MINIMUM_IDENTIFICATIONS` (currently 1) is one rule for the whole credit union, not per product. No duplicate ID types, no expired IDs, and exactly one primary.
- **Parish** is required only when the country is Grenada.
- **Business-only expense types** are hidden on personal loans, via `applies_to`.
- **Third-party collateral assets** are saved as an Asset owned 100% by the third-party Party, and are **kept out of the application's `asset_ids`** so they don't count toward the applicant's net worth.
- The server should **recalculate** `assessed_payment`, `nis_deduction`, and `income_tax_deduction` before underwriting uses them. The browser's figures are estimates.

## Lessons learned (don't repeat these)

- **The developer can't change SystemConfiguration.** Don't put settings there; use constants in the code or fields on resources the developer controls.
- **Ask for the Network tab response body** when a Saturn request fails. The upload 400 was only solved by comparing against a working request.
- **Required links need a save order.** PartyIdentification requires `party`, so the Party is saved first, then the IDs, then `Party.ids` is updated.
- **Element Plus `el-radio`** changed its value prop between versions (`label` vs `value`). Prefer `el-select` or buttons for choices.
- **The Saturn guide's `FormField`** is unclear about its update event payload (`{ property, data }` vs the value). Test before relying on it.
- **Saturn composables can't call each other** and can't see the `composables` object. A split into five composables failed, and then a single `useLoanIntake` composable was reverted too. Keep all shared code in the main form.
- **Don't use the spread operator (`...`)** anywhere in Saturn code: it fails at runtime ("Spread syntax requires ...iterable[Symbol.iterator] to be a function"). Use `concat`, `slice()`, `Object.assign`, and `Array.from(new Set(...))` instead. To be safe, also avoid destructuring by position (`const [a] = list`, `for (const [i, x] of list.entries())`); use indexes instead.
- **When a Saturn error mentions a missing name, log the object first** (e.g. `console.log(composables)`) to see its real shape before guessing.
- **Dates:** compare `YYYY-MM-DD` strings. `new Date().toISOString()` is UTC, which is 4 hours ahead of Grenada, so "today" is wrong after 8pm. Prefer a local date.

## Status

### Done

- Initial review and bug fixes: deletion and restore, zero values, stale assignments, and leftover collateral.
- LiabilityType and ExpenseType resources, credit limits, the revolving assessed payment, and expense filtering by loan category.
- Server-assigned application numbers, created in the submit workflow `MXHGYH`.
- Per-section document uploads with the reusable requirements component, and per-item saving.
- Multiple IDs (PartyIdentification), the split address, the NIS number, and calculated NIS and income tax.
- Collateral: removed the category dropdown, added insurance details, and added third-party collateral.
- Tried moving shared code into a `useLoanIntake` composable, then reverted. Everything lives in the main form again.
- Rebuilt `AdaptiveLoanCollateralSection.vue` to match the main form: the owner choice (applicant or third party), the third-party asset and owner fields, and the insurance fields. The old Category dropdown is gone. The repo copy had been an old version, so the form warned about insurance fields that weren't on screen.

### Decisions still waiting on the developer

1. **Income tax rates:** 10% or 15% for the middle band, and 30% or 28% for the top band. Sources disagree; confirm with the Inland Revenue Division.
2. **Minimum IDs:** 1 or 2.
3. **Credit bureau consent:** signed in the form, or a form downloaded, signed, and uploaded. Skipped for now.
4. **Review screen:** Saturn's `ResourceViewInline` or expanding our own.
5. **Input boxes:** Saturn `FormField` or our hand-built ones.
6. **Upload boxes:** Saturn `typed_file_upload` or ours.
7. **Submit failure:** have the workflow set the status and the number together (recommended), or keep the current order and let officers spot stuck applications.

### Next up

- **Projected insurance expense:** turn each collateral insurance premium into a read-only monthly expense marked `is_projected`.
- **Remove spread syntax (`...`):** the reverted code still uses it (about 25 places in the main form, plus one each in `AdaptiveLoanAllocationEditor.vue` and `AdaptiveLoanApplicantsSection.vue`), and Saturn fails on it at runtime.
- **Add missing files to the repo:** `AdaptiveLoanDocumentRequirements.vue` is used by the main form but isn't in the repo yet.
- **Cleanup:** remove the dead CSS from the old Documents screen (e.g. `.legacy-queue`), and reorganize the main form into labelled sections.

### Later (Phase 3: back office)

Valuations and appraisals (including the minimum required value, and vehicle appraisal for auto loans), an underwriting summary (DSR and LTV, which needs a product interest rate and monthly-equivalent amounts), staff search by application number, and department queues by loan category.

### Known issues

- **Submit order:** `submit()` sets status "submitted" before the workflow runs, so a failed workflow can leave an application submitted without a number.
- **Changing an item's type** after uploading leaves the old document linked.
- **Drafts only resume in the same browser.** Cross-device resume would need an emailed link or a login.