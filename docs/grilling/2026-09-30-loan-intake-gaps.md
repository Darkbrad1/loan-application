---
date: 2026-09-30
topic: loan intake gaps
tags: [grilling]
---

# Loan intake gaps

The parts of the Adaptive Loan intake form (branch `redesign`) that aren't fully worked out yet. The starting point is the "Decisions still waiting" list in `claude.md`, gaps in the new credit card, overdraft, and student screens, and the full Saturn field list the developer shared (19 resources).

Settled from the code or the field list, so not asked:

- Input boxes use Saturn `FormField` on `redesign`, with `el-select` for the lists the form decides.
- The AML and declaration questions need the compliance officer, so they're out of scope for this session.
- `Reference` has no `application` field. The form sends one anyway and Saturn drops it. References are linked only through `Application.reference_ids`, which is enough.
- `Party.years_at_address` isn't used. The form saves `address_since` and works the years out from it.
- `LoanCategory` has a text field called `property` that nothing uses. It looks like it was added by mistake.
- Staff-only fields (`Application.financial_snapshot`, `priority`, `current_owner_id`, `Asset.verified_value`, Collateral appraisal and lien fields, the `Loan` and `Interaction` resources) belong to the back office, not the intake form.

## Round 1 (asked again with the Saturn field list)

❓ **Q1** - **"Loan type" or "Product"**: The code uses both words for the same Saturn `LoanType` record (for example "Auto loan, 5 years"). Which word do we use when talking about the form?

- [ ] Product
- [ ] Loan type

➡️ Product. It's the word on the cards the applicant picks from. "Loan category" stays the word for the group it sits in (for example Automotive), and `LoanType` stays only as the Saturn resource name.

**Answer:** Loan type. "Product" is the word to avoid.

---

❓ **Q2** - **Who issues the credit card**: `Application` has only `name_on_card` and `card_collection_method` for cards. Does the credit union issue its own cards, or go through a partner (a bank or card company)?

- [ ] Its own cards
- [ ] Through a partner

➡️ Through a partner. Small credit unions usually do this, and the two fields already in Saturn are what a partner needs from the application.

**Answer:** It differs by credit union: some offer cards and some don't, so it has to be configurable.

---

❓ **Q3** - **Guarantor on student loans**: `LoanType.requires_guarantor` is a yes/no on the product. Does a student loan always need a guarantor, or only when the student has no income?

- [ ] Always (switch it on for the student product)
- [ ] Only when the student has no income

➡️ Always. It's one switch in Saturn with no new code. "Only with no income" needs a new rule and leaves a grey area about how much income is enough.

**Answer:** Always.

---

❓ **Q4** - **Students and "What is your work situation?"**: `ApplicationParty.employment_status` is a text field whose answers come from its Saturn list. The form treats only "Unemployed" and "Retired" as having no job, so a full-time student has no honest answer.

- [ ] Add a "Student" answer in Saturn that works like Unemployed (no job or pay screens, no NIS)
- [ ] Add "Student" and still ask about any job
- [ ] Leave it; students pick Unemployed

➡️ Add "Student", working like Unemployed. A part-time job or an allowance goes under other income (`IncomeSource`).

**Answer:** Add "Student", working like Unemployed.

---

❓ **Q5** - **Tuition compared with the amount asked**: `tuition_amount` and `other_study_costs` are numbers, with `tuition_currency` as text. Nothing checks them against `requested_loan_amount`, and there's no field for an exchange rate.

- [ ] No check; staff compare them
- [ ] A warning the applicant can ignore
- [ ] Block the application if the amount is more than the costs

➡️ No check. With fees in US$ and no exchange rate, any check would be wrong.

**Answer:** No check; staff compare them.

---

❓ **Q6** - **Savings held against a card or overdraft**: When `is_secured_by_savings` is yes, the form only checks that `secured_savings_amount` is above 0. Should it have a limit?

- [ ] It can't be more than `requested_credit_limit`
- [ ] It must equal `requested_credit_limit` (fully secured)
- [ ] No rule; staff decide

➡️ It can't be more than the limit asked for. Holding more savings than the limit makes no sense, and anything less is staff's call.

**Answer:** It can't be more than `requested_credit_limit`.

---

❓ **Q7** - **Whose account the overdraft is on**: `linked_account_number` is plain text, and the form can't see the credit union's accounts, so it can't check who owns the account.

- [ ] Any account held by someone on this application; the form just asks
- [ ] Only the primary applicant's account; say so on the screen

➡️ Only the primary applicant's account, and say so under the question. A joint account is fine if the primary applicant is on it.

