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

---

❓ **Q2** - **One Saturn install per lender, or one shared**: Does each lender get its own Saturn install (its own records, logo, and settings), or do several lenders share one install?

- [ ] One install per lender
- [ ] Several lenders share one install

➡️ One install per lender. Logo and colours already come from SystemConfiguration, which is one per install, and sharing would mean every record needs a lender link so applicants never see another lender's loan types.

---

❓ **Q3** - **Are all lenders in Grenada?**: NIS and income tax rules, the EC$ currency, and "parish is required in Grenada" are all Grenadian.

- [ ] Yes, Grenada only for now
- [ ] No, other countries too

➡️ Grenada only for now. Other countries would need their own tax and social security rules, which is a separate piece of work.

---

❓ **Q4** - **Turning credit cards on or off**: Is switching the Credit card loan category to inactive enough for a lender without cards? And for a lender with cards, do the questions stay the same (name on the card, how to get it) whether it issues them itself or through a partner?

- [ ] Yes to both: category on or off, same questions for everyone
- [ ] A lender that issues its own cards needs extra questions

➡️ Yes to both. It needs no new setting, and the two card questions cover what either kind of issuer needs from the application.

---

❓ **Q5** - **Where the number of IDs is set**: Some lenders need 1 ID and some need 2. Today it's one number in the code (`MINIMUM_IDENTIFICATIONS`).

- [ ] A new number field on `LoanType`, `minimum_identifications` (empty means 1)
- [ ] A per-lender setting in a new Saturn resource
- [ ] Keep the constant; change it in the code for each lender

➡️ A field on `LoanType`. The developer controls LoanType, it needs no new resource, and a lender sets the same number on all its loan types. It also lets one loan type ask for more if a lender ever wants that.

---

❓ **Q6** - **Trading name when it's the same as the legal name**: You made trading name required. Many small businesses trade under their legal name.

- [ ] Required, with a "Same as the legal name" tick box that copies it
- [ ] Optional; empty means the same as the legal name
- [ ] Required, typed every time

➡️ Required, with a "Same as the legal name" tick box. Staff always see a trading name, and the applicant doesn't type it twice.

---

❓ **Q7** - **Industry answers**: `Party.industry` is text. Should the answers come from a Saturn list (for example Retail, Construction, Agriculture, Tourism, Other), or be typed?

- [ ] A Saturn list, shown as a dropdown
- [ ] Typed

➡️ A Saturn list. Staff can then group and report by industry, which typed answers won't allow.

---

❓ **Q8** - **Signing authority answers**: `ApplicationParty.signing_authority` is text. Should "What's your position in the business?" use a Saturn list (for example Owner, Director, Partner, Manager, Other), or be typed?

- [ ] A Saturn list, shown as answer cards
- [ ] Typed

➡️ A Saturn list, as answer cards. It's a short list, and cards are easier than typing for the people this form is for.

---

❓ **Q9** - **NIS card for students**: Every applicant must give an NIS number and upload the NIS card (the `applicant-all` document). A full-time student who has never worked may not have one.

- [ ] Keep it required for students
- [ ] Optional when the work situation is Student

➡️ Keep it required. Grenada issues NIS numbers to school leavers, and the lender uses the NIS card as a standard ID check.

---

❓ **Q10** - **Trying again after a failed send**: With the workflow setting status, date, and number together, what happens when it fails?

- [ ] The form says "We couldn't send your application. Please try again.", the application stays a draft, and pressing Send again reruns the workflow. The workflow gives a number only when the application has none, so a retry never gets a second number.
- [ ] Something else

➡️ The first option. The applicant can fix it by pressing Send again, and the "only when it has none" rule stops double numbers.
