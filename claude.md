# CLAUDE.md: Adaptive Loan Application (Saturn)

Context for AI assistants working on this project. Read this before making changes.

## How to work with the developer

These are the developer's standing preferences. Follow them in every session.

- **Explain at a beginner level.** Write the way you'd explain it to a first-year student: plain words, short sentences, everyday comparisons. Avoid jargon; when a technical term is needed, say what it means.
- **Ask before product decisions.** If a choice changes how the app behaves for the applicant (how a feature works, what happens in an edge case, what something is called), lay out the options, say which you'd pick and why, and let the developer choose. Don't decide quietly.
- **Technical choices are yours.** If a choice doesn't change what the user sees or experiences, decide it, then say what you picked.
- **Make the requested changes and push them to `main`.** Don't paste code into the chat; edit the files in the repo, commit, and push to the `main` branch. If pushing to `main` fails for any reason, tell the developer what went wrong. **Exception:** experiments the developer puts on the `dev` branch are committed and pushed to `dev`, not `main`, until the developer says to merge them. The **`design` branch** (made from `dev`) holds the user-friendly redesign, and **`design-style-tags`** (made from `design`) is the same redesign with every style in `<style>` tags and no Tailwind. The **`redesign`** branch (made from `design-style-tags`) is a full visual redesign; push design work there until the developer says otherwise.
- **Flag any Saturn resource changes.** When a change needs a new resource, a new or changed field (property) on a resource, a new dropdown option (`lookup_reference`), a new AttachmentGroup, or a workflow change, say so clearly in a "Changes to make in Saturn" list: which resource, which field, its type, and the options if it's a dropdown. The developer makes these by hand, so the code won't work until they're done.
- **After finishing anything, list the files changed.** Label each one NEW, UPDATED, or DELETED, with a line on what changed, and say which files didn't change.
- **Keep this file up to date as we go.** Whenever a change affects anything described here (components, rules, data model, decisions, lessons learned, or status), update this file in the same session and push it with the code. When a decision is made, move it out of "Decisions still waiting" and record the outcome.

## What this is

An online loan application form for lenders in Grenada (credit unions and other lending institutions). Applicants apply from anywhere on their own device, not at a kiosk. It runs inside **Saturn**, a low-code platform, as a set of Saturn dynamic components.