**Answer:** Only the primary applicant's account; say so on the screen.

---

❓ **Q8** - **Guarantor's share of the loan**: Borrowers split 100% of `ownership_percentage` evenly. Guarantors get 0% for now, and their promise is saved in `guarantee_type` and `guarantee_amount`.

- [ ] 0%
- [ ] A share, like a borrower

➡️ 0%. A guarantor only pays if the borrowers don't, and `guarantee_amount` already records how much they promise.

**Answer:** 0%.

---

❓ **Q9** - **The person filling in a business loan**: `ApplicationParty` already has `signing_authority` (text) and `is_primary_contact` (yes/no), and the form uses neither for this. Today the person is saved as "Primary Applicant", and every "primary applicant" rule applies to them: references, household bills, and the purchase asset. Options:

- [ ] Keep the role "Primary Applicant", and ask "What's your position in the business?" into `signing_authority`
- [ ] A new role "Authorised Signatory" with `signing_authority`, and change every primary-applicant rule to cope with it

➡️ Keep "Primary Applicant" and add the `signing_authority` question (for example Director, Owner, Manager). The field is already in Saturn. A new role touches every rule that looks for the primary applicant, for little gain.

**Answer:** Keep "Primary Applicant", and ask their position into `signing_authority`.

---

❓ **Q10** - **Extra business details**: `Party` has four business fields the form never asks: `trading_name`, `industry`, `business_license_number`, `website`. Which should a business loan ask?

- [ ] Trading name
- [ ] Industry
- [ ] Business licence number
- [ ] Website

➡️ Trading name and industry, both required. Put the licence number behind "+ Add more details" as optional. Skip the website, since it tells a loan officer little.

**Answer:** All four. The website is optional.

---

❓ **Q11** - **How to contact the applicant**: `Application.preferred_contact_method` exists but the form never asks it.

- [ ] Ask it on the contact details screen, as answer cards from its Saturn list
- [ ] Don't ask it

➡️ Ask it, with the answers set in Saturn (for example Phone call, WhatsApp, Email). It's one tap, and staff then know how to reach people for follow-up.

**Answer:** Ask it on the contact details screen, as answer cards from its Saturn list.

---

❓ **Q12** - **Products that aren't offered any more**: `LoanType` has a text `status` (no `is_active`), and the form shows every product in the category whatever its status.

- [ ] Show only products whose status is "Active"; treat an empty status as active
- [ ] Show every product (staff delete ones they stop offering)

➡️ Show only "Active", with empty counted as active so nothing disappears before the statuses are filled in. Staff can then retire a product without deleting it.

**Answer:** Show only "Active"; treat an empty status as active.

---

❓ **Q13** - **Documents staff ask for after sending**: `Application.requested_documents` links to `DocumentRequest` (document type, status, reason). The form doesn't use it. Should the intake form show these requests and let the applicant upload them?

- [ ] Not now; it's back-office work (Phase 3)
- [ ] Yes, on a returning visit, before the review screen

➡️ Not now. Drafts only resume in the same browser, so an applicant could miss the requests anyway. It fits with the back office and a login or emailed link.

**Answer:** Not now (Phase 3).

---

❓ **Q14** - **How many IDs each applicant gives**: The NIS card is already required for everyone. `MINIMUM_IDENTIFICATIONS` is 1 on top of that.

- [ ] 1 photo ID (plus the NIS card)
- [ ] 2 IDs (plus the NIS card)

➡️ 1 photo ID. With the NIS card that's already two documents, and each extra one loses applicants who don't have it.

**Answer:** It differs by lending institution: some require 2 IDs and some require 1.

---

❓ **Q15** - **When submitting fails**: Today the form sets `status` to "submitted" and then runs the workflow that fills `application_number`. If the workflow fails, the application is submitted with no number.

- [ ] The workflow sets `status`, `submitted_at`, and `application_number` together
- [ ] Keep the current order; staff spot stuck applications

➡️ The workflow sets all three together. Then an application is never "submitted" without a number, and the applicant can simply try again.

**Answer:** The workflow sets `status`, `submitted_at`, and `application_number` together.

---

