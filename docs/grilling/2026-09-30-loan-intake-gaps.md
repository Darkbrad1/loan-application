---
date: 2026-09-30
topic: loan intake gaps
tags: [grilling]
---

# Loan intake gaps

The parts of the Adaptive Loan intake form (branch `redesign`) that aren't fully worked out yet. The starting point is the "Decisions still waiting" list in `claude.md`, plus gaps found in the new credit card, overdraft, and student screens.

Already settled from the code, so not asked:

- Input boxes use Saturn `FormField` on `redesign`, with `el-select` for the lists the form decides. That's treated as decided.
- Income tax and NIS rates are facts, so a helper is looking them up instead of asking.
- The AML and declaration questions need the compliance officer, so they're out of scope for this session.

## Round 1

❓ **Q1** - **"Loan type" or "Product"**: The code uses both words for the same Saturn `LoanType` record (for example "Auto loan, 5 years"). Which word do we use when talking about the form?

- [ ] Product
- [ ] Loan type

➡️ Product. It's the word on the cards the applicant picks from. "Loan category" stays the word for the group it sits in (for example Automotive).

---

❓ **Q2** - **Who issues the credit card**: Does the credit union issue its own cards, or go through a partner (a bank or card company)? This decides which card details the form asks for later.

- [ ] Its own cards
- [ ] Through a partner

➡️ Through a partner. Small credit unions usually do this. Keep asking only the name on the card and how to get it, since the partner collects anything else.

---

❓ **Q3** - **Guarantor on student loans**: Does a student loan always need a guarantor, or only when the student has no income?

- [ ] Always
- [ ] Only when the student has no income

➡️ Always. It's one switch on the student product in Saturn (`requires_guarantor`) with no new code. "Only with no income" leaves a grey area about how much income is enough.

---

❓ **Q4** - **Students and "What is your work situation?"**: The answers come from Saturn's `employment_status` list. The form only treats "Unemployed" and "Retired" as having no job, so a full-time student has no honest answer.

- [ ] Add a "Student" answer that works like Unemployed (no job or pay screens, no NIS)
- [ ] Add "Student" and still ask about any job
- [ ] Leave it; students pick Unemployed

➡️ Add "Student", working like Unemployed. A part-time job or an allowance goes under other income.

---

❓ **Q5** - **Tuition compared with the amount asked**: Nothing checks the amount asked for against tuition plus other costs. Tuition can be in another currency (for example US$).

- [ ] No check; staff compare them
- [ ] A warning the applicant can ignore
- [ ] Block the application if the amount is more than the costs

➡️ No check. The form has no exchange rates, so any check would be wrong for fees in US$.

---

❓ **Q6** - **Savings held against a card or overdraft**: When the applicant says "secure it with my savings", the form only checks the amount is above 0. Should the savings amount have a limit?

- [ ] It can't be more than the limit asked for
- [ ] It must equal the limit (fully secured)
- [ ] No rule; staff decide

➡️ It can't be more than the limit asked for. Holding more savings than the limit makes no sense, and anything less is staff's call.

---

❓ **Q7** - **Whose account the overdraft is on**: The form asks for an account number and only checks it isn't empty. It can't see the credit union's accounts, so it can't check who owns it.

- [ ] Any account held by someone on this application; the form just asks
- [ ] Only the primary applicant's account; say so on the screen

➡️ Only the primary applicant's account, and say so under the question. A joint account is fine if the primary applicant is on it.

---

❓ **Q8** - **Guarantor's share of the loan**: Borrowers split 100% of the loan evenly. Guarantors get 0% for now. Should they keep 0%?

- [ ] 0%
- [ ] A share, like a borrower

➡️ 0%. A guarantor only pays if the borrowers don't, and their promise is already recorded as the guarantee amount.

---

❓ **Q9** - **Who fills in a business loan**: On a business loan, the person filling in the form is saved as "Primary Applicant", and the business is saved as the "Business Borrower".

- [ ] Keep "Primary Applicant"
- [ ] A new role, "Authorised Signatory" (a director or someone allowed to sign for the business)

➡️ Authorised Signatory. The business owes the money, so calling the person the primary applicant says they owe it, which is wrong unless they also guarantee it.

---

❓ **Q10** - **How many IDs each applicant gives**: The NIS card is already required for everyone. `MINIMUM_IDENTIFICATIONS` is 1 on top of that.

- [ ] 1 photo ID (plus the NIS card)
- [ ] 2 IDs (plus the NIS card)

➡️ 1 photo ID. With the NIS card that's already two documents, and each extra one loses applicants who don't have it.

---

❓ **Q11** - **When submitting fails**: Today the form sets the status to "submitted" and then runs the workflow that gives the application number. If the workflow fails, the application is submitted with no number.

- [ ] The workflow sets the status and the number together
- [ ] Keep the current order; staff spot stuck applications

➡️ The workflow sets both together. Then an application is never "submitted" without a number, and the applicant can simply try again.

---

❓ **Q12** - **Upload boxes**: Uploads use our own `AdaptiveLoanDocumentRequirements` card, which lives only in Saturn (it's not in the repo). The other choice is Saturn's built-in `typed_file_upload`.

- [ ] Keep ours, and add its file to the repo
- [ ] Switch to Saturn's `typed_file_upload`

➡️ Keep ours and add the file to the repo. It already handles the locks, per-item scopes, and the upload fields Saturn needs, and switching would redo all of that.

---

❓ **Q13** - **Top income tax rate**: The form charges 30% on monthly income above EC$5,000. A World Bank paper (2022) says Grenada cut the top rate from 30% to 28% in 2019, and the middle band from 15% to 10% in 2016. The IRD site confirms the EC$3,000 a month tax-free amount. Some private tax sites still show 15% and 30%, but those match the rates from before the cuts. The helper couldn't open the IRD tax tables, because the network blocks ird.gd. NIS at 6.25%, capped at EC$5,200 a month (EC$325 at most), matches the NIS site for 2026. With 28%, the EC$6,000 worked example becomes EC$480 tax instead of EC$500.

- [ ] Change to 28%, and update the worked example
- [ ] Keep 30% until IRD confirms

➡️ Change to 28%. The only dated source for the cut is a World Bank paper, and the 30% figures are all from sites that are out of date. Ask IRD to confirm it anyway.