- **One Saturn install per lender, the same code for every lender** ([ADR 0001](docs/adr/0001-one-install-per-lender.md)). Everything that differs between lenders lives in that lender's Saturn records: which loan categories and loan types are active, `LoanType.minimum_identifications`, `LiabilityType.revolving_rate`, and SystemConfiguration (name, logo, colours). Don't add per-lender constants to the code.
- **The project's words** are in [`CONTEXT.md`](CONTEXT.md) (Lender, Loan category, Loan type, Primary Applicant, Business Borrower, Guarantor). Say "loan type", not "product".
- **The September 2026 design session** (every decision, with the reasoning) is in [`docs/grilling/2026-09-30-loan-intake-gaps.md`](docs/grilling/2026-09-30-loan-intake-gaps.md). Its "Settled" list is the spec for this batch.

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
|`AdaptiveLoanApplicantsSection.vue`|Applicants step: one tab per applicant, plus that applicant's documents, then the primary applicant's **references** (a personal reference and next of kin). Shows a note that only people listed here can own assets, so owners who aren't borrowing are added as Third Party Owners.|
|`AdaptiveLoanApplicantEditor.vue`|One applicant: personal details, membership, citizenship and residency, address (with years there, housing, dependants, previous and mailing addresses), NIS number, IDs (with scans), employment (type, start date, previous job), pay and pay frequency, other income (with proof), estimated deductions, guarantee (guarantors), declarations (PEP, bankruptcy, and so on), and consents. A Third Party Owner gets a short form instead (see Business rules).|
|`AdaptiveLoanRequestSection`|Amount, term, purpose, auto or home details (with registration/chassis or block and parcel/deed numbers), purchase details (price, down payment and its source, seller), and business details (saved on the business's own Party).|
|`AdaptiveLoanAssetsSection.vue`|Assets with ownership splits, identifiers for vehicles and land, existing loans secured on them, and per-asset documents. On loans that need collateral, each asset has a "Use this asset as collateral" checkbox that reveals the insurance questions, collateral notes, and collateral documents. The vehicle or property being bought is shown read-only and is always collateral.|
|`AdaptiveLoanLiabilitiesSection.vue`|Liabilities with responsibility splits, credit limits for revolving credit, "paid off by this loan", "secured on" (an asset), notes, and documents.|
|`AdaptiveLoanExpensesSection.vue`|Expenses, filtered by loan category, with a "shared household expense" option and documents. Shows the projected collateral insurance read-only.|
|`AdaptiveLoanAllocationEditor`|Percentage split editor (ownership and responsibility) that must total 100%.|
|`AdaptiveLoanDocumentRequirements.vue`|Reusable document upload cards for one owner (the application, an applicant, an ID, an income, or an item).|
|`AdaptiveLoanReviewSection.vue`|Final review step: every section with everything entered (labels, not IDs), an "Edit" link back to each step, and a monthly summary. The main form builds the content (`reviewSummary`).|

**On `dev`, inputs are Saturn's `FormField`** (see below). The Product step (clickable cards) and the Review step (read-only) have no inputs.

`AdaptiveLoanDocumentsSection` was replaced by `AdaptiveLoanDocumentRequirements`, and `AdaptiveLoanCollateralSection` was removed (collateral is now on the Assets step). Both are deleted from Saturn and from the repo.

### Everything lives in the main form

The developer tried moving shared code into a Saturn composable (`useLoanIntake`) and **reverted it**. All shared code is back inside the main form, and that's the setup to work with.

- **Don't create composables or move code out of the main form.** Keep helpers, rules, and settings in the main form.
- The settings sit as constants at the top of the main form's `<script>`: `DEFAULT_REVOLVING_RATE` (the same 3% fallback for every lender), `TRACKED_RESOURCES`, `MONTHLY_FACTORS` (with `monthlyAmount()`), `PURCHASE_ASSET_STATUS`, `COLLATERAL_INSURANCE_CODE`, `GUARANTOR_ROLE`, `DEFAULT_MINIMUM_IDENTIFICATIONS` (1, used when the loan type sets none), `NOT_WORKING_STATUSES` (unemployed, retired, student), `STATUTORY_DEDUCTIONS` (NIS and tax), `DEFAULT_COUNTRY`, `TEST_MODE_AVAILABLE`, and `THIRD_PARTY_OWNER_ROLE`, plus the `createEmpty…` factories for new rows (applicant, identification, income, reference, collateral, application).

### The section pattern

Section components keep a local `draft` (a deep copy of `modelValue`), edit it, and emit a fresh copy with `update:modelValue`. A deep watcher re-copies when the parent changes. Consequences:

- **A section's new rows must have the same shape as the main form's `createEmpty…` factory.** If a field the main form checks is missing from the screen, the applicant sees a warning they can't fix.
- **Never store `File` objects inside section data.** The JSON copy turns them into `{}`. Staged files live only in the parent's `documentState`.
- **The parent's objects can be replaced mid-operation** when the applicant types. After async saves, `syncSavedIds(scopeKey, saved)` copies server IDs onto the current object.
- Document events (`stage-file`, `remove-file`, `request-file-upload`, `file-rejected`) are passed straight up from sections to the parent.

### Saturn `FormField` (on `dev`)

Every section renders its inputs with Saturn's built-in `FormField` instead of `el-input`, `el-select`, `el-checkbox`, and so on. The request section was tested in Saturn first and worked; the other sections followed the same pattern.

- **The main form passes Saturn's field definitions** to the sections: `applicationProps` (Application) and `resourceProps` (every resource by name: Application, Party, ApplicationParty, PartyIdentification, Asset, Liability, Expense, Collateral, IncomeSource, Reference). The request section's `FIELDS` list says which resource each field's definition comes from (e.g. business fields use Party, vehicle identifiers use Asset).
- **Each field uses Saturn's own definition** of that property when it exists, with our label, so the input type and dropdown options follow the resource settings in Saturn.
- **Identification type uses Saturn's own `PartyIdentification.identification_type` setup.** The form's own list didn't work in Saturn. Don't point it at `Party.ids`: that's the link to the applicant's saved ID records, so the dropdown shows whole records as `[object Object]`. Using a type twice is caught when the applicant continues.
- **`FormField`'s own option lists don't work in Saturn.** The guide's `lookup_type: "values"` with `map.values` showed broken dropdowns. **A `FormField` dropdown only works with Saturn's own definition of the property.**
- **So dropdowns whose choices the form decides stay as `el-select`:** role (no "Primary Applicant"), "Owner is a" (person or business), liability type (active LiabilityType records), "Secured on" (this application's assets), expense type (filtered by loan category, with group headings), the responsible applicant, and the owner in ownership splits. Saturn's own definitions of these would list every record in the system (including other applicants' names), so they can't be used.
- **Without a Saturn definition,** a basic one is built from the Saturn guide: `type: "number"`, `type: "date"`, `type: "boolean"` with `input_properties.type: "check-box"`, or `input_properties.type` of `input`/`textarea`. A dropdown without a Saturn definition becomes a text box.
- **Configs are reused while unchanged** (`fieldCache`, set in `created()`), so `FormField` isn't handed a new object on every keystroke. The request section does the same with a computed `fields` map.
- **The same helpers are copied into each section** (`valueOf`, `toDateString`, `savedProperty`, `field`), since Saturn components can't share code. Keep the copies the same.
- **The update event payload is unclear** in the guide (the value, or `{ property, data }`), so `valueOf()` accepts both. Dates are stored as `YYYY-MM-DD` strings (the local date for `Date` objects).
- Each `FormField` sits inside an `el-form-item`, which shows the label and required star.
- **What was lost:** `FormField` has no min/max, so amounts, terms, and percentages aren't limited as they're typed (the main form still checks them on Continue). Placeholders are gone from `FormField` inputs.

## Test mode

A **Test mode** switch in the header fills each step with sample data as you reach it (`fillTestData()`). It only fills empty fields, and only adds an asset, liability, or expense when the list is empty, so nothing typed is overwritten. It picks an auto loan when there is one, so collateral gets exercised.

- **Documents are optional in test mode:** `firstIncompleteScope()` returns nothing, so Continue and Submit don't ask for uploads. Uploading still works.
- **A second switch, "Remember draft on reload",** appears while test mode is on. It's **off** by default: the draft's location isn't kept in `localStorage` (and any saved one is removed), so reloading the page starts from the beginning. The draft is still saved on the server. Turning it on, or turning test mode off, remembers the current draft again. All `localStorage` writes go through `rememberDraftLocation()`.
- Test mode itself is off after every reload.

- **The switch shows while `TEST_MODE_AVAILABLE` is `true`** (top of the main form). **Set it to `false` before real applicants use the form.**
- Asset owners and responsible applicants are filled with the primary applicant, whose IDs exist once the Applicants step has been saved.
- It also fills membership, residency, housing, employment, pay, the two references, and (for auto and home loans) a purchase price, down payment, and seller, so the purchase asset gets exercised. Insurance is filled on every collateral asset.

## The redesign (on the `design` branch)

The form was redesigned for people who aren't comfortable with technology. Decisions: **short screens** (one topic each), **Yes/No questions first** that skip a section on "No", and **everyday words**. Saving, restoring, rules, and Saturn records are unchanged.

- **Steps and screens:** the wizard still has the same steps (`path`), now titled Choose a loan, About you, The loan you need, Things you own, Money you owe, Your bills, (Documents), Check and send. Each step is split into short screens by `screensFor(stepId)`; `screenIndex` is the position within the current step. `next()` checks the screen (`screenIssue`), and on a step's last screen also the whole step (`validate()`), then saves and moves on as before. `back()` goes back one screen. Always change steps with `setStep(index, screenIndex)`.
- **About you:** the applicant's details are 8 screens, one per part in `APPLICANT_PARTS` (about, home, membership, ids, job, pay, declarations, consent); `applicantPartIssue(person, part)` checks each, and `applicantIssue()` checks them all. Then "Is anyone else on this loan?" (Yes/No); on Yes, a "people" screen (role and name for each), then each person's own screens (a Third Party Owner gets one "owner" screen). Then the references screen.
- **Yes/No questions ("gate" screens):** `answers` holds `others`, `assets`, `debts`, `bills` (null until answered). "No" clears that section on Next (`clearForNo`), with a warning if anything was added. Secured loans don't get the assets question (they must list something). Answers aren't saved to Saturn; restore works them out from the data and the saved step.
- **Sections take a `screen` prop** (the applicant editor, applicants section, request section) to show just one part, and a `showErrors` prop that shows "This is needed" under empty required boxes after a failed Next (`need()`).
- **Simpler inputs:** ownership and "who pays" use the allocation editor's `simple` mode ("Just me" / "Shared with someone else"; percentages only when shared). Bills only ask "Who pays it?" when someone else is borrowing (with "Shared by the household" as a choice). Optional details (descriptions, tax number, mailing address, ID issue date and country, insurance extras, notes) are behind "+ Add ..." links. Loan products are big cards; a category with one product picks it automatically. The list screens start with one empty item after a "Yes".
- **Uploads on a person's ID screen** unlock once their name, contact details, NIS number, and IDs are in (`readyForUploads`), not after consent.
### The full visual redesign (`redesign` branch)

The developer asked for a complete redesign (anything may change except: colours and logo come from the system configuration, and the same components keep the same names). Logic, saving, and Saturn records are unchanged.

- **Styling rules (Saturn):** **Tailwind classes + flex only. No `<style>` CSS and no CSS grid.** The developer confirmed Saturn doesn't apply the components' `<style>` blocks (only some old, cached rules from the original version still show). So every element is styled in its template: Tailwind utility classes (from Saturn's documented set: flex, gap, padding, margin, text sizes and weights, rounded, border, grays and other colour shades, `md:` prefixes) for layout, spacing, and text; **inline `style`** for exact sizes (for example `min-height:56px`) and for the brand colours. The style tags are left empty with a note. The named classes (`choice`, `item-card`, ...) are kept on the elements so the code stays readable, but they have no CSS. Selected and open states are `:style` bindings (see the answer cards and list cards). **Never put `style="..."` and `:style="..."` on the same element**: Saturn drops the fixed one (that's what stretched the answer circles and shrank the loan boxes). Combine them as `:style="['fixed styles', changingStyles]"`. Saturn's page also passes flex settings down to elements inside the form, which stretched the answer ticks: fixed-size things (ticks, icons) pin their size inline (`width`, `min-width`, `max-width`, `height`, `flex:0 0 <size>`), and the words beside them are a `choice-text` span with `flex:1 1 auto`. Claude's preview includes a deliberately hostile flex rule to catch this. Every section that uses answer cards needs its own `choices(key)` method, or opening the card fails with a `_ctx` error. The form uses `el-form size="large"` for bigger boxes. **Loading spinners use Vuetify's `<v-progress-circular indeterminate>`**, which brings its own animation; an icon with a CSS class like `spin` doesn't turn in Saturn.
- **Colours from the system configuration:** `brandStyle` (an inline style on the form's root, so it always applies) turns the SystemConfiguration colours into CSS variables that the inline styles use, for example `style="background:var(--brand)"`. The primary colour is `--brand` (buttons, selected answers, borders). The secondary colour is `--accent` (the progress bar). The form also works out:
  - `--brand-ink`: black or white, whichever is readable on `--brand` (`readableTextOn()`)
  - `--brand-text`: the brand colour for text on white, darkened if the brand colour is too light to read
  - `--brand-soft` / `--brand-tint`: light tints for backgrounds (`tint()`)
  The logo (and company name) sits in a header in the brand colour (the developer's choice), with text in `--brand-ink`; on phones the name is hidden when there's a logo.
- **Layout:** one centred column (max 680px) on a light grey page, the screen inside a white panel. At the top: a "Back" link, "Part 2 of 7 · About you", and a segmented progress bar (one segment per part, the current one filling up by screen, `segmentFill()`). At the bottom: one full-width primary button ("Start my application", "Continue", or "Send my application"). Messages (`alert`) show as a coloured box under the screen title. No sidebar, no Save draft button; a "Saved" chip appears in the header once a draft exists.
- **Welcome screen:** the loan step starts with a `welcome` screen (what to have ready, about 15 minutes, answers saved as you go) until a draft has been saved (`formData.id`).
- **Answer cards instead of dropdowns:** a question whose Saturn list has 2 to 7 answers is shown as big tappable cards (`.choice-list` / `.choice`, radio style); longer lists (country, parish, relationship) stay as Saturn's FormField dropdown. Each section has a `choices(key)` helper that returns the list from `lookups`, or null to fall back to FormField. Yes/No questions, "Just me / Shared", "Who pays it?", the kind of debt, "Which thing?", and the consent tick boxes use the same cards (consent uses the square `.choice-mark.square`).
- **Fold-up lists:** things you own, debts, and bills show each item as a one-line summary card (icon, name, summary, Change / Remove). When the item still has documents to upload (`docsNeeded()`), the summary says how many and the button reads "Upload documents"; the uploads are inside the open card. Only one is open at a time (`openKey`); "Done" folds it up, or shows what's missing (`checking`). A failed Continue opens the first item with missing details, and folded items with missing details say so. IDs, other income, people, and references stay open.
- **Named classes** (no CSS; each element carries its own Tailwind classes and inline styles): `panel`, `screen-title`, `screen-help`, `hint`, `helper`, `question`, `subheading`, `field-grid` (two boxes side by side when there's room), `choice-list` (`inline` for short answers), `choice`, `choice-mark`, `item-card` (`open`, `needs-work`), `item-head`, `item-icon`, `item-text`, `item-actions`, `link-btn`, `small-btn`, `add-button`, `more-link`, `soft-box` (`info`, `warn`), `sum-row`, `tag`, `loan-card`, `check-*` (review page), and `btn btn-primary`.
- **The document upload card** (`AdaptiveLoanDocumentRequirements`) keeps its own look for now: it uses its own class names (`.document-section`, `.requirement-card`, ...), not the redesign's Tailwind classes and inline styles. Its file is in the repo, so it can be restyled like the other sections. An optional requirement (the NIS card for students) shows "Optional" and doesn't stop the card counting as complete.
- **Checking the look:** Claude keeps a local preview (Vite + Element Plus + Vuetify + a Tailwind build of the classes used, with Saturn mocked and no component CSS) and takes screenshots at phone and desktop widths before pushing.

## Wizard steps

Choose loan → Applicants → Your request → Assets → Liabilities → Expenses → Documents (only when the product has application-level documents) → Review.

- **There is no Collateral step.** Collateral is marked on the Assets step, with a checkbox on each asset.
- **Collateral is required** for auto and home loans, or when the product has `secured === true`. At least one asset must then be marked as collateral before leaving the Assets step. On other loans the checkbox isn't shown.
- **Leaving "Your request"** runs `syncPurchaseAsset()`: for auto and home loans with a purchase price, the vehicle or property being bought becomes an asset (status `To be purchased`, `is_purchase` in the form), valued at the price, owned 100% by the primary applicant, and marked as collateral. Its details follow the request step. Without a price (e.g. a refinance) it's removed.
- **Leaving each data step** (parties, assets, liabilities, expenses) auto-saves the draft.

## Saving, restoring, and deleting

- **The draft is created on the server** the first time it's saved: leaving the Applicants step, or uploading a document. Nothing is created on page load.
- **Restoring uses the record ID,** kept in `localStorage` as `<company>_draft_app_id`, with the step in `<company>_draft_step`. `<company>` is the SystemConfiguration name in lower case with anything that isn't a letter or number turned into `_` (for example `grenada_credit_union_`), or `loan` when there's no name (`draftKey()`). Renaming the company makes browsers forget older drafts. The application number is **not** used for restoring and must never be put in localStorage or a resume link, because numbers are guessable.
- **Restore reads the application's own lists** (`asset_ids`, `liability_ids`, `expense_ids`, `reference_ids`) and `ApplicationParty.income_ids`, which are rewritten on every save, so removed items can't come back.
- **Save order:** Party, IDs, ApplicationParty, then its IncomeSource records (they need the ApplicationParty ID); the business Party; the Application; assets, collateral, and projected insurance expenses; liabilities; expenses; references (they need the Application ID); then the ID lists on the Application.
- **Only changes are sent.** `upsert()` remembers what it last wrote to each record (`savedPayloads`, keyed by resource, ID, and the fields sent) and skips a request when nothing changed; `saveRecord()` returns `{ id, wrote }`. When a person's Party or IDs change, their ApplicationParty is sent again anyway (`force`), so Saturn's `partysnapshot` workflow refreshes. A new Business Borrower link is always sent twice. The first save after a restore sends everything.
- **Independent records are saved side by side** (`Promise.all`): the people, each person's IDs and incomes, the assets (each with its own ownerships, collateral, and insurance in order), the liabilities, the expenses, and the references. The order that matters stays: Party before its IDs, ApplicationParty before its incomes, people before the Application, the Application before assets and references.
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
|`collateral-<asset type>`|Assets of that type marked as collateral (shown in the asset's card)|
|`collateral-third party`|Collateral assets that a Third Party Owner owns all or part of (e.g. the owner's consent)|
|`income-<income type>`|Each other income of that type (e.g. a pension statement)|
|`applicant-all`|Every applicant. The mandatory NIS card goes here.|

- **Scope keys** are `application` or `<kind>:<client_key>`. Upload state is keyed `<scopeKey>::<attachmentTypeId>`. Collateral scopes use the asset's key (`collateral:<asset client_key>`), and their owner record is `asset.collateral`.
- **Third Party Owners have no document or ID scopes** for now.
- **Uploads save only the owning record** (plus what it depends on), not the whole draft. Uploads stay locked, with a message, until that item's required fields are filled in. Co-applicant uploads also need the primary applicant to be complete first.
- **Upload endpoint:** `POST /uploads/<Resource>/<ownerId>/documents` with multipart fields **`file`, `name`, `saturn_file_type`, `tags` (`"[]"`), and `meta_data` (`"{}"`)**. Missing `tags` or `meta_data` gives a 400 whose response is `{"status":"FAILURE","type":"single","message":{}}`, with no useful message.
- After uploading, the form confirms by re-reading the owner record, tags the Upload with `saturn_file_type`, and updates the owner's `documents` list.

## Data model (Saturn resources)

Only fields this form relies on are listed.

- **Application:** `status`, `loan_type_id` (the loan category and name are read from the linked loan type; the form keeps them in memory as `loan_category` and `loan_name` but doesn't save them), amounts and terms, category-specific fields, `purchase_price`, `down_payment_amount`, `source_of_funds` and `source_of_funds_details` (asked only with a down payment), `seller_type`, `seller_name`, `parties` (links to ApplicationParty, including the Business Borrower), `asset_ids`, `liability_ids`, `expense_ids`, `reference_ids`, `documents`, `submitted_at` (set by the submit workflow), `consent_forms_sent_at` (for the future consent workflow), and `application_number` (**assigned by the submit workflow; the form never sends it**). Credit cards and overdrafts send `requested_credit_limit`, `is_secured_by_savings`, `secured_savings_amount`, `linked_account_number` (and for cards `name_on_card`, `card_collection_method`) **instead of** `requested_loan_amount`, `requested_loan_term`, and `repayment_frequency`. Student loans also send `institution_name`, `institution_country`, `program_name`, `program_level`, `student_id_number`, `enrollment_start_date`, `expected_graduation_date`, `tuition_amount`, `tuition_currency`, `other_study_costs`, `disbursement_schedule`, `pay_to_institution`, `grace_period_months`. `interest_rate` and `interest_rate_type` are staff overrides the form never sends. `loan_name`, `loan_category`, `business_party`, and `preferred_contact_method` were removed from Application.
- **Party** (shared across applications): `kind` (`PERSON` or `ORGANIZATION`), names (`legal_name` for a business; `business_name` and `years_at_address` were removed), `date_of_birth`, `marital_status`, `email`, `phone`, `address`, `parish`, `country`, `nis_number`, `is_member`, `member_number`, `citizenship`, `residency_status`, `tin`, `address_since` (date they moved in; asked as "When did you move there?"), `previous_address`/`previous_parish`/`previous_country` (only under 2 years at the address), `mailing_address`, and `ids` (links to PartyIdentification). **A business Party** also uses `legal_name`, `trading_name` (empty means the same as the legal name), `industry` (typed), `business_license_number`, `website`, `registration_number`, `business_type`, `incorporation_date`, and `number_of_employees`.
- **PartyIdentification:** `party` (**required**), `identification_type`, `identification_number`, `issuing_country`, `issue_date`, `expiry_date`, `is_primary`, `documents`.
- **ApplicationParty:** `party`, `role` (including `Third Party Owner`, `Guarantor`, and `Business Borrower`), `ownership_percentage` (**the share of the loan, see Business rules**), `preferred_contact_method` (everyone, from its Saturn list), `signing_authority` (the Primary Applicant's position in the business, business loans only), `relationship_to_applicant` (Third Party Owners only), `housing_status`, `number_of_dependants`, `employment_status`, `employment_type`, `employment_start_date`, `employer_name`, `job_title`, `years_employed` (**worked out from the start date**), `previous_employer_name`/`previous_job_title`/`previous_employment_years` (only under 2 years in the job), `gross_pay` and `pay_frequency` (entered), `gross_monthly_income` (**worked out from them**), `annual_revenue` (self-employed), `guarantee_type` and `guarantee_amount` (guarantors), `income_ids` (links to IncomeSource), `is_pep`, `pep_details`, the four `declared_*` yes/no fields and `declaration_details`, `nis_deduction` and `income_tax_deduction` (**calculated, not entered**), consent fields, `documents`.
- **IncomeSource:** `application_party`, `income_type`, `description`, `amount`, `frequency`, `monthly_equivalent` (worked out), `documents`.
- **Reference:** `application`, `reference_type` (`Personal reference` or `Next of kin`), `name`, `relationship`, `phone`, `email`, `address`.
- **Asset / AssetOwnership:** ownership percentages link to Party. Asset also has `registration_number` and `chassis_number` (vehicles), `block_and_parcel` and `deed_number` (land and property), and `status` (`Declared`, or `To be purchased` for the asset a purchase loan is buying).
- **Liability:** `liability_type` (link to LiabilityType), `credit_limit` and `assessed_payment` (revolving only), `monthly_equivalent` (worked out), `balance_as_of` (**set to the application date on save**), `description`, `is_to_be_paid_off`, `is_secured` and `secured_asset` (link to the Asset it's secured on), `documents`.
- **LiabilityResponsibility:** responsibility percentages link to ApplicationParty.
- **LoanCategory:** `code`, `name`, `description`, `icon`, `minimum_interest_rate`, `maximum_interest_rate`, `is_revolving`, `is_active`, `sort_order`. The category boxes on the first step come from the active records that have at least one loan type on offer. If none load, the first step says "We can't show our loans right now. Please try again later." (there are no built-in categories).
- **LoanType:** `name`, `status` (only "Active" or empty is offered), `loan_category` (link to LoanCategory; the old text `category` was removed), amount and term limits, `default_interest_rate`, `interest_rate_type`, `revolving_rate`, `requires_guarantor`, `minimum_identifications` (empty means 1).
- **LiabilityType:** `name`, `code`, `is_revolving`, `revolving_rate` (e.g. 3 or 0.03), `is_active`, `sort_order`.
- **Expense:** `expense_type` (link to ExpenseType), `applicationpartiesid`, `amount`, `frequency`, `monthly_equivalent` (worked out), `is_household` (shared; linked to the primary applicant), `is_projected` and `collateral` (projected insurance only), `documents`.
- **ExpenseType:** `name`, `code`, `applies_to` (loan categories; empty means all), `group`, `user_selectable`, `is_active`, `sort_order`.
- **Collateral:** `application`, `asset_id`, `category` (derived from the asset type, with no dropdown), `description` (collateral notes), the `insurance_*` fields, `status`, and `documents`. One record per asset marked as collateral. It has **no value of its own**: the asset's `declared_value` is used. `ownership`, `third_party_owner`, `third_party_relationship`, and `estimated_value` are no longer used.
- **In the form,** collateral details live on each asset as `asset.collateral` (`enabled`, `id`, `description`, `insurance`, `document_ids`, `projected_expense_id`), not in a separate list.

Dropdown options come from each resource's property `lookup_reference`, loaded with `loadResourceProps()` and parsed by `options()`.

### Fields added to Saturn for the form

Checked against the developer's full property list in September 2026. These were missing, so Saturn silently dropped them (and restores came back empty); **the developer has since added them**:

- **Application:** `asset_ids`, `liability_ids`, `expense_ids` (without `asset_ids`, assets and collateral didn't restore).
- **Party:** `nis_number`.
- **ApplicationParty:** `relationship_to_applicant`.
- **Collateral:** the eight `insurance_*` fields.

The form also sends `application_parties` on Application and `party_id` on ApplicationParty and PartyIdentification. Those resources don't have them; it's harmless (`parties` and `party` hold the same values).

The developer then added the fields for membership, other income, monthly equivalents, AML, declarations, housing, references, guarantors, purchase details, liens, and business Parties (the IncomeSource and Reference resources are new). Resource fields that exist but the form doesn't use yet include Application `preferred_contact_method`, and Collateral `appraised_value`, `appraisal_date`, `appraisal_source`, `lien_position` and `lien_status` (back office, Phase 3).

**ExpenseTypes are set up** (September 2026): the Utilities and Insurance types and `COLLATERAL_INSURANCE` (not user-selectable) exist; Interest expense and Bad debt don't exist in this install. Every revolving LiabilityType has `revolving_rate` 0.03.

### Link lists Saturn fills itself (the form must not write them)

Saturn fills these lists automatically from the records that point to them, so the form never sets them:
- `Asset.owners`, `Party.assets`: from `AssetOwnership` (`asset`, `party`)
- `Liability.owners`: from `LiabilityResponsibility` (`liability`)
- `Application.collateral`: from `Collateral.application`
- `ApplicationParty.liability_ids`, `ApplicationParty.expense_ids`: from `LiabilityResponsibility.application_party` and `Expense.applicationpartiesid`

A Party can be a person or a business, so Party keeps its business fields.

## Business rules

- **Revolving credit** (credit cards, overdrafts): the assessed repayment is `credit_limit × rate`. The rate is the type's `revolving_rate`, falling back to `DEFAULT_REVOLVING_RATE` (3%).
- **NIS and income tax are calculated** from gross monthly income and shown read-only. The settings are in `STATUTORY_DEDUCTIONS`, near the top of the main form:
    - **NIS:** 6.25% of income up to EC$5,200 (maximum EC$325). Self-employed pay 13.5%. Nothing for the unemployed, the retired, anyone under 16, or anyone at or over 65. 65 is used because pensionable age is being phased from 60 to 65 by birth year, and 65 never understates NIS.
    - **Income tax:** 0% on the first EC$3,000 a month, 10% on the next EC$2,000, and 30% above EC$5,000.
    - Worked examples that must hold: EC$4,000 gives EC$250 NIS, EC$100 tax, EC$3,650 net. EC$6,000 gives EC$325 NIS, EC$500 tax, EC$5,175 net.
- **IDs:** the chosen loan type's `minimum_identifications` (empty means 1) applies to the Primary Applicant, every co-borrower, and every Guarantor. Third Party Owners give none, and the NIS card doesn't count. No duplicate ID types, no expired IDs, and exactly one primary.
- **Loan types on offer:** only loan types with status "Active" (or empty) show, and a category with none is hidden. A restored draft whose loan type isn't on offer opens on the first step with "The loan you picked, <name>, isn't offered any more. Please choose another."; the loan type, amount, term, and limit are cleared and everything else stays.
- **Students:** the work situation "Student" works like Unemployed (no job details, no NIS deduction; the pay screen still asks, and 0 is fine). For a student, the NIS number and the NIS card upload are optional (the card shows as "(optional)"). The student loan type needs a guarantor (`requires_guarantor`). The amount isn't checked against tuition.
- **Cards and overdrafts:** the savings held can't be more than the limit asked for. The overdraft must be on an account in the applicant's name (a joint account is fine); the form says so but can't check it.
- **Contact method:** everyone, including Third Party Owners, must choose how they'd like to be contacted.
- **Parish** is required only when the country is Grenada.
- **Business-only expense types** are hidden on personal loans, via `applies_to`.
- **Only applicants can own assets.** Someone who isn't borrowing but owns all or part of an asset is added on the Applicants step with the role **Third Party Owner** (`THIRD_PARTY_OWNER_ROLE`).
    - **Their short form:** person or business, name (first and last, or business name), relationship to the primary applicant, phone (all required), and email (optional). No address, date of birth, NIS, IDs, documents, employment, income, or consents.
    - They appear in asset ownership splits, but **not** in liability or expense assignments, since they owe nothing on the loan.
    - **An asset owned only by Third Party Owners must be marked as collateral,** or it can't stay on the application.
    - All assets, including third-party-owned ones, go in the application's `asset_ids`. **Net worth and underwriting should count only the borrowers' ownership share,** using the AssetOwnership percentages. (The form doesn't calculate net worth yet.)
- **Monthly figures:** every amount with a frequency is turned into a monthly figure with `monthlyAmount()` and saved as `monthly_equivalent` (Expense, Liability, IncomeSource). The factors in `MONTHLY_FACTORS` are keyed by the standard frequency list: Weekly, Fortnightly, Twice monthly, Monthly, Quarterly, Twice yearly, Yearly (unknown frequencies count as monthly). A revolving liability's monthly figure is its assessed payment.
- **Pay:** applicants enter gross pay and how often they're paid; `grossMonthly()` works out the monthly income that NIS and income tax use. Drafts saved before that restore as a monthly figure.
- **Under 2 years:** at the current address (worked out from `address_since`) asks for the previous address; in the current job asks for the previous job.
- **Membership:** non-members can apply ("Not a member yet", staff follow up). A member must give their member number.
- **AML and declarations:** citizenship and residency status are required; a PEP "yes" or any declaration "yes" needs an explanation. (The question set is typical; the credit union's compliance officer should confirm it.) A down payment needs its source; "Other" needs details.
- **Housing:** no rent or mortgage amount is asked with the housing question; rent is an expense and a mortgage a liability, so nothing is counted twice.
- **References:** one personal reference and one next of kin, for the primary applicant only, both with name, relationship, and phone.
- **Guarantors** (role `Guarantor`) fill in the full applicant form plus the guarantee type and amount, and can be assigned liabilities and expenses.
- **Liabilities** record whether this loan pays them off (their payment isn't counted in the monthly summary) and which asset they're secured on (shown on that asset as an existing loan).
- **Shared household expenses** aren't assigned to one applicant (saved against the primary applicant, marked `is_household`), so joint applications don't split them by percentage.
- **Projected insurance:** each collateral asset with a premium gets an Expense marked `is_projected`, linked to its Collateral, with the premium as a monthly figure. It's shown read-only on the Expenses step and counted in the monthly summary.
- **Business loans:** the business is its own Party (kind `ORGANIZATION`), joined to the application by an ApplicationParty with the role **`Business Borrower`** (`BUSINESS_BORROWER_ROLE`), whose ID goes in `Application.parties`. It's never deleted by the form. The person applying stays "Primary Applicant" and must give their position in the business (typed, saved to `signing_authority`). Trading name, industry, licence number, and website are optional.
    - The business's figures when it applied, `number_of_employees` and `annual_revenue` ("Yearly sales"), are saved on the Business Borrower link. The employee count is also kept on the Party as the latest figure.
    - A new Business Borrower link is saved twice. Saturn's `partysnapshot` workflow runs when a link is updated, not when it's created. People's links always get a second save (their income list).
    - "Business Borrower" is removed from the role choices for other people. Only the form creates it.
- **Links to a Party use `party` only.** The form doesn't send or read `party_id` on ApplicationParty or PartyIdentification, since Saturn doesn't define it. (`party_id` inside the form is the form's own in-memory name for a Party's ID.)
- **Loan kinds:** the form picks its screens by *kind*, the LoanCategory code in lower case with spaces and dashes as `_` (`categoryKind()`). Only seven codes have their own screens: `PROPERTY`, `AUTOMOTIVE`, `PERSONAL`, `ORGANIZATION`, `CREDIT_CARD`, `OVERDRAFT`, `STUDENT`. Any other code behaves like a personal loan. The old alternative codes (`auto`, `home`, `business`, `card`, ...) were removed; `ExpenseType.applies_to` uses the seven codes too. Credit cards and overdrafts are revolving (`REVOLVING_KINDS`, or `LoanCategory.is_revolving`): they ask for a limit, not an amount and term.
- **Required guarantor:** when the product's `requires_guarantor` is on, the applicant can't pass "Is anyone else on this loan?" or submit without someone with the role Guarantor.
- **Share of the loan** (`ApplicationParty.ownership_percentage`): the borrowers (primary applicant and co-borrowers) split 100% evenly, with any rounding on the primary applicant (e.g. 33.34 / 33.33 / 33.33). Guarantors and Third Party Owners get 0. On a business loan the Business Borrower gets 100% and every person gets 0 (a director who backs the loan personally is added as a Guarantor).
- **Sending:** the form saves the draft and runs the submit workflow `MXHGYH`, which sets `status`, `submitted_at`, and `application_number` together, giving a number only when there's none (`E6M2NS`). The form never writes the status itself. If sending fails, it says "We couldn't send your application. Please try again." and the application stays a draft.
- **Interest rate for estimates (DSR):** `Application.interest_rate` if staff set one, otherwise the product's `default_interest_rate`. The form doesn't show a payment.
- The server should **recalculate** `assessed_payment`, `nis_deduction`, `income_tax_deduction`, and the `monthly_equivalent` figures before underwriting uses them. The browser's figures are estimates.

## Lessons learned (don't repeat these)

- **The developer can't change SystemConfiguration.** Don't put settings there; use constants in the code or fields on resources the developer controls.
- **Ask for the Network tab response body** when a Saturn request fails. The upload 400 was only solved by comparing against a working request.
- **Required links need a save order.** PartyIdentification requires `party`, so the Party is saved first, then the IDs, then `Party.ids` is updated.
- **Element Plus `el-radio`** changed its value prop between versions (`label` vs `value`). Prefer `el-select` or buttons for choices.
- **The Saturn guide's `FormField`** is unclear about its update event payload (`{ property, data }` vs the value). Test before relying on it.
- **Saturn composables can't call each other** and can't see the `composables` object. A split into five composables failed, and then a single `useLoanIntake` composable was reverted too. Keep all shared code in the main form.
- **Don't use the spread operator (`...`)** anywhere in Saturn code: it fails at runtime ("Spread syntax requires ...iterable[Symbol.iterator] to be a function"). Use `concat`, `slice()`, `Object.assign`, and `Array.from(new Set(...))` instead. To be safe, also avoid destructuring by position (`const [a] = list`, `for (const [i, x] of list.entries())`); use indexes instead.
- **Don't give `FormField` its own option list** (`lookup_type: "values"`); it doesn't work in Saturn. Use Saturn's definition of the property, or an `el-select`.
- **A dropdown showing `[object Object]`** means its options are records or objects, not text. Check that the field points at the right property (a list of choices, not a link to other records). `options()` now always turns labels and values into plain text.
- **Saturn can reply to `create` with a failure instead of throwing** (e.g. `{"status":"FAILURE","type":"single","message":{}}`), so there's no record ID. `upsert()` looks for the ID in several places (`createdIdOf()`), and otherwise throws with Saturn's reply; the failed save's message and the console (`[Loan form] …`) show which resource failed and why. This first showed up creating Reference records. **If `create` returns nothing at all** (`undefined`) and no request shows in the Network tab, Saturn didn't send it: the resource name doesn't match exactly (e.g. `Reference` vs `References`), the resource isn't published, or the user's role can't create it.
- **Saturn only accepts values of a field's own type; it rejects `null`.** So every save goes through `upsert()`, which leaves empty values out of the message entirely (`withoutEmptyValues()`: `null`, `undefined`, `""`, and `NaN`; lists, `true`/`false`, and `0` are kept). Don't call `Resource.create`/`update` directly; use `upsert()` (it also reports failures clearly). **Side effect:** on an update, a field left out keeps what was saved before, so a value the applicant empties (e.g. unticking "member" after typing a number, or a credit limit after switching to a non-revolving type) stays on the Saturn record. The form ignores these leftovers (it checks the related yes/no or type), but staff looking at the raw record may see them.
- **Failed saves are logged:** `saveDraft()` shows the real reason in the alert and logs it to the console. Don't swallow errors without logging them.
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
- Rebuilt `AdaptiveLoanCollateralSection.vue` to match the main form (it had been an old version, so the form warned about insurance fields that weren't on screen). Then replaced it entirely, below.
- **Collateral moved onto the Assets step,** and third-party owners became applicants with the role Third Party Owner and a short form. The Collateral step and `AdaptiveLoanCollateralSection.vue` were removed. Collateral uses the asset's value; its own value fields were dropped.
- **On `dev`:** Saturn `FormField` for inputs; test mode (optional documents, "Remember draft on reload").
- **On `dev`, the gap-analysis batch:** membership, other income, pay frequency and monthly figures, debts paid off by the loan, AML questions, declarations, housing and dependants, previous address and job, references, guarantors, purchase details and the automatic purchase asset, vehicle and land identifiers, liens, business Parties, liability notes and balance date, shared household expenses, projected collateral insurance, and the full review screen.

### Decisions made

- **Non-members can apply** (staff follow up); members give their number.
- **Pay is entered per pay period** with its frequency; the form works out the monthly figure.
- **No rent or mortgage amount on the housing question** (rent is an expense, a mortgage a liability).
- **References:** one personal reference and one next of kin, for the primary applicant only.
- **Guarantors** fill in the full applicant form plus the guarantee.
- **The vehicle or property being bought** is added automatically as a collateral asset.
- **Business details** live on the business's own Party.
- **Review screen:** our own full review (not `ResourceViewInline`).
- **Consent forms** will be emailed by a workflow after submitting (see Next up).
- **Every party joins an application through ApplicationParty**, never linked straight to the Application, so the application keeps what was true when the person applied. The business on a business loan is a Party with an ApplicationParty whose role is **`Business Borrower`**; `Application.business_party` is removed. Facts that matter to the decision and can change later (housing, dependants, PEP, job and pay, the business's revenue and employees) stay on ApplicationParty.
- **`ApplicationParty.partysnapshot`** is filled by a Saturn workflow, not the form.
- **Loan categories** will be a `LoanCategory` resource (code, name, description, icon, interest rate range, `is_revolving`, active, order), linked from `LoanType.loan_category`. Each LoanType has a `default_interest_rate` inside its category's range.
- **Interest rate:** `Application.interest_rate` and `interest_rate_type` are for staff overrides; the form never writes them. Estimates use the override if set, otherwise the product default. Overrides outside the range need privilege escalation, handled in Saturn, not the form.
- **Applicants don't see an estimated monthly payment.**
- **Credit cards and overdrafts** ask for a limit in its own field, `Application.requested_credit_limit` (not `requested_loan_amount`).
- **Categories renamed** to PROPERTY, AUTOMOTIVE, PERSONAL, ORGANIZATION; new ones: CREDIT_CARD, OVERDRAFT, STUDENT (screens built on `redesign`).

- **September 2026 design session** (full list in `docs/grilling/2026-09-30-loan-intake-gaps.md`): one install per lender; credit cards are switched on or off per lender by the Credit card category, with the same card questions for all; the student loan always needs a guarantor; guarantors get 0% of the loan; on business loans the person applying stays Primary Applicant; the number of IDs is set per loan type; input boxes stay `FormField`; uploads keep our own card; the submit workflow sets status, date, and number together; the top tax rate stays 30% until IRD confirms.

- **Public applicant form, October 2026 (planned, not built yet; don't start until the developer says go).** Applicants fill in the form themselves on a public "apply" page:
  - **No Saturn logins for applicants** (Saturn supports it, but it isn't allowed). The page is public and uses `pageHttp` with saved Requests and Secrets; the browser never sees the bearer token.
  - **Applicants start their own application.** No staff invite and no one-time code (too cumbersome). Spam guards that add no steps: a hidden honeypot field, nothing created until the Applicants step is left (as today), limits in the start step, and cleanup of expired drafts.
  - **Option A, one gateway Path** (`loan_intake`, a Saturn HTTP WorkflowTrigger → workflow action → Code operators). The form keeps its save, restore, and delete logic; `Resource` is swapped for a helper with the same methods that sends `{ application, token, op, resource, id, data }`. Uploads (`loan_upload`) and submit (`loan_submit`) get their own Paths; the submit logic becomes a reusable command.
  - **Every Path checks everything itself,** because anyone can run them through the public page's saved Requests (Saturn requires the bearer token on `/api/_paths/...`, but the page lends it to every visitor; without a token Saturn answers `Route not found` for every route, real or not): token, expiry, status Draft, allowed resource, the ID is in the application's own list (`intake_record_ids`), and staff-only fields (`status`, `application_number`, `submitted_at`, `interest_rate`, `interest_rate_type`, token fields) are stripped. The start step never looks up existing Parties. Errors come back as `{"status":"FAILURE"}`, often with HTTP 200.
  - **Link token:** `randomBytes(32)`, stored only as a SHA-256 hash (`access_token_hash`), compared with `timingSafeEqual`, **valid 30 days** (`access_token_expires_at`), cleared on submit. Kept in `localStorage` for resuming; a "Finish later" button and submitting remove it from the browser.
  - **Resume link by email,** sent by a Saturn workflow from the server-side start step (never from the browser). The address is shown back to the applicant before sending. The workflow ID goes in a marked placeholder (`WORKFLOW_ACTIONS.resumeLinkEmail`, next to `submit: "MXHGYH"`).
  - **Public only:** staff won't keep a separate logged-in version of the form.
  - **The API key** belongs to a limited User Group, not an admin.
  - Background and concepts: `docs/learn.md`.

### Decisions still waiting on the developer

1. **Income tax rates:** the form uses 10% for the middle band and 30% for the top band. A 2022 World Bank paper says the top rate was cut to 28% in 2019. Keep 30% until the Inland Revenue Division confirms.
2. **AML and declaration questions:** confirm the set with the compliance officer.
3. **Public form, largest upload a Path accepts:** unknown. Base64 adds about 33%; test a real 5–10 MB scan through a test-only Path.
4. **Public form, what a failing Path returns to the page** (a `throw` in a Code operator): unknown. Check the Network tab on the MediPal page with a made-up link. Until then, handle both an HTTP error and `{"status":"FAILURE"}`.
5. **Public form, does saving restart the 30-day clock?** Suggested yes.
6. **Public form, data-protection sign-off** before going live: who approves it.

### Next up

- **Test the gap-analysis batch in Saturn** on `dev`, then merge to `main`.
- **Publish the September 2026 batch in Saturn and test it,** using the Nexa test loan types for cards, overdrafts, and student loans. Test resuming a draft too.
- **Switch off duplicate revolving LiabilityType records** if they're active, so the same kind of debt doesn't show twice.
- **Consent forms on submit:** a workflow that emails the Credit Bureau consent form (and the Valuation authorization when property is collateral) and sets `Application.consent_forms_sent_at`. First check that Saturn workflows can send emails with attachments.
- **Reference and next-of-kin emails:** the developer will build a Saturn workflow that emails them. The form then just calls it on submit (like `MXHGYH`); waiting on the workflow ID.
- **Remove spread syntax (`...`):** the reverted code still uses it (about 14 places left in the main form; the sections are clean), and Saturn fails on it at runtime.
- **Cleanup:** remove the dead CSS from the old Documents screen (e.g. `.legacy-queue`), and reorganize the main form into labelled sections.

### Later (Phase 3: back office)

Valuations and appraisals (including the minimum required value, and vehicle appraisal for auto loans), an underwriting summary (DSR and LTV, which needs a product interest rate and monthly-equivalent amounts), staff search by application number, and department queues by loan category.

### Known issues

- **Students and the NIS number:** the NIS number is asked on the ID screen, before the work situation, so a student who hasn't chosen "Student" yet is still asked for it. It becomes optional once "Student" is chosen.
- **Changing an item's type** after uploading leaves the old document linked.
- **Drafts only resume in the same browser.** Cross-device resume would need an emailed link or a login. (The planned public form fixes this with the emailed resume link.)