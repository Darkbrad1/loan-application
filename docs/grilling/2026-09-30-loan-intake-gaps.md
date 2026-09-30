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

---

❓ **Q2** - **Who issues the credit card**: `Application` has only `name_on_card` and `card_collection_method` for cards. Does the credit union issue its own cards, or go through a partner (a bank or card company)?

- [ ] Its own cards
- [ ] Through a partner

➡️ Through a partner. Small credit unions usually do this, and the two fields already in Saturn are what a partner needs from the application.

---

❓ **Q3** - **Guarantor on student loans**: `LoanType.requires_guarantor` is a yes/no on the product. Does a student loan always need a guarantor, or only when the student has no income?

- [ ] Always (switch it on for the student product)
- [ ] Only when the student has no income

➡️ Always. It's one switch in Saturn with no new code. "Only with no income" needs a new rule and leaves a grey area about how much income is enough.

---

❓ **Q4** - **Students and "What is your work situation?"**: `ApplicationParty.employment_status` is a text field whose answers come from its Saturn list. The form treats only "Unemployed" and "Retired" as having no job, so a full-time student has no honest answer.

- [ ] Add a "Student" answer in Saturn that works like Unemployed (no job or pay screens, no NIS)
- [ ] Add "Student" and still ask about any job
- [ ] Leave it; students pick Unemployed

➡️ Add "Student", working like Unemployed. A part-time job or an allowance goes under other income (`IncomeSource`).

---

❓ **Q5** - **Tuition compared with the amount asked**: `tuition_amount` and `other_study_costs` are numbers, with `tuition_currency` as text. Nothing checks them against `requested_loan_amount`, and there's no field for an exchange rate.

- [ ] No check; staff compare them
- [ ] A warning the applicant can ignore
- [ ] Block the application if the amount is more than the costs

➡️ No check. With fees in US$ and no exchange rate, any check would be wrong.

---

❓ **Q6** - **Savings held against a card or overdraft**: When `is_secured_by_savings` is yes, the form only checks that `secured_savings_amount` is above 0. Should it have a limit?

- [ ] It can't be more than `requested_credit_limit`
- [ ] It must equal `requested_credit_limit` (fully secured)
- [ ] No rule; staff decide

➡️ It can't be more than the limit asked for. Holding more savings than the limit makes no sense, and anything less is staff's call.

---

❓ **Q7** - **Whose account the overdraft is on**: `linked_account_number` is plain text, and the form can't see the credit union's accounts, so it can't check who owns the account.

- [ ] Any account held by someone on this application; the form just asks
- [ ] Only the primary applicant's account; say so on the screen

➡️ Only the primary applicant's account, and say so under the question. A joint account is fine if the primary applicant is on it.

---

❓ **Q8** - **Guarantor's share of the loan**: Borrowers split 100% of `ownership_percentage` evenly. Guarantors get 0% for now, and their promise is saved in `guarantee_type` and `guarantee_amount`.

- [ ] 0%
- [ ] A share, like a borrower

➡️ 0%. A guarantor only pays if the borrowers don't, and `guarantee_amount` already records how much they promise.

---

❓ **Q9** - **The person filling in a business loan**: `ApplicationParty` already has `signing_authority` (text) and `is_primary_contact` (yes/no), and the form uses neither for this. Today the person is saved as "Primary Applicant", and every "primary applicant" rule applies to them: references, household bills, and the purchase asset. Options:

- [ ] Keep the role "Primary Applicant", and ask "What's your position in the business?" into `signing_authority`
- [ ] A new role "Authorised Signatory" with `signing_authority`, and change every primary-applicant rule to cope with it

➡️ Keep "Primary Applicant" and add the `signing_authority` question (for example Director, Owner, Manager). The field is already in Saturn. A new role touches every rule that looks for the primary applicant, for little gain.

---

❓ **Q10** - **Extra business details**: `Party` has four business fields the form never asks: `trading_name`, `industry`, `business_license_number`, `website`. Which should a business loan ask?

- [ ] Trading name
- [ ] Industry
- [ ] Business licence number
- [ ] Website

➡️ Trading name and industry, both required. Put the licence number behind "+ Add more details" as optional. Skip the website, since it tells a loan officer little.

---

❓ **Q11** - **How to contact the applicant**: `Application.preferred_contact_method` exists but the form never asks it.

- [ ] Ask it on the contact details screen, as answer cards from its Saturn list
- [ ] Don't ask it

➡️ Ask it, with the answers set in Saturn (for example Phone call, WhatsApp, Email). It's one tap, and staff then know how to reach people for follow-up.

---

❓ **Q12** - **Products that aren't offered any more**: `LoanType` has a text `status` (no `is_active`), and the form shows every product in the category whatever its status.

- [ ] Show only products whose status is "Active"; treat an empty status as active
- [ ] Show every product (staff delete ones they stop offering)

➡️ Show only "Active", with empty counted as active so nothing disappears before the statuses are filled in. Staff can then retire a product without deleting it.

---

❓ **Q13** - **Documents staff ask for after sending**: `Application.requested_documents` links to `DocumentRequest` (document type, status, reason). The form doesn't use it. Should the intake form show these requests and let the applicant upload them?

- [ ] Not now; it's back-office work (Phase 3)
- [ ] Yes, on a returning visit, before the review screen

➡️ Not now. Drafts only resume in the same browser, so an applicant could miss the requests anyway. It fits with the back office and a login or emailed link.

---

❓ **Q14** - **How many IDs each applicant gives**: The NIS card is already required for everyone. `MINIMUM_IDENTIFICATIONS` is 1 on top of that.

- [ ] 1 photo ID (plus the NIS card)
- [ ] 2 IDs (plus the NIS card)

➡️ 1 photo ID. With the NIS card that's already two documents, and each extra one loses applicants who don't have it.

---

❓ **Q15** - **When submitting fails**: Today the form sets `status` to "submitted" and then runs the workflow that fills `application_number`. If the workflow fails, the application is submitted with no number.

- [ ] The workflow sets `status`, `submitted_at`, and `application_number` together
- [ ] Keep the current order; staff spot stuck applications

➡️ The workflow sets all three together. Then an application is never "submitted" without a number, and the applicant can simply try again.

---

❓ **Q16** - **Upload boxes**: Uploads use our own `AdaptiveLoanDocumentRequirements` card, which lives only in Saturn (it's not in the repo). The other choice is Saturn's built-in `typed_file_upload`.

- [ ] Keep ours, and add its file to the repo
- [ ] Switch to Saturn's `typed_file_upload`

➡️ Keep ours and add the file to the repo. It already handles the locks, per-item scopes, and the upload fields Saturn needs.

---

❓ **Q17** - **Top income tax rate**: The form charges 30% on monthly income above EC$5,000. A World Bank paper (2022) says Grenada cut the top rate from 30% to 28% in 2019, and the middle band from 15% to 10% in 2016. The IRD site confirms the EC$3,000 a month tax-free amount, and NIS at 6.25% capped at EC$5,200 a month (EC$325 at most) matches the NIS site for 2026. The helper couldn't open IRD's tax tables, because the network blocks ird.gd. With 28%, the EC$6,000 worked example becomes EC$480 tax instead of EC$500.

- [ ] Change to 28%, and update the worked example
- [ ] Keep 30% until IRD confirms

➡️ Change to 28%, and still ask IRD to confirm it. The only dated source for the cut is the World Bank paper, and the sites still showing 30% look out of date.