❓ **Q16** - **Upload boxes**: Uploads use our own `AdaptiveLoanDocumentRequirements` card, which lives only in Saturn (it's not in the repo). The other choice is Saturn's built-in `typed_file_upload`.

- [ ] Keep ours, and add its file to the repo
- [ ] Switch to Saturn's `typed_file_upload`

➡️ Keep ours and add the file to the repo. It already handles the locks, per-item scopes, and the upload fields Saturn needs.

**Answer:** Keep ours, and add its file to the repo.

---

❓ **Q17** - **Top income tax rate**: The form charges 30% on monthly income above EC$5,000. A World Bank paper (2022) says Grenada cut the top rate from 30% to 28% in 2019, and the middle band from 15% to 10% in 2016. The IRD site confirms the EC$3,000 a month tax-free amount, and NIS at 6.25% capped at EC$5,200 a month (EC$325 at most) matches the NIS site for 2026. The helper couldn't open IRD's tax tables, because the network blocks ird.gd. With 28%, the EC$6,000 worked example becomes EC$480 tax instead of EC$500.

- [ ] Change to 28%, and update the worked example
- [ ] Keep 30% until IRD confirms

➡️ Change to 28%, and still ask IRD to confirm it. The only dated source for the cut is the World Bank paper, and the sites still showing 30% look out of date.

**Answer:** Keep 30% until IRD confirms.


## Round 2

Facts found before this round:

- A lender that doesn't offer cards can already hide them: switch its Credit card `LoanCategory` to inactive (`is_active` off) and the box disappears from the first step. A category that's active but has no loan types still shows as an empty box.
- `claude.md` describes one Grenadian credit union, and records that the developer can't change SystemConfiguration. Logo and colours come from SystemConfiguration, so they are already one set per Saturn install.

❓ **Q1** - **The word for the organisation that lends**: You wrote "credit union" in one answer and "lending institution" in another. Which is the project's word?

- [ ] Lender (credit unions and any other lending institution)
- [ ] Credit union
- [ ] Lending institution

➡️ Lender. It covers credit unions and anything else that lends, and it's short enough for everyday use.

**Answer:** Lender.

---

❓ **Q2** - **One Saturn install per lender, or one shared**: Does each lender get its own Saturn install (its own records, logo, and settings), or do several lenders share one install?

- [ ] One install per lender
- [ ] Several lenders share one install

➡️ One install per lender. Logo and colours already come from SystemConfiguration, which is one per install, and sharing would mean every record needs a lender link so applicants never see another lender's loan types.

**Answer:** One install per lender.

---

❓ **Q3** - **Are all lenders in Grenada?**: NIS and income tax rules, the EC$ currency, and "parish is required in Grenada" are all Grenadian.

- [ ] Yes, Grenada only for now
- [ ] No, other countries too

➡️ Grenada only for now. Other countries would need their own tax and social security rules, which is a separate piece of work.

**Answer:** Grenada only for now.

---

❓ **Q4** - **Turning credit cards on or off**: Is switching the Credit card loan category to inactive enough for a lender without cards? And for a lender with cards, do the questions stay the same (name on the card, how to get it) whether it issues them itself or through a partner?

- [ ] Yes to both: category on or off, same questions for everyone
- [ ] A lender that issues its own cards needs extra questions

➡️ Yes to both. It needs no new setting, and the two card questions cover what either kind of issuer needs from the application.

**Answer:** Yes to both.

---

❓ **Q5** - **Where the number of IDs is set**: Some lenders need 1 ID and some need 2. Today it's one number in the code (`MINIMUM_IDENTIFICATIONS`).

- [ ] A new number field on `LoanType`, `minimum_identifications` (empty means 1)
- [ ] A per-lender setting in a new Saturn resource
- [ ] Keep the constant; change it in the code for each lender

➡️ A field on `LoanType`. The developer controls LoanType, it needs no new resource, and a lender sets the same number on all its loan types. It also lets one loan type ask for more if a lender ever wants that.

**Answer:** Keep the constant, and change it in the code for each lender.

---

❓ **Q6** - **Trading name when it's the same as the legal name**: You made trading name required. Many small businesses trade under their legal name.

- [ ] Required, with a "Same as the legal name" tick box that copies it
- [ ] Optional; empty means the same as the legal name
- [ ] Required, typed every time

➡️ Required, with a "Same as the legal name" tick box. Staff always see a trading name, and the applicant doesn't type it twice.

**Answer:** Optional; empty means the same as the legal name.

---

❓ **Q7** - **Industry answers**: `Party.industry` is text. Should the answers come from a Saturn list (for example Retail, Construction, Agriculture, Tourism, Other), or be typed?

- [ ] A Saturn list, shown as a dropdown
- [ ] Typed

➡️ A Saturn list. Staff can then group and report by industry, which typed answers won't allow.

**Answer:** Typed.

---

❓ **Q8** - **Signing authority answers**: `ApplicationParty.signing_authority` is text. Should "What's your position in the business?" use a Saturn list (for example Owner, Director, Partner, Manager, Other), or be typed?

- [ ] A Saturn list, shown as answer cards
- [ ] Typed

➡️ A Saturn list, as answer cards. It's a short list, and cards are easier than typing for the people this form is for.

**Answer:** Typed.

---

❓ **Q9** - **NIS card for students**: Every applicant must give an NIS number and upload the NIS card (the `applicant-all` document). A full-time student who has never worked may not have one.

- [ ] Keep it required for students
- [ ] Optional when the work situation is Student

➡️ Optional when the work situation is Student. A student who has never worked can't get past the form otherwise, and the guarantor, who does give an NIS number, carries the repayment.

**Answer:** Optional when the work situation is Student.

---

❓ **Q10** - **Trying again after a failed send**: With the workflow setting status, date, and number together, what happens when it fails?

- [ ] The form says "We couldn't send your application. Please try again.", the application stays a draft, and pressing Send again reruns the workflow. The workflow gives a number only when the application has none, so a retry never gets a second number.
- [ ] Something else

➡️ The first option. The applicant can fix it by pressing Send again, and the "only when it has none" rule stops double numbers.

**Answer:** The first option: stays a draft, Send again reruns the workflow, a number only when there's none.


## Round 3

Facts found before this round:

- The "Business name" box saves the same text into both `Party.business_name` and `Party.legal_name`.
- The browser remembers a draft under keys starting `gccu_`, which is one lender's initials.
- `STATUTORY_DEDUCTIONS`, `DEFAULT_COUNTRY`, and the tax bands are Grenadian, so they stay the same for every lender while all lenders are in Grenada.
- ADR 0001 (one install per lender, lender differences in the code) is written as "proposed". Q2 below confirms its reason.

❓ **Q1** - **Which settings differ per lender**: Which constants at the top of the main form does each lender set for itself?

- [ ] `MINIMUM_IDENTIFICATIONS` (1 or 2)
- [ ] `DEFAULT_REVOLVING_RATE` (the 3% used for credit cards and overdrafts when the liability type has no rate)
- [ ] `TEST_MODE_AVAILABLE` (on while testing, off when real applicants use it)

➡️ All three, grouped together at the top of the main form under a "Lender settings" heading, so setting up a new lender means changing only that block.

**Answer:** `MINIMUM_IDENTIFICATIONS` and `DEFAULT_REVOLVING_RATE`. Not `TEST_MODE_AVAILABLE`.

---

❓ **Q2** - **Why a code constant and not a `LoanType` field**: ADR 0001 says it keeps the lender's setup with the developer and adds no Saturn fields. Is that the reason?

- [ ] Yes
- [ ] Another reason (say which)

➡️ Only you know this one, so there's no recommendation. The ADR stays "proposed" until you answer.

**Answer:** Another reason: adding and connecting a new property seemed like a hassle, but a `LoanType` field would be a good idea.

---

❓ **Q3** - **Keeping track of each lender's settings**: With one copy of the code per lender, how do we keep track of which lender has which settings?

- [ ] One copy of the code in this repo, plus a table in `claude.md` listing each lender's values
- [ ] A branch per lender

➡️ One copy plus a table. A branch per lender means copying every fix into every branch.

**Answer:** Not answered (see Round 4, Q2).

---

❓ **Q4** - **The `gccu_` draft keys**: Each lender has its own web address, so the keys never clash, but they carry one lender's name. Renaming them makes browsers forget drafts saved before the change.

- [ ] Rename to `loan_` now, while only test drafts exist
- [ ] Keep `gccu_`

➡️ Rename to `loan_` now. Later, real applicants would lose their place.

**Answer:** The prefix should be the company name from SystemConfiguration, followed by `_`.

---

❓ **Q5** - **The "Business name" box**: Since trading name is optional and empty means "same as the legal name", the main box has to be the legal name.

- [ ] Label it "Registered business name", saved to `legal_name` and `business_name` as now
- [ ] Keep "Business name"

➡️ "Registered business name". It tells the applicant which name to type, and makes the empty trading name mean something.

**Answer:** Is it necessary to have both `legal_name` and `business_name`? Keep one and remove the other, and do the same in other resources, since there are too many duplicate properties.

---

❓ **Q6** - **Industry and licence number**: You ticked both. Are they required?

- [ ] Industry required, licence number required
- [ ] Industry required, licence number optional
- [ ] Both optional

➡️ Industry required, licence number optional. Some sole traders and new businesses don't have a licence number yet, and blocking them loses real applicants.

**Answer:** Both optional.

---

❓ **Q7** - **The business's share of the loan**: On a business loan, the Business Borrower has no `ownership_percentage` set. The Primary Applicant gets the whole 100%, as if they were borrowing personally.

- [ ] Business Borrower 100%; the people on the application 0% (a director who backs the loan personally is added as a Guarantor)
- [ ] Business Borrower and the Primary Applicant split it

➡️ Business Borrower 100%, people 0%. The business owes the money, and personal backing already has its own role.

**Answer:** Business Borrower 100%; the people on the application 0%.

---

❓ **Q8** - **Exactly what "optional NIS for students" covers**: Restated precisely. When a person's work situation is Student, the NIS number and the NIS card upload are both optional, and their NIS deduction is 0. If they do give a number, the form checks it the same way as for anyone else.

- [ ] That's right
- [ ] Not quite (say what's different)

➡️ That's right.

**Answer:** That's right.

---

❓ **Q9** - **A loan type retired while someone's draft uses it**: An applicant saves a draft for "Auto loan, 5 years", staff then set its status to something other than Active, and the applicant comes back.

- [ ] They keep it and can send the application; staff decide
- [ ] They're told it's no longer offered and must pick another loan type

➡️ They keep it and can send. They chose it while it was offered, and staff can still turn it down.

**Answer:** They're told it's no longer offered and must pick another loan type.

---

❓ **Q10** - **A loan category with no active loan types**: Today it still shows as a box on the first step, and picking it leads nowhere.

- [ ] Hide it
- [ ] Show it, greyed out, with "Not available right now"

➡️ Hide it. An applicant can't do anything with it.

**Answer:** Hide it.


## Round 4

Facts found before this round:

- Revolving rates already live on each `LiabilityType` record (`revolving_rate`). `DEFAULT_REVOLVING_RATE` is only used when a type has no rate.
- The company name comes from SystemConfiguration (`config.name`). When it's missing, the form shows "Loan Application".
- Switching loan category clears the loan type, amount, term, and limit. Details that belong to another category (for example vehicle details) stay in the draft but are only saved for the category that's chosen.
- `TEST_MODE_AVAILABLE` is still `true`, which suggests no real applicants use the form yet.

❓ **Q1** - **Number of IDs on `LoanType` after all**: Round 2 chose a code constant, and you've now said a `LoanType` field would be a good idea. Which one?

- [ ] A number field `LoanType.minimum_identifications`, where empty means 1
- [ ] Keep the code constant

➡️ The `LoanType` field. Each lender sets it in Saturn with no code change, and it's one new number property (the form already loads every `LoanType` field).

**Answer:** The `LoanType` field, empty means 1.

---

❓ **Q2** - **Revolving rate per lender**: Each lender already sets `revolving_rate` on its own liability types. Do we still need a per-lender default in the code?

- [ ] No: keep 3% as one fallback for every lender, and each lender sets `revolving_rate` on its revolving types
- [ ] Yes: a per-lender constant

➡️ No. If Q1 and Q2 both go this way, the code is identical for every lender, and every lender difference lives in Saturn (records and SystemConfiguration). ADR 0001 would then be rewritten to say that, and the "table of lender settings" question goes away.

**Answer:** No: 3% stays as one fallback for every lender.

---

❓ **Q3** - **Exactly how the draft keys are named**: Stated precisely: take the company name from SystemConfiguration, make it lower case, turn anything that isn't a letter or a number into `_`, and add `_draft_app_id` and `_draft_step` (for example `grenada_credit_union_draft_app_id`). With no company name, use `loan_`. Renaming the company in SystemConfiguration makes browsers forget drafts saved before the change.

- [ ] That's right
- [ ] Not quite (say what's different)

➡️ That's right.

**Answer:** That's right.

---

❓ **Q4** - **Are real applicants using the form yet?**: Removing old fields safely depends on whether any real application uses them.

- [ ] No, only test applications so far
- [ ] Yes, it's live somewhere

➡️ No. Test mode is still switched on in the code.

**Answer:** No, only test applications so far.

---

❓ **Q5** - **Duplicate fields to remove**: These pairs hold the same thing. Which of the extras should go?

- [ ] `Party.business_name` (keep `legal_name`; a business Third Party Owner's name goes there too)
- [ ] `Party.years_at_address` (keep `address_since`; the years are worked out)
- [ ] `LoanCategory.property` (unused)

➡️ All three. Each has a partner field that already does the job. (Two more depend on Q4: `Application.business_party` and `LoanType.category`, the old versions of the Business Borrower link and `loan_category`. They're for Round 5.)

**Answer:** All three.

---

❓ **Q6** - **Duplicates to keep on purpose**: These look like duplicates, but each records something the other can't.

- `Application.loan_category` and `loan_name`: what the applicant chose, even if staff later rename the loan type.
- `ApplicationParty.gross_monthly_income`, `years_employed`, `nis_deduction`, `income_tax_deduction`, and every `monthly_equivalent`: worked-out figures, so staff can read them without doing the maths.
- `Party.number_of_employees` and `ApplicationParty.number_of_employees`: the business today, and the business when it applied.

- [ ] Keep all of them
- [ ] Remove some (say which)

➡️ Keep all of them. Removing them loses either history or the figures staff read at a glance.

**Answer:** Keep all of them.

---

❓ **Q7** - **Exactly what happens with a retired loan type**: Stated precisely: when a draft is restored and its loan type's status isn't Active (empty counts as Active), the form opens the first step with "The loan you picked, <name>, isn't offered any more. Please choose another." The loan type, amount, term, and limit are cleared, as when switching category. Everything else in the draft stays. The applicant can't go past the first step until they pick an active loan type.

- [ ] That's right
- [ ] Not quite (say what's different)

➡️ That's right.

**Answer:** That's right.


## Round 5

ADR 0001 is rewritten and accepted: one install per lender, the same code for every lender, and every lender difference in Saturn records.

Facts found before this round:

- Restoring a draft still reads two old things: `Application.business_party` (before the Business Borrower link) and `party_id` on ApplicationParty (before `party`), as a fallback in two lines of the restore code (`AdaptiveLoanApplcationFrom.vue` lines 6081 and 6313). PartyIdentification restore doesn't read it. The many other `party_id` names in the code are the form's own in-memory fields, which hold the Party ID, and aren't read from Saturn.
- The form also understands old category codes such as `home`, `auto`, `vehicle`, `business`, and `card`. These are other names a lender might give a `LoanCategory` code, not old data.

❓ **Q1** - **Removing the old fields**: With only test applications so far, should `Application.business_party` and `LoanType.category` (the old text category) be deleted in Saturn, and the form stop reading them and `party_id` when restoring?

- [ ] Yes, remove all of it; old test drafts may lose their business or category
- [ ] Keep reading them for now

➡️ Yes, remove it. There's no real data to protect, and every old path left in is code that has to keep working. The old category codes (`auto`, `home`, and so on) stay, since they aren't old data.

**Answer:** Yes, remove all of it. Also remove the alternative category codes, since they aren't used and aren't in the LoanCategory resource.

---

❓ **Q2** - **Exactly who the number of IDs applies to**: Stated precisely: the chosen loan type's `minimum_identifications` (empty means 1) applies to the Primary Applicant, every co-borrower, and every Guarantor. Third Party Owners give no IDs. The NIS card doesn't count toward the number. If the applicant switches loan type after entering IDs, the new number is checked the next time they press Continue.

- [ ] That's right
- [ ] Not quite (say what's different)

➡️ That's right.

**Answer:** That's right.

---

❓ **Q3** - **The exact wording for the overdraft and savings rules**: Stated precisely:

- Under "Which account is the overdraft for?": "It must be an account in your name. A joint account is fine."
- If the savings amount is more than the limit: "The savings held can't be more than the limit you asked for."

- [ ] Use this wording
- [ ] Change it (say how)

➡️ Use this wording. It's short, and it says what to do.

**Answer:** Use this wording.


## Round 6

Facts found before this round:

- Removing the alternative codes leaves seven: `PROPERTY`, `AUTOMOTIVE`, `PERSONAL`, `ORGANIZATION`, `CREDIT_CARD`, `OVERDRAFT`, `STUDENT`. Anything else behaves like a personal loan.
- The same code list also filters bills: `ExpenseType.applies_to` holds category codes, and a bill type only shows on a loan whose category matches one of them. The form can't see what the ExpenseType records hold. If any still say `business`, `auto`, or `home`, those bill types would disappear once the alternative codes are gone.
- When no LoanCategory records load, the form shows four built-in categories (Personal, Auto, Home, Business). With every lender's categories set in Saturn, those four could offer loans a lender doesn't have.

❓ **Q1** - **Old codes in `ExpenseType.applies_to`**: Before the alternative codes are removed, what do the ExpenseType records hold?

- [ ] Only the new codes (or nothing)
- [ ] Some old codes; I'll change them to the new ones in Saturn first
- [ ] Not sure; I'll check in Saturn first

➡️ Check in Saturn and change any old ones (`business` to `ORGANIZATION`, `auto` or `vehicle` to `AUTOMOTIVE`, `home` to `PROPERTY`) before the code change goes in.

**Answer:** Only the new codes (or nothing).

---

❓ **Q2** - **The four built-in categories**: What should the first step show when no LoanCategory records load?

- [ ] A message: "We can't show our loans right now. Please try again later." and no categories
- [ ] Keep the four built-in categories

➡️ The message. The built-in four may not match what the lender offers, and an applicant could start an application for a loan that doesn't exist.

**Answer:** The message, and no categories.


## Settled

Lenders and setup

- The form serves several lenders. Each lender has its own Saturn install, and the code is the same for every lender ([ADR 0001](../adr/0001-one-install-per-lender.md)).
- All lenders are in Grenada for now. Tax, NIS, EC$, and the parish rule stay the same for everyone.
- Number of IDs: a new number field `LoanType.minimum_identifications`, where empty means 1. It applies to the Primary Applicant, every co-borrower, and every Guarantor. Third Party Owners give none. The NIS card doesn't count. After a loan type switch, the form checks the new number on the next Continue.
- Revolving rate: each lender sets `revolving_rate` on its revolving liability types. 3% stays as one fallback for everyone.
- `TEST_MODE_AVAILABLE` isn't a lender setting.
- Draft keys in the browser: the company name from SystemConfiguration, lower case, anything not a letter or number turned into `_`, then `_draft_app_id` and `_draft_step`. With no company name, `loan_`. Renaming the company forgets older drafts.

Loan categories and loan types

- Only seven category codes: `PROPERTY`, `AUTOMOTIVE`, `PERSONAL`, `ORGANIZATION`, `CREDIT_CARD`, `OVERDRAFT`, `STUDENT`. The alternative codes (`auto`, `home`, `business`, `card`, and so on) are removed. `ExpenseType.applies_to` already uses only the new codes.
- Only loan types whose `status` is "Active" show (empty counts as Active).
- A loan category with no active loan types is hidden.
- If no LoanCategory records load, the first step shows "We can't show our loans right now. Please try again later." with no categories. The four built-in categories are removed.
- A restored draft whose loan type isn't Active opens on the first step with "The loan you picked, <name>, isn't offered any more. Please choose another." The loan type, amount, term, and limit are cleared, everything else stays, and the applicant can't go past the first step until they pick an active loan type.
- Credit cards: a lender without cards switches its Credit card category off. The card questions (name on the card, how to get it) are the same for every lender.

Student loans

- The student loan type always needs a guarantor (`requires_guarantor` on).
- Add "Student" to the `employment_status` list in Saturn. It works like Unemployed: no job or pay screens, no NIS deduction.
- For a Student, the NIS number and the NIS card upload are optional. A number that is given is checked as for anyone else.
- No check of the amount asked against tuition and other costs.

Cards and overdrafts

- `secured_savings_amount` can't be more than `requested_credit_limit`. Message: "The savings held can't be more than the limit you asked for."
- The overdraft account must be the Primary Applicant's. Under the question: "It must be an account in your name. A joint account is fine."

Business loans

- The person applying stays "Primary Applicant". A typed question "What's your position in the business?" is saved to `ApplicationParty.signing_authority`.
- Business Borrower gets 100% of `ownership_percentage`. The people on a business loan get 0%. A director who backs the loan personally is added as a Guarantor.
- Ask trading name, industry (typed), licence number, and website. All four are optional. An empty trading name means the same as the legal name.

People and shares

- Guarantors get 0% of the loan.

Other questions and fields

- Ask each person's preferred contact method on their contact details screen, as answer cards from its Saturn list, saved to `ApplicationParty.preferred_contact_method`. Everyone is asked, including Third Party Owners. `Application.preferred_contact_method` is removed (see Round 7).
- Remove from Saturn and the form: `Party.business_name` (use `legal_name`, including for business Third Party Owners), `Party.years_at_address`, `LoanCategory.property`, `Application.business_party`, `LoanType.category`, `Application.loan_name`, `Application.loan_category` (Round 8; the form reads the name and category from the linked loan type). The form stops reading `party_id` when restoring.
- Keep on purpose: the worked-out figures (`gross_monthly_income`, `years_employed`, `nis_deduction`, `income_tax_deduction`, every `monthly_equivalent`), and `number_of_employees` on both Party and ApplicationParty.
- Documents staff ask for after sending (`DocumentRequest`) are not in the form for now (Phase 3).

Sending and uploads

- The submit workflow sets `status`, `submitted_at`, and `application_number` together, and gives a number only when the application has none. If it fails, the form says "We couldn't send your application. Please try again.", the application stays a draft, and Send runs the workflow again.
- Keep our upload card, and add `AdaptiveLoanDocumentRequirements.vue` to the repo.
- Input boxes stay Saturn `FormField`.

Rates

- The top income tax rate stays 30% until IRD confirms. The World Bank says 28% since 2019.

Out of scope for this session

- The AML and declaration questions (need the compliance officer).


## Round 7

The developer asked whether `preferred_contact_method` belongs on ApplicationParty rather than Application.

❓ **Q1** - **Where the preferred contact method lives**: `Application.preferred_contact_method` holds one answer for the whole application. With a co-borrower or guarantor, each person may want to be reached differently.

- [ ] `ApplicationParty.preferred_contact_method`: one answer per person on this application
- [ ] `Party.preferred_contact_method`: one answer per person, kept across all their applications
- [ ] Keep it on Application

➡️ ApplicationParty. Each person answers for themselves, and it follows the project's rule that facts which can change later are kept on ApplicationParty, as they were when the person applied. `Application.preferred_contact_method` is then removed.

**Answer:** `ApplicationParty.preferred_contact_method`.

---

❓ **Q2** - **Who is asked**: Which people get the question on their contact details screen?

- [ ] Everyone on the application, including Third Party Owners (their short form already asks for phone and email)
- [ ] Everyone except Third Party Owners

➡️ Everyone, including Third Party Owners. Staff may need to reach an owner about the collateral, and it's one tap.

**Answer:** Everyone, including Third Party Owners. (The reply was "q1a q1a"; read as Q1 a, Q2 a.)

Done in Saturn (2026-09-30): the developer moved `preferred_contact_method` from Application to ApplicationParty.


## Round 8

The developer asked whether `Application.loan_name` is needed when `loan_type_id` already points at the loan type. This reopens Round 4, Q6.

Facts found before this round:

- `loan_type_id` is the link to the LoanType record. `loan_name` is a copy of that record's `name`, taken when the applicant picks it. The applicant never types it.
- The form uses the name for three things: the words on screen ("Please upload these for your Auto loan"), the review heading, and finding the document groups (`application-<loan name>`, `applicant-<loan name>`). It can read all three from the linked loan type instead.
- `loan_category` is the same kind of copy: the linked loan type already has `loan_category`.
- The Saturn guide in the repo doesn't say whether a staff list of applications can show a linked record's name as a column.

❓ **Q1** - **Keep the copies of the loan type's name and category?**:

- [ ] Remove `loan_name` and `loan_category` from Application; the form and staff read them from the linked loan type
- [ ] Keep both as a record of what the applicant picked

➡️ Remove both, if Saturn's staff list can show the loan type's name through the link. Nothing in the form needs the copies, and a copy can drift from the real record. Keep them only if staff lists can't show linked names, because staff would otherwise see just an ID.

**Answer:** Remove both.

## Done in Saturn (reported 2026-09-30)

- Added `LoanType.minimum_identifications` (empty means 1). The seven original loan types are set to 1.
- `employment_status` answers: Employed, Self-employed, Unemployed, Retired, Student.
- `preferred_contact_method` answers: Phone call, WhatsApp, Email. They are on `ApplicationParty.preferred_contact_method` (confirmed by the developer).
- Every offered loan type has status Active. Three test loan types were added: Nexa Classic Credit Card, Nexa Flex Overdraft, and Nexa Student Loan (2 IDs, guarantor required).
- Every loan type is linked to a LoanCategory through `loan_category`. The seven category codes are normalised.
- `revolving_rate` is 0.03 on all six revolving LiabilityType records.
- ExpenseTypes are checked: Utilities, Insurance, and `COLLATERAL_INSURANCE` (not user-selectable) exist. Interest expense and Bad debt don't exist.
- Submit workflow: `E6M2NS` gives a number only when `application_number` is blank. `MXHGYH` runs that first, then sets `status` to submitted and `submitted_at`.
- Removed: `Party.business_name`, `Party.years_at_address`, `LoanCategory.property`, `Application.business_party`, `LoanType.category`, and the `AdaptiveLoanDocumentsSection` component. `AdaptiveLoanCollateralSection` is confirmed gone.

- Removed `Application.loan_name` and `Application.loan_category` (Round 8), before the new form was published. Until it is, a restored draft comes back with no loan category or loan name.
