<template>
    <main class="adaptive-form min-h-screen text-gray-900" :style="['background:#f4f6f8;font-family:Inter,system-ui,-apple-system,Segoe UI,Roboto,sans-serif;line-height:1.5', brandStyle]">
        <!-- Header: the logo, name, and colours come from the system configuration -->
        <header class="topbar" style="background:var(--brand);color:var(--brand-ink)">
            <div class="topbar-inner flex items-center justify-between gap-4 px-5 py-2" style="max-width:960px;min-height:68px;margin:0 auto">
                <div class="brand flex items-center gap-3" style="min-width:0">
                    <img v-if="companyLogo" :src="companyLogo" :alt="companyName" class="brand-logo block rounded" style="max-width:150px;max-height:44px;object-fit:contain;background:var(--logo-bg)" />
                    <span class="brand-name text-lg font-bold" style="white-space:nowrap;overflow:hidden;text-overflow:ellipsis" :class="companyLogo ? 'hidden md:block' : ''">{{ companyName }}</span>
                </div>
                <div class="topbar-side flex items-center justify-end gap-3">
                    <!-- Testing only: fills each step with sample data -->
                    <label v-if="testModeAvailable" class="test-toggle flex items-center gap-2 px-3 rounded-full text-xs" style="border:1px dashed currentColor;cursor:pointer;opacity:0.85">
                        <el-switch :model-value="testMode" size="small" @update:model-value="toggleTestMode" />
                        <span class="hidden md:inline">Test mode</span>
                    </label>
                    <label v-if="testMode" class="test-toggle flex items-center gap-2 px-3 rounded-full text-xs" style="border:1px dashed currentColor;cursor:pointer;opacity:0.85">
                        <el-switch :model-value="testKeepDraft" size="small" @update:model-value="toggleTestKeepDraft" />
                        <span class="hidden md:inline">Remember draft</span>
                    </label>
                    <span v-if="formData.id && !receipt" class="saved-chip flex items-center gap-1 px-3 py-1 rounded-full text-sm font-semibold bg-green-100 text-green-800">
                        <v-icon size="16">mdi-cloud-check-outline</v-icon>
                        <span class="hidden md:inline">Saved</span>
                    </span>
                </div>
            </div>
        </header>

        <!-- After sending: a clear receipt -->
        <section v-if="receipt" class="page px-3 pt-6 pb-12" style="max-width:680px;margin:0 auto">
            <div class="panel receipt bg-white p-5 md:p-8 shadow flex flex-col items-center text-center" style="border:1px solid #e5e7eb;border-radius:20px">
                <div class="receipt-icon flex items-center justify-center rounded-full mb-5 bg-green-100 text-green-700" style="width:72px;min-width:72px;max-width:72px;height:72px;min-height:72px;flex:0 0 72px;flex-grow:0;flex-shrink:0;align-self:center;padding:0;margin:0;box-sizing:border-box"><v-icon size="40">mdi-check</v-icon></div>
                <h1 class="screen-title text-2xl md:text-3xl font-bold mb-2" style="line-height:1.25;color:#111827">Application sent</h1>
                <p class="screen-help text-base md:text-lg text-gray-600 mb-6">
                    Thank you. We'll look at your application and contact you soon.
                </p>
                <template v-if="receipt.number">
                    <p class="receipt-label mt-5 mb-2 text-gray-600 font-semibold">Your reference number</p>
                    <strong class="reference-card block px-6 py-3 rounded-lg text-xl font-bold" style="border:2px dashed var(--brand);background:var(--brand-tint);font-family:ui-monospace,monospace;letter-spacing:0.04em">{{ receipt.number }}</strong>
                    <p class="hint mt-2 text-base text-gray-600">Keep this number. You'll need it if you contact us.</p>
                </template>
                <p v-else class="hint mt-2 text-base text-gray-600">
                    Your reference number is still being assigned. Contact us if you need it before we get in touch.
                </p>
                <button type="button" class="btn btn-secondary inline-flex items-center justify-center gap-2 px-6 rounded-lg text-lg font-bold bg-white mt-6" style="min-height:56px;cursor:pointer;color:var(--brand-text);border:2px solid var(--brand)" @click="reset">
                    Start another application
                </button>
            </div>
        </section>

        <section v-else class="page px-3 pt-6 pb-12" style="max-width:680px;margin:0 auto">
            <!-- Where they are: a short label and one bar per part -->
            <div v-if="currentScreen.kind !== 'welcome'" class="progress-block mb-5">
                <button v-if="!isFirstScreen" type="button" class="back-link inline-flex items-center gap-1 mb-3 py-1 text-base font-semibold" style="color:var(--brand-text);background:transparent;border:0;cursor:pointer" :disabled="saving" @click="back">
                    <v-icon size="20">mdi-chevron-left</v-icon>
                    Back
                </button>
                <p class="progress-label mb-2 text-base font-bold text-gray-700">
                    Part {{ step + 1 }} of {{ path.length }}
                    <span class="font-normal text-gray-600">· {{ current.title }}</span>
                </p>
                <div class="progress-segments flex gap-2" aria-hidden="true">
                    <span v-for="(stepItem, stepIndex) in path" :key="stepItem.id" class="segment flex-1 rounded-full overflow-hidden bg-gray-200" style="height:6px">
                        <i class="block h-full rounded-full" :style="{ width: segmentFill(stepIndex), background: 'var(--accent)' }"></i>
                    </span>
                </div>
            </div>

            <el-form label-position="top" size="large" class="panel bg-white p-5 md:p-8 shadow" style="border:1px solid #e5e7eb;border-radius:20px" @submit.prevent>
                <!-- Welcome: what to have ready, before starting -->
                <div v-if="currentScreen.kind === 'welcome'" class="welcome">
                    <div class="welcome-icon flex items-center justify-center rounded-full mb-5" style="width:72px;min-width:72px;max-width:72px;height:72px;min-height:72px;flex:0 0 72px;flex-grow:0;flex-shrink:0;align-self:center;padding:0;margin:0;box-sizing:border-box;background:var(--brand-tint);color:var(--brand-text)"><v-icon size="36">mdi-hand-wave-outline</v-icon></div>
                    <h1 class="screen-title text-2xl md:text-3xl font-bold mb-2" style="line-height:1.25;color:#111827">Apply for a loan</h1>
                    <p class="screen-help text-base md:text-lg text-gray-600 mb-6">
                        It takes about 15 minutes. Your answers are saved as you go, so you can stop and come back.
                    </p>
                    <p class="welcome-subtitle mb-3 text-lg font-bold">Have these ready</p>
                    <ul class="ready-list flex flex-col gap-3" style="list-style:none;padding:0;margin:0">
                        <li class="ready-item flex items-center gap-3 px-4 py-3 rounded-lg bg-gray-50 text-base"><v-icon class="ready-icon" style="color:var(--brand-text)">mdi-card-account-details-outline</v-icon> A photo ID, like a passport or driver's licence</li>
                        <li class="ready-item flex items-center gap-3 px-4 py-3 rounded-lg bg-gray-50 text-base"><v-icon class="ready-icon" style="color:var(--brand-text)">mdi-numeric</v-icon> Your NIS number</li>
                        <li class="ready-item flex items-center gap-3 px-4 py-3 rounded-lg bg-gray-50 text-base"><v-icon class="ready-icon" style="color:var(--brand-text)">mdi-file-document-outline</v-icon> A recent payslip or proof of income</li>
                        <li class="ready-item flex items-center gap-3 px-4 py-3 rounded-lg bg-gray-50 text-base"><v-icon class="ready-icon" style="color:var(--brand-text)">mdi-credit-card-outline</v-icon> Details of any loans or cards you're paying</li>
                    </ul>
                </div>

                <template v-else>
                    <h1 class="screen-title text-2xl md:text-3xl font-bold mb-2" style="line-height:1.25;color:#111827">{{ currentScreen.title }}</h1>
                    <p v-if="currentScreen.help" class="screen-help text-base md:text-lg text-gray-600 mb-6">{{ currentScreen.help }}</p>
                </template>

                <!-- Problems with this screen, in one place -->
                <div v-if="alert.text" class="notice flex items-start gap-3 mb-5 p-4 rounded-lg border font-semibold text-base" style="border-left-width:5px" :class="noticeClass" role="alert">
                    <v-icon size="22">{{ alert.type === 'success' ? 'mdi-check-circle-outline' : 'mdi-alert-circle-outline' }}</v-icon>
                    <span>{{ alert.text }}</span>
                </div>

                <!-- A Yes/No question. "No" skips the section after it. -->
                <div v-if="currentScreen.kind === 'gate'" class="gate flex flex-col gap-3" role="radiogroup">
                    <button
                        v-for="option in [{ value: true, label: 'Yes', icon: 'mdi-check' }, { value: false, label: 'No', icon: 'mdi-close' }]"
                        :key="option.label"
                        type="button"
                        role="radio"
                        class="gate-option flex items-center gap-4 w-full px-5 py-4 rounded-lg text-xl font-semibold text-left"
                        :style="['min-height:68px;border-width:2px;border-style:solid;cursor:pointer;justify-content:flex-start;color:#111827', (answers[currentScreen.answer] === option.value) ? 'border-color:var(--brand);background:var(--brand-tint);box-shadow:inset 0 0 0 1px var(--brand)' : 'border-color:#d1d5db;background:#ffffff']"
                        :aria-checked="answers[currentScreen.answer] === option.value"
                        @click="answer(currentScreen.answer, option.value)"
                    >
                        <span class="radio-mark flex items-center justify-center rounded-full" :style="['width:26px;min-width:26px;max-width:26px;height:26px;min-height:26px;flex:0 0 26px;flex-grow:0;flex-shrink:0;align-self:center;padding:0;margin:0;box-sizing:border-box;border-width:2px;border-style:solid', (answers[currentScreen.answer] === option.value) ? 'background:var(--brand);border-color:var(--brand);color:var(--brand-ink)' : 'background:#ffffff;border-color:#d1d5db;color:transparent']"><v-icon size="18">{{ option.icon }}</v-icon></span>
                        <span class="choice-text" style="flex:1 1 auto;min-width:0;text-align:left;display:block">{{ option.label }}</span>
                    </button>
                    <p
                        v-if="answers[currentScreen.answer] === false && gateWarning(currentScreen.answer)"
                        class="gate-warning flex items-start gap-2 mt-2 p-3 rounded-lg bg-orange-50 text-orange-800 font-semibold text-base"
                    >
                        <v-icon size="20">mdi-alert-outline</v-icon>
                        {{ gateWarning(currentScreen.answer) }}
                    </p>
                </div>

                    <AdaptiveLoanProductSection
                        v-else-if="currentScreen.kind === 'loan'"
                        :loans="loans"
                        :products="productsForLoan"
                        :loan-category="selectedCategory ? selectedCategory.id : ''"
                        :loan-type-id="formData.loan_type_id"
                        :loaded="optionsLoaded"
                        @select-category="chooseLoan"
                        @select-product="selectProduct"
                    />

                    <!-- The other people on the loan, or the references -->
                    <AdaptiveLoanApplicantsSection
                        v-else-if="currentScreen.kind === 'people' || currentScreen.kind === 'references'"
                        :screen="currentScreen.kind"
                        :primary="formData.primary"
                        :parties="formData.parties"
                        :references="formData.references"
                        :lookups="lookups"
                        :resource-props="resourceProps"
                        :show-errors="showErrors"
                        @update:parties="formData.parties = $event"
                        @update:references="formData.references = $event"
                        @request-add="addApplicant"
                        @request-remove="removeApplicant"
                    />

                    <!-- One part of one person's details -->
                    <AdaptiveLoanApplicantEditor
                        v-else-if="currentScreen.kind === 'person' && currentPerson"
                        :key="currentScreen.id"
                        :model-value="currentPerson"
                        :screen="currentScreen.part"
                        :is-primary="currentScreen.personKey === formData.primary.client_key"
                        :show-errors="showErrors"
                        :lookups="lookups"
                        :resource-props="resourceProps"
                        :minimum-identifications="minimumIdentifications"
                        :deductions="deductionsByApplicant[currentScreen.personKey] || null"
                        :document-scopes="documentScopes"
                        :uploading-key="uploadingDocumentKey"
                        :documents-disabled="documentsDisabled"
                        @update:model-value="updatePerson(currentScreen.personKey, $event)"
                        @stage-file="stageDocument"
                        @remove-file="removeStagedDocument"
                        @request-file-upload="uploadScopedDocument"
                        @file-rejected="handleRejectedDocument"
                    />

                    <AdaptiveLoanRequestSection
                        v-else-if="currentScreen.kind === 'request'"
                        :screen="currentScreen.part"
                        :show-errors="showErrors"
                        :model-value="requestData"
                        :loan-category="loanKind"
                        :revolving="isRevolving"
                        :selected-product="selectedProduct"
                        :lookups="lookups"
                        :application-props="applicationProps"
                        :resource-props="resourceProps"
                        :amount-minimum="amountMinimum"
                        :amount-maximum="amountMaximum"
                        :term-minimum="termMinimum"
                        :term-maximum="termMaximum"
                        @update:model-value="patchRequest"
                    />

                    <AdaptiveLoanAssetsSection
                        v-else-if="currentScreen.kind === 'assets'"
                        :model-value="formData.assets"
                        :party-options="partyOptions"
                        :primary-party-id="toId(formData.primary.party_id) || ''"
                        :third-party-owner-ids="thirdPartyOwnerPartyIds"
                        :requires-collateral="requiresCollateral"
                        :liens="liensByAsset"
                        :show-errors="showErrors"
                        :lookups="lookups"
                        :resource-props="resourceProps"
                        @update:model-value="formData.assets = $event"
                        :document-scopes="documentScopes"
                        :uploading-key="uploadingDocumentKey"
                        :documents-disabled="documentsDisabled"
                        @stage-file="stageDocument"
                        @remove-file="removeStagedDocument"
                        @request-file-upload="uploadScopedDocument"
                        @file-rejected="handleRejectedDocument"
                    />

                    <!--
                    Liability types are LiabilityType records. Revolving
                    types (credit cards, overdrafts) require a credit limit
                    and show an assessed repayment based on a % of the limit.
                    -->
                    <AdaptiveLoanLiabilitiesSection
                        v-else-if="currentScreen.kind === 'liabilities'"
                        :model-value="formData.liabilities"
                        :application-party-options="applicationPartyOptions"
                        :primary-link-id="toId(formData.primary.application_party_id) || ''"
                        :liability-types="liabilityTypes"
                        :default-revolving-rate="defaultRevolvingRate"
                        :asset-options="assetOptions"
                        :show-errors="showErrors"
                        :lookups="lookups"
                        :resource-props="resourceProps"
                        @update:model-value="formData.liabilities = $event"
                        :document-scopes="documentScopes"
                        :uploading-key="uploadingDocumentKey"
                        :documents-disabled="documentsDisabled"
                        @stage-file="stageDocument"
                        @remove-file="removeStagedDocument"
                        @request-file-upload="uploadScopedDocument"
                        @file-rejected="handleRejectedDocument"
                    />

                    <!--
                    Expense types are ExpenseType records, filtered by the
                    selected loan category and user_selectable.
                    -->
                    <AdaptiveLoanExpensesSection
                        v-else-if="currentScreen.kind === 'expenses'"
                        :model-value="formData.expenses"
                        :application-party-options="applicationPartyOptions"
                        :has-others="applicationPartyOptions.length > 1"
                        :expense-type-options="expenseTypeOptions"
                        :expense-types="expenseTypes"
                        :loan-category-label="loanCategoryLabel"
                        :projected-expenses="projectedInsuranceExpenses"
                        :show-errors="showErrors"
                        :lookups="lookups"
                        :resource-props="resourceProps"
                        @update:model-value="formData.expenses = $event"
                        :document-scopes="documentScopes"
                        :uploading-key="uploadingDocumentKey"
                        :documents-disabled="documentsDisabled"
                        @stage-file="stageDocument"
                        @remove-file="removeStagedDocument"
                        @request-file-upload="uploadScopedDocument"
                        @file-rejected="handleRejectedDocument"
                    />

                    <!--
                    Application-level documents only. Applicant, asset,
                    liability, expense, and collateral documents are
                    uploaded on their own screens.
                    -->
                    <AdaptiveLoanDocumentRequirements
                        v-else-if="currentScreen.kind === 'documents'"
                        standalone
                        title="Documents for this loan"
                        :description="`Please upload these for your ${formData.loan_name || 'loan'}.`"
                        :scope="documentScopes.application"
                        :uploading-key="uploadingDocumentKey"
                        :disabled="documentsDisabled"
                        @stage-file="stageDocument"
                        @remove-file="removeStagedDocument"
                        @request-file-upload="uploadScopedDocument"
                        @file-rejected="handleRejectedDocument"
                    />

                    <!-- Final step: check everything before sending -->
                    <AdaptiveLoanReviewSection
                        v-else-if="currentScreen.kind === 'review'"
                        :application="formData"
                        :applicants="allApplicants"
                        :requires-collateral="requiresCollateral"
                        :summary="reviewSummary"
                        @edit-step="goToStep"
                    />

                <footer class="actions flex mt-8">
                    <button
                        v-if="currentScreen.kind === 'welcome'"
                        type="button"
                        class="btn btn-primary inline-flex items-center justify-center gap-2 px-6 rounded-lg text-lg font-bold flex-1" style="min-height:56px;cursor:pointer;background:var(--brand);color:var(--brand-ink);border:0"
                        @click="next"
                    >
                        Start my application
                        <v-icon size="22">mdi-arrow-right</v-icon>
                    </button>
                    <button
                        v-else-if="!isLastScreen"
                        type="button"
                        class="btn btn-primary inline-flex items-center justify-center gap-2 px-6 rounded-lg text-lg font-bold flex-1" style="min-height:56px;cursor:pointer;background:var(--brand);color:var(--brand-ink);border:0"
                        :disabled="saving || documentsLoading"
                        @click="next"
                    >
                        <v-icon v-if="saving" size="22" class="spin">mdi-loading</v-icon>
                        {{ saving ? 'Saving' : 'Continue' }}
                        <v-icon v-if="!saving" size="22">mdi-arrow-right</v-icon>
                    </button>
                    <button
                        v-else
                        type="button"
                        class="btn btn-primary inline-flex items-center justify-center gap-2 px-6 rounded-lg text-lg font-bold flex-1" style="min-height:56px;cursor:pointer;background:var(--brand);color:var(--brand-ink);border:0"
                        :disabled="saving"
                        @click="submit"
                    >
                        <v-icon v-if="saving" size="22" class="spin">mdi-loading</v-icon>
                        {{ saving ? 'Sending' : 'Send my application' }}
                        <v-icon v-if="!saving" size="22">mdi-send</v-icon>
                    </button>
                </footer>
            </el-form>

            <p class="page-foot flex items-center justify-center gap-2 mt-5 text-sm text-gray-500">
                <v-icon size="16">mdi-lock-outline</v-icon>
                Your information is kept private and secure.
            </p>
        </section>
    </main>
</template>

<script>
/**
 * Fallback revolving-credit rate (3% of the limit), used only when a
 * revolving LiabilityType has no revolving_rate set.
 */
const DEFAULT_REVOLVING_RATE = 0.03;

/**
 * Child records the form creates for a draft, in the order stale ones are
 * deleted: records that point at others go first, so nothing is left
 * referencing a record that has already been deleted.
 */
const TRACKED_RESOURCES = [
    // Projected insurance expenses point at their Collateral, so expenses go first.
    "Expense",
    "Collateral",
    "AssetOwnership",
    "LiabilityResponsibility",
    // Liabilities can point at the asset that secures them.
    "Liability",
    "Asset",
    "IncomeSource",
    "Reference",
    "ApplicationParty",
];

/**
 * How many times a month each payment frequency happens, used to turn any
 * amount into a monthly figure (monthly_equivalent). Keys are the standard
 * frequency choices in Saturn, lowercased, plus a few common spellings.
 */
const MONTHLY_FACTORS = {
    weekly: 52 / 12,
    fortnightly: 26 / 12,
    "bi-weekly": 26 / 12,
    biweekly: 26 / 12,
    "twice monthly": 2,
    "semi-monthly": 2,
    monthly: 1,
    quarterly: 1 / 3,
    "twice yearly": 1 / 6,
    "semi-annually": 1 / 6,
    yearly: 1 / 12,
    annually: 1 / 12,
    annual: 1 / 12,
};

/**
 * An amount paid at some frequency, as a monthly figure (rounded to cents).
 * Unknown or missing frequencies count as monthly. Returns null without an
 * amount.
 */
const monthlyAmount = (amount, frequency) => {
    if (amount === null || amount === undefined || amount === "") return null;
    const key = String(frequency || "monthly").trim().toLowerCase();
    const factor = MONTHLY_FACTORS[key] ?? 1;
    return Math.round(Number(amount) * factor * 100) / 100;
};

/** Asset status for the vehicle or property a purchase loan is buying. */
const PURCHASE_ASSET_STATUS = "To be purchased";

/** ExpenseType code for the projected monthly cost of collateral insurance. */
const COLLATERAL_INSURANCE_CODE = "COLLATERAL_INSURANCE";

/**
 * The parts of an applicant's details, one short screen each, in order.
 * applicantPartIssue() checks each part.
 */
const APPLICANT_PARTS = [
    "about",
    "home",
    "membership",
    "ids",
    "job",
    "pay",
    "declarations",
    "consent",
];

/** Applicant roles with their own meaning in the form. */
const GUARANTOR_ROLE = "Guarantor";

/** ApplicationParty role of the business on a business (ORGANIZATION) loan. */
const BUSINESS_BORROWER_ROLE = "Business Borrower";

/** Work situations with no job details and no NIS (lower case). */
const NOT_WORKING_STATUSES = ["unemployed", "retired", "student"];

/** Loan kinds that give a limit to draw on, with no term (revolving credit). */
const REVOLVING_KINDS = ["credit_card", "overdraft"];

/**
 * The form's kind for a LoanCategory code, e.g. "CREDIT_CARD" -> "credit_card".
 * The kind picks the screens. The seven codes with their own screens are
 * PROPERTY, AUTOMOTIVE, PERSONAL, ORGANIZATION, CREDIT_CARD, OVERDRAFT, and
 * STUDENT; any other code gets the personal loan screens.
 */
function categoryKind(code) {
    return String(code || "")
        .trim()
        .toLowerCase()
        .replace(/[\s-]+/g, "_");
}

/** An icon for each kind, when the LoanCategory record has none. */
const KIND_ICONS = {
    personal: "mdi-account-cash",
    automotive: "mdi-car",
    property: "mdi-home",
    organization: "mdi-storefront",
    credit_card: "mdi-credit-card-outline",
    overdraft: "mdi-bank-transfer",
    student: "mdi-school-outline",
};

/**
 * Generates a unique client-side key for list rows that have no server ID yet.
 */
const generateRowKey = (prefix = "row") =>
    `${prefix}_${Date.now()}_${Math.random().toString(36).slice(2, 9)}`;

/**
 * Forms of identification each borrower and guarantor must give when the
 * loan type's minimum_identifications is empty. Each lender sets the number
 * on its own loan types in Saturn. The NIS card doesn't count toward it.
 */
const DEFAULT_MINIMUM_IDENTIFICATIONS = 1;

/**
 * Grenada statutory deduction settings, used to estimate each applicant's
 * monthly NIS and income tax (PAYE) from their gross monthly income.
 * Confirm these with NIS Grenada and the Inland Revenue Division before
 * relying on them: published sources disagree on the income tax bands.
 */
const STATUTORY_DEDUCTIONS = {
    nis: {
        employeeRate: 0.0625, // employee share, 6.25%
        selfEmployedRate: 0.135, // self-employed pay both shares, 13.5%
        monthlyInsurableCap: 5200, // earnings above this aren't insurable
        minimumAge: 16,
        // Pensionable age is being phased from 60 to 65 (2024 to 2029) by
        // birth year. 65 is the conservative choice: it never understates NIS.
        pensionableAge: 65,
    },
    // Monthly bands, applied progressively to gross monthly income.
    incomeTaxBands: [
        { upTo: 3000, rate: 0 }, // EC$36,000 a year personal allowance
        { upTo: 5000, rate: 0.1 }, // next EC$24,000 a year
        { upTo: Infinity, rate: 0.3 }, // above EC$60,000 a year
    ],
};

/** Default country for new addresses and identifications. */
const DEFAULT_COUNTRY = "Grenada";

/**
 * Shows the "Test mode" switch in the header. When it's on, each step is
 * filled with sample data as you reach it, so the form can be run through
 * quickly. Set this to false before real applicants use the form.
 */
const TEST_MODE_AVAILABLE = true;

/**
 * Role for someone who isn't borrowing but owns (or part-owns) an asset
 * offered as collateral. Only applicants can own assets on the
 * application, so these owners are added on the Applicants step with a
 * short form: name, relationship, and contact details only.
 */
const THIRD_PARTY_OWNER_ROLE = "Third Party Owner";

/**
 * Creates an empty identification row (a PartyIdentification record).
 */
const createEmptyIdentification = (isPrimary = false) => ({
    client_key: generateRowKey("ident"),
    id: null,
    identification_type: "",
    identification_number: "",
    issuing_country: DEFAULT_COUNTRY,
    issue_date: "",
    expiry_date: "",
    is_primary: isPrimary,
    // IDs of scans uploaded against this identification
    document_ids: [],
});

/**
 * Creates an empty applicant object. Used for both the primary applicant
 * and any additional parties on the application.
 */
const createEmptyApplicant = (role = "") => ({
    client_key: generateRowKey("party"),
    party_id: null,
    application_party_id: null,
    role,
    // PERSON or ORGANIZATION. Only a Third Party Owner can be a business.
    kind: "PERSON",
    first_name: "",
    last_name: "",
    business_name: "",
    // Business loans, Primary Applicant only: their position in the business.
    signing_authority: "",
    // Everyone: how they'd like to be contacted (ApplicationParty).
    preferred_contact_method: "",
    // Third Party Owners only: how they're related to the primary applicant.
    relationship_to_applicant: "",
    email: "",
    phone: "",
    date_of_birth: "",
    marital_status: "",
    // Address, split into street, parish, and country
    address: "",
    parish: "",
    country: DEFAULT_COUNTRY,
    nis_number: "",
    // Credit union membership (non-members may apply; staff follow up)
    is_member: false,
    member_number: "",
    // Anti-money-laundering questions
    citizenship: "",
    residency_status: "",
    tin: "",
    is_pep: false,
    pep_details: "",
    // Housing. The previous address is asked for under 2 years at this one.
    housing_status: "",
    // The date they moved to their current address (Party.address_since).
    address_since: "",
    previous_address: "",
    previous_parish: "",
    previous_country: DEFAULT_COUNTRY,
    mailing_address: "",
    number_of_dependants: null,
    // Linked to Party.ids. saved_identification_ids is what was on the
    // server at the last save or restore, so removed IDs can be deleted.
    identifications: [createEmptyIdentification(true)],
    saved_identification_ids: [],
    employment_status: "",
    employment_type: "",
    employment_start_date: "",
    employer_name: "",
    job_title: "",
    // Worked out from employment_start_date when it's saved
    years_employed: null,
    // Pay per pay period; gross_monthly_income is worked out from these.
    gross_pay: null,
    pay_frequency: "",
    gross_monthly_income: null,
    annual_revenue: null,
    // Asked for under 2 years in the current job
    previous_employer_name: "",
    previous_job_title: "",
    previous_employment_years: null,
    // Other income (IncomeSource records, linked through income_ids)
    incomes: [],
    // Guarantors only
    guarantee_type: "",
    guarantee_amount: null,
    // Declarations (a "yes" needs an explanation in declaration_details)
    declared_bankruptcy: false,
    declared_judgments: false,
    declared_arrears: false,
    declared_other_applications: false,
    declaration_details: "",
    consent_accuracy_confirmation: false,
    consent_credit_check: false,
    consent_data_processing: false,
    consented_at: null,
    consent_policy_version: "",
    // IDs of documents uploaded against this applicant's ApplicationParty record
    document_ids: [],
});

/**
 * Creates the collateral details kept on each asset (asset.collateral).
 * An asset is offered as collateral when "enabled" is on; it's then saved
 * as a Collateral record linked to the application and the asset. The
 * asset's own name, type, and declared value are used; collateral has no
 * value of its own.
 */
const createEmptyCollateral = () => ({
    enabled: false,
    id: null,
    description: "",
    document_ids: [],
    // The projected monthly insurance Expense saved for this collateral
    projected_expense_id: null,
    // Insurance policy, or a quote when there's no policy yet
    insurance: {
        type: "",
        status: "Quote",
        provider: "",
        reference: "",
        coverage_amount: null,
        premium: null,
        premium_frequency: "",
        expiry_date: "",
    },
});

/** Creates an empty other-income row (an IncomeSource record). */
const createEmptyIncome = () => ({
    client_key: generateRowKey("income"),
    id: null,
    income_type: "",
    description: "",
    amount: null,
    frequency: "",
    document_ids: [],
});

/** Creates an empty reference (a Reference record) of the given type. */
const createEmptyReference = (referenceType) => ({
    client_key: generateRowKey("reference"),
    id: null,
    reference_type: referenceType,
    name: "",
    relationship: "",
    phone: "",
    email: "",
    address: "",
});

/**
 * Creates a fresh, empty application form state.
 */
const createEmptyApplication = () => ({
    id: null,
    // Assigned by the server's submit workflow, never by the form.
    application_number: "",
    status: "draft",
    loan_category: "",
    loan_type_id: "",
    loan_name: "",
    requested_loan_amount: null,
    requested_loan_term: null,
    repayment_frequency: "",
    loan_purpose: "",
    // Auto-loan-specific fields
    vehicle_make: "",
    vehicle_model: "",
    vehicle_year: "",
    vehicle_condition: "",
    // Identifies the vehicle being bought; saved on its Asset.
    vehicle_registration_number: "",
    vehicle_chassis_number: "",
    // Home-loan-specific fields
    property_address: "",
    property_type: "",
    property_value: null,
    // Identifies the property being bought; saved on its Asset.
    property_block_and_parcel: "",
    property_deed_number: "",
    // Purchase loans (auto and home). With a purchase price, the form adds
    // the vehicle or property being bought as a collateral asset.
    purchase_price: null,
    down_payment_amount: null,
    source_of_funds: "",
    source_of_funds_details: "",
    seller_type: "",
    seller_name: "",
    // Credit cards and overdrafts: the limit asked for (instead of an
    // amount and term), and how the card or overdraft is set up.
    requested_credit_limit: null,
    is_secured_by_savings: false,
    secured_savings_amount: null,
    linked_account_number: "",
    name_on_card: "",
    card_collection_method: "",
    // Student loans: the school, the course, and the costs.
    institution_name: "",
    institution_country: "",
    program_name: "",
    program_level: "",
    student_id_number: "",
    enrollment_start_date: "",
    expected_graduation_date: "",
    tuition_amount: null,
    tuition_currency: "",
    other_study_costs: null,
    disbursement_schedule: "",
    pay_to_institution: false,
    grace_period_months: null,
    // Business-loan-specific fields. Saved on the business's own Party
    // (kind ORGANIZATION), joined to the application through an
    // ApplicationParty with the role "Business Borrower".
    business_party_id: null,
    business_application_party_id: null,
    // Saved on the Party as legal_name.
    business_name: "",
    // Saved on the Primary Applicant's ApplicationParty (signing_authority).
    business_signing_authority: "",
    // Optional. An empty trading name means the same as the legal name.
    business_trading_name: "",
    business_industry: "",
    business_license_number: "",
    business_website: "",
    business_registration_number: "",
    business_type: "",
    business_incorporation_date: "",
    business_employee_count: null,
    // Saved on the Business Borrower's ApplicationParty (annual_revenue).
    business_annual_revenue: null,
    // Nested collections
    primary: createEmptyApplicant("Primary Applicant"),
    parties: [],
    // One personal reference and one next of kin, for the primary applicant
    references: [
        createEmptyReference("Personal reference"),
        createEmptyReference("Next of kin"),
    ],
    assets: [],
    liabilities: [],
    expenses: [],
    document_ids: [],
});

export default {
    data() {
        const formData = createEmptyApplication();

        return {
            // Branding pulled from the system configuration
            companyName: "",
            // Start of the localStorage keys that remember the draft, from
            // the company name (see draftKey).
            draftKeyPrefix: "loan_",
            companyLogo: "",
            primaryColor: "",
            secondaryColor: "",
            logoBackground: "",

            // Form state and UI state
            formData,
            activePartyTab: formData.primary.client_key,
            step: 0,
            // Step saved in localStorage, applied after document uploads are
            // hydrated (the documents step may not exist in the path until then)
            pendingDraftStep: null,
            // Position within the current step's short screens (see screensFor)
            screenIndex: 0,
            // Shows "this is needed" under empty required boxes after a
            // failed Next; cleared when the screen changes.
            showErrors: false,
            // Answers to the Yes/No questions that skip sections: null until
            // answered. Not saved to Saturn; worked out again on restore.
            answers: {
                others: null,
                assets: null,
                debts: null,
                bills: null,
            },
            saving: false,
            savingDraft: false,
            receipt: null,
            alert: { text: "", type: "warning" },

            // Test mode (see TEST_MODE_AVAILABLE)
            testModeAvailable: TEST_MODE_AVAILABLE,
            testMode: false,
            // In test mode, whether the draft is remembered in localStorage.
            // Off by default, so reloading the page starts from the beginning.
            testKeepDraft: false,

            // Server data. Every loan type, including ones no longer offered,
            // so a restored draft can tell its loan type was retired.
            products: [],
            // True once the loan categories and types have been loaded.
            optionsLoaded: false,
            // The name of a restored draft's loan type that isn't offered
            // any more, until the applicant picks another one.
            retiredLoanName: "",

            // Type resources (replace the old liability_type / expense_type
            // string lookups). Records are normalized in loadOptions().
            liabilityTypes: [],
            expenseTypes: [],
            // Fallback % of credit limit for revolving repayments, used only
            // when a revolving LiabilityType has no revolving_rate.
            defaultRevolvingRate: DEFAULT_REVOLVING_RATE,

            // Attachment types indexed by normalized AttachmentGroup label,
            // e.g. "asset-vehicle" or "applicant-all". See loadAttachmentCatalog().
            attachmentTypesByGroup: {},
            documentsLoading: false,

            // Upload state keyed by "scopeKey::attachmentTypeId", where the
            // scope key is "application" or "<kind>:<client_key>".
            documentState: {},
            uploadingDocumentKey: "",

            // Server IDs saved for this draft, per resource, as of the last
            // save or restore. Anything in here that's no longer in the form
            // was removed by the applicant and is deleted on the next save.
            persistedIds: {},

            // Saturn's own field definitions, so the steps can render them
            // with Saturn's FormField. applicationProps is the Application
            // list; resourceProps holds every resource's list by name.
            applicationProps: [],
            resourceProps: {},

            // Dropdown option lists keyed by field name
            lookups: {
                role: [],
                preferred_contact_method: [],
                identification_type: [],
                marital_status: [],
                parish: [],
                country: [],
                employment_status: [],
                repayment_frequency: [],
                vehicle_condition: [],
                property_type: [],
                business_type: [],
                asset_type: [],
                payment_frequency: [],
                expense_frequency: [],
                insurance_type: [],
                insurance_status: [],
                insurance_premium_frequency: [],
                relationship_to_applicant: [],
                residency_status: [],
                housing_status: [],
                employment_type: [],
                pay_frequency: [],
                guarantee_type: [],
                income_type: [],
                income_frequency: [],
                reference_type: [],
                reference_relationship: [],
                source_of_funds: [],
                seller_type: [],
                card_collection_method: [],
                program_level: [],
                tuition_currency: [],
                disbursement_schedule: [],
            },

            // LoanCategory records (see the loans computed property)
            loanCategoryRecords: [],
        };
    },

    computed: {
        /** CSS custom properties derived from the tenant's branding. */
        brandStyle() {
            const brand = this.primaryColor || "#1178bd";
            const accent = this.secondaryColor || brand;
            return {
                "--brand": brand,
                "--brand-ink": this.readableTextOn(brand),
                // Brand-coloured text on white: darkened when the brand is light.
                "--brand-text":
                    this.readableTextOn(brand) === "#ffffff" ? brand : this.tint(brand, -0.55),
                "--brand-soft": this.tint(brand, 0.82),
                "--brand-tint": this.tint(brand, 0.92),
                "--accent": accent,
                "--accent-ink": this.readableTextOn(accent),
                "--logo-bg": this.logoBackground || "transparent",
            };
        },

        /** Colours for the message box, by its type (Tailwind classes). */
        noticeClass() {
            const type = this.alert.type;
            if (type === "success") return "bg-green-50 border-green-200 text-green-800";
            if (type === "info") return "bg-blue-50 border-blue-200 text-blue-800";
            return "bg-red-50 border-red-200 text-red-700";
        },

        /** Primary applicant plus all additional parties. */
        allApplicants() {
            return [this.formData.primary].concat(this.formData.parties);
        },

        /** Party IDs of the Third Party Owners (people who aren't borrowing). */
        thirdPartyOwnerPartyIds() {
            return this.allApplicants
                .filter((person) => this.isThirdPartyOwner(person))
                .map((person) => this.toId(person.party_id))
                .filter(Boolean);
        },

        /**
         * The loan categories on step 1, from the active LoanCategory records
         * in sort_order. A category with no loan type on offer is left out.
         * Each is { id: its code, recordId, kind, title, note, icon,
         * isRevolving }.
         */
        loans() {
            const categories = this.loanCategoryRecords.map((record) => {
                const kind = categoryKind(record.code);
                return {
                    id: String(record.code || ""),
                    recordId: this.toId(record) || null,
                    kind,
                    title: record.name || record.code,
                    note: record.description || "",
                    icon: record.icon || KIND_ICONS[kind] || "mdi-cash",
                    isRevolving:
                        this.toBool(record.is_revolving, false) ||
                        REVOLVING_KINDS.includes(kind),
                };
            });
            return categories.filter(
                (category) => this.productsIn(category).length > 0,
            );
        },

        /**
         * How many forms of identification each borrower and guarantor must
         * give, from the chosen loan type (empty means the default of 1).
         */
        minimumIdentifications() {
            const value = Number(this.selectedProduct?.minimum_identifications);
            return value > 0 ? Math.floor(value) : DEFAULT_MINIMUM_IDENTIFICATIONS;
        },

        /** The chosen loan category (from loans), or null. */
        selectedCategory() {
            const kind = categoryKind(this.formData.loan_category);
            if (!kind) return null;
            return this.loans.find((loan) => loan.kind === kind) || null;
        },

        /**
         * The kind of loan chosen: property, automotive, personal,
         * organization, credit_card, overdraft, or student. The screens and
         * rules follow this, not the category's name.
         */
        loanKind() {
            return categoryKind(this.formData.loan_category);
        },

        /** Credit cards and overdrafts: a limit to draw on, with no term. */
        isRevolving() {
            return Boolean(
                (this.selectedCategory && this.selectedCategory.isRevolving) ||
                    REVOLVING_KINDS.includes(this.loanKind),
            );
        },

        /**
         * Loan types on offer in the selected category, matched through
         * LoanType.loan_category (a link to LoanCategory).
         */
        productsForLoan() {
            return this.productsIn(this.selectedCategory);
        },

        /** Products that need a guarantor (LoanType.requires_guarantor). */
        requiresGuarantor() {
            return this.toBool(this.selectedProduct?.requires_guarantor, false);
        },

        /** The full product record matching the selected loan type ID. */
        selectedProduct() {
            const selectedId = this.toId(this.formData.loan_type_id);
            return (
                this.products.find(
                    (product) => this.toId(product) === selectedId,
                ) || null
            );
        },

        /** Human-readable name of the selected loan category, e.g. "personal loan". */
        loanCategoryLabel() {
            const loan = this.selectedCategory;
            return loan ? String(loan.title).toLowerCase() : "this loan";
        },

        /**
         * Expense type options the applicant may pick: active, user-selectable,
         * and applicable to the selected loan category.
         */
        expenseTypeOptions() {
            return this.expenseTypes
                .filter((type) => type.user_selectable)
                .filter((type) => this.expenseTypeAppliesToLoan(type))
                .map((type) => ({
                    value: this.toId(type),
                    label: type.name,
                    group: type.group || "",
                }));
        },

        /** Amount/term bounds, falling back to permissive defaults. */
        amountMinimum() {
            const minimum = Number(this.selectedProduct?.minimum_amount);
            return this.selectedProduct && minimum >= 0 ? minimum : 0;
        },
        amountMaximum() {
            const maximum = Number(this.selectedProduct?.maximum_amount);
            return this.selectedProduct && maximum > 0 ? maximum : 999999999;
        },
        termMinimum() {
            const minimum = Number(this.selectedProduct?.minimum_term_months);
            return this.selectedProduct && minimum > 0 ? minimum : 1;
        },
        termMaximum() {
            const maximum = Number(this.selectedProduct?.maximum_term_months);
            return this.selectedProduct && maximum > 0 ? maximum : 600;
        },

        /** Auto and property loans (or explicitly secured products) need collateral. */
        requiresCollateral() {
            return (
                ["automotive", "property"].includes(this.loanKind) ||
                Boolean(this.selectedProduct?.secured === true)
            );
        },

        securityLabel() {
            if (this.requiresCollateral) return "Secured";
            return this.selectedProduct ? "Unsecured" : "To be confirmed";
        },

        /** Select options for asset ownership, keyed by Party ID. */
        partyOptions() {
            return this.allApplicants
                .filter((person) => person.party_id)
                .map((person) => ({
                    value: this.toId(person.party_id),
                    label: this.applicantLabel(person, person.role),
                }));
        },

        /**
         * Select options for liabilities/expenses, keyed by ApplicationParty
         * ID. Third Party Owners aren't borrowing, so they're left out.
         */
        applicationPartyOptions() {
            return this.allApplicants
                .filter(
                    (person) =>
                        person.application_party_id &&
                        !this.isThirdPartyOwner(person),
                )
                .map((person) => ({
                    value: this.toId(person.application_party_id),
                    label: this.applicantLabel(person, person.role),
                }));
        },

        /** Subset of form fields edited on the "Your request" step. */
        requestData() {
            const form = this.formData;
            return {
                requested_credit_limit: form.requested_credit_limit,
                is_secured_by_savings: form.is_secured_by_savings,
                secured_savings_amount: form.secured_savings_amount,
                linked_account_number: form.linked_account_number,
                name_on_card: form.name_on_card,
                card_collection_method: form.card_collection_method,
                institution_name: form.institution_name,
                institution_country: form.institution_country,
                program_name: form.program_name,
                program_level: form.program_level,
                student_id_number: form.student_id_number,
                enrollment_start_date: form.enrollment_start_date,
                expected_graduation_date: form.expected_graduation_date,
                tuition_amount: form.tuition_amount,
                tuition_currency: form.tuition_currency,
                other_study_costs: form.other_study_costs,
                disbursement_schedule: form.disbursement_schedule,
                pay_to_institution: form.pay_to_institution,
                grace_period_months: form.grace_period_months,
                requested_loan_amount: form.requested_loan_amount,
                requested_loan_term: form.requested_loan_term,
                repayment_frequency: form.repayment_frequency,
                loan_purpose: form.loan_purpose,
                vehicle_make: form.vehicle_make,
                vehicle_model: form.vehicle_model,
                vehicle_year: form.vehicle_year,
                vehicle_condition: form.vehicle_condition,
                vehicle_registration_number: form.vehicle_registration_number,
                vehicle_chassis_number: form.vehicle_chassis_number,
                property_address: form.property_address,
                property_type: form.property_type,
                property_value: form.property_value,
                property_block_and_parcel: form.property_block_and_parcel,
                property_deed_number: form.property_deed_number,
                purchase_price: form.purchase_price,
                down_payment_amount: form.down_payment_amount,
                source_of_funds: form.source_of_funds,
                source_of_funds_details: form.source_of_funds_details,
                seller_type: form.seller_type,
                seller_name: form.seller_name,
                business_name: form.business_name,
                business_signing_authority: form.business_signing_authority,
                business_trading_name: form.business_trading_name,
                business_industry: form.business_industry,
                business_license_number: form.business_license_number,
                business_website: form.business_website,
                business_registration_number: form.business_registration_number,
                business_type: form.business_type,
                business_incorporation_date: form.business_incorporation_date,
                business_employee_count: form.business_employee_count,
                business_annual_revenue: form.business_annual_revenue,
            };
        },

        /** Select options for the asset that secures a liability (by client key). */
        assetOptions() {
            return this.formData.assets.map((asset, index) => ({
                value: asset.client_key,
                label: asset.name || `Asset ${index + 1}`,
            }));
        },

        /**
         * Existing loans secured on each asset (liens), keyed by the asset's
         * client key: a list of "Creditor (EC$ balance)" labels.
         */
        liensByAsset() {
            const result = {};
            this.formData.liabilities.forEach((liability) => {
                if (!liability.is_secured || !liability.secured_asset_ref) return;
                const asset = this.assetForRef(liability.secured_asset_ref);
                if (!asset) return;
                if (!result[asset.client_key]) result[asset.client_key] = [];
                result[asset.client_key].push(
                    `${liability.creditor_name || "Unnamed creditor"} (${this.money(
                        liability.outstanding_balance,
                    )} owing)`,
                );
            });
            return result;
        },

        /** The ExpenseType for projected collateral insurance, if it's set up. */
        collateralInsuranceType() {
            return (
                this.expenseTypes.find(
                    (type) =>
                        String(type.code || "").trim().toUpperCase() ===
                        COLLATERAL_INSURANCE_CODE,
                ) || null
            );
        },

        /**
         * The projected monthly insurance cost of each collateral asset with
         * a premium, shown read-only on the Expenses step and saved as an
         * Expense marked is_projected.
         */
        projectedInsuranceExpenses() {
            return this.formData.assets
                .filter(
                    (asset) =>
                        this.isCollateral(asset) &&
                        Number(asset.collateral.insurance.premium) > 0,
                )
                .map((asset) => {
                    const insurance = asset.collateral.insurance;
                    return {
                        asset_key: asset.client_key,
                        name: `Insurance for ${asset.name || "collateral"} (projected)`,
                        amount: Number(insurance.premium),
                        frequency: insurance.premium_frequency || "Monthly",
                        monthly: monthlyAmount(
                            insurance.premium,
                            insurance.premium_frequency,
                        ),
                    };
                });
        },

        /**
         * Monthly totals for the review step: income (gross and after
         * estimated deductions), expenses, and liability payments.
         * Third Party Owners aren't borrowing, so their income isn't counted.
         */
        monthlyTotals() {
            const borrowers = this.allApplicants.filter(
                (person) => !this.isThirdPartyOwner(person),
            );
            let grossIncome = 0;
            let netIncome = 0;
            borrowers.forEach((person) => {
                const deductions = this.statutoryDeductions(person);
                grossIncome += Number(this.grossMonthly(person)) || 0;
                netIncome += deductions.net;
                (person.incomes || []).forEach((income) => {
                    const monthly = monthlyAmount(income.amount, income.frequency) || 0;
                    grossIncome += monthly;
                    netIncome += monthly;
                });
            });
            const expenses =
                this.formData.expenses.reduce(
                    (sum, item) =>
                        sum + (monthlyAmount(item.amount, item.frequency) || 0),
                    0,
                ) +
                this.projectedInsuranceExpenses.reduce(
                    (sum, item) => sum + (item.monthly || 0),
                    0,
                );
            const liabilityPayments = this.formData.liabilities
                .filter((item) => !item.is_to_be_paid_off)
                .reduce(
                    (sum, item) => sum + (this.liabilityMonthlyPayment(item) || 0),
                    0,
                );
            return {
                grossIncome: this.roundMoney(grossIncome),
                netIncome: this.roundMoney(netIncome),
                expenses: this.roundMoney(expenses),
                liabilityPayments: this.roundMoney(liabilityPayments),
                remaining: this.roundMoney(netIncome - expenses - liabilityPayments),
            };
        },

        /**
         * Everything entered, with dropdown values turned into labels, for the
         * review step. Each section is { title, step, items }, where each
         * item is { heading, rows: [[label, value]] }.
         */
        reviewSummary() {
            const form = this.formData;
            const money = this.money;
            const label = this.lookupLabel;
            const yesNo = (value) => (value ? "Yes" : "No");
            const partyName = (partyId) => {
                const option = this.partyOptions.find(
                    (entry) => entry.value === this.toId(partyId),
                );
                return option ? option.label : "Not assigned";
            };
            const linkName = (linkId) => {
                const option = this.applicationPartyOptions.find(
                    (entry) => entry.value === this.toId(linkId),
                );
                return option ? option.label : "Not assigned";
            };
            const rows = (pairs) =>
                pairs.filter(
                    (pair) =>
                        pair[1] !== "" &&
                        pair[1] !== null &&
                        pair[1] !== undefined &&
                        pair[1] !== "—",
                );

            const loan = {
                title: "The loan you need",
                step: "request",
                items: [
                    {
                        heading: form.loan_name || "Loan",
                        rows: rows([
                            ["Limit", this.isRevolving && form.requested_credit_limit ? money(form.requested_credit_limit) : ""],
                            ["Amount", this.isRevolving ? "" : money(form.requested_loan_amount)],
                            [
                                "Term",
                                !this.isRevolving && form.requested_loan_term
                                    ? `${form.requested_loan_term} months`
                                    : "",
                            ],
                            ["Repayment frequency", this.isRevolving ? "" : form.repayment_frequency],
                            ["Purpose", form.loan_purpose],
                            ["Name on the card", this.loanKind === "credit_card" ? form.name_on_card : ""],
                            ["How to get the card", this.loanKind === "credit_card" ? label("card_collection_method", form.card_collection_method) : ""],
                            ["Account", this.isRevolving ? form.linked_account_number : ""],
                            [
                                "Secured by savings",
                                this.isRevolving && form.is_secured_by_savings
                                    ? money(form.secured_savings_amount)
                                    : "",
                            ],
                            ["School", this.loanKind === "student" ? [form.institution_name, label("country", form.institution_country)].filter(Boolean).join(", ") : ""],
                            ["Course", this.loanKind === "student" ? [form.program_name, label("program_level", form.program_level)].filter(Boolean).join(", ") : ""],
                            ["Student ID", this.loanKind === "student" ? form.student_id_number : ""],
                            ["Course dates", this.loanKind === "student" ? [form.enrollment_start_date, form.expected_graduation_date].filter(Boolean).join(" to ") : ""],
                            ["Tuition fees", this.loanKind === "student" && form.tuition_amount ? `${Number(form.tuition_amount).toLocaleString(undefined, { minimumFractionDigits: 2, maximumFractionDigits: 2 })} ${form.tuition_currency || ""}`.trim() : ""],
                            ["Other study costs", this.loanKind === "student" && form.other_study_costs ? money(form.other_study_costs) : ""],
                            ["Payout schedule", this.loanKind === "student" ? label("disbursement_schedule", form.disbursement_schedule) : ""],
                            ["Paid to the school", this.loanKind === "student" && form.pay_to_institution ? "Yes" : ""],
                            ["Grace period", this.loanKind === "student" && form.grace_period_months ? `${form.grace_period_months} months` : ""],
                            ["Purchase price", form.purchase_price ? money(form.purchase_price) : ""],
                            ["Down payment", form.down_payment_amount ? money(form.down_payment_amount) : ""],
                            ["Source of down payment", form.source_of_funds],
                            ["Seller", [form.seller_type, form.seller_name].filter(Boolean).join(": ")],
                            ["Vehicle", [form.vehicle_year, form.vehicle_make, form.vehicle_model].filter(Boolean).join(" ")],
                            ["Vehicle condition", form.vehicle_condition],
                            ["Property address", form.property_address],
                            ["Property type", form.property_type],
                            ["Property value", form.property_value ? money(form.property_value) : ""],
                            ["Business", form.business_name],
                            ["Your position", this.loanKind === "organization" ? form.business_signing_authority : ""],
                            ["Trading name", form.business_trading_name],
                            ["What the business does", form.business_industry],
                            ["Licence number", form.business_license_number],
                            ["Website", form.business_website],
                            ["Registration number", form.business_registration_number],
                            ["Business type", form.business_type],
                            ["Employees", form.business_employee_count],
                            ["Yearly sales", form.business_annual_revenue ? money(form.business_annual_revenue) : ""],
                        ]),
                    },
                ],
            };

            const applicants = {
                title: "People on the loan",
                step: "parties",
                items: this.allApplicants.map((person, index) => {
                    const heading = this.applicantLabel(
                        person,
                        index === 0 ? "Primary Applicant" : person.role,
                    );
                    if (this.isThirdPartyOwner(person)) {
                        return {
                            heading,
                            rows: rows([
                                ["Relationship", label("relationship_to_applicant", person.relationship_to_applicant)],
                                ["Phone", person.phone],
                                ["Email", person.email],
                                ["Contact by", label("preferred_contact_method", person.preferred_contact_method)],
                            ]),
                        };
                    }
                    const deductions = this.statutoryDeductions(person);
                    return {
                        heading,
                        rows: rows([
                            ["Member", person.is_member ? `Yes (${person.member_number})` : "Not yet a member"],
                            ["Email", person.email],
                            ["Phone", person.phone],
                            ["Contact by", label("preferred_contact_method", person.preferred_contact_method)],
                            ["Date of birth", person.date_of_birth],
                            ["Address", [person.address, person.parish, person.country].filter(Boolean).join(", ")],
                            ["Housing", label("housing_status", person.housing_status)],
                            ["Living there since", person.address_since],
                            ["Dependants", person.number_of_dependants],
                            ["Citizenship", label("country", person.citizenship)],
                            ["Residency", label("residency_status", person.residency_status)],
                            ["NIS number", person.nis_number],
                            [
                                "Identification",
                                (person.identifications || [])
                                    .map((row) => `${label("identification_type", row.identification_type)} ${row.identification_number}`.trim())
                                    .join("; "),
                            ],
                            ["Employment", [label("employment_status", person.employment_status), label("employment_type", person.employment_type)].filter(Boolean).join(", ")],
                            ["Employer", [person.employer_name, person.job_title].filter(Boolean).join(", ")],
                            ["Started", person.employment_start_date],
                            [
                                "Gross pay",
                                person.gross_pay !== null && person.gross_pay !== undefined
                                    ? `${money(person.gross_pay)} ${label("pay_frequency", person.pay_frequency).toLowerCase()}`.trim()
                                    : "",
                            ],
                            ["Gross monthly income", money(this.grossMonthly(person))],
                            ["Estimated NIS and income tax", `${money(deductions.nis)} and ${money(deductions.incomeTax)} a month`],
                            [
                                "Other income",
                                (person.incomes || [])
                                    .map((income) => `${label("income_type", income.income_type)}: ${money(income.amount)} ${String(income.frequency || "").toLowerCase()}`)
                                    .join("; "),
                            ],
                            ["Guarantee", person.role === GUARANTOR_ROLE ? `${label("guarantee_type", person.guarantee_type)} ${money(person.guarantee_amount)}` : ""],
                            ["Politically exposed person", yesNo(person.is_pep)],
                            [
                                "Declarations",
                                [
                                    person.declared_bankruptcy ? "bankruptcy" : "",
                                    person.declared_judgments ? "court judgments" : "",
                                    person.declared_arrears ? "arrears" : "",
                                    person.declared_other_applications ? "other applications" : "",
                                ].filter(Boolean).join(", ") || "None",
                            ],
                        ]),
                    };
                }),
            };

            const references = {
                title: "People who know you",
                step: "parties",
                items: (form.references || []).map((row) => ({
                    heading: `${row.reference_type}: ${row.name || "Not entered"}`,
                    rows: rows([
                        ["Relationship", label("reference_relationship", row.relationship)],
                        ["Phone", row.phone],
                        ["Email", row.email],
                    ]),
                })),
            };

            const assets = {
                title: "Things you own",
                step: "assets",
                items: form.assets.map((asset, index) => {
                    const insurance = asset.collateral.insurance;
                    const liens = this.liensByAsset[asset.client_key] || [];
                    return {
                        heading: `${asset.name || `Asset ${index + 1}`}${this.isCollateral(asset) ? " (collateral)" : ""}`,
                        rows: rows([
                            ["Type", label("asset_type", asset.asset_type)],
                            ["Value", money(asset.declared_value)],
                            ["Owners", (asset.owners || []).map((row) => `${partyName(row.party_id)} ${row.percentage}%`).join("; ")],
                            ["Registration / chassis", [asset.registration_number, asset.chassis_number].filter(Boolean).join(" / ")],
                            ["Block and parcel / deed", [asset.block_and_parcel, asset.deed_number].filter(Boolean).join(" / ")],
                            ["Existing loans on it", liens.join("; ")],
                            ["Insurance", this.isCollateral(asset) ? [insurance.status, insurance.provider, insurance.reference].filter(Boolean).join(", ") : ""],
                            ["Premium", this.isCollateral(asset) && insurance.premium ? `${money(insurance.premium)} ${String(insurance.premium_frequency || "").toLowerCase()}` : ""],
                        ]),
                    };
                }),
            };

            const liabilities = {
                title: "Money you owe",
                step: "liabilities",
                items: form.liabilities.map((item, index) => ({
                    heading: item.creditor_name || `Liability ${index + 1}`,
                    rows: rows([
                        ["Type", this.liabilityTypeFor(item.liability_type)?.name || ""],
                        ["Balance", money(item.outstanding_balance)],
                        ["Credit limit", this.isRevolvingType(item.liability_type) ? money(item.credit_limit) : ""],
                        ["Monthly payment", money(this.liabilityMonthlyPayment(item))],
                        ["Paid off by this loan", yesNo(item.is_to_be_paid_off)],
                        ["Secured on", item.is_secured ? this.assetForRef(item.secured_asset_ref)?.name || "Yes" : ""],
                        ["Responsible", (item.responsibilities || []).map((row) => `${linkName(row.application_party_id)} ${row.percentage}%`).join("; ")],
                        ["Notes", item.description],
                    ]),
                })),
            };

            const expenses = {
                title: "Your bills",
                step: "expenses",
                items: form.expenses
                    .map((item, index) => ({
                        heading: item.expense_name || this.expenseTypeName(item.expense_type) || `Expense ${index + 1}`,
                        rows: rows([
                            ["Amount", `${money(item.amount)} ${String(item.frequency || "").toLowerCase()}`.trim()],
                            ["Monthly", money(monthlyAmount(item.amount, item.frequency))],
                            ["Paid by", item.is_household ? "Shared household" : linkName(item.application_party_id)],
                        ]),
                    }))
                    .concat(
                        this.projectedInsuranceExpenses.map((item) => ({
                            heading: item.name,
                            rows: [["Monthly", money(item.monthly)]],
                        })),
                    ),
            };

            const totals = this.monthlyTotals;
            const monthly = {
                title: "Your month at a glance",
                step: "",
                items: [
                    {
                        heading: "An estimate, each month",
                        rows: [
                            ["Money coming in, before tax", money(totals.grossIncome)],
                            ["Take-home pay (after NIS and tax)", money(totals.netIncome)],
                            ["Bills", money(totals.expenses)],
                            ["Loan and card payments", money(totals.liabilityPayments)],
                            ["Left over each month", money(totals.remaining)],
                        ],
                    },
                ],
            };

            return [loan, applicants, references, assets, liabilities, expenses, monthly].filter(
                (section) => section.items.length,
            );
        },

        /** Estimated statutory deductions per applicant, keyed by client_key. */
        deductionsByApplicant() {
            return Object.fromEntries(
                this.allApplicants.map((person) => [
                    person.client_key,
                    this.statutoryDeductions(person),
                ]),
            );
        },

        /** Document uploads are paused while the draft is saving. */
        documentsDisabled() {
            return this.saving || this.savingDraft || this.documentsLoading;
        },

        /**
         * Every document scope in the application, in wizard order: the
         * application itself, each applicant, then each asset (and its
         * collateral, when it's offered as collateral), liability, and
         * expense. Scopes with no required documents are left out.
         *
         * Each scope carries its attachment types, requirement view models
         * with upload state, and a lockedMessage while its owner isn't
         * complete enough to be saved (uploads need a saved owner record).
         */
        documentScopeList() {
            const form = this.formData;
            const scopes = [];
            const add = (key, kind, label, types, lockedMessage = "", optionalIds = []) => {
                if (!types.length) return;
                scopes.push({
                    key,
                    kind,
                    label,
                    types,
                    lockedMessage,
                    requirements: this.requirementsFor(key, types, optionalIds),
                });
            };

            add(
                "application",
                "application",
                "the application",
                this.requirementTypesFor("application", form.loan_name),
            );

            const applicantTypes = this.requirementTypesFor(
                "applicant",
                form.loan_name,
            );
            // Uploads unlock once the name, contact details, NIS number, and
            // IDs are in: the screens before the documents are asked for.
            const primaryReady = this.readyForUploads(form.primary);
            this.allApplicants.forEach((person, index) => {
                // Third Party Owners give no documents or IDs for now.
                if (this.isThirdPartyOwner(person)) return;

                let locked = "";
                if (!this.readyForUploads(person)) {
                    locked =
                        "Fill in the name, contact details, NIS number, and ID first, then you can upload.";
                } else if (index > 0 && !primaryReady) {
                    locked = "Fill in your own details first.";
                }
                // The NIS card is optional for students.
                const optionalIds = this.isStudent(person)
                    ? applicantTypes
                          .filter((type) => /\bnis\b/i.test(type.name || ""))
                          .map(this.toId)
                    : [];
                add(
                    `applicant:${person.client_key}`,
                    "applicant",
                    this.applicantName(person, index),
                    applicantTypes,
                    locked,
                    optionalIds,
                );

                // Scans for each of this applicant's identifications.
                (person.identifications || []).forEach((row, rowIndex) => {
                    const typeLabel = this.lookupLabel(
                        "identification_type",
                        row.identification_type,
                    );
                    add(
                        `identification:${row.client_key}`,
                        "identification",
                        `${this.applicantName(person, index)}'s ${
                            typeLabel || `identification ${rowIndex + 1}`
                        }`,
                        this.requirementTypesFor("identification", typeLabel),
                        locked ||
                            (this.identificationRowIssue(row)
                                ? "Complete this identification to upload a scan."
                                : ""),
                    );
                });

                // Proof of each other income ("income-<income type>").
                (person.incomes || []).forEach((income, incomeIndex) => {
                    const typeLabel = this.lookupLabel("income_type", income.income_type);
                    add(
                        `income:${income.client_key}`,
                        "income",
                        `${this.applicantName(person, index)}'s ${
                            typeLabel || `other income ${incomeIndex + 1}`
                        }`,
                        this.requirementTypesFor("income", typeLabel),
                        locked ||
                            (this.incomeRowIssue(income)
                                ? "Complete this income to upload proof."
                                : ""),
                    );
                });
            });

            form.assets.forEach((asset, index) => {
                const assetLabel = asset.name || `asset ${index + 1}`;
                const typeLabel = this.assetTypeLabel(asset.asset_type);
                const assetLocked = this.assetReady(asset)
                    ? ""
                    : "Enter this asset's name, type, and value to upload documents.";

                add(
                    `asset:${asset.client_key}`,
                    "asset",
                    assetLabel,
                    this.requirementTypesFor("asset", typeLabel),
                    assetLocked,
                );

                // Collateral documents, shown inside the asset's card.
                if (!this.isCollateral(asset)) return;

                // "collateral-<asset type>" and "collateral-all", plus
                // "collateral-third party" when a Third Party Owner owns part
                // of it (e.g. the owner's signed consent to pledge it).
                const types = this.requirementTypesFor("collateral", typeLabel);
                if (this.hasThirdPartyOwner(asset)) {
                    const seen = new Set(types.map(this.toId));
                    this.requirementTypesFor("collateral", "third party").forEach(
                        (type) => {
                            if (!seen.has(this.toId(type))) types.push(type);
                        },
                    );
                }

                add(
                    `collateral:${asset.client_key}`,
                    "collateral",
                    `collateral (${assetLabel})`,
                    types,
                    assetLocked,
                );
            });

            form.liabilities.forEach((liability, index) =>
                add(
                    `liability:${liability.client_key}`,
                    "liability",
                    liability.creditor_name || `liability ${index + 1}`,
                    this.requirementTypesFor(
                        "liability",
                        this.liabilityTypeFor(liability.liability_type)?.name,
                    ),
                    this.liabilityReady(liability)
                        ? ""
                        : "Enter this liability's creditor, type, balance, and payment to upload documents.",
                ),
            );

            form.expenses.forEach((expense, index) =>
                add(
                    `expense:${expense.client_key}`,
                    "expense",
                    expense.expense_name ||
                        this.expenseTypeName(expense.expense_type) ||
                        `expense ${index + 1}`,
                    this.requirementTypesFor(
                        "expense",
                        this.expenseTypeName(expense.expense_type),
                    ),
                    this.expenseReady(expense)
                        ? ""
                        : "Choose the responsible applicant, type, and amount to upload documents.",
                ),
            );

            return scopes;
        },

        /** The same scopes keyed by scope key, for lookups from the sections. */
        documentScopes() {
            return Object.fromEntries(
                this.documentScopeList.map((scope) => [scope.key, scope]),
            );
        },

        requiredDocumentCount() {
            return this.documentScopeList.reduce(
                (total, scope) => total + scope.requirements.length,
                0,
            );
        },

        uploadedDocumentCount() {
            return this.documentScopeList.reduce(
                (total, scope) =>
                    total +
                    scope.requirements.filter(
                        (item) => item.status === "uploaded",
                    ).length,
                0,
            );
        },

        /**
         * Ordered list of wizard steps. The documents step is inserted only
         * when the product has application-level documents. Collateral is
         * chosen on the Assets step.
         */
        path() {
            const steps = [
                { id: "loan", title: "Choose a loan" },
                { id: "parties", title: "About you" },
                { id: "request", title: "The loan you need" },
                { id: "assets", title: "Things you own" },
                { id: "liabilities", title: "Money you owe" },
                { id: "expenses", title: "Your bills" },
            ];

            // Only application-level documents have their own step.
            if (this.documentScopes.application) {
                steps.push({ id: "documents", title: "Documents" });
            }

            steps.push({ id: "review", title: "Check and send" });

            return steps;
        },

        /** The step descriptor for the current position in the wizard. */
        current() {
            return this.path[Math.min(this.step, this.path.length - 1)];
        },

        /** The short screens of the current step. */
        screens() {
            return this.screensFor(this.current.id);
        },

        /** The screen being shown. */
        currentScreen() {
            const screens = this.screens;
            return screens[Math.min(this.screenIndex, screens.length - 1)] || {};
        },

        /** The person the current screen is about, if it's a person screen. */
        currentPerson() {
            const key = this.currentScreen.personKey;
            if (!key) return null;
            return (
                this.allApplicants.find((person) => person.client_key === key) ||
                null
            );
        },

        isFirstScreen() {
            return this.step === 0 && this.screenIndex === 0;
        },

        isLastScreen() {
            return (
                this.step >= this.path.length - 1 &&
                this.screenIndex >= this.screens.length - 1
            );
        },

        /** How far through all the screens the applicant is. */
        progressWidth() {
            let done = 0;
            let total = 0;
            this.path.forEach((stepItem, index) => {
                const count = this.screensFor(stepItem.id).length;
                total += count;
                if (index < this.step) done += count;
            });
            done += Math.min(this.screenIndex, this.screens.length - 1) + 1;
            return `${Math.round((done / Math.max(total, 1)) * 100)}%`;
        },
    },

    watch: {
        // In test mode, fill each step with sample data as it's reached.
        "current.id"() {
            if (this.testMode) this.fillTestData();
        },
    },

    async mounted() {
        await Promise.all([
            this.loadBranding(),
            this.loadOptions(),
            this.loadAttachmentCatalog(),
        ]);
        await this.restoreDraft();
        // Mark documents already linked to a restored draft as uploaded
        // before applying the saved step.
        await this.hydrateAllDocumentUploads();
        if (Number.isFinite(this.pendingDraftStep)) {
            this.setStep(Math.min(this.pendingDraftStep, this.path.length - 1));
        }
        if (this.retiredLoanName) {
            this.alert = {
                text: `The loan you picked, ${this.retiredLoanName}, isn't offered any more. Please choose another.`,
                type: "warning",
            };
        }
    },

    methods: {
        /**
         * Normalizes an ID reference. Handles raw IDs, record objects,
         * and { data: record } API wrappers.
         */
        toId(value) {
            if (!value) return null;
            if (typeof value !== "object") return value;
            return value.id || value.data?.id || null;
        },

        /** Unwraps a list API response into a plain array. */
        toList(value) {
            const rows = value?.data || value;
            return Array.isArray(rows) ? rows : [];
        },

        /** Unwraps a single-record API response. */
        recordOf(value) {
            return value?.data || value;
        },

        /** Formats a number as an EC dollar amount for display. */
        money(value) {
            if (value === null || value === undefined || value === "")
                return "—";
            return `EC$ ${Number(value).toLocaleString(undefined, {
                minimumFractionDigits: 2,
                maximumFractionDigits: 2,
            })}`;
        },

        /** Shows a dismissible alert message at the top of the form. */
        warn(text, type = "warning") {
            this.alert = { text, type };
        },

        /** A person's full name, or the business name for a business owner. */
        fullName(person) {
            if (person.kind === "ORGANIZATION") {
                return String(person.business_name || "").trim();
            }
            return `${person.first_name || ""} ${person.last_name || ""}`.trim();
        },

        /** "Jane Doe · Co-Applicant", falling back to just the role. */
        applicantLabel(person, fallback) {
            const name = this.fullName(person);
            return name
                ? `${name} · ${person.role || fallback}`
                : person.role || fallback;
        },

        /** Display name for a person, with positional fallbacks. */
        applicantName(person, index) {
            const name = this.fullName(person);
            return (
                name ||
                (index === 0 ? "Primary Applicant" : `Co-applicant ${index}`)
            );
        },

        /** True for someone who owns collateral but isn't borrowing. */
        isThirdPartyOwner(person) {
            return (
                String(person?.role || "")
                    .trim()
                    .toLowerCase() === THIRD_PARTY_OWNER_ROLE.toLowerCase()
            );
        },

        /** True when an asset is offered as collateral on a loan that needs it. */
        isCollateral(asset) {
            return Boolean(this.requiresCollateral && asset?.collateral?.enabled);
        },

        /** True when a Third Party Owner owns any share of the asset. */
        hasThirdPartyOwner(asset) {
            const ownerIds = this.thirdPartyOwnerPartyIds;
            return (asset.owners || []).some((row) =>
                ownerIds.includes(this.toId(row.party_id)),
            );
        },

        /** True when every owner of the asset is a Third Party Owner. */
        ownedOnlyByThirdParty(asset) {
            const ownerIds = this.thirdPartyOwnerPartyIds;
            const rows = (asset.owners || []).filter((row) => row.party_id);
            return (
                rows.length > 0 &&
                rows.every((row) => ownerIds.includes(this.toId(row.party_id)))
            );
        },

        /** True for a guarantor (full applicant form plus the guarantee). */
        isGuarantor(person) {
            return (
                String(person?.role || "")
                    .trim()
                    .toLowerCase() === GUARANTOR_ROLE.toLowerCase()
            );
        },

        /** The asset a reference points at: its client key or server ID. */
        assetForRef(ref) {
            const value = this.toId(ref);
            if (!value) return null;
            return (
                this.formData.assets.find(
                    (asset) =>
                        asset.client_key === value || this.toId(asset.id) === value,
                ) || null
            );
        },

        /**
         * A liability's monthly payment: the assessed payment for revolving
         * credit, otherwise its payment turned into a monthly figure.
         */
        liabilityMonthlyPayment(liability) {
            if (this.isRevolvingType(liability.liability_type)) {
                return this.assessedPayment(liability);
            }
            return monthlyAmount(liability.payment_amount, liability.payment_frequency);
        },

        /** Whole years from a YYYY-MM-DD date until today, or null. */
        yearsSince(date) {
            if (!date) return null;
            const start = new Date(`${String(date).slice(0, 10)}T00:00:00`);
            if (Number.isNaN(start.getTime())) return null;
            const now = new Date();
            let years = now.getFullYear() - start.getFullYear();
            const anniversaryPassed =
                now.getMonth() > start.getMonth() ||
                (now.getMonth() === start.getMonth() &&
                    now.getDate() >= start.getDate());
            if (!anniversaryPassed) years--;
            return Math.max(0, years);
        },

        /** Under 2 years in the current job, so the previous job is asked for. */
        needsPreviousEmployment(person) {
            const years = this.yearsSince(person.employment_start_date);
            return (
                this.showsEmploymentDetails(person.employment_status) &&
                years !== null &&
                years < 2
            );
        },

        /** Under 2 years at the current address, so the previous one is asked for. */
        needsPreviousAddress(person) {
            const years = this.yearsSince(person.address_since);
            return years !== null && years < 2;
        },

        /**
         * "" unless the chosen product needs a guarantor
         * (LoanType.requires_guarantor) and nobody on the loan is one.
         */
        guarantorIssue() {
            if (!this.requiresGuarantor) return "";
            if (this.formData.parties.some((person) => this.isGuarantor(person))) {
                return "";
            }
            return `${this.formData.loan_name || "This loan"} needs a guarantor. Please add someone with the role "Guarantor".`;
        },

        /** What's missing from one other-income row, or "". */
        incomeRowIssue(income) {
            if (
                !income.income_type ||
                income.amount === null ||
                income.amount === undefined ||
                !income.frequency
            ) {
                return "Complete the type, amount, and frequency for each other income.";
            }
            return "";
        },

        /** Jumps to a step by its ID (used by "Edit" on the review step). */
        goToStep(id) {
            const index = this.path.findIndex((item) => item.id === id);
            if (index >= 0) this.setStep(index);
        },

        /** Moves to a step, at the given screen (the first by default). */
        setStep(index, screenIndex = 0) {
            this.step = index;
            this.screenIndex = screenIndex;
            this.showErrors = false;
            this.scrollToTop();
        },

        /**
         * How much of one part's progress bar is filled: full for finished
         * parts, partly for the current part (by screen), empty after it.
         */
        segmentFill(index) {
            if (index < this.step) return "100%";
            if (index > this.step) return "0%";
            const total = Math.max(this.screens.length, 1);
            const done = Math.min(this.screenIndex, total - 1) + 1;
            return `${Math.round((done / total) * 100)}%`;
        },

        /** A colour mixed with white (amount 0 to 1), for soft backgrounds. */
        tint(color, amount) {
            const hex = String(color || "").trim().replace("#", "");
            const full =
                hex.length === 3
                    ? hex
                          .split("")
                          .map((c) => c + c)
                          .join("")
                    : hex;
            if (!/^[0-9a-f]{6}$/i.test(full)) return "#eef4fb";
            // A negative amount mixes with black instead (a darker shade).
            const mix = (start) => {
                const value = parseInt(full.slice(start, start + 2), 16);
                const target = amount < 0 ? 0 : 255;
                return Math.round(value + (target - value) * Math.abs(amount))
                    .toString(16)
                    .padStart(2, "0");
            };
            return `#${mix(0)}${mix(2)}${mix(4)}`;
        },

        /**
         * Black or white, whichever is easier to read on a colour, so text on
         * the system's brand colours always has enough contrast.
         */
        readableTextOn(color) {
            const hex = String(color || "").trim().replace("#", "");
            const full =
                hex.length === 3
                    ? hex
                          .split("")
                          .map((c) => c + c)
                          .join("")
                    : hex;
            if (!/^[0-9a-f]{6}$/i.test(full)) return "#ffffff";
            const channel = (start) => {
                const value = parseInt(full.slice(start, start + 2), 16) / 255;
                return value <= 0.03928
                    ? value / 12.92
                    : Math.pow((value + 0.055) / 1.055, 2.4);
            };
            const luminance =
                0.2126 * channel(0) + 0.7152 * channel(2) + 0.0722 * channel(4);
            return luminance > 0.4 ? "#111827" : "#ffffff";
        },

        scrollToTop() {
            if (typeof window !== "undefined" && window.scrollTo) {
                window.scrollTo(0, 0);
            }
        },

        /** True when an asset type's label looks like a vehicle. */
        isVehicleType(value) {
            return /vehicle|car|auto|truck|motor/i.test(this.assetTypeLabel(value));
        },

        /** True when an asset type's label looks like land or property. */
        isPropertyType(value) {
            return /property|land|house|home|real estate|building|lot/i.test(
                this.assetTypeLabel(value),
            );
        },

        /** The asset for the vehicle or property this loan is buying, or null. */
        purchaseAsset() {
            return this.formData.assets.find((asset) => asset.is_purchase) || null;
        },

        /**
         * For auto and home loans with a purchase price, keeps an asset for
         * the vehicle or property being bought, owned 100% by the primary
         * applicant and marked as collateral. Its name, type, value, and
         * identifiers follow the "Your request" step. Without a purchase
         * price (e.g. a refinance), any such asset is removed.
         */
        syncPurchaseAsset() {
            const form = this.formData;
            const isAuto = this.loanKind === "automotive";
            const isHome = this.loanKind === "property";
            const price = Number(form.purchase_price);
            let asset = this.purchaseAsset();

            if (!(isAuto || isHome) || !(price > 0)) {
                if (asset) {
                    form.assets = form.assets.filter((entry) => entry !== asset);
                }
                return;
            }

            if (!asset) {
                asset = {
                    client_key: generateRowKey("asset"),
                    id: null,
                    is_purchase: true,
                    name: "",
                    asset_type: "",
                    description: "",
                    declared_value: null,
                    registration_number: "",
                    chassis_number: "",
                    block_and_parcel: "",
                    deed_number: "",
                    document_ids: [],
                    owners: [
                        {
                            client_key: generateRowKey("owner"),
                            id: null,
                            party_id: this.toId(form.primary.party_id) || "",
                            percentage: 100,
                        },
                    ],
                    collateral: createEmptyCollateral(),
                };
                form.assets.unshift(asset);
            }

            const typeFor = (pattern) => {
                const option = (this.lookups.asset_type || []).find((entry) =>
                    pattern.test(String(entry.label)),
                );
                return option ? option.value : asset.asset_type;
            };

            if (isAuto) {
                Object.assign(asset, {
                    name:
                        [form.vehicle_year, form.vehicle_make, form.vehicle_model]
                            .filter(Boolean)
                            .join(" ") || "Vehicle being purchased",
                    asset_type: typeFor(/vehicle|car|auto|motor/i),
                    registration_number: form.vehicle_registration_number,
                    chassis_number: form.vehicle_chassis_number,
                    block_and_parcel: "",
                    deed_number: "",
                });
            } else {
                Object.assign(asset, {
                    name: form.property_address || "Property being purchased",
                    asset_type: typeFor(/property|land|house|home|real estate/i),
                    registration_number: "",
                    chassis_number: "",
                    block_and_parcel: form.property_block_and_parcel,
                    deed_number: form.property_deed_number,
                });
            }
            asset.declared_value = price;
            asset.description = "Being purchased with this loan.";
            asset.collateral.enabled = true;

            // Owned by the primary applicant until someone changes the split.
            const primaryId = this.toId(form.primary.party_id);
            if (primaryId && asset.owners.length === 1 && !asset.owners[0].party_id) {
                asset.owners[0].party_id = primaryId;
            }
        },

        /**
         * Unemployed, retired, and student applicants don't need employer
         * details, and pay no NIS.
         */
        showsEmploymentDetails(status) {
            const value = String(status || "")
                .trim()
                .toLowerCase();
            return !NOT_WORKING_STATUSES.includes(value);
        },

        /** True when the person's work situation is Student. */
        isStudent(person) {
            return (
                String(person?.employment_status || "")
                    .trim()
                    .toLowerCase() === "student"
            );
        },

        /** Sums percentage allocations (ownership/responsibility splits). */
        allocationTotal(rows) {
            return (rows || []).reduce(
                (sum, row) => sum + (Number(row.percentage) || 0),
                0,
            );
        },

        /**
         * True when allocations total 100%. Uses a small tolerance so splits
         * like 33.33 / 33.33 / 33.34 aren't rejected by floating-point error.
         */
        allocationComplete(rows) {
            return Math.abs(this.allocationTotal(rows) - 100) < 0.01;
        },

        /** Quotes a value for a list query, escaping single quotes. */
        quote(value) {
            return `'${String(value ?? "").replace(/'/g, "''")}'`;
        },

        // ---- Liability and expense type resources ----

        /**
         * Reads a boolean that may arrive as true/false, "true"/"false" or 1/0.
         * Returns the fallback when the value is missing.
         */
        toBool(value, fallback = false) {
            if (value === null || value === undefined || value === "")
                return fallback;
            if (typeof value === "string")
                return ["true", "1", "yes"].includes(value.toLowerCase());
            return Boolean(value);
        },

        /**
         * Normalizes a rate to a fraction. Accepts 0.03 or 3 (meaning 3%).
         * Returns null for missing or non-positive values.
         */
        normalizeRate(value) {
            const rate = Number(value);
            if (!Number.isFinite(rate) || rate <= 0) return null;
            return rate > 1 ? rate / 100 : rate;
        },

        /** Normalizes applies_to (array or comma-separated string) to lowercase keys. */
        normalizeCategories(value) {
            const entries = Array.isArray(value)
                ? value
                : String(value || "").split(",");
            return entries
                .map((entry) =>
                    String(
                        (entry && typeof entry === "object"
                            ? entry.value || entry.name
                            : entry) || "",
                    )
                        .trim()
                        .toLowerCase(),
                )
                .filter(Boolean);
        },

        /** Keeps active type records and sorts them by sort_order, then name. */
        activeSortedTypes(rows) {
            return this.toList(rows)
                .filter((type) => this.toBool(type.is_active, true))
                .sort(
                    (left, right) =>
                        (Number(left.sort_order) || 0) -
                            (Number(right.sort_order) || 0) ||
                        String(left.name || "").localeCompare(
                            String(right.name || ""),
                        ),
                );
        },

        /**
         * Resolves a stored type value to a type record ID. Handles IDs,
         * record objects, and legacy string values saved before the type
         * resources existed (matched by name or code). Unmatched values are
         * returned unchanged so validation can flag them.
         */
        resolveTypeId(value, types) {
            const id = this.toId(value);
            if (!id) return "";
            if (types.some((type) => this.toId(type) === id)) return id;

            const text = String(id).trim().toLowerCase();
            const match = types.find(
                (type) =>
                    String(type.name || "").trim().toLowerCase() === text ||
                    String(type.code || "").trim().toLowerCase() === text,
            );
            return match ? this.toId(match) : id;
        },

        liabilityTypeFor(typeId) {
            const id = this.toId(typeId);
            return (
                this.liabilityTypes.find((type) => this.toId(type) === id) ||
                null
            );
        },

        isRevolvingType(typeId) {
            return Boolean(this.liabilityTypeFor(typeId)?.is_revolving);
        },

        /** Rate applied to the credit limit, or null for non-revolving types. */
        revolvingRateFor(typeId) {
            const type = this.liabilityTypeFor(typeId);
            if (!type?.is_revolving) return null;
            return type.revolving_rate || this.defaultRevolvingRate;
        },

        /**
         * Assessed monthly repayment for revolving credit: limit × rate.
         * This is a client-side figure; the server should recalculate it
         * before it's used for underwriting.
         */
        assessedPayment(liability) {
            const rate = this.revolvingRateFor(liability.liability_type);
            const limit = Number(liability.credit_limit);
            if (rate === null || !(limit > 0)) return null;
            return Math.round(limit * rate * 100) / 100;
        },

        expenseTypeFor(typeId) {
            const id = this.toId(typeId);
            return (
                this.expenseTypes.find((type) => this.toId(type) === id) || null
            );
        },

        expenseTypeName(typeId) {
            return this.expenseTypeFor(typeId)?.name || "";
        },

        /** An empty applies_to list means the type applies to every loan category. */
        expenseTypeAppliesToLoan(type) {
            if (!type) return false;
            return (
                !type.applies_to.length ||
                type.applies_to.some((code) => categoryKind(code) === this.loanKind)
            );
        },

        /** Merges partial updates from the request step into the form. */
        patchRequest(patch) {
            Object.assign(this.formData, patch);
        },

        /** The loan types on offer in one loan category (an entry of loans), or []. */
        productsIn(category) {
            if (!category || !category.recordId) return [];
            return this.products.filter(
                (product) =>
                    this.isOffered(product) &&
                    this.toId(product.loan_category) === category.recordId,
            );
        },

        /** A loan type is on offer when its status is Active or empty. */
        isOffered(product) {
            const status = String(product?.status || "").trim().toLowerCase();
            return status === "" || status === "active";
        },

        /**
         * Switching categories clears the product and dependent values. The
         * category's code (e.g. "PROPERTY") is what's saved.
         */
        chooseLoan(category) {
            Object.assign(this.formData, {
                loan_category: category,
                loan_type_id: "",
                loan_name: "",
                requested_loan_amount: null,
                requested_loan_term: null,
                requested_credit_limit: null,
            });
            // Only one loan of this type: choose it for them.
            if (this.productsForLoan.length === 1) {
                this.selectProduct(this.toId(this.productsForLoan[0]));
            }
        },

        /**
         * Requirements follow the product name automatically. For a saved
         * draft, documents already linked may satisfy the new product's
         * requirements, so re-check them.
         */
        selectProduct(id) {
            this.formData.loan_type_id = id;
            if (this.retiredLoanName) {
                this.retiredLoanName = "";
                this.alert = { text: "", type: "warning" };
            }
            this.chooseProduct();
            if (this.formData.id) this.hydrateAllDocumentUploads();
        },

        /**
         * Syncs the loan name from the selected product and clamps any
         * previously entered amount/term to the product's allowed range.
         */
        chooseProduct() {
            this.formData.loan_name = this.selectedProduct?.name || "";

            if (this.formData.requested_loan_amount !== null) {
                this.formData.requested_loan_amount = Math.min(
                    this.amountMaximum,
                    Math.max(
                        this.amountMinimum,
                        Number(this.formData.requested_loan_amount),
                    ),
                );
            }
            if (this.formData.requested_loan_term !== null) {
                this.formData.requested_loan_term = Math.min(
                    this.termMaximum,
                    Math.max(
                        this.termMinimum,
                        Number(this.formData.requested_loan_term),
                    ),
                );
            }
        },

        addApplicant() {
            const newApplicant = createEmptyApplicant("");
            this.formData.parties.push(newApplicant);
            this.activePartyTab = newApplicant.client_key;
        },

        /**
         * Removes an additional applicant and clears every asset ownership,
         * liability responsibility, and expense assigned to them, so those
         * rows fail validation until they're reassigned instead of silently
         * pointing at someone who is no longer on the application.
         */
        removeApplicant(index) {
            const removed = this.formData.parties.splice(index, 1)[0];
            this.activePartyTab = this.formData.primary.client_key;
            if (!removed) return;

            const partyId = this.toId(removed.party_id);
            const linkId = this.toId(removed.application_party_id);
            let cleared = 0;

            if (partyId) {
                this.formData.assets.forEach((asset) =>
                    (asset.owners || []).forEach((owner) => {
                        if (this.toId(owner.party_id) === partyId) {
                            owner.party_id = "";
                            cleared++;
                        }
                    }),
                );
            }

            if (linkId) {
                this.formData.liabilities.forEach((liability) =>
                    (liability.responsibilities || []).forEach((row) => {
                        if (this.toId(row.application_party_id) === linkId) {
                            row.application_party_id = "";
                            cleared++;
                        }
                    }),
                );
                this.formData.expenses.forEach((expense) => {
                    if (this.toId(expense.application_party_id) === linkId) {
                        expense.application_party_id = "";
                        cleared++;
                    }
                });
            }

            if (cleared) {
                const name = this.fullName(removed) || "The applicant";
                this.warn(
                    `${name} was removed. Reassign the ${cleared} asset, liability, or expense ${
                        cleared === 1 ? "entry" : "entries"
                    } that belonged to them.`,
                    "warning",
                );
            }
        },

        // ---- Test mode ----

        /**
         * Turns test mode on or off. Turning it on fills the current step,
         * makes documents optional, and (unless "Remember draft on reload"
         * is on) forgets the saved draft so a reload starts from the beginning.
         */
        toggleTestMode(value) {
            this.testMode = Boolean(value);
            if (this.testMode) {
                if (!this.testKeepDraft) this.forgetDraftLocation();
                this.fillTestData();
                this.warn(
                    "Test mode is on. Each step is filled with sample data as you reach it, and documents are optional.",
                    "info",
                );
            } else {
                this.alert = { text: "", type: "warning" };
                // Back to normal: remember the current draft again.
                if (this.formData.id) this.rememberDraftLocation();
            }
        },

        /** Test mode only: whether a reload brings back the current draft. */
        toggleTestKeepDraft(value) {
            this.testKeepDraft = Boolean(value);
            if (this.testKeepDraft) {
                if (this.formData.id) this.rememberDraftLocation();
            } else {
                this.forgetDraftLocation();
            }
        },

        /**
         * Saves the draft's record ID and step in localStorage, so the draft
         * can be restored after a reload. Skipped in test mode unless
         * "Remember draft on reload" is on.
         */
        rememberDraftLocation() {
            if (this.testMode && !this.testKeepDraft) return;
            localStorage.setItem(this.draftKey("draft_app_id"), this.formData.id);
            localStorage.setItem(this.draftKey("draft_step"), String(this.step));
        },

        /** Removes the saved draft location, so a reload starts fresh. */
        forgetDraftLocation() {
            localStorage.removeItem(this.draftKey("draft_app_id"));
            localStorage.removeItem(this.draftKey("draft_step"));
        },

        /**
         * The key prefix for a company name: lower case, with anything that
         * isn't a letter or a number turned into "_", then "_" (for example
         * "grenada_credit_union_"). With no name, "loan_". Renaming the
         * company makes browsers forget drafts saved under the old name.
         */
        draftKeyPrefixFor(name) {
            const key = String(name || "")
                .trim()
                .toLowerCase()
                .replace(/[^a-z0-9]/g, "_");
            return key ? `${key}_` : "loan_";
        },

        /** A localStorage key for the draft, e.g. "grenada_credit_union_draft_step". */
        draftKey(suffix) {
            return `${this.draftKeyPrefix}${suffix}`;
        },

        /**
         * Value of the first dropdown option whose label matches the pattern,
         * else the first option, else the fallback.
         */
        testOption(field, pattern, fallback = "") {
            const options = this.lookups[field] || [];
            const match = pattern
                ? options.find((option) => pattern.test(String(option.label)))
                : null;
            if (match) return match.value;
            return options.length ? options[0].value : fallback;
        },

        /** Sets each value on the target only where the field is still empty. */
        fillBlanks(target, values) {
            Object.keys(values).forEach((key) => {
                const current = target[key];
                if (current === "" || current === null || current === undefined) {
                    target[key] = values[key];
                }
            });
        },

        /**
         * Fills the current step with sample data. Only empty fields are
         * filled and lists are only added to when they're empty, so nothing
         * already typed is overwritten.
         */
        fillTestData() {
            const form = this.formData;
            const primary = form.primary;
            const id = this.current.id;

            if (id === "loan") {
                if (!form.loan_category) {
                    // An auto loan exercises collateral too, when it has products.
                    const withProducts = this.loans.filter(
                        (loan) => this.productsIn(loan).length > 0,
                    );
                    const pick =
                        withProducts.find((loan) => loan.kind === "automotive") ||
                        withProducts[0] ||
                        this.loans[0];
                    if (pick) this.chooseLoan(pick.id);
                }
                if (!this.toId(form.loan_type_id) && this.productsForLoan.length) {
                    this.selectProduct(this.toId(this.productsForLoan[0]));
                }
            }

            if (id === "parties") {
                // Just the one applicant, unless someone was already added,
                // or the product needs a guarantor (one is added below).
                if (this.requiresGuarantor && !form.parties.length) {
                    form.parties.push(createEmptyApplicant(GUARANTOR_ROLE));
                }
                if (this.answers.others === null) {
                    this.answers.others = form.parties.length > 0;
                }
                const sample = {
                    first_name: "Test",
                    last_name: "Applicant",
                    email: "test.applicant@example.com",
                    phone: "473-555-0100",
                    preferred_contact_method: this.testOption("preferred_contact_method", /phone/i),
                    date_of_birth: "1990-05-15",
                    marital_status: this.testOption("marital_status", /single/i),
                    address: "12 Test Street, Grand Anse",
                    country: DEFAULT_COUNTRY,
                    parish: this.testOption("parish", /george/i, "St. George"),
                    nis_number: "TEST123456",
                    member_number: "M-TEST-001",
                    citizenship: this.testOption("country", /grenada/i, DEFAULT_COUNTRY),
                    residency_status: this.testOption("residency_status", /citizen/i, "Citizen"),
                    housing_status: this.testOption("housing_status", /rent/i, "Rent"),
                    address_since: "2019-03-01",
                    number_of_dependants: 1,
                    employment_status: this.testOption("employment_status", /^employed/i),
                    employment_type: this.testOption("employment_type", /permanent/i, "Permanent"),
                    employment_start_date: "2019-03-01",
                    employer_name: "Test Employer Ltd",
                    job_title: "Clerk",
                    gross_pay: 6000,
                    pay_frequency: this.testOption("pay_frequency", /^monthly/i, "Monthly"),
                };
                this.fillBlanks(primary, sample);
                const people = [primary];
                form.parties.forEach((person) => {
                    if (!this.isGuarantor(person)) return;
                    this.fillBlanks(
                        person,
                        Object.assign({}, sample, {
                            first_name: "Test",
                            last_name: "Guarantor",
                            email: "test.guarantor@example.com",
                            phone: "473-555-0103",
                            nis_number: "TEST654321",
                            member_number: "M-TEST-002",
                            guarantee_type: this.testOption("guarantee_type", /full/i),
                            guarantee_amount: 10000,
                        }),
                    );
                    if (!person.identifications.length) {
                        person.identifications.push(createEmptyIdentification(true));
                    }
                    this.fillBlanks(person.identifications[0], {
                        identification_type: this.testOption("identification_type", /passport/i),
                        identification_number: "TEST-0002",
                        issuing_country: DEFAULT_COUNTRY,
                        issue_date: "2022-01-10",
                        expiry_date: "2032-01-10",
                    });
                    people.push(person);
                });
                people.forEach((person) => {
                    person.is_member = true;
                    person.consent_accuracy_confirmation = true;
                    person.consent_credit_check = true;
                    person.consent_data_processing = true;
                });

                const relationship = (pattern, fallback) =>
                    this.testOption("reference_relationship", pattern, fallback);
                const samples = {
                    "Personal reference": {
                        name: "Test Reference",
                        relationship: relationship(/friend/i, "Friend"),
                        phone: "473-555-0101",
                    },
                    "Next of kin": {
                        name: "Test Next Of Kin",
                        relationship: relationship(/parent/i, "Parent"),
                        phone: "473-555-0102",
                    },
                };
                (form.references || []).forEach((row) => {
                    if (samples[row.reference_type]) {
                        this.fillBlanks(row, samples[row.reference_type]);
                    }
                });

                if (!primary.identifications.length) {
                    primary.identifications.push(createEmptyIdentification(true));
                }
                this.fillBlanks(primary.identifications[0], {
                    identification_type: this.testOption("identification_type", /passport/i),
                    identification_number: "TEST-0001",
                    issuing_country: DEFAULT_COUNTRY,
                    issue_date: "2022-01-10",
                    expiry_date: "2032-01-10",
                });
            }

            if (id === "request") {
                this.fillBlanks(form, {
                    requested_loan_amount: Math.min(
                        this.amountMaximum,
                        Math.max(this.amountMinimum, 25000),
                    ),
                    requested_loan_term: Math.min(
                        this.termMaximum,
                        Math.max(this.termMinimum, 60),
                    ),
                    repayment_frequency: this.testOption("repayment_frequency", /month/i),
                    loan_purpose: "Test application created in test mode.",
                });
                if (this.isRevolving) {
                    this.fillBlanks(form, {
                        requested_credit_limit: Math.min(
                            this.amountMaximum,
                            Math.max(this.amountMinimum, 5000),
                        ),
                        linked_account_number: "TEST-ACCT-0001",
                    });
                }
                if (this.loanKind === "credit_card") {
                    this.fillBlanks(form, {
                        name_on_card: "TEST APPLICANT",
                        card_collection_method: this.testOption("card_collection_method", /branch/i),
                    });
                }
                if (this.loanKind === "student") {
                    this.fillBlanks(form, {
                        institution_name: "Test University",
                        institution_country: DEFAULT_COUNTRY,
                        program_name: "Test Studies",
                        program_level: this.testOption("program_level", /bachelor|degree/i),
                        enrollment_start_date: "2026-09-01",
                        expected_graduation_date: "2030-06-30",
                        tuition_amount: 20000,
                        tuition_currency: this.testOption("tuition_currency", /xcd|ec/i),
                        disbursement_schedule: this.testOption("disbursement_schedule", /semester|term/i),
                    });
                }
                if (this.loanKind === "automotive") {
                    this.fillBlanks(form, {
                        vehicle_make: "Toyota",
                        vehicle_model: "Corolla",
                        vehicle_year: "2020",
                        vehicle_condition: this.testOption("vehicle_condition", /used/i),
                        vehicle_chassis_number: "TESTCHASSIS0001",
                        purchase_price: 45000,
                        down_payment_amount: 5000,
                        seller_name: "Test Motors Ltd",
                    });
                }
                if (this.loanKind === "property") {
                    this.fillBlanks(form, {
                        property_address: "5 Test Road, St. George's",
                        property_type: this.testOption("property_type", /house|single/i),
                        property_value: 350000,
                        property_block_and_parcel: "Block 1234 Parcel 56",
                        purchase_price: 350000,
                        down_payment_amount: 35000,
                        seller_name: "Test Seller",
                    });
                }
                if (this.loanKind === "automotive" || this.loanKind === "property") {
                    this.fillBlanks(form, {
                        source_of_funds: this.testOption("source_of_funds", /savings/i, "Savings"),
                        seller_type: this.testOption(
                            "seller_type",
                            this.loanKind === "automotive" ? /dealer/i : /private/i,
                        ),
                    });
                }
                if (this.loanKind === "organization") {
                    this.fillBlanks(form, {
                        business_name: "Test Business Ltd",
                        business_signing_authority: "Director",
                        business_industry: "Retail",
                        business_registration_number: "TEST-REG-001",
                        business_type: this.testOption("business_type", null),
                        business_incorporation_date: "2015-06-01",
                        business_employee_count: 5,
                        business_annual_revenue: 250000,
                    });
                }
            }

            if (id === "assets") {
                this.answers.assets = true;
                // A purchase loan already has the asset being bought.
                if (!form.assets.length) {
                    form.assets.push({
                        client_key: generateRowKey("asset"),
                        id: null,
                        name: "Test vehicle",
                        asset_type: this.testOption("asset_type", /vehicle|car|auto/i),
                        description: "Sample asset added in test mode.",
                        declared_value: 45000,
                        registration_number: "PTEST1",
                        chassis_number: "",
                        block_and_parcel: "",
                        deed_number: "",
                        document_ids: [],
                        owners: [
                            {
                                client_key: generateRowKey("owner"),
                                id: null,
                                party_id: this.toId(primary.party_id) || "",
                                percentage: 100,
                            },
                        ],
                        collateral: Object.assign(createEmptyCollateral(), {
                            enabled: this.requiresCollateral,
                        }),
                    });
                }
                // Fill the insurance on every asset used as collateral.
                form.assets
                    .filter((asset) => this.isCollateral(asset))
                    .forEach((asset) => {
                        this.fillBlanks(asset.collateral, {
                            description: "Sample collateral.",
                        });
                        this.fillBlanks(asset.collateral.insurance, {
                            type: this.testOption("insurance_type", /comprehensive/i, "Comprehensive"),
                            provider: "Test Insurance Co.",
                            reference: "QUOTE-TEST-001",
                            coverage_amount: asset.declared_value,
                            premium: 150,
                            premium_frequency: this.testOption(
                                "insurance_premium_frequency",
                                /^monthly/i,
                                "Monthly",
                            ),
                        });
                    });
            }

            if (id === "liabilities") this.answers.debts = true;
            if (id === "liabilities" && !form.liabilities.length) {
                const type =
                    this.liabilityTypes.find((entry) => !entry.is_revolving) ||
                    this.liabilityTypes[0] ||
                    null;
                form.liabilities.push({
                    client_key: generateRowKey("liability"),
                    id: null,
                    creditor_name: "Test Lender",
                    liability_type: type ? this.toId(type) : "",
                    outstanding_balance: 5000,
                    payment_amount: 250,
                    payment_frequency: this.testOption("payment_frequency", /month/i),
                    credit_limit: type && type.is_revolving ? 5000 : null,
                    assessed_payment: null,
                    description: "",
                    is_to_be_paid_off: false,
                    is_secured: false,
                    secured_asset_ref: "",
                    document_ids: [],
                    responsibilities: [
                        {
                            client_key: generateRowKey("resp"),
                            id: null,
                            application_party_id:
                                this.toId(primary.application_party_id) || "",
                            percentage: 100,
                            responsibility_type: "Borrower",
                        },
                    ],
                });
            }

            if (id === "expenses") this.answers.bills = true;
            if (id === "expenses" && !form.expenses.length) {
                const type = this.expenseTypeOptions[0] || null;
                form.expenses.push({
                    client_key: generateRowKey("expense"),
                    id: null,
                    application_party_id: this.toId(primary.application_party_id) || "",
                    expense_name: "",
                    expense_type: type ? type.value : "",
                    amount: 800,
                    frequency: this.testOption("expense_frequency", /^monthly/i),
                    is_household: false,
                    document_ids: [],
                });
            }
        },

        // ---- Document requirements ----

        /**
         * Normalizes attachment group labels for comparison, so
         * "Application - Auto Loan" matches "application-auto loan".
         */
        normalizeGroupLabel(value) {
            return String(value || "")
                .trim()
                .toLowerCase()
                .replace(/\s*-\s*/g, "-")
                .replace(/\s+/g, " ");
        },

        /**
         * Loads every AttachmentGroup and Attachment_type once and indexes the
         * attachment types by normalized group label, so requirement lookups
         * are instant. Group labels follow "<kind>-<name>":
         *
         *   application-<loan name>    applicant-<loan name>
         *   asset-<asset type>         liability-<liability type>
         *   expense-<expense type>     collateral-<asset type>
         *
         * "<kind>-all" (e.g. "applicant-all") applies to every item of that kind.
         */
        async loadAttachmentCatalog() {
            this.documentsLoading = true;

            try {
                const [groupRows, typeRows] = await Promise.all([
                    new Resource(this, "AttachmentGroup").list({ limit: 200 }),
                    new Resource(this, "Attachment_type")
                        .list({ limit: 200 })
                        .catch(() => []),
                ]);

                const typesById = {};
                this.toList(typeRows).forEach((type) => {
                    const id = this.toId(type);
                    if (id) typesById[id] = type;
                });

                const isRecord = (value) =>
                    value && typeof value === "object" && value.name;
                const groups = this.toList(groupRows);

                // Fetch any referenced types the list didn't include.
                const missingIds = new Set();
                groups.forEach((group) =>
                    (group.attachment_types || []).forEach((value) => {
                        const id = this.toId(value);
                        if (!isRecord(value) && id && !typesById[id]) {
                            missingIds.add(id);
                        }
                    }),
                );
                (await this.byIds("Attachment_type", [...missingIds])).forEach(
                    (type) => {
                        typesById[this.toId(type)] = type;
                    },
                );

                const index = {};
                groups.forEach((group) => {
                    const label = this.normalizeGroupLabel(group.label);
                    if (!label) return;
                    const types = (group.attachment_types || [])
                        .map((value) =>
                            isRecord(value) ? value : typesById[this.toId(value)],
                        )
                        .filter(Boolean);
                    index[label] = [...(index[label] || []), ...types];
                });

                this.attachmentTypesByGroup = index;
            } catch (error) {
                this.attachmentTypesByGroup = {};
                this.warn(
                    "Document requirements could not be loaded. Refresh the page to try again.",
                    "warning",
                );
            } finally {
                this.documentsLoading = false;
            }
        },

        /**
         * Attachment types required for one item: its "<kind>-all" group plus
         * its "<kind>-<name>" group, deduplicated by ID.
         */
        requirementTypesFor(kind, name) {
            const labels = [`${kind}-all`];
            if (name) labels.push(`${kind}-${name}`);

            const seenIds = new Set();
            return labels
                .flatMap(
                    (label) =>
                        this.attachmentTypesByGroup[
                            this.normalizeGroupLabel(label)
                        ] || [],
                )
                .filter((type) => {
                    const id = this.toId(type);
                    if (!id || seenIds.has(id)) return false;
                    seenIds.add(id);
                    return true;
                });
        },

        /** Display label of an asset type value (used to match group labels). */
        assetTypeLabel(value) {
            const option = (this.lookups.asset_type || []).find(
                (entry) => entry.value === value,
            );
            return option ? option.label : value || "";
        },

        // An item can only receive uploads once the fields it needs to be
        // saved on its own are filled in.

        assetReady(asset) {
            return Boolean(
                asset &&
                    asset.name &&
                    asset.asset_type &&
                    asset.declared_value !== null &&
                    asset.declared_value !== undefined,
            );
        },

        liabilityReady(liability) {
            return Boolean(
                liability &&
                    liability.creditor_name &&
                    this.liabilityTypeFor(liability.liability_type) &&
                    liability.outstanding_balance !== null &&
                    liability.outstanding_balance !== undefined &&
                    liability.payment_amount !== null &&
                    liability.payment_amount !== undefined &&
                    (!this.isRevolvingType(liability.liability_type) ||
                        Number(liability.credit_limit) > 0),
            );
        },

        expenseReady(expense) {
            return Boolean(
                expense &&
                    expense.application_party_id &&
                    expense.expense_type &&
                    expense.amount !== null &&
                    expense.amount !== undefined,
            );
        },

        /** What's missing from a collateral item's insurance, or "". */
        insuranceIssue(collateral) {
            const insurance = collateral.insurance || {};
            if (!insurance.type || !insurance.status || !insurance.provider) {
                return "Enter the insurance type, whether it's a policy or a quote, and the insurer.";
            }
            const isPolicy =
                String(insurance.status).trim().toLowerCase() === "policy";
            if (isPolicy && !insurance.reference) {
                return "Enter the policy number.";
            }
            if (
                insurance.premium === null ||
                insurance.premium === undefined ||
                !insurance.premium_frequency
            ) {
                return "Enter the insurance premium and how often it's paid.";
            }
            if (
                isPolicy &&
                insurance.expiry_date &&
                insurance.expiry_date < this.today()
            ) {
                return "The insurance policy has expired. Enter a current policy or a quote.";
            }
            return "";
        },

        /** True when every required document in the scope has been uploaded. */
        scopeComplete(scope) {
            return (
                !scope ||
                scope.requirements.every(
                    (item) => item.optional || item.status === "uploaded",
                )
            );
        },

        /**
         * First scope (optionally of the given kinds) still missing documents.
         * Documents aren't required in test mode, so it returns null then.
         */
        firstIncompleteScope(kinds = null) {
            if (this.testMode) return null;
            return (
                this.documentScopeList.find(
                    (scope) =>
                        (!kinds || kinds.includes(scope.kind)) &&
                        !this.scopeComplete(scope),
                ) || null
            );
        },

        /** Composite key for one requirement within one scope. */
        requirementStateKey(scopeKey, typeId) {
            return `${scopeKey}::${typeId}`;
        },

        /**
         * Builds the requirement view models for one scope by merging each
         * attachment type's metadata with its current upload state.
         */
        requirementsFor(scopeKey, types, optionalIds = []) {
            return (types || []).map((type) => {
                const typeId = this.toId(type);
                const optional = optionalIds.includes(typeId);
                const state = this.documentState[
                    this.requirementStateKey(scopeKey, typeId)
                ] || {
                    status: "pending",
                    file: null,
                    error: "",
                };

                return Object.assign(
                    {
                        key: typeId,
                        id: typeId,
                        attachmentTypeId: typeId,
                        name: `${type.name || "Supporting document"}${optional ? " (optional)" : ""}`,
                        description: type.description || "",
                        allowed_file_types: type.allowed_file_types || [],
                        optional,
                    },
                    state,
                );
            });
        },

        /** Returns (creating if needed) the mutable state for one requirement. */
        stateFor(scopeKey, requirementKey) {
            const key = this.requirementStateKey(scopeKey, requirementKey);
            if (!this.documentState[key]) {
                this.documentState[key] = {
                    status: "pending",
                    file: null,
                    error: "",
                };
            }
            return this.documentState[key];
        },

        /** Records a locally selected file, ready to be uploaded. */
        stageDocument(payload) {
            Object.assign(
                this.stateFor(payload.scopeKey, payload.requirementKey),
                {
                    file: payload.file,
                    fileName: payload.file?.name || "",
                    fileSize: payload.file?.size || 0,
                    status: "ready",
                    error: "",
                },
            );
        },

        /** Clears a staged file before it was uploaded. */
        removeStagedDocument(payload) {
            Object.assign(
                this.stateFor(payload.scopeKey, payload.requirementKey),
                {
                    file: null,
                    fileName: "",
                    fileSize: 0,
                    status: "pending",
                    error: "",
                },
            );
        },

        handleRejectedDocument(payload) {
            this.warn(
                payload.message || "That file type is not accepted.",
                "warning",
            );
        },

        /**
         * Marks requirements as uploaded based on document IDs already linked
         * to the owner record. Uploads are matched to types via
         * saturn_file_type; untyped uploads fill remaining slots in order.
         */
        async hydrateScopeUploads(scopeKey, documentIds, types) {
            const ids = (documentIds || []).map(this.toId).filter(Boolean);
            if (!ids.length || !types.length) return;

            const uploads = await this.byIds("Upload", ids);
            const allowedTypeIds = types.map(this.toId).filter(Boolean);
            const unassignedTypeIds = [...allowedTypeIds];

            uploads.forEach((upload) => {
                let typeId = this.toId(upload.saturn_file_type);
                if (!typeId) typeId = unassignedTypeIds[0] || null;
                if (!typeId || !allowedTypeIds.includes(typeId)) return;

                // Each upload fills one type slot.
                const slotIndex = unassignedTypeIds.indexOf(typeId);
                if (slotIndex >= 0) unassignedTypeIds.splice(slotIndex, 1);

                Object.assign(this.stateFor(scopeKey, typeId), {
                    status: "uploaded",
                    uploadId: this.toId(upload),
                    uploadedFileName: upload.file_name || "Document attached",
                    file: null,
                    error: "",
                });
            });
        },

        /** Hydrates upload state for every scope (after restoring a draft). */
        async hydrateAllDocumentUploads() {
            for (const scope of this.documentScopeList) {
                const owner = this.scopeOwner(scope.key);
                await this.hydrateScopeUploads(
                    scope.key,
                    owner.record?.document_ids,
                    scope.types,
                );
            }
        },

        /**
         * Extracts an upload ID from an upload endpoint response. The response
         * shape varies between backends, so this walks common wrappers
         * (data/upload/record/result) looking for an "upload-" prefixed ID.
         */
        uploadIdFromResponse(result) {
            const queue = [result];
            const visited = new Set();

            while (queue.length) {
                const value = queue.shift();
                if (!value) continue;

                if (typeof value === "string") {
                    if (value.startsWith("upload-")) return value;
                    continue;
                }

                if (typeof value !== "object" || visited.has(value)) continue;
                visited.add(value);

                if (value.id && String(value.id).startsWith("upload-"))
                    return value.id;
                if (value.upload_id) return this.toId(value.upload_id);
                if (value.uploadId) return this.toId(value.uploadId);

                if (Array.isArray(value)) {
                    queue.push(...value);
                    continue;
                }

                ["data", "upload", "resource", "record", "result"].forEach(
                    (key) => {
                        if (value[key]) queue.push(value[key]);
                    },
                );
            }

            return null;
        },

        /**
         * Confirms an upload completed by re-reading the owner record. If the
         * response didn't yield an ID, falls back to diffing the document list
         * against what was there before, then to matching by file name.
         */
        async resolveCompletedUpload(
            result,
            resourceName,
            ownerId,
            beforeIds,
            fileName,
        ) {
            let uploadedId = this.uploadIdFromResponse(result);
            let linkedDocuments = [];

            try {
                const owner = this.recordOf(
                    await new Resource(this, resourceName).get(ownerId),
                );

                linkedDocuments = Array.isArray(owner?.documents)
                    ? owner.documents
                    : [];

                const linkedIds = linkedDocuments
                    .map(this.toId)
                    .filter(Boolean);

                // A document ID that wasn't there before must be the new upload.
                if (!uploadedId) {
                    uploadedId =
                        linkedIds.find((id) => !beforeIds.includes(id)) || null;
                }

                // Last resort: match by file name, newest first.
                if (!uploadedId && fileName) {
                    const matching = linkedDocuments
                        .filter(
                            (document) =>
                                document && document.file_name === fileName,
                        )
                        .sort((left, right) => {
                            const leftDate =
                                left.created_at || left.date_uploaded || "";
                            const rightDate =
                                right.created_at || right.date_uploaded || "";
                            return String(rightDate).localeCompare(
                                String(leftDate),
                            );
                        });

                    uploadedId = this.toId(matching[0]);
                }
            } catch (error) {
                // The response ID can still be used if re-reading fails.
            }

            return {
                uploadedId,
                linkedIds: linkedDocuments.map(this.toId).filter(Boolean),
            };
        },

        // ---- Upload owners ----

        /**
         * Resolves a scope key ("application" or "<kind>:<client_key>") to the
         * form record that owns its documents and the Saturn resource the
         * documents attach to.
         */
        scopeOwner(scopeKey) {
            const [kind, ref] = String(scopeKey || "").split(":");
            const form = this.formData;
            const find = (rows) =>
                (rows || []).find((row) => row.client_key === ref) || null;

            const owners = {
                application: () => ({ resourceName: "Application", record: form }),
                applicant: () => ({
                    resourceName: "ApplicationParty",
                    record: find(this.allApplicants),
                }),
                identification: () => {
                    const person =
                        this.allApplicants.find((entry) =>
                            (entry.identifications || []).some(
                                (row) => row.client_key === ref,
                            ),
                        ) || null;
                    return {
                        resourceName: "PartyIdentification",
                        record: person ? find(person.identifications) : null,
                        person,
                    };
                },
                // Other income is saved as part of its applicant.
                income: () => {
                    const person =
                        this.allApplicants.find((entry) =>
                            (entry.incomes || []).some(
                                (row) => row.client_key === ref,
                            ),
                        ) || null;
                    return {
                        resourceName: "IncomeSource",
                        record: person ? find(person.incomes) : null,
                        person,
                    };
                },
                asset: () => ({ resourceName: "Asset", record: find(form.assets) }),
                liability: () => ({
                    resourceName: "Liability",
                    record: find(form.liabilities),
                }),
                expense: () => ({
                    resourceName: "Expense",
                    record: find(form.expenses),
                }),
                // Collateral details live on their asset (asset.collateral).
                collateral: () => {
                    const asset = find(form.assets);
                    return {
                        resourceName: "Collateral",
                        record: asset ? asset.collateral || null : null,
                        asset,
                    };
                },
            };

            const owner = owners[kind]
                ? owners[kind]()
                : { resourceName: "", record: null };
            return Object.assign({ kind }, owner);
        },

        /** The server ID documents are attached to for an owner. */
        ownerIdOf(owner) {
            if (!owner.record) return null;
            if (owner.kind === "application") return this.toId(this.formData.id);
            if (owner.kind === "applicant") {
                return this.toId(owner.record.application_party_id);
            }
            return this.toId(owner.record.id);
        },

        /**
         * Links every saved applicant to the application, creating the
         * application first if this is the first thing saved.
         */
        async linkApplicantsToApplication() {
            const partyLinks = this.allApplicants
                .map((person) => this.toId(person.application_party_id))
                .concat(
                    this.loanKind === "organization"
                        ? [this.toId(this.formData.business_application_party_id)]
                        : [],
                )
                .filter(Boolean);
            const links = { parties: partyLinks };

            if (this.formData.id) {
                await this.upsert("Application", this.formData.id, links);
                return;
            }

            const createdId = await this.upsert(
                "Application",
                null,
                Object.assign(this.appPayload(), links),
            );
            this.formData.id = createdId;
            this.rememberDraftLocation();
        },

        /** Updates the application's asset/liability/expense ID lists. */
        async linkFinancialsToApplication() {
            const ids = this.collectPersistedIds();
            await this.upsert("Application", this.formData.id, {
                // Every declared asset, including ones part- or fully owned
                // by a Third Party Owner. Ownership rows say whose share is whose.
                asset_ids: this.formData.assets
                    .map((asset) => this.toId(asset.id))
                    .filter(Boolean),
                liability_ids: ids.Liability,
                expense_ids: ids.Expense,
            });
        },

        /**
         * Saves just the record that will own an upload (plus anything it
         * depends on) instead of the whole draft, so half-finished items
         * elsewhere in the form aren't pushed to the server.
         */
        async ensureOwnerSaved(owner) {
            const record = owner.record;

            if (
                owner.kind === "applicant" ||
                owner.kind === "identification" ||
                owner.kind === "income"
            ) {
                // Identifications and other income are saved with their applicant.
                const person = owner.kind === "applicant" ? record : owner.person;
                const primary = this.formData.primary;
                const isPrimary = person.client_key === primary.client_key;
                // The primary applicant must exist first, so a restored draft
                // never mistakes a co-applicant for the primary.
                if (!isPrimary && !primary.application_party_id) {
                    await this.saveApplicant(primary, true);
                    this.syncSavedIds(`applicant:${primary.client_key}`, primary);
                }
                await this.saveApplicant(person, isPrimary);
                this.syncSavedIds(`applicant:${person.client_key}`, person);
                await this.linkApplicantsToApplication();
            } else if (!this.formData.id) {
                // Everything else hangs off the application, which is created
                // the first time the draft is saved.
                await this.saveDraft(true);
            } else if (owner.kind === "asset") {
                await this.saveAsset(record);
                await this.linkFinancialsToApplication();
            } else if (owner.kind === "liability") {
                await this.saveLiability(record);
                await this.linkFinancialsToApplication();
            } else if (owner.kind === "expense") {
                await this.saveExpense(record);
                await this.linkFinancialsToApplication();
            } else if (owner.kind === "collateral") {
                // The Collateral record points at its asset, so save the
                // asset first if it's new.
                const asset = owner.asset;
                if (!asset.id) {
                    await this.saveAsset(asset);
                    this.syncSavedIds(`asset:${asset.client_key}`, asset);
                    await this.linkFinancialsToApplication();
                }
                await this.saveCollateral(asset);
            }

            // Track new IDs so removing the item later deletes it.
            this.rememberPersistedIds(this.persistedIds);
        },

        /**
         * Sections replace their arrays with fresh copies whenever the
         * applicant edits something, so the object saved during an upload
         * may no longer be the one in the form. Copies the server IDs onto
         * the current object so the next save updates instead of duplicating.
         */
        syncSavedIds(scopeKey, saved) {
            const current = this.scopeOwner(scopeKey).record;
            if (!current || current === saved) return current || saved;

            [
                "id",
                "party_id",
                "application_party_id",
                "consented_at",
                "consent_policy_version",
                "saved_identification_ids",
            ].forEach((field) => {
                if (saved[field] !== undefined && saved[field] !== null) {
                    current[field] = saved[field];
                }
            });

            // An asset's collateral ID lives in a nested object.
            if (saved.collateral && current.collateral && saved.collateral.id) {
                current.collateral.id = saved.collateral.id;
                if (saved.collateral.projected_expense_id) {
                    current.collateral.projected_expense_id =
                        saved.collateral.projected_expense_id;
                }
            }

            ["owners", "responsibilities", "identifications", "incomes"].forEach((listKey) =>
                (saved[listKey] || []).forEach((row) => {
                    const match = (current[listKey] || []).find(
                        (entry) => entry.client_key === row.client_key,
                    );
                    if (match && row.id) match.id = row.id;
                }),
            );

            return current;
        },

        /**
         * Uploads one staged document for one requirement: saves the owner
         * record if needed, uploads the file, verifies completion by
         * re-reading the owner, tags the upload with its attachment type,
         * and syncs the owner's document list.
         */
        async uploadScopedDocument(payload) {
            const state = this.stateFor(
                payload.scopeKey,
                payload.requirementKey,
            );

            // Ignore clicks with no staged file or while another upload runs.
            if (!state.file || this.uploadingDocumentKey) return;

            const selectedFile = state.file;

            this.uploadingDocumentKey = this.requirementStateKey(
                payload.scopeKey,
                payload.requirementKey,
            );
            state.status = "uploading";
            state.error = "";

            try {
                const scope = this.documentScopes[payload.scopeKey];
                if (!scope) throw Error("This document is no longer required.");
                if (scope.lockedMessage) throw Error(scope.lockedMessage);

                const owner = this.scopeOwner(payload.scopeKey);
                if (!owner.record) throw Error("This item no longer exists.");

                // Upload targets must exist on the server before files attach.
                await this.ensureOwnerSaved(owner);
                const record = this.syncSavedIds(payload.scopeKey, owner.record);
                const resourceName = owner.resourceName;
                const ownerId = this.ownerIdOf(
                    Object.assign({}, owner, { record }),
                );

                if (!ownerId) {
                    throw Error("The document owner has not been saved.");
                }

                const existingIds = [...(record.document_ids || [])];

                /*
                 * The upload endpoint requires tags and meta_data as well as
                 * the file, even when there's nothing to put in them, and
                 * rejects the request with a 400 when they're missing.
                 */
                const body = new FormData();
                body.append("file", selectedFile);
                body.append("name", selectedFile.name);
                body.append("saturn_file_type", payload.requirementKey);
                body.append("tags", JSON.stringify([]));
                body.append("meta_data", JSON.stringify({}));

                let result = null;
                let requestError = null;

                try {
                    result = await new Resource(this).request(
                        "post",
                        `/uploads/${resourceName}/${ownerId}/documents`,
                        {
                            data: body,
                            headers: { "Content-Type": "multipart/form-data" },
                        },
                    );
                } catch (error) {
                    /*
                     * The server may have completed the upload even when the client
                     * could not interpret the response. Re-read the owner before
                     * treating it as a failure.
                     */
                    requestError = error;
                }

                const resolved = await this.resolveCompletedUpload(
                    result,
                    resourceName,
                    ownerId,
                    existingIds,
                    selectedFile.name,
                );

                const uploadedId = resolved.uploadedId;
                if (!uploadedId) {
                    throw (
                        requestError ||
                        Error("The uploaded document could not be resolved.")
                    );
                }

                // Tag the Upload with its Attachment_type after resolving it.
                let typeSaved = true;
                try {
                    await this.upsert("Upload", uploadedId, {
                        saturn_file_type: payload.requirementKey,
                    });
                } catch (error) {
                    typeSaved = false;
                }

                /*
                 * The upload endpoint normally links the document automatically.
                 * This update ensures the local and persisted arrays agree.
                 */
                const documentIds = [
                    ...new Set([
                        ...resolved.linkedIds,
                        ...existingIds,
                        uploadedId,
                    ]),
                ];
                await this.upsert(resourceName, ownerId, {
                    documents: documentIds,
                });

                const target = this.scopeOwner(payload.scopeKey).record || record;
                target.document_ids = documentIds;

                Object.assign(state, {
                    status: "uploaded",
                    uploadId: uploadedId,
                    uploadedFileName: selectedFile.name,
                    fileName: selectedFile.name,
                    file: null,
                    error: "",
                });

                if (typeSaved) {
                    this.warn("Document uploaded successfully.", "success");
                } else {
                    this.warn(
                        "The document uploaded, but its document type could not be saved.",
                        "warning",
                    );
                }
            } catch (error) {
                state.status = "error";
                state.error =
                    error?.message ||
                    "The document could not be uploaded. Try again.";
                this.warn("The document could not be uploaded.", "error");
            } finally {
                this.uploadingDocumentKey = "";
            }
        },

        // ---- Branding and lookups ----

        /**
         * Loads tenant branding (name, colors, logo) from the most recently
         * updated SystemConfiguration record. Falls back to defaults.
         */
        async loadBranding() {
            const api = new Resource(this);
            let config = api.config || {};

            try {
                const rows = this.toList(
                    await new Resource(this, "SystemConfiguration").list({
                        limit: 1,
                        sort: "updated_at DESC",
                    }),
                );
                if (rows.length) config = rows[0];
            } catch (error) {
                // Branding is cosmetic; keep defaults on failure.
            }

            this.companyName = config.name || "Loan Application";
            this.draftKeyPrefix = this.draftKeyPrefixFor(config.name);
            this.primaryColor =
                config.primary_color || config.primaryColor || "";
            this.secondaryColor =
                config.secondary_color || config.secondaryColor || "";
            this.logoBackground = config.logo_bg_color || "";

            // The logo may be a record object or a bare upload ID.
            if (config.logo && typeof config.logo === "object") {
                this.companyLogo = config.logo.url || "";
            } else {
                const logoId = this.toId(config.logo);
                if (logoId) {
                    try {
                        this.companyLogo =
                            this.recordOf(
                                await new Resource(this, "Upload").get(logoId),
                            )?.url || "";
                    } catch (error) {
                        // Ignore; the header just won't show a logo.
                    }
                }
            }
        },

        /**
         * Extracts dropdown options for one field from a resource's property
         * metadata. lookup_reference may be an array or a comma-separated string.
         */
        options(source, name) {
            const target = String(name).toLowerCase();
            const label = target.replace(/_/g, " ");

            const property = this.toList(source).find(
                (row) =>
                    String(
                        row.property || row.key || row.name || "",
                    ).toLowerCase() === target ||
                    String(row.label || "").toLowerCase() === label,
            );
            if (!property) return [];

            const raw =
                property.lookup_reference || property.lookupReference || [];
            const entries = Array.isArray(raw) ? raw : String(raw).split(",");

            // Labels and values are always plain text, so a nested object
            // can never show up as "[object Object]" in a dropdown.
            const text = (value) => {
                if (value && typeof value === "object") {
                    return text(value.label || value.name || value.value || value.id);
                }
                return String(value ?? "").trim();
            };

            return entries
                .map((entry) =>
                    entry && typeof entry === "object"
                        ? {
                              label: text(
                                  entry.label || entry.name || entry.value,
                              ),
                              value: text(
                                  entry.value ||
                                      entry.id ||
                                      entry.name ||
                                      entry.label,
                              ),
                          }
                        : { label: text(entry), value: text(entry) },
                )
                .filter((entry) => entry.value !== "");
        },

        /**
         * Loads loan products, liability/expense type records, and all lookup
         * metadata in parallel. Individual failures degrade gracefully to
         * empty option lists.
         */
        async loadOptions() {
            const api = new Resource(this);
            const safeLoadProps = (resourceName) =>
                api.loadResourceProps(resourceName).catch(() => []);
            const safeList = (resourceName) =>
                new Resource(this, resourceName)
                    .list({ limit: 100 })
                    .catch(() => []);

            const [
                products,
                liabilityTypeRows,
                expenseTypeRows,
                applicationPartyProps,
                partyProps,
                identificationProps,
                applicationProps,
                assetProps,
                liabilityProps,
                expenseProps,
                collateralProps,
                incomeProps,
                referenceProps,
                categoryRows,
            ] = await Promise.all([
                safeList("LoanType"),
                safeList("LiabilityType"),
                safeList("ExpenseType"),
                safeLoadProps("ApplicationParty"),
                safeLoadProps("Party"),
                safeLoadProps("PartyIdentification"),
                safeLoadProps("Application"),
                safeLoadProps("Asset"),
                safeLoadProps("Liability"),
                safeLoadProps("Expense"),
                safeLoadProps("Collateral"),
                safeLoadProps("IncomeSource"),
                safeLoadProps("Reference"),
                safeList("LoanCategory"),
            ]);

            this.products = this.toList(products);
            // Active categories in sort_order.
            this.loanCategoryRecords = this.activeSortedTypes(categoryRows).filter(
                (row) => String(row.code || "").trim() !== "",
            );
            this.applicationProps = this.toList(applicationProps);
            this.resourceProps = {
                Application: this.toList(applicationProps),
                Party: this.toList(partyProps),
                ApplicationParty: this.toList(applicationPartyProps),
                PartyIdentification: this.toList(identificationProps),
                Asset: this.toList(assetProps),
                Liability: this.toList(liabilityProps),
                Expense: this.toList(expenseProps),
                Collateral: this.toList(collateralProps),
                IncomeSource: this.toList(incomeProps),
                Reference: this.toList(referenceProps),
            };

            this.liabilityTypes = this.activeSortedTypes(liabilityTypeRows).map(
                (type) =>
                    Object.assign({}, type, {
                        is_revolving: this.toBool(type.is_revolving),
                        revolving_rate: this.normalizeRate(type.revolving_rate),
                    }),
            );

            this.expenseTypes = this.activeSortedTypes(expenseTypeRows).map(
                (type) =>
                    Object.assign({}, type, {
                        applies_to: this.normalizeCategories(type.applies_to),
                        user_selectable: this.toBool(type.user_selectable, true),
                    }),
            );

            if (!this.liabilityTypes.length || !this.expenseTypes.length) {
                this.warn(
                    "Some options could not be loaded. Refresh the page before adding liabilities or expenses.",
                    "warning",
                );
            }

            Object.assign(this.lookups, {
                // "Business Borrower" is only for the business on a business
                // loan, which the form links itself, so it's never offered.
                role: this.options(applicationPartyProps, "role").filter(
                    (option) =>
                        String(option.value).trim().toLowerCase() !==
                        BUSINESS_BORROWER_ROLE.toLowerCase(),
                ),
                employment_status: this.options(
                    applicationPartyProps,
                    "employment_status",
                ),
                identification_type: this.options(
                    identificationProps,
                    "identification_type",
                ),
                marital_status: this.options(partyProps, "marital_status"),
                parish: this.options(partyProps, "parish"),
                country: this.options(partyProps, "country"),
                repayment_frequency: this.options(
                    applicationProps,
                    "repayment_frequency",
                ),
                vehicle_condition: this.options(
                    applicationProps,
                    "vehicle_condition",
                ),
                property_type: this.options(applicationProps, "property_type"),
                // The business is its own Party now; fall back to Application.
                business_type: this.options(partyProps, "business_type").length
                    ? this.options(partyProps, "business_type")
                    : this.options(applicationProps, "business_type"),
                asset_type: this.options(assetProps, "asset_type"),
                payment_frequency: this.options(
                    liabilityProps,
                    "payment_frequency",
                ),
                expense_frequency: this.options(expenseProps, "frequency"),
                insurance_type: this.options(collateralProps, "insurance_type"),
                insurance_status: this.options(
                    collateralProps,
                    "insurance_status",
                ),
                // Falls back to the expense frequencies if Collateral has none.
                insurance_premium_frequency: this.options(
                    collateralProps,
                    "insurance_premium_frequency",
                ).length
                    ? this.options(collateralProps, "insurance_premium_frequency")
                    : this.options(expenseProps, "frequency"),
                relationship_to_applicant: this.options(
                    applicationPartyProps,
                    "relationship_to_applicant",
                ),
                residency_status: this.options(partyProps, "residency_status"),
                housing_status: this.options(applicationPartyProps, "housing_status"),
                employment_type: this.options(applicationPartyProps, "employment_type"),
                pay_frequency: this.options(applicationPartyProps, "pay_frequency"),
                guarantee_type: this.options(applicationPartyProps, "guarantee_type"),
                income_type: this.options(incomeProps, "income_type"),
                income_frequency: this.options(incomeProps, "frequency"),
                reference_type: this.options(referenceProps, "reference_type"),
                preferred_contact_method: this.options(applicationPartyProps, "preferred_contact_method"),
                reference_relationship: this.options(referenceProps, "relationship"),
                source_of_funds: this.options(applicationProps, "source_of_funds"),
                seller_type: this.options(applicationProps, "seller_type"),
                card_collection_method: this.options(applicationProps, "card_collection_method"),
                program_level: this.options(applicationProps, "program_level"),
                tuition_currency: this.options(applicationProps, "tuition_currency"),
                disbursement_schedule: this.options(applicationProps, "disbursement_schedule"),
            });
            this.optionsLoaded = true;
        },

        // ---- Validation and navigation ----

        // ---- Statutory deductions ----

        /** Whole years of age today, or null without a valid date of birth. */
        ageOn(dateOfBirth) {
            if (!dateOfBirth) return null;
            const birth = new Date(dateOfBirth);
            if (Number.isNaN(birth.getTime())) return null;
            const now = new Date();
            let age = now.getFullYear() - birth.getFullYear();
            const birthdayPassed =
                now.getMonth() > birth.getMonth() ||
                (now.getMonth() === birth.getMonth() &&
                    now.getDate() >= birth.getDate());
            if (!birthdayPassed) age--;
            return age;
        },

        roundMoney(value) {
            return Math.round(value * 100) / 100;
        },

        /**
         * Estimated monthly NIS contribution from gross monthly income.
         * Employees pay 6.25% and the self-employed 13.5%, both on income up
         * to the insurable cap. Nothing is due for the unemployed, the
         * retired, students, or anyone under 16 or at pensionable age.
         * Returns { amount, basis } where basis explains the figure.
         */
        estimateNis(person) {
            const settings = STATUTORY_DEDUCTIONS.nis;
            const gross = Number(this.grossMonthly(person)) || 0;
            const status = String(person.employment_status || "")
                .trim()
                .toLowerCase();
            const age = this.ageOn(person.date_of_birth);

            if (gross <= 0) return { amount: 0, basis: "No income entered" };
            if (NOT_WORKING_STATUSES.includes(status)) {
                return { amount: 0, basis: "Not deducted when not employed" };
            }
            if (
                age !== null &&
                (age < settings.minimumAge || age >= settings.pensionableAge)
            ) {
                return { amount: 0, basis: "Not deducted at this age" };
            }

            const selfEmployed = status.includes("self");
            const rate = selfEmployed
                ? settings.selfEmployedRate
                : settings.employeeRate;
            const insurable = Math.min(gross, settings.monthlyInsurableCap);
            const percent = `${Number((rate * 100).toFixed(2))}%`;
            const cap = settings.monthlyInsurableCap.toLocaleString();

            return {
                amount: this.roundMoney(insurable * rate),
                basis:
                    `${percent} of income up to EC$${cap}` +
                    (selfEmployed ? " (self-employed rate)" : ""),
            };
        },

        /** Estimated monthly income tax (PAYE), applied band by band. */
        estimateIncomeTax(person) {
            const gross = Number(this.grossMonthly(person)) || 0;
            let tax = 0;
            let lower = 0;
            for (const band of STATUTORY_DEDUCTIONS.incomeTaxBands) {
                if (gross <= lower) break;
                tax += (Math.min(gross, band.upTo) - lower) * band.rate;
                lower = band.upTo;
            }
            return this.roundMoney(tax);
        },

        /**
         * Estimated NIS, income tax, and net income for one applicant. These
         * are estimates for display and underwriting; the server should
         * recalculate them from the saved gross income.
         */
        statutoryDeductions(person) {
            const gross = Number(this.grossMonthly(person)) || 0;
            const nis = this.estimateNis(person);
            const incomeTax = this.estimateIncomeTax(person);
            return {
                gross,
                nis: nis.amount,
                nisBasis: nis.basis,
                incomeTax,
                net: this.roundMoney(gross - nis.amount - incomeTax),
            };
        },

        /** Label of a lookup value, falling back to the value itself. */
        lookupLabel(field, value) {
            const option = (this.lookups[field] || []).find(
                (entry) => entry.value === value,
            );
            return option ? option.label : value || "";
        },

        /** Parish is only asked for (and required) for addresses in Grenada. */
        isGrenada(country) {
            return (
                !country ||
                String(country).trim().toLowerCase() ===
                    DEFAULT_COUNTRY.toLowerCase()
            );
        },

        /**
         * Today's local date as YYYY-MM-DD, for comparing dates. (toISOString
         * is UTC, which is a day ahead in Grenada after 8pm.)
         */
        today() {
            const now = new Date();
            const pad = (number) => String(number).padStart(2, "0");
            return `${now.getFullYear()}-${pad(now.getMonth() + 1)}-${pad(now.getDate())}`;
        },

        /**
         * An applicant's gross monthly income: their pay per pay period
         * turned into a monthly figure, or the monthly figure saved before
         * pay frequency was asked for.
         */
        grossMonthly(person) {
            if (person.gross_pay !== null && person.gross_pay !== undefined && person.gross_pay !== "") {
                return monthlyAmount(person.gross_pay, person.pay_frequency || "Monthly");
            }
            const saved = person.gross_monthly_income;
            return saved === null || saved === undefined || saved === "" ? null : Number(saved);
        },

        /** What's wrong with one identification row, or "" if it's complete. */
        identificationRowIssue(row) {
            if (!row.identification_type || !row.identification_number) {
                return "Complete the type and number for each identification.";
            }
            if (row.expiry_date && row.expiry_date < this.today()) {
                return "Remove or replace any expired identification.";
            }
            return "";
        },

        /** What's wrong with an applicant's identifications, or "". */
        identificationsIssue(person) {
            const rows = person.identifications || [];
            const minimum = this.minimumIdentifications;
            if (rows.length < minimum) {
                return minimum === 1
                    ? "Add a form of identification."
                    : `Add at least ${minimum} forms of identification.`;
            }
            for (const row of rows) {
                const issue = this.identificationRowIssue(row);
                if (issue) return issue;
            }
            const types = rows.map((row) => row.identification_type);
            if (new Set(types).size !== types.length) {
                return "Each identification must be a different type.";
            }
            if (rows.filter((row) => row.is_primary).length !== 1) {
                return "Mark one identification as primary.";
            }
            return "";
        },

        /**
         * The first thing an applicant still needs to fix, as a sentence,
         * or "" when they're complete (including all three consents).
         */
        applicantIssue(person) {
            for (const part of APPLICANT_PARTS) {
                const issue = this.applicantPartIssue(person, part);
                if (issue) return issue;
            }
            return "";
        },

        /**
         * What's missing from one part (one screen) of an applicant's
         * details, or "". The parts are listed in APPLICANT_PARTS.
         */
        applicantPartIssue(person, part) {
            const blank = (value) => value === null || value === undefined || value === "";

            if (part === "about") {
                if (!person.first_name || !person.last_name) {
                    return "Please enter the first and last name.";
                }
                if (!person.phone) return "Please enter a phone number.";
                if (!person.email) return "Please enter an email address.";
                if (!person.preferred_contact_method) {
                    return "Please choose how we should contact you.";
                }
                if (!person.date_of_birth) return "Please enter the date of birth.";
            }

            if (part === "home") {
                if (!person.address) return "Please enter the home address.";
                if (!person.country) return "Please choose the country.";
                if (this.isGrenada(person.country) && !person.parish) {
                    return "Please choose the parish.";
                }
                if (!person.address_since) {
                    return "When did you move to this address?";
                }
                if (String(person.address_since).slice(0, 10) > this.today()) {
                    return "The date you moved there can't be in the future.";
                }
                if (this.needsPreviousAddress(person) && !person.previous_address) {
                    return "You've lived here under 2 years, so please enter the previous address.";
                }
                if (!person.housing_status) return "Please choose your housing situation.";
                if (blank(person.number_of_dependants)) {
                    return "How many people depend on you? Enter 0 if none.";
                }
            }

            if (part === "membership") {
                if (person.is_member && !person.member_number) {
                    return "Please enter the member number, or untick \"I'm a member\".";
                }
                if (!person.citizenship) return "Please choose the country of citizenship.";
                if (!person.residency_status) return "Please choose the residency status.";
            }

            if (part === "ids") {
                // Students may not have an NIS number yet.
                if (!person.nis_number && !this.isStudent(person)) {
                    return "Please enter the NIS number.";
                }
                const identificationIssue = this.identificationsIssue(person);
                if (identificationIssue) return identificationIssue;
            }

            if (part === "job") {
                if (!person.employment_status) return "Please choose the work situation.";
                if (this.showsEmploymentDetails(person.employment_status)) {
                    if (!person.employment_type) return "Please choose the type of job.";
                    if (!person.employment_start_date) {
                        return "When did the job start? Please enter the start date.";
                    }
                    if (this.needsPreviousEmployment(person) && !person.previous_employer_name) {
                        return "The job started under 2 years ago, so please enter the previous employer.";
                    }
                }
            }

            if (part === "pay") {
                if (blank(person.gross_pay)) {
                    return "Please enter the pay before tax. Enter 0 if none.";
                }
                if (Number(person.gross_pay) > 0 && !person.pay_frequency) {
                    return "How often is the pay received?";
                }
                for (const income of person.incomes || []) {
                    const issue = this.incomeRowIssue(income);
                    if (issue) return issue;
                }
                if (
                    this.isGuarantor(person) &&
                    (!person.guarantee_type || !(Number(person.guarantee_amount) > 0))
                ) {
                    return "Please enter the type and amount of the guarantee.";
                }
            }

            if (part === "declarations") {
                if (person.is_pep && !person.pep_details) {
                    return "Please explain the public official answer.";
                }
                const declaredYes =
                    person.declared_bankruptcy ||
                    person.declared_judgments ||
                    person.declared_arrears ||
                    person.declared_other_applications;
                if (declaredYes && !person.declaration_details) {
                    return "Please explain each \"yes\" answer.";
                }
            }

            if (part === "consent") {
                if (
                    !person.consent_accuracy_confirmation ||
                    !person.consent_credit_check ||
                    !person.consent_data_processing
                ) {
                    return "Please tick all three boxes to continue.";
                }
            }

            return "";
        },

        /** True once a person's details are far enough along to upload documents. */
        readyForUploads(person) {
            return (
                !this.applicantPartIssue(person, "about") &&
                !this.applicantPartIssue(person, "ids")
            );
        },

        /**
         * What's missing from the amount, term, and purpose, or "". Credit
         * cards and overdrafts ask for a limit instead of an amount and term.
         */
        requestMainIssue() {
            const form = this.formData;
            if (this.isRevolving) {
                const limit = Number(form.requested_credit_limit);
                if (!limit) return "Please enter the limit you'd like.";
                if (limit < this.amountMinimum || limit > this.amountMaximum) {
                    return `The limit must be between ${this.money(
                        this.amountMinimum,
                    )} and ${this.money(this.amountMaximum)}.`;
                }
                if (!form.loan_purpose) return "Please tell us what you'll use it for.";
                return "";
            }
            const amount = Number(form.requested_loan_amount);
            const term = Number(form.requested_loan_term);
            if (!amount) return "Please enter how much you'd like to borrow.";
            if (amount < this.amountMinimum || amount > this.amountMaximum) {
                return `The amount must be between ${this.money(
                    this.amountMinimum,
                )} and ${this.money(this.amountMaximum)}.`;
            }
            if (!term) return "Please enter how many months you need to pay it back.";
            if (term < this.termMinimum || term > this.termMaximum) {
                return `The time to pay back must be between ${this.termMinimum} and ${this.termMaximum} months.`;
            }
            if (!form.loan_purpose) return "Please tell us what the loan is for.";
            return "";
        },

        /** What's missing from the vehicle, property, or business details, or "". */
        requestDetailsIssue() {
            const form = this.formData;
            const price = Number(form.purchase_price) || 0;
            const downPayment = Number(form.down_payment_amount) || 0;
            if (price > 0 && downPayment >= price) {
                return "The down payment must be less than the price.";
            }
            if (downPayment > 0 && !form.source_of_funds) {
                return "Where is the down payment coming from?";
            }
            if (
                downPayment > 0 &&
                String(form.source_of_funds).toLowerCase() === "other" &&
                !form.source_of_funds_details
            ) {
                return "Please describe where the down payment is coming from.";
            }
            if (this.loanKind === "organization" && !form.business_name) {
                return "Please enter the business name.";
            }
            if (this.loanKind === "organization" && !form.business_signing_authority) {
                return "Please tell us your position in the business.";
            }
            if (this.loanKind === "credit_card" && !form.name_on_card) {
                return "Please enter the name to print on the card.";
            }
            if (this.loanKind === "overdraft" && !form.linked_account_number) {
                return "Please enter the account the overdraft is for.";
            }
            if (this.isRevolving && form.is_secured_by_savings) {
                const savings = Number(form.secured_savings_amount);
                if (!savings) return "Please enter how much of your savings to hold against it.";
                if (savings > Number(form.requested_credit_limit)) {
                    return "The savings held can't be more than the limit you asked for.";
                }
            }
            if (this.loanKind === "student") {
                if (!form.institution_name) return "Please enter the name of the school.";
                if (!form.program_name) return "Please enter the name of the course.";
                if (!form.expected_graduation_date) {
                    return "Please enter when you expect to finish.";
                }
                if (
                    form.enrollment_start_date &&
                    form.expected_graduation_date <= form.enrollment_start_date
                ) {
                    return "The finish date must be after the start date.";
                }
                if (!Number(form.tuition_amount)) return "Please enter the tuition fees.";
            }
            return "";
        },

        /**
         * What a Third Party Owner still needs, or "". They aren't borrowing,
         * so only their name, relationship, and phone number are required.
         */
        thirdPartyOwnerIssue(person) {
            const named =
                person.kind === "ORGANIZATION"
                    ? person.business_name
                    : person.first_name && person.last_name;
            if (!named) {
                return person.kind === "ORGANIZATION"
                    ? "Please enter the business name."
                    : "Please enter their first and last name.";
            }
            if (!person.relationship_to_applicant) {
                return "Please choose how they're related to you.";
            }
            if (!person.phone) return "Please enter their phone number.";
            if (!person.preferred_contact_method) {
                return "Please choose how we should contact them.";
            }
            return "";
        },

        /** All required applicant fields, IDs, deductions, and consents. */
        validApplicant(person) {
            if (this.isThirdPartyOwner(person)) {
                return !this.thirdPartyOwnerIssue(person);
            }
            return !this.applicantIssue(person);
        },

        /** True when an ApplicationParty ID belongs to a borrower (not a Third Party Owner). */
        isBorrowerLink(applicationPartyId) {
            const id = this.toId(applicationPartyId);
            return Boolean(
                id &&
                    this.applicationPartyOptions.some(
                        (option) => option.value === id,
                    ),
            );
        },

        /**
         * Validates the current step only. Returns an error message string,
         * or an empty string when the step is valid.
         */
        validate() {
            const form = this.formData;

            if (this.current.id === "loan") {
                if (!form.loan_category || !this.toId(form.loan_type_id)) {
                    return "Select a loan category and product.";
                }
            }

            if (this.current.id === "parties") {
                const primaryIssue = this.applicantIssue(form.primary);
                if (primaryIssue) return `Primary applicant: ${primaryIssue}`;

                for (let index = 0; index < form.parties.length; index++) {
                    const person = form.parties[index];
                    const name = this.applicantName(person, index + 1);
                    if (
                        !person.role ||
                        String(person.role).toLowerCase() === "primary applicant"
                    ) {
                        return `${name}: select a role other than Primary Applicant.`;
                    }
                    const issue = this.isThirdPartyOwner(person)
                        ? this.thirdPartyOwnerIssue(person)
                        : this.applicantIssue(person);
                    if (issue) return `${name}: ${issue}`;
                }

                const guarantorIssue = this.guarantorIssue();
                if (guarantorIssue) return guarantorIssue;

                for (const row of form.references || []) {
                    if (!row.name || !row.relationship || !row.phone) {
                        return `${row.reference_type}: enter their name, relationship, and phone number.`;
                    }
                }
            }

            if (this.current.id === "request") {
                const issue = this.requestMainIssue() || this.requestDetailsIssue();
                if (issue) return issue;
            }

            if (this.current.id === "assets") {
                const hasInvalidAsset = form.assets.some(
                    (item) =>
                        !item.name ||
                        !item.asset_type ||
                        item.declared_value === null ||
                        !this.allocationComplete(item.owners) ||
                        item.owners.some((row) => !row.party_id),
                );
                if (hasInvalidAsset) {
                    return "Complete every asset and total ownership to 100%.";
                }

                for (let index = 0; index < form.assets.length; index++) {
                    const asset = form.assets[index];
                    const name = asset.name || `Asset ${index + 1}`;

                    // Someone else's asset is only here to secure the loan.
                    if (this.ownedOnlyByThirdParty(asset) && !this.isCollateral(asset)) {
                        return this.requiresCollateral
                            ? `${name} is owned only by a Third Party Owner, so it can only be on this application as collateral. Mark it as collateral or remove it.`
                            : `${name} is owned only by a Third Party Owner. Remove it, since this loan doesn't use collateral.`;
                    }

                    if (this.isCollateral(asset)) {
                        const issue = this.insuranceIssue(asset.collateral);
                        if (issue) return `${name}: ${issue}`;
                    }
                }

                if (
                    this.requiresCollateral &&
                    !form.assets.some((asset) => this.isCollateral(asset))
                ) {
                    return form.assets.length
                        ? "This loan needs collateral. Mark at least one asset as collateral."
                        : "This loan needs collateral. Add the asset securing it and mark it as collateral.";
                }
            }

            if (this.current.id === "liabilities") {
                const hasUnknownType = form.liabilities.some(
                    (item) =>
                        item.liability_type &&
                        !this.liabilityTypeFor(item.liability_type),
                );
                if (hasUnknownType) {
                    return "Choose a liability type from the list for every liability.";
                }

                const missingLimit = form.liabilities.some(
                    (item) =>
                        this.isRevolvingType(item.liability_type) &&
                        !(Number(item.credit_limit) > 0),
                );
                if (missingLimit) {
                    return "Enter the credit limit for every credit card, overdraft, or line of credit.";
                }

                const hasInvalidLiability = form.liabilities.some(
                    (item) =>
                        !item.creditor_name ||
                        !item.liability_type ||
                        item.outstanding_balance === null ||
                        item.payment_amount === null ||
                        !this.allocationComplete(item.responsibilities) ||
                        item.responsibilities.some(
                            (row) => !this.isBorrowerLink(row.application_party_id),
                        ),
                );
                if (hasInvalidLiability) {
                    return "Complete every liability and total responsibility to 100%.";
                }

                const missingSecurity = form.liabilities.find(
                    (item) => item.is_secured && !this.assetForRef(item.secured_asset_ref),
                );
                if (missingSecurity) {
                    return `${missingSecurity.creditor_name || "A secured liability"}: choose the asset it's secured on (add it on the Assets step if it's missing).`;
                }
            }

            if (this.current.id === "expenses") {
                const hasInapplicableType = form.expenses.some(
                    (item) =>
                        item.expense_type &&
                        !this.expenseTypeAppliesToLoan(
                            this.expenseTypeFor(item.expense_type),
                        ),
                );
                if (hasInapplicableType) {
                    return `Remove or change expenses that don't apply to a ${this.loanCategoryLabel}.`;
                }

                // A shared household expense isn't assigned to one person.
                const hasInvalidExpense = form.expenses.some(
                    (item) =>
                        (!item.is_household &&
                            !this.isBorrowerLink(item.application_party_id)) ||
                        !item.expense_type ||
                        item.amount === null ||
                        item.amount === undefined,
                );
                if (hasInvalidExpense) {
                    return "Complete and assign every expense.";
                }
            }

            // Required documents for the items on this step. Field checks
            // above run first, since items can't receive uploads until
            // they're complete.
            const kindsByStep = {
                parties: ["applicant", "identification", "income"],
                assets: ["asset", "collateral"],
                liabilities: ["liability"],
                expenses: ["expense"],
                documents: ["application"],
            };
            const kinds = kindsByStep[this.current.id];
            const missing = kinds ? this.firstIncompleteScope(kinds) : null;
            if (missing) {
                return missing.kind === "application"
                    ? "Upload every required application document before continuing."
                    : `Upload the required documents for ${missing.label}.`;
            }

            return "";
        },

        /**
         * Goes to the next screen. The current screen is checked first; on
         * a step's last screen the whole step is checked too (validate()),
         * the draft is saved when leaving a step that holds server data, and
         * the wizard moves to the next step.
         */
        async next() {
            const screen = this.currentScreen;
            const lastScreen = this.screenIndex >= this.screens.length - 1;
            let issue = this.screenIssue(screen);
            if (!issue && lastScreen) issue = this.validate();
            if (issue) {
                this.showErrors = true;
                return this.warn(issue);
            }

            this.alert = { text: "", type: "warning" };
            this.showErrors = false;

            // "No" to a Yes/No question clears that section.
            if (screen.kind === "gate" && this.answers[screen.answer] === false) {
                this.clearForNo(screen.answer);
            }

            if (!lastScreen) {
                this.screenIndex++;
                this.scrollToTop();
                return;
            }

            // The vehicle or property being bought becomes a collateral asset.
            if (this.current.id === "request") this.syncPurchaseAsset();

            const persistedSteps = [
                "parties",
                "assets",
                "liabilities",
                "expenses",
            ];
            if (persistedSteps.includes(this.current.id)) {
                this.saving = true;
                try {
                    await this.saveDraft(true);
                } catch (error) {
                    this.saving = false;
                    return; // saveDraft already surfaced an error alert
                }
                this.saving = false;
            }

            this.setStep(this.step + 1);
            if (this.formData.id) this.rememberDraftLocation();
        },

        /** Goes back one screen (to the last screen of the previous step). */
        back() {
            this.alert = { text: "", type: "warning" };
            if (this.screenIndex > 0) {
                this.screenIndex--;
                this.showErrors = false;
                this.scrollToTop();
                return;
            }
            if (this.step > 0) {
                const previous = this.path[this.step - 1];
                this.setStep(this.step - 1, this.screensFor(previous.id).length - 1);
            }
        },

        // ---- Short screens ----

        /**
         * The short screens that make up one step, in order. Each is
         * { id, kind, title, help } plus, depending on the kind, the part of
         * a person's details (part, personKey) or the Yes/No answer it sets
         * (answer). Yes/No questions ("gate" screens) skip the section after
         * them when the answer is No.
         */
        screensFor(stepId) {
            const form = this.formData;

            if (stepId === "loan") {
                // A welcome screen first, until a draft has been saved.
                const welcome = form.id
                    ? []
                    : [{ id: "welcome", kind: "welcome", title: "", help: "" }];
                return welcome.concat([
                    {
                        id: "loan",
                        kind: "loan",
                        title: "What kind of loan do you need?",
                        help: "Choose the type of loan, then the loan that fits you best.",
                    },
                ]);
            }

            if (stepId === "parties") {
                let screens = this.personScreens(form.primary, true).concat([
                    {
                        id: "others-gate",
                        kind: "gate",
                        answer: "others",
                        title: "Is anyone else on this loan?",
                        help: this.requiresGuarantor
                            ? "This loan needs a guarantor, so please answer Yes and add them. You can also add someone borrowing with you, or someone who co-owns what secures it."
                            : "For example, someone borrowing with you, someone guaranteeing the loan, or someone who co-owns what secures it.",
                    },
                ]);
                if (this.answers.others) {
                    screens.push({
                        id: "people",
                        kind: "people",
                        title: "Who else is on this loan?",
                        help: "Add each person and choose how they're involved. You'll fill in their details next.",
                    });
                    form.parties.forEach((person) => {
                        screens = screens.concat(this.personScreens(person, false));
                    });
                }
                screens.push({
                    id: "references",
                    kind: "references",
                    title: "People who know you",
                    help: "Someone we can contact about you, and your next of kin. They shouldn't be applying with you.",
                });
                return screens;
            }

            if (stepId === "request") {
                const screens = [
                    {
                        id: "request-main",
                        kind: "request",
                        part: "main",
                        title: this.isRevolving
                            ? this.loanKind === "overdraft"
                                ? "Your overdraft"
                                : "Your credit card"
                            : "Your loan",
                        help: this.isRevolving
                            ? "The limit you'd like, and what you'll use it for."
                            : "How much you need, how long to pay it back, and what it's for.",
                    },
                ];
                const details = {
                    automotive: ["The vehicle", "Tell us about the vehicle and who you're buying it from."],
                    property: ["The property", "Tell us about the property and who you're buying it from."],
                    organization: ["Your business", "Tell us about the business the loan is for."],
                    credit_card: ["Your card", "A few details for your card."],
                    overdraft: ["Your overdraft", "Which account it's for, and how it's secured."],
                    student: ["Your studies", "Tell us about your school, your course, and the costs."],
                };
                const extra = details[this.loanKind];
                if (extra) {
                    screens.push({
                        id: "request-details",
                        kind: "request",
                        part: "details",
                        title: extra[0],
                        help: extra[1],
                    });
                }
                return screens;
            }

            if (stepId === "assets") {
                const list = {
                    id: "assets",
                    kind: "assets",
                    title: "Things you own",
                    help: this.requiresCollateral
                        ? "This loan needs something valuable to secure it, like the vehicle or property you're buying."
                        : "Add each valuable thing you own, like a house, land, a vehicle, or savings.",
                };
                // A secured loan always needs an asset, so there's no question.
                if (this.requiresCollateral) return [list];
                const gate = {
                    id: "assets-gate",
                    kind: "gate",
                    answer: "assets",
                    title: "Do you own a house, land, a vehicle, or savings?",
                    help: "Things you own help show you can repay the loan.",
                };
                return this.answers.assets ? [gate, list] : [gate];
            }

            if (stepId === "liabilities") {
                const gate = {
                    id: "debts-gate",
                    kind: "gate",
                    answer: "debts",
                    title: "Do you owe money on any loans or cards?",
                    help: "For example a car loan, mortgage, credit card, overdraft, or hire purchase.",
                };
                const list = {
                    id: "liabilities",
                    kind: "liabilities",
                    title: "Money you owe",
                    help: "Add each loan or card, how much is left to pay, and what you pay.",
                };
                return this.answers.debts ? [gate, list] : [gate];
            }

            if (stepId === "expenses") {
                const gate = {
                    id: "bills-gate",
                    kind: "gate",
                    answer: "bills",
                    title: "Do you pay regular bills or living costs?",
                    help: "For example rent, electricity, water, phone, groceries, or school fees.",
                };
                const list = {
                    id: "expenses",
                    kind: "expenses",
                    title: "Your bills",
                    help: "Add each regular bill and how much you pay.",
                };
                return this.answers.bills ? [gate, list] : [gate];
            }

            if (stepId === "documents") {
                return [
                    {
                        id: "documents",
                        kind: "documents",
                        title: "Upload your documents",
                        help: "Take a clear photo or upload a file for each one.",
                    },
                ];
            }

            return [
                {
                    id: "review",
                    kind: "review",
                    title: "Check your answers",
                    help: "Make sure everything is right. Tap \"Change\" to fix anything, then send your application.",
                },
            ];
        },

        /**
         * The screens for one person's details. A Third Party Owner has one
         * short screen; everyone else has the full set.
         */
        personScreens(person, isPrimary) {
            const key = person.client_key;
            const name = this.fullName(person) || "this person";
            if (!isPrimary && this.isThirdPartyOwner(person)) {
                return [
                    {
                        id: `person-${key}-owner`,
                        kind: "person",
                        part: "owner",
                        personKey: key,
                        title: `About ${name}`,
                        help: "They co-own something that secures the loan, but aren't borrowing. We only need a few details.",
                    },
                ];
            }
            const titles = isPrimary
                ? {
                      about: ["About you", "Your name and how to contact you."],
                      home: ["Where you live", "Your home address and who lives with you."],
                      membership: ["Membership and citizenship", "Are you a member of the credit union, and where are you a citizen?"],
                      ids: ["Your NIS number and ID", "Your NIS number is on your NIS card. Then add a photo ID, like a passport or driver's licence."],
                      job: ["Your work", "Tell us about your job, or choose unemployed, retired, or student."],
                      pay: ["Your pay", "What you earn before tax, and any other money you receive."],
                      declarations: ["A few questions", "Please answer honestly. A \"yes\" won't stop your application."],
                      consent: ["Your agreement", "Please read and tick each box."],
                  }
                : {
                      about: [`About ${name}`, "Their name and how to contact them."],
                      home: [`Where ${name} lives`, "Their home address and who lives with them."],
                      membership: [`${name}: membership and citizenship`, ""],
                      ids: [`${name}: NIS number and ID`, "Their NIS number, and a photo ID."],
                      job: [`${name}'s work`, ""],
                      pay: [`${name}'s pay`, "What they earn before tax, and any other money they receive."],
                      declarations: [`A few questions for ${name}`, "Please answer honestly."],
                      consent: [`${name}'s agreement`, "They need to read and agree to these."],
                  };
            return Object.keys(titles).map((part) => ({
                id: `person-${key}-${part}`,
                kind: "person",
                part,
                personKey: key,
                title: titles[part][0],
                help: titles[part][1],
            }));
        },

        /** Records the answer to a Yes/No question. */
        answer(key, value) {
            this.answers[key] = value;
            this.alert = { text: "", type: "warning" };
        },

        /** What answering "No" will remove, if anything was added. */
        gateWarning(key) {
            const form = this.formData;
            const count = {
                others: form.parties.length,
                assets: form.assets.filter((asset) => !asset.is_purchase).length,
                debts: form.liabilities.length,
                bills: form.expenses.length,
            }[key];
            if (!count) return "";
            return `Choosing "No" will remove the ${count} ${count === 1 ? "entry" : "entries"} you added here.`;
        },

        /** Clears a section after its Yes/No question is answered "No". */
        clearForNo(key) {
            const form = this.formData;
            if (key === "others") {
                for (let index = form.parties.length - 1; index >= 0; index--) {
                    this.removeApplicant(index);
                }
                this.alert = { text: "", type: "warning" };
            }
            if (key === "assets") {
                form.assets = form.assets.filter((asset) => asset.is_purchase);
            }
            if (key === "debts") form.liabilities = [];
            if (key === "bills") form.expenses = [];
        },

        /** Replaces one person's details with an edited copy. */
        updatePerson(key, value) {
            const form = this.formData;
            if (form.primary.client_key === key) {
                form.primary = value;
                return;
            }
            const index = form.parties.findIndex((person) => person.client_key === key);
            if (index >= 0) form.parties.splice(index, 1, value);
        },

        /**
         * What's missing on one screen, or "". The last screen of a step is
         * also checked with validate(), which covers the whole step.
         */
        screenIssue(screen) {
            if (screen.kind === "gate") {
                if (
                    this.answers[screen.answer] === null ||
                    this.answers[screen.answer] === undefined
                ) {
                    return "Please choose Yes or No.";
                }
                if (screen.answer === "others" && !this.answers.others) {
                    return this.guarantorIssue();
                }
                return "";
            }
            if (screen.kind === "person") {
                const person = this.allApplicants.find(
                    (entry) => entry.client_key === screen.personKey,
                );
                if (!person) return "";
                const issue =
                    screen.part === "owner"
                        ? this.thirdPartyOwnerIssue(person)
                        : this.applicantPartIssue(person, screen.part);
                if (issue) return issue;
                return this.personDocumentsIssue(person, screen.part);
            }
            if (screen.kind === "people") {
                if (!this.formData.parties.length) {
                    return "Add each person, or go back and answer \"No\".";
                }
                const missingRole = this.formData.parties.some(
                    (person) =>
                        !person.role ||
                        String(person.role).toLowerCase() === "primary applicant",
                );
                if (missingRole) return "Choose how each person is involved.";
                const guarantorIssue = this.guarantorIssue();
                if (guarantorIssue) return guarantorIssue;
            }
            if (screen.kind === "request") {
                return screen.part === "main"
                    ? this.requestMainIssue()
                    : this.requestDetailsIssue();
            }
            return "";
        },

        /**
         * Required documents still to upload on a person's screen: their
         * applicant documents and ID scans on the ID screen, and proof of
         * other income on the pay screen.
         */
        personDocumentsIssue(person, part) {
            let keys = [];
            if (part === "ids") {
                keys = [`applicant:${person.client_key}`].concat(
                    (person.identifications || []).map(
                        (row) => `identification:${row.client_key}`,
                    ),
                );
            }
            if (part === "pay") {
                keys = (person.incomes || []).map((row) => `income:${row.client_key}`);
            }
            if (!keys.length || this.testMode) return "";
            const missing = this.documentScopeList.find(
                (scope) => keys.includes(scope.key) && !this.scopeComplete(scope),
            );
            return missing ? `Please upload the documents for ${missing.label}.` : "";
        },

        // ---- Persistence payloads ----

        /**
         * Builds the Application resource payload. Only fields relevant to the
         * chosen kind of loan (vehicle, property, card, overdraft, studies) are
         * included. The business on a business loan is saved separately as a
         * "Business Borrower" ApplicationParty (see saveBusinessParty).
         */
        appPayload() {
            const form = this.formData;

            // application_number is deliberately not sent: the submit
            // workflow assigns it, and sending it would overwrite it.
            const payload = {
                status: form.status || "draft",
                loan_type_id: this.toId(form.loan_type_id),
                loan_purpose: form.loan_purpose,
                documents: form.document_ids.map(this.toId).filter(Boolean),
            };

            // Credit cards and overdrafts have a limit, not an amount and term.
            // (Application.interest_rate and interest_rate_type are staff
            // overrides, so the form never sends them.)
            if (this.isRevolving) {
                Object.assign(payload, {
                    requested_credit_limit: form.requested_credit_limit,
                    is_secured_by_savings: !!form.is_secured_by_savings,
                    secured_savings_amount: form.is_secured_by_savings
                        ? form.secured_savings_amount
                        : null,
                    linked_account_number: form.linked_account_number,
                });
                if (this.loanKind === "credit_card") {
                    Object.assign(payload, {
                        name_on_card: form.name_on_card,
                        card_collection_method: form.card_collection_method,
                    });
                }
            } else {
                Object.assign(payload, {
                    requested_loan_amount: form.requested_loan_amount,
                    requested_loan_term: form.requested_loan_term,
                    repayment_frequency: form.repayment_frequency,
                });
            }
            if (this.loanKind === "student") {
                Object.assign(payload, {
                    institution_name: form.institution_name,
                    institution_country: form.institution_country,
                    program_name: form.program_name,
                    program_level: form.program_level,
                    student_id_number: form.student_id_number,
                    enrollment_start_date: form.enrollment_start_date,
                    expected_graduation_date: form.expected_graduation_date,
                    tuition_amount: form.tuition_amount,
                    tuition_currency: form.tuition_currency,
                    other_study_costs: form.other_study_costs,
                    disbursement_schedule: form.disbursement_schedule,
                    grace_period_months: form.grace_period_months,
                    pay_to_institution: !!form.pay_to_institution,
                });
            }
            if (this.loanKind === "automotive") {
                Object.assign(payload, {
                    vehicle_make: form.vehicle_make,
                    vehicle_model: form.vehicle_model,
                    vehicle_year: form.vehicle_year,
                    vehicle_condition: form.vehicle_condition,
                });
            }
            if (this.loanKind === "property") {
                Object.assign(payload, {
                    property_address: form.property_address,
                    property_type: form.property_type,
                    property_value: form.property_value,
                });
            }
            // Purchase details (auto and home). The down payment's source is
            // an anti-money-laundering question.
            if (this.loanKind === "automotive" || this.loanKind === "property") {
                const hasDownPayment = Number(form.down_payment_amount) > 0;
                Object.assign(payload, {
                    purchase_price: form.purchase_price,
                    down_payment_amount: form.down_payment_amount,
                    source_of_funds: hasDownPayment ? form.source_of_funds : "",
                    source_of_funds_details: hasDownPayment
                        ? form.source_of_funds_details
                        : "",
                    seller_type: form.seller_type,
                    seller_name: form.seller_name,
                });
            }
            return payload;
        },

        /**
         * Saves the business on a business loan as its own Party (kind
         * ORGANIZATION), joined to the application by an ApplicationParty
         * with the role "Business Borrower" (a Party is never linked straight
         * to the Application). Returns that ApplicationParty's ID, or null.
         * Party records are shared, so the business is never deleted by the
         * form.
         */
        async saveBusinessParty() {
            const form = this.formData;
            if (this.loanKind !== "organization" || !form.business_name) {
                return null;
            }
            form.business_party_id = await this.upsert("Party", form.business_party_id, {
                kind: "ORGANIZATION",
                legal_name: form.business_name,
                trading_name: form.business_trading_name,
                industry: form.business_industry,
                business_license_number: form.business_license_number,
                website: form.business_website,
                registration_number: form.business_registration_number,
                business_type: form.business_type,
                incorporation_date: form.business_incorporation_date || null,
                number_of_employees: form.business_employee_count,
            });
            const payload = {
                party: this.toId(form.business_party_id),
                role: BUSINESS_BORROWER_ROLE,
                relationship_status: "Active",
                // The business owes the money, so it holds the whole loan.
                ownership_percentage: 100,
                is_primary_contact: false,
                // The business's figures when it applied (the Party keeps
                // the latest employee count too).
                number_of_employees: form.business_employee_count,
                annual_revenue: form.business_annual_revenue,
            };
            const isNew = !this.toId(form.business_application_party_id);
            form.business_application_party_id = await this.upsert(
                "ApplicationParty",
                form.business_application_party_id,
                payload,
            );
            // Saturn fills ApplicationParty.partysnapshot when a link is
            // updated, not when it's created. People's links are always
            // saved a second time (their income list); the business's link
            // has nothing else to save, so it's saved again here.
            if (isNew) {
                await this.upsert(
                    "ApplicationParty",
                    form.business_application_party_id,
                    payload,
                );
            }
            return form.business_application_party_id;
        },

        /** Identity data stored on the Party resource (shared across applications). */
        partyPayload(person) {
            // Third Party Owners give only their name and contact details.
            // Other Party fields are left as they are, not blanked.
            if (this.isThirdPartyOwner(person)) {
                const isBusiness = person.kind === "ORGANIZATION";
                return {
                    kind: isBusiness ? "ORGANIZATION" : "PERSON",
                    first_name: isBusiness ? "" : person.first_name,
                    last_name: isBusiness ? "" : person.last_name,
                    legal_name: isBusiness ? person.business_name : "",
                    email: person.email,
                    phone: person.phone,
                };
            }

            return {
                kind: "PERSON",
                first_name: person.first_name,
                last_name: person.last_name,
                date_of_birth: person.date_of_birth,
                marital_status: person.marital_status,
                email: person.email,
                phone: person.phone,
                address: person.address,
                parish: this.isGrenada(person.country) ? person.parish : "",
                country: person.country,
                nis_number: person.nis_number,
                is_member: Boolean(person.is_member),
                member_number: person.is_member ? person.member_number : "",
                citizenship: person.citizenship,
                residency_status: person.residency_status,
                tin: person.tin,
                address_since: person.address_since || null,
                previous_address: this.needsPreviousAddress(person)
                    ? person.previous_address
                    : "",
                previous_parish:
                    this.needsPreviousAddress(person) &&
                    this.isGrenada(person.previous_country)
                        ? person.previous_parish
                        : "",
                previous_country: this.needsPreviousAddress(person)
                    ? person.previous_country
                    : "",
                mailing_address: person.mailing_address,
                // ids are set after the identifications are saved, since a
                // PartyIdentification can't exist before its Party.
            };
        },

        /** One PartyIdentification record, linked to its owning Party. */
        identificationPayload(row, partyId) {
            return {
                party: this.toId(partyId),
                identification_type: row.identification_type,
                identification_number: row.identification_number,
                issuing_country: row.issuing_country,
                issue_date: row.issue_date || null,
                expiry_date: row.expiry_date || null,
                is_primary: Boolean(row.is_primary),
                documents: this.documentIdsOf(row),
            };
        },

        /**
         * Employment, consent, and per-applicant document data stored on the
         * ApplicationParty link record. (No longer dependent on Application ID).
         */
        /**
         * A person's share of the loan (ApplicationParty.ownership_percentage).
         * The borrowers (the primary applicant and any co-borrowers) share it
         * evenly, and the shares always add up to 100. Guarantors and Third
         * Party Owners aren't borrowing, so they get 0. On a business loan the
         * Business Borrower holds the whole loan, so every person gets 0.
         */
        loanShareOf(person, isPrimary) {
            if (this.loanKind === "organization") return 0;
            if (!isPrimary && (this.isGuarantor(person) || this.isThirdPartyOwner(person))) {
                return 0;
            }
            const borrowers = [this.formData.primary].concat(
                this.formData.parties.filter(
                    (other) => !this.isGuarantor(other) && !this.isThirdPartyOwner(other),
                ),
            );
            const count = borrowers.length || 1;
            const even = Math.floor((100 / count) * 100) / 100;
            // The primary applicant takes any rounding left over (e.g. 33.34).
            if (isPrimary) return Math.round((100 - even * (count - 1)) * 100) / 100;
            return even;
        },

        applicationPartyPayload(person, isPrimary) {
            // Third Party Owners aren't borrowing: no employment, income,
            // deductions, or consents are asked for or kept.
            if (!isPrimary && this.isThirdPartyOwner(person)) {
                return {
                    party: this.toId(person.party_id),
                    role: THIRD_PARTY_OWNER_ROLE,
                    relationship_status: "Active",
                    ownership_percentage: 0,
                    is_primary_contact: false,
                    relationship_to_applicant: person.relationship_to_applicant,
                    preferred_contact_method: person.preferred_contact_method,
                    employment_status: "",
                    employer_name: "",
                    job_title: "",
                    years_employed: null,
                    gross_monthly_income: null,
                    nis_deduction: null,
                    income_tax_deduction: null,
                    consent_accuracy_confirmation: false,
                    consent_credit_check: false,
                    consent_data_processing: false,
                    consented_at: null,
                    consent_policy_version: "",
                    documents: this.documentIdsOf(person),
                };
            }

            const allConsentsGiven =
                person.consent_accuracy_confirmation &&
                person.consent_credit_check &&
                person.consent_data_processing;

            const showEmployment = this.showsEmploymentDetails(
                person.employment_status,
            );
            const showPrevious = this.needsPreviousEmployment(person);
            const guarantor = !isPrimary && this.isGuarantor(person);

            return {
                party: this.toId(person.party_id),
                role: isPrimary ? "Primary Applicant" : person.role,
                relationship_status: "Active",
                // This person's share of the loan (see loanShareOf).
                ownership_percentage: this.loanShareOf(person, isPrimary),
                is_primary_contact: isPrimary,
                preferred_contact_method: person.preferred_contact_method,
                // Business loans: the Primary Applicant's position in the business.
                signing_authority:
                    isPrimary && this.loanKind === "organization"
                        ? this.formData.business_signing_authority
                        : "",
                relationship_to_applicant: "",
                housing_status: person.housing_status,
                number_of_dependants: person.number_of_dependants,
                employment_status: person.employment_status,
                employment_type: showEmployment ? person.employment_type : "",
                employment_start_date: showEmployment
                    ? person.employment_start_date || null
                    : null,
                employer_name: showEmployment ? person.employer_name : "",
                job_title: showEmployment ? person.job_title : "",
                // Worked out from the start date.
                years_employed: showEmployment
                    ? this.yearsSince(person.employment_start_date)
                    : null,
                previous_employer_name: showPrevious ? person.previous_employer_name : "",
                previous_job_title: showPrevious ? person.previous_job_title : "",
                previous_employment_years: showPrevious
                    ? person.previous_employment_years
                    : null,
                gross_pay: person.gross_pay,
                pay_frequency: person.pay_frequency,
                // Worked out from gross pay and pay frequency.
                gross_monthly_income: this.grossMonthly(person),
                annual_revenue: person.annual_revenue,
                guarantee_type: guarantor ? person.guarantee_type : "",
                guarantee_amount: guarantor ? person.guarantee_amount : null,
                is_pep: Boolean(person.is_pep),
                pep_details: person.is_pep ? person.pep_details : "",
                declared_bankruptcy: Boolean(person.declared_bankruptcy),
                declared_judgments: Boolean(person.declared_judgments),
                declared_arrears: Boolean(person.declared_arrears),
                declared_other_applications: Boolean(person.declared_other_applications),
                declaration_details: person.declaration_details,
                // Calculated from gross monthly income, not entered.
                nis_deduction: this.estimateNis(person).amount,
                income_tax_deduction: this.estimateIncomeTax(person),
                consent_accuracy_confirmation:
                    person.consent_accuracy_confirmation,
                consent_credit_check: person.consent_credit_check,
                consent_data_processing: person.consent_data_processing,
                consented_at: allConsentsGiven
                    ? person.consented_at || new Date().toISOString()
                    : null,
                consent_policy_version: allConsentsGiven
                    ? person.consent_policy_version || "2026-09"
                    : "",
                documents: (person.document_ids || [])
                    .map(this.toId)
                    .filter(Boolean),
            };
        },

        /**
         * Creates or updates a resource record and returns its ID. A failure
         * is logged to the console with the resource, the data sent, and
         * the server's reply, and rethrown with the resource's name, so a
         * failed save says which record it was.
         */
        async upsert(resourceName, id, fullPayload) {
            const resource = new Resource(this, resourceName);
            const action = id ? "updating" : "creating";
            const payload = this.withoutEmptyValues(fullPayload);

            try {
                if (id) {
                    await resource.update(this.toId(id), payload);
                    return this.toId(id);
                }

                const result = await resource.create(payload);
                const createdId = this.createdIdOf(result);
                if (!createdId) {
                    // Saturn can reply with a failure instead of throwing,
                    // e.g. { status: "FAILURE", message: ... }.
                    console.error(
                        `[Loan form] ${resourceName} was not created. Saturn replied:`,
                        result,
                    );
                    const failed =
                        String(result?.status || "").toUpperCase() === "FAILURE";
                    if (failed) {
                        throw Error(
                            `Saturn refused it: ${this.errorText({ data: result })}`,
                        );
                    }
                    // No reply at all: Saturn didn't send the request, which
                    // happens when it doesn't know the resource name or the
                    // user isn't allowed to create it.
                    if (result === undefined || result === null) {
                        throw Error(
                            `Saturn didn't send the request. Check that a resource named exactly "${resourceName}" exists and is published, and that your role can create ${resourceName} records.`,
                        );
                    }
                    throw Error(
                        `the server didn't return a record ID (reply: ${JSON.stringify(result).slice(0, 300)})`,
                    );
                }
                return createdId;
            } catch (error) {
                console.error(
                    `[Loan form] Failed ${action} ${resourceName}`,
                    { id: this.toId(id), payload, error },
                );
                const failure = Error(
                    `${resourceName} (${action}): ${this.errorText(error)}`,
                );
                failure.cause = error;
                throw failure;
            }
        },

        /**
         * A copy of a payload with every empty value left out: null,
         * undefined, "" (an empty text box or unchosen dropdown), and numbers
         * that aren't numbers (NaN). Saturn only accepts values of the
         * field's own type, and rejects null. Lists (even empty ones),
         * true/false, and 0 are kept. On an update, a field left out keeps
         * whatever was saved before, so a value can't be cleared this way.
         */
        withoutEmptyValues(payload) {
            const result = {};
            Object.keys(payload || {}).forEach((field) => {
                const value = payload[field];
                if (value === null || value === undefined || value === "") return;
                if (typeof value === "number" && Number.isNaN(value)) return;
                result[field] = value;
            });
            return result;
        },

        /**
         * The ID of a record Saturn just created. Replies come in a few
         * shapes, so this checks the usual places: id, data.id,
         * data.data.id, record.id, resource.id, result.id, and _id.
         */
        createdIdOf(result) {
            if (!result) return null;
            if (typeof result !== "object") return result;
            const candidates = [
                result,
                result.data,
                result.data?.data,
                result.record,
                result.resource,
                result.result,
            ];
            for (const candidate of candidates) {
                if (!candidate || typeof candidate !== "object") continue;
                const id = candidate.id || candidate._id;
                if (id && typeof id !== "object") return id;
            }
            return null;
        },

        /** The most useful message in an error, including the server's reply. */
        errorText(error) {
            if (!error) return "unknown error";
            const reply = error.response?.data ?? error.data;
            if (reply) {
                const message = reply.message ?? reply.error ?? reply;
                const text =
                    typeof message === "string" ? message : JSON.stringify(message);
                if (text && text !== "{}") return text;
                // An empty message: show the whole reply instead.
                return JSON.stringify(reply).slice(0, 300);
            }
            return error.message || JSON.stringify(error).slice(0, 300);
        },

        /**
         * Persists one applicant: their identifications, then the Party
         * record (linking the identifications through Party.ids), then the
         * ApplicationParty link. Identifications the applicant removed are
         * deleted afterwards. Mutates the person object with server IDs.
         */
        async saveApplicant(person, isPrimary) {
            // The Party is saved first: PartyIdentification requires the
            // party it belongs to, so it can't be created before the Party
            // exists. Party.ids is then filled in once the IDs are saved.
            person.party_id = await this.upsert(
                "Party",
                person.party_id,
                this.partyPayload(person),
            );

            // Third Party Owners give no identification, so their IDs are
            // skipped (any saved earlier under another role are left alone).
            if (!isPrimary && this.isThirdPartyOwner(person)) {
                return this.saveApplicationParty(person, isPrimary);
            }

            for (const row of person.identifications || []) {
                row.id = await this.upsert(
                    "PartyIdentification",
                    row.id,
                    this.identificationPayload(row, person.party_id),
                );
            }

            // Link the identifications back onto the Party.
            await this.upsert("Party", person.party_id, {
                ids: (person.identifications || [])
                    .map((row) => this.toId(row.id))
                    .filter(Boolean),
            });

            // Delete identifications removed since the last save. Party
            // records are shared across applications, so only IDs the
            // applicant removed here are deleted; failures are retried.
            const currentIds = (person.identifications || [])
                .map((row) => this.toId(row.id))
                .filter(Boolean);
            const failedIds = [];
            for (const id of person.saved_identification_ids || []) {
                if (currentIds.includes(id)) continue;
                try {
                    await new Resource(this, "PartyIdentification").delete(id);
                } catch (error) {
                    const status = error?.response?.status || error?.status;
                    if (status !== 404) failedIds.push(id);
                }
            }
            person.saved_identification_ids = currentIds.concat(failedIds);

            return this.saveApplicationParty(person, isPrimary);
        },

        /** Saves the ApplicationParty link for one applicant and returns its ID. */
        async saveApplicationParty(person, isPrimary) {
            const applicationPartyPayload = this.applicationPartyPayload(
                person,
                isPrimary,
            );
            person.application_party_id = await this.upsert(
                "ApplicationParty",
                person.application_party_id,
                applicationPartyPayload,
            );

            // Keep the local consent metadata in sync with what was saved.
            person.consented_at = applicationPartyPayload.consented_at;
            person.consent_policy_version =
                applicationPartyPayload.consent_policy_version;

            // Other income hangs off the ApplicationParty, so it's saved after.
            await this.saveIncomes(person, isPrimary);

            return person.application_party_id;
        },

        /**
         * Saves an applicant's other income as IncomeSource records and links
         * them through ApplicationParty.income_ids. Third Party Owners aren't
         * borrowing, so they have none. Removed rows are deleted by
         * deleteStaleRecords().
         */
        async saveIncomes(person, isPrimary) {
            const incomes =
                !isPrimary && this.isThirdPartyOwner(person) ? [] : person.incomes || [];
            for (const income of incomes) {
                income.id = await this.upsert("IncomeSource", income.id, {
                    application_party: this.toId(person.application_party_id),
                    income_type: income.income_type,
                    description: income.description,
                    amount: income.amount,
                    frequency: income.frequency,
                    monthly_equivalent: monthlyAmount(income.amount, income.frequency),
                    documents: this.documentIdsOf(income),
                });
            }
            await this.upsert(
                "ApplicationParty",
                person.application_party_id,
                {
                    income_ids: incomes
                        .map((income) => this.toId(income.id))
                        .filter(Boolean),
                },
            );
        },

        /**
         * Saves the references (personal reference and next of kin) as
         * Reference records linked to the application, and returns their IDs.
         */
        async saveReferences() {
            const ids = [];
            for (const row of this.formData.references || []) {
                if (!row.name) continue;
                row.id = await this.upsert("Reference", row.id, {
                    application: this.toId(this.formData.id),
                    reference_type: row.reference_type,
                    name: row.name,
                    relationship: row.relationship,
                    phone: row.phone,
                    email: row.email,
                    address: row.address,
                });
                ids.push(row.id);
            }
            return ids;
        },

        /**
         * Saves one liability responsibility row, then re-reads it to confirm
         * the server stored the values we sent.
         */
        async persistLiabilityResponsibility(liabilityId, row) {
            const payload = {
                liability: this.toId(liabilityId),
                application_party: this.toId(row.application_party_id),
                responsibility_percentage: Number(row.percentage),
                responsibility_type: row.responsibility_type || "Borrower",
                status: "Active",
            };

            row.id = await this.upsert(
                "LiabilityResponsibility",
                row.id,
                payload,
            );

            const saved = this.recordOf(
                await new Resource(this, "LiabilityResponsibility").get(row.id),
            );
            const verified =
                saved &&
                this.toId(saved.liability) === payload.liability &&
                this.toId(saved.application_party) ===
                    payload.application_party &&
                Number(saved.responsibility_percentage) ===
                    payload.responsibility_percentage;

            if (!verified) {
                throw Error(
                    "Liability responsibility could not be verified after saving.",
                );
            }
        },

        /** Server IDs of the documents linked to a form record. */
        documentIdsOf(record) {
            return (record.document_ids || []).map(this.toId).filter(Boolean);
        },

        /**
         * Saves one asset and its ownership rows. Rows without an owner yet
         * are skipped; they're saved once assigned.
         */
        async saveAsset(asset) {
            // Identifiers only apply to their kind of asset.
            const vehicle = this.isVehicleType(asset.asset_type);
            const property = this.isPropertyType(asset.asset_type);
            asset.id = await this.upsert("Asset", asset.id, {
                name: asset.name,
                asset_type: asset.asset_type,
                description: asset.description,
                declared_value: asset.declared_value,
                registration_number: vehicle ? asset.registration_number || "" : "",
                chassis_number: vehicle ? asset.chassis_number || "" : "",
                block_and_parcel: property ? asset.block_and_parcel || "" : "",
                deed_number: property ? asset.deed_number || "" : "",
                // The vehicle or property a purchase loan is buying.
                status: asset.is_purchase ? PURCHASE_ASSET_STATUS : "Declared",
                documents: this.documentIdsOf(asset),
            });

            for (const owner of asset.owners || []) {
                if (!owner.party_id) continue;
                owner.id = await this.upsert("AssetOwnership", owner.id, {
                    asset: asset.id,
                    party: this.toId(owner.party_id),
                    ownership_percentage: Number(owner.percentage),
                });
            }

            return asset.id;
        },

        /**
         * Saves one liability and its responsibility rows. Rows without a
         * responsible applicant yet are skipped.
         */
        async saveLiability(liability) {
            const revolving = this.isRevolvingType(liability.liability_type);

            liability.id = await this.upsert("Liability", liability.id, {
                creditor_name: liability.creditor_name,
                liability_type: this.toId(liability.liability_type),
                outstanding_balance: liability.outstanding_balance,
                payment_amount: liability.payment_amount,
                payment_frequency: liability.payment_frequency,
                // Revolving credit only. The server should recalculate
                // assessed_payment before it's used for underwriting.
                credit_limit: revolving ? liability.credit_limit : null,
                assessed_payment: revolving
                    ? this.assessedPayment(liability)
                    : null,
                monthly_equivalent: this.liabilityMonthlyPayment(liability),
                // The balance is as of the date the application is filled in.
                balance_as_of: this.today(),
                description: liability.description,
                is_to_be_paid_off: Boolean(liability.is_to_be_paid_off),
                is_secured: Boolean(liability.is_secured),
                secured_asset: liability.is_secured
                    ? this.toId(this.assetForRef(liability.secured_asset_ref)?.id)
                    : null,
                documents: this.documentIdsOf(liability),
            });

            for (const responsibility of liability.responsibilities || []) {
                if (!responsibility.application_party_id) continue;
                await this.persistLiabilityResponsibility(
                    liability.id,
                    responsibility,
                );
            }

            return liability.id;
        },

        async saveExpense(expense) {
            // A shared household expense is linked to the primary applicant
            // (so it can be found again) and marked is_household.
            const linkId = expense.is_household
                ? this.toId(this.formData.primary.application_party_id)
                : this.toId(expense.application_party_id);
            expense.id = await this.upsert("Expense", expense.id, {
                applicationpartiesid: linkId,
                is_household: Boolean(expense.is_household),
                expense_name:
                    expense.expense_name ||
                    this.expenseTypeName(expense.expense_type),
                expense_type: this.toId(expense.expense_type),
                amount: expense.amount,
                frequency: expense.frequency,
                monthly_equivalent: monthlyAmount(expense.amount, expense.frequency),
                is_projected: false,
                status: "Declared",
                documents: this.documentIdsOf(expense),
            });
            return expense.id;
        },

        /**
         * Saves the projected monthly insurance cost of a collateral asset as
         * an Expense marked is_projected, linked to its Collateral record.
         * Uses the COLLATERAL_INSURANCE expense type when it's set up in
         * Saturn. Returns the Expense ID, or null when there's no premium
         * (a projected expense saved earlier is then deleted on this save).
         */
        async saveProjectedInsurance(asset) {
            const collateral = asset.collateral;
            const projected = this.projectedInsuranceExpenses.find(
                (item) => item.asset_key === asset.client_key,
            );
            if (!projected) {
                collateral.projected_expense_id = null;
                return null;
            }
            const type = this.collateralInsuranceType;
            collateral.projected_expense_id = await this.upsert(
                "Expense",
                collateral.projected_expense_id,
                {
                    applicationpartiesid: this.toId(
                        this.formData.primary.application_party_id,
                    ),
                    expense_name: projected.name,
                    expense_type: type ? this.toId(type) : null,
                    amount: projected.amount,
                    frequency: projected.frequency,
                    monthly_equivalent: projected.monthly,
                    is_projected: true,
                    is_household: false,
                    collateral: this.toId(collateral.id),
                    status: "Projected",
                },
            );
            return collateral.projected_expense_id;
        },

        /**
         * Saves the Collateral record for an asset offered as collateral. It
         * uses the asset's own details: collateral has no value of its own,
         * and its category follows the asset's type.
         */
        async saveCollateral(asset) {
            const collateral = asset.collateral;
            const insurance = collateral.insurance || {};

            collateral.id = await this.upsert("Collateral", collateral.id, {
                application: this.formData.id,
                asset_id: this.toId(asset.id),
                category: this.assetTypeLabel(asset.asset_type),
                description: collateral.description,
                insurance_type: insurance.type,
                insurance_status: insurance.status,
                insurance_provider: insurance.provider,
                insurance_reference: insurance.reference,
                insurance_coverage_amount: insurance.coverage_amount,
                insurance_premium: insurance.premium,
                insurance_premium_frequency: insurance.premium_frequency,
                insurance_expiry_date: insurance.expiry_date || null,
                status: "Pending Valuation",
                documents: this.documentIdsOf(collateral),
            });
            return collateral.id;
        },

        /**
         * Persists assets (with ownership splits), liabilities (with
         * responsibility splits), expenses, and collateral. Returns the IDs
         * the application links to.
         */
        async saveFinancials() {
            const assetIds = [];
            const liabilityIds = [];
            const expenseIds = [];

            // Collateral is saved with its asset, and only while the loan
            // requires it. If the applicant switched to an unsecured loan,
            // deleteStaleRecords() removes any collateral saved earlier.
            // Projected insurance expenses are saved after their collateral.
            for (const asset of this.formData.assets) {
                assetIds.push(await this.saveAsset(asset));
                if (this.isCollateral(asset)) {
                    await this.saveCollateral(asset);
                    const projectedId = await this.saveProjectedInsurance(asset);
                    if (projectedId) expenseIds.push(projectedId);
                } else {
                    asset.collateral.projected_expense_id = null;
                }
            }
            for (const liability of this.formData.liabilities) {
                liabilityIds.push(await this.saveLiability(liability));
            }
            for (const expense of this.formData.expenses) {
                expenseIds.push(await this.saveExpense(expense));
            }

            return { assetIds, liabilityIds, expenseIds };
        },

        // ---- Tracking and deleting removed records ----

        /** Server IDs currently in the form, per tracked resource. */
        collectPersistedIds() {
            const form = this.formData;
            const idsOf = (rows) =>
                (rows || []).map((row) => this.toId(row.id)).filter(Boolean);

            return {
                Asset: idsOf(form.assets),
                AssetOwnership: form.assets.flatMap((asset) =>
                    idsOf(asset.owners),
                ),
                Liability: idsOf(form.liabilities),
                LiabilityResponsibility: form.liabilities.flatMap(
                    (liability) => idsOf(liability.responsibilities),
                ),
                // Declared expenses plus the projected insurance expenses of
                // collateral that still has a premium.
                Expense: idsOf(form.expenses).concat(
                    form.assets
                        .filter((asset) => this.isCollateral(asset))
                        .map((asset) => this.toId(asset.collateral.projected_expense_id))
                        .filter(Boolean),
                ),
                // Collateral counts only while its asset is marked as
                // collateral and the loan requires it.
                Collateral: idsOf(
                    form.assets
                        .filter((asset) => this.isCollateral(asset))
                        .map((asset) => asset.collateral),
                ),
                // Third Party Owners have no other income.
                IncomeSource: this.allApplicants
                    .filter((person) => !this.isThirdPartyOwner(person))
                    .flatMap((person) => idsOf(person.incomes)),
                Reference: idsOf(form.references),
                ApplicationParty: this.allApplicants
                    .map((person) => this.toId(person.application_party_id))
                    .filter(Boolean),
            };
        },

        /**
         * Records what's saved on the server: the IDs now in the form plus
         * any extra IDs still to be dealt with (deletes that failed, or
         * everything previously known if a save failed part-way).
         */
        rememberPersistedIds(extra = {}) {
            const current = this.collectPersistedIds();
            const merged = {};
            TRACKED_RESOURCES.forEach((resourceName) => {
                merged[resourceName] = Array.from(
                    new Set(
                        (current[resourceName] || []).concat(
                            extra[resourceName] || [],
                        ),
                    ),
                );
            });
            this.persistedIds = merged;
        },

        /**
         * Deletes records that were saved earlier but are no longer in the
         * form (removed assets, liabilities, expenses, collateral, allocation
         * rows, and applicants). Party records are never deleted, since they
         * can belong to other applications. Returns the IDs that couldn't be
         * deleted so they're retried on the next save.
         */
        async deleteStaleRecords() {
            const current = this.collectPersistedIds();
            const deleted = {};
            const failed = {};

            for (const resourceName of TRACKED_RESOURCES) {
                const keep = new Set(current[resourceName] || []);
                const stale = (this.persistedIds[resourceName] || []).filter(
                    (id) => !keep.has(id),
                );
                if (!stale.length) continue;

                const resource = new Resource(this, resourceName);
                deleted[resourceName] = new Set();

                for (const id of stale) {
                    try {
                        await resource.delete(id);
                        deleted[resourceName].add(id);
                    } catch (error) {
                        // Already gone counts as deleted.
                        const status =
                            error?.response?.status || error?.status;
                        if (status === 404) {
                            deleted[resourceName].add(id);
                        } else {
                            if (!failed[resourceName]) failed[resourceName] = [];
                            failed[resourceName].push(id);
                        }
                    }
                }
            }

            this.forgetDeletedIds(deleted);
            return failed;
        },

        /**
         * Clears the server ID from any local row whose record was deleted
         * (e.g. collateral kept in the form after switching to an unsecured
         * loan), so it's created fresh if it's saved again.
         */
        forgetDeletedIds(deleted) {
            const clear = (rows, resourceName) => {
                const ids = deleted[resourceName];
                if (!ids || !ids.size) return;
                (rows || []).forEach((row) => {
                    if (row.id && ids.has(this.toId(row.id))) row.id = null;
                });
            };

            const form = this.formData;
            clear(form.assets, "Asset");
            form.assets.forEach((asset) =>
                clear(asset.owners, "AssetOwnership"),
            );
            clear(form.liabilities, "Liability");
            form.liabilities.forEach((liability) =>
                clear(liability.responsibilities, "LiabilityResponsibility"),
            );
            clear(form.expenses, "Expense");
            clear(
                form.assets
                    .map((asset) => asset.collateral)
                    .filter(Boolean),
                "Collateral",
            );
            form.assets.forEach((asset) => {
                const collateral = asset.collateral;
                if (
                    collateral &&
                    collateral.projected_expense_id &&
                    deleted.Expense &&
                    deleted.Expense.has(this.toId(collateral.projected_expense_id))
                ) {
                    collateral.projected_expense_id = null;
                }
            });
            this.allApplicants.forEach((person) =>
                clear(person.incomes, "IncomeSource"),
            );
            clear(form.references, "Reference");
        },

        /**
         * Creates or updates the Application, saves every applicant, links
         * them to the application, persists financials, and records the draft
         * location in localStorage for later restoration.
         */
        async saveDraft(silent = false) {
            this.savingDraft = true;

            try {
                // 1. Persist parties and application-parties FIRST
                const partyLinks = [];
                partyLinks.push(
                    await this.saveApplicant(this.formData.primary, true),
                );
                for (const person of this.formData.parties) {
                    partyLinks.push(await this.saveApplicant(person, false));
                }

                // The business on a business loan is its own Party, joined
                // through a "Business Borrower" ApplicationParty.
                const businessLink = await this.saveBusinessParty();
                if (businessLink) partyLinks.push(businessLink);

                // 2. Bundle the newly minted party IDs into the core payload
                const payload = Object.assign(this.appPayload(), {
                    parties: partyLinks,
                    documents: this.formData.document_ids,
                });

                // 3. Create or update the Application in a single shot
                this.formData.id = await this.upsert(
                    "Application",
                    this.formData.id,
                    payload,
                );

                // 4. Save financials and references (these need this.formData.id)
                const financials = await this.saveFinancials();
                const referenceIds = await this.saveReferences();

                // Link them back to the application
                await this.upsert("Application", this.formData.id, {
                    asset_ids: financials.assetIds,
                    liability_ids: financials.liabilityIds,
                    expense_ids: financials.expenseIds,
                    reference_ids: referenceIds,
                });

                // 5. Delete what the applicant removed since the last save.
                // Runs last, once nothing on the application points at them.
                const failedDeletes = await this.deleteStaleRecords();
                this.rememberPersistedIds(failedDeletes);

                this.rememberDraftLocation();

                if (!silent) this.warn("Draft saved successfully.", "success");
                return this.formData.id;
            } catch (error) {
                // Keep track of everything known so far, including records
                // created before the failure, so none are orphaned.
                this.rememberPersistedIds(this.persistedIds);
                // Show and log the real reason, so it can be tracked down.
                console.error("[Loan form] The draft could not be saved:", error);
                this.warn(
                    `The draft could not be saved. ${this.errorText(error)}`,
                    "error",
                );
                throw error;
            } finally {
                this.savingDraft = false;
            }
        },

        /** Fetches full records for a list of IDs, skipping any that fail. */
        async byIds(resourceName, values) {
            const resource = new Resource(this, resourceName);
            const records = [];

            for (const value of values || []) {
                const id = this.toId(value);
                if (!id) continue;
                try {
                    records.push(this.recordOf(await resource.get(id)));
                } catch (error) {
                    // Skip records that no longer exist or can't be read.
                }
            }

            return records;
        },

        /** Runs a filtered list query, returning [] on failure. */
        async listFor(resourceName, query) {
            try {
                return this.toList(
                    await new Resource(this, resourceName).list({
                        query,
                        limit: 100,
                    }),
                );
            } catch (error) {
                return [];
            }
        },

        /**
         * Restores a previously saved draft from localStorage, rebuilding the
         * full nested form state (applicants, assets, liabilities, expenses,
         * collateral) from the server. Submitted applications are ignored.
         * The saved step is deferred to pendingDraftStep and applied after
         * document requirements load in mounted().
         */
        async restoreDraft() {
            const applicationId = localStorage.getItem(this.draftKey("draft_app_id"));
            if (!applicationId) return;

            try {
                const record = this.recordOf(
                    await new Resource(this, "Application").get(applicationId),
                );

                // Never restore a draft that was already submitted.
                if (
                    !record ||
                    String(record.status || "").toLowerCase() === "submitted"
                ) {
                    return;
                }

                const form = this.formData;

                // Copy flat fields, but never overwrite the nested collections.
                const nestedKeys = [
                    "primary",
                    "parties",
                    "assets",
                    "liabilities",
                    "expenses",
                    "references",
                ];
                Object.keys(form).forEach((fieldName) => {
                    if (
                        record[fieldName] !== undefined &&
                        !nestedKeys.includes(fieldName)
                    ) {
                        form[fieldName] = record[fieldName];
                    }
                });

                form.id = this.toId(record);
                form.loan_type_id = this.toId(record.loan_type_id);
                form.is_secured_by_savings = this.toBool(record.is_secured_by_savings, false);
                form.pay_to_institution = this.toBool(record.pay_to_institution, false);
                ["enrollment_start_date", "expected_graduation_date"].forEach((key) => {
                    form[key] = form[key] ? String(form[key]).slice(0, 10) : "";
                });
                // The loan category and name come from the linked loan type.
                const product = this.selectedProduct;
                const categoryRecord = product
                    ? this.loanCategoryRecords.find(
                          (row) => this.toId(row) === this.toId(product.loan_category),
                      )
                    : null;
                form.loan_category = categoryRecord ? String(categoryRecord.code || "") : "";
                form.loan_name = product ? product.name || "" : "";

                // A loan type that isn't offered any more: the applicant picks
                // another one on the first step. Everything else stays.
                if (form.loan_type_id && (!product || !this.isOffered(product))) {
                    this.retiredLoanName = form.loan_name || "your loan";
                    Object.assign(form, {
                        loan_type_id: "",
                        loan_name: "",
                        requested_loan_amount: null,
                        requested_loan_term: null,
                        requested_credit_limit: null,
                    });
                    if (!this.selectedCategory) form.loan_category = "";
                }
                form.document_ids = (record.documents || [])
                    .map(this.toId)
                    .filter(Boolean);

                // --- Rebuild applicants ---
                // Trust the array of IDs stored on the application record
                const partyIds = record.parties || [];

                const links = await this.byIds("ApplicationParty", partyIds);

                // Join each link record with its Party record. The business
                // on a business loan is a "Business Borrower" link, not a
                // person, so it's kept aside and restored further down.
                const people = [];
                let businessLink = null;
                for (const link of links) {
                    const partyId = this.toId(link.party);
                    if (!partyId) continue;
                    if (
                        String(link.role || "").trim().toLowerCase() ===
                        BUSINESS_BORROWER_ROLE.toLowerCase()
                    ) {
                        businessLink = link;
                        continue;
                    }

                    const party = this.recordOf(
                        await new Resource(this, "Party").get(partyId),
                    );
                    if (!party) continue;

                    // Identifications are linked from Party.ids.
                    const identificationRows = (
                        await this.byIds("PartyIdentification", party.ids || [])
                    ).map((row) => ({
                        client_key: generateRowKey("ident"),
                        id: this.toId(row),
                        identification_type: row.identification_type || "",
                        identification_number: row.identification_number || "",
                        issuing_country: row.issuing_country || "",
                        issue_date: row.issue_date || "",
                        expiry_date: row.expiry_date || "",
                        is_primary: row.is_primary === true,
                        document_ids: (row.documents || [])
                            .map(this.toId)
                            .filter(Boolean),
                    }));
                    if (
                        identificationRows.length &&
                        !identificationRows.some((row) => row.is_primary)
                    ) {
                        identificationRows[0].is_primary = true;
                    }

                    // Other income is linked from ApplicationParty.income_ids.
                    const incomeRows = (
                        await this.byIds("IncomeSource", link.income_ids || [])
                    ).map((row) =>
                        Object.assign(createEmptyIncome(), {
                            id: this.toId(row),
                            income_type: row.income_type || "",
                            description: row.description || "",
                            amount: row.amount ?? null,
                            frequency: row.frequency || "",
                            document_ids: (row.documents || [])
                                .map(this.toId)
                                .filter(Boolean),
                        }),
                    );

                    // Drafts saved before pay frequency existed only have a
                    // monthly figure.
                    const hasGrossPay =
                        link.gross_pay !== null &&
                        link.gross_pay !== undefined &&
                        link.gross_pay !== "";

                    people.push(
                        Object.assign(createEmptyApplicant(link.role || ""), {
                            party_id: partyId,
                            application_party_id: this.toId(link),
                            role: link.role || "",
                            kind:
                                party.kind === "ORGANIZATION"
                                    ? "ORGANIZATION"
                                    : "PERSON",
                            first_name: party.first_name || "",
                            last_name: party.last_name || "",
                            business_name: party.legal_name || "",
                            signing_authority: link.signing_authority || "",
                            preferred_contact_method: link.preferred_contact_method || "",
                            relationship_to_applicant:
                                link.relationship_to_applicant || "",
                            email: party.email || "",
                            phone: party.phone || "",
                            date_of_birth: party.date_of_birth || "",
                            marital_status: party.marital_status || "",
                            address: party.address || "",
                            parish: party.parish || "",
                            country: party.country || DEFAULT_COUNTRY,
                            nis_number: party.nis_number || "",
                            is_member: party.is_member === true,
                            member_number: party.member_number || "",
                            citizenship: party.citizenship || "",
                            residency_status: party.residency_status || "",
                            tin: party.tin || "",
                            address_since: party.address_since
                                ? String(party.address_since).slice(0, 10)
                                : "",
                            previous_address: party.previous_address || "",
                            previous_parish: party.previous_parish || "",
                            previous_country:
                                party.previous_country || DEFAULT_COUNTRY,
                            mailing_address: party.mailing_address || "",
                            housing_status: link.housing_status || "",
                            number_of_dependants: link.number_of_dependants ?? null,
                            identifications: identificationRows.length
                                ? identificationRows
                                : [createEmptyIdentification(true)],
                            saved_identification_ids: identificationRows.map(
                                (row) => row.id,
                            ),
                            employment_status: link.employment_status || "",
                            employment_type: link.employment_type || "",
                            employment_start_date: link.employment_start_date
                                ? String(link.employment_start_date).slice(0, 10)
                                : "",
                            employer_name: link.employer_name || "",
                            job_title: link.job_title || "",
                            years_employed: link.years_employed ?? null,
                            gross_pay: hasGrossPay
                                ? link.gross_pay
                                : link.gross_monthly_income ?? null,
                            pay_frequency: hasGrossPay
                                ? link.pay_frequency || ""
                                : link.gross_monthly_income !== null &&
                                    link.gross_monthly_income !== undefined
                                  ? "Monthly"
                                  : "",
                            gross_monthly_income:
                                link.gross_monthly_income ?? null,
                            annual_revenue: link.annual_revenue ?? null,
                            previous_employer_name: link.previous_employer_name || "",
                            previous_job_title: link.previous_job_title || "",
                            previous_employment_years:
                                link.previous_employment_years ?? null,
                            incomes: incomeRows,
                            guarantee_type: link.guarantee_type || "",
                            guarantee_amount: link.guarantee_amount ?? null,
                            is_pep: link.is_pep === true,
                            pep_details: link.pep_details || "",
                            declared_bankruptcy: link.declared_bankruptcy === true,
                            declared_judgments: link.declared_judgments === true,
                            declared_arrears: link.declared_arrears === true,
                            declared_other_applications:
                                link.declared_other_applications === true,
                            declaration_details: link.declaration_details || "",
                            consent_accuracy_confirmation:
                                link.consent_accuracy_confirmation === true,
                            consent_credit_check:
                                link.consent_credit_check === true,
                            consent_data_processing:
                                link.consent_data_processing === true,
                            consented_at: link.consented_at || null,
                            consent_policy_version:
                                link.consent_policy_version || "",
                            document_ids: (link.documents || [])
                                .map(this.toId)
                                .filter(Boolean),
                        }),
                    );
                }

                const primaryApplicant =
                    people.find(
                        (person) =>
                            String(person.role).toLowerCase() ===
                            "primary applicant",
                    ) || people[0];

                if (primaryApplicant) form.primary = primaryApplicant;
                form.business_signing_authority = form.primary.signing_authority || "";
                form.parties = people.filter(
                    (person) =>
                        person !== primaryApplicant &&
                        String(person.role).toLowerCase() !==
                            "primary applicant",
                );
                this.activePartyTab = form.primary.client_key;

                // --- Rebuild assets and their ownership splits ---
                form.assets = (await this.byIds("Asset", record.asset_ids)).map(
                    (item) => ({
                        client_key: generateRowKey("asset"),
                        id: this.toId(item),
                        document_ids: (item.documents || [])
                            .map(this.toId)
                            .filter(Boolean),
                        name: item.name || "",
                        asset_type: item.asset_type || "",
                        description: item.description || "",
                        declared_value: item.declared_value ?? null,
                        registration_number: item.registration_number || "",
                        chassis_number: item.chassis_number || "",
                        block_and_parcel: item.block_and_parcel || "",
                        deed_number: item.deed_number || "",
                        is_purchase: item.status === PURCHASE_ASSET_STATUS,
                        owners: [],
                        collateral: createEmptyCollateral(),
                    }),
                );

                // The vehicle or property being bought keeps its identifiers
                // on its Asset; copy them back to the "Your request" step.
                const purchased = form.assets.find((asset) => asset.is_purchase);
                if (purchased) {
                    Object.assign(form, {
                        vehicle_registration_number: purchased.registration_number,
                        vehicle_chassis_number: purchased.chassis_number,
                        property_block_and_parcel: purchased.block_and_parcel,
                        property_deed_number: purchased.deed_number,
                    });
                }

                // --- Rebuild references ---
                const referenceRows = await this.byIds(
                    "Reference",
                    record.reference_ids || [],
                );
                form.references = ["Personal reference", "Next of kin"].map(
                    (referenceType) => {
                        const row = referenceRows.find(
                            (entry) => entry.reference_type === referenceType,
                        );
                        const reference = createEmptyReference(referenceType);
                        if (!row) return reference;
                        return Object.assign(reference, {
                            id: this.toId(row),
                            name: row.name || "",
                            relationship: row.relationship || "",
                            phone: row.phone || "",
                            email: row.email || "",
                            address: row.address || "",
                        });
                    },
                );

                // --- Rebuild the business (business loans) ---
                const businessId = businessLink ? this.toId(businessLink.party) : null;
                form.business_party_id = businessId;
                form.business_application_party_id = businessLink
                    ? this.toId(businessLink)
                    : null;
                if (businessId) {
                    const business = (await this.byIds("Party", [businessId]))[0];
                    if (business) {
                        Object.assign(form, {
                            business_name: business.legal_name || "",
                            business_trading_name: business.trading_name || "",
                            business_industry: business.industry || "",
                            business_license_number: business.business_license_number || "",
                            business_website: business.website || "",
                            business_registration_number:
                                business.registration_number || "",
                            business_type: business.business_type || "",
                            business_incorporation_date: business.incorporation_date
                                ? String(business.incorporation_date).slice(0, 10)
                                : "",
                            business_employee_count: business.number_of_employees ?? null,
                        });
                    }
                }
                // The figures given when applying are on the Business
                // Borrower link; use them over the Party's latest count.
                if (businessLink) {
                    if (
                        businessLink.number_of_employees !== null &&
                        businessLink.number_of_employees !== undefined &&
                        businessLink.number_of_employees !== ""
                    ) {
                        form.business_employee_count = businessLink.number_of_employees;
                    }
                    form.business_annual_revenue = businessLink.annual_revenue ?? null;
                }

                for (const asset of form.assets) {
                    const ownershipRows = await this.listFor(
                        "AssetOwnership",
                        `asset = ${this.quote(asset.id)}`,
                    );
                    asset.owners = ownershipRows.map((row) => ({
                        client_key: generateRowKey("owner"),
                        id: this.toId(row),
                        party_id: this.toId(row.party),
                        percentage: Number(row.ownership_percentage) || 0,
                    }));

                    // Default to a single 100% owner row when nothing was saved.
                    if (!asset.owners.length) {
                        asset.owners = [
                            {
                                client_key: generateRowKey("owner"),
                                id: null,
                                party_id: "",
                                percentage: 100,
                            },
                        ];
                    }
                }

                // --- Rebuild liabilities ---
                // The application's liability_ids list is authoritative: it's
                // rewritten on every save, so removed liabilities aren't in it.
                // Only drafts saved before that list existed fall back to
                // discovering liabilities through responsibility rows.
                let liabilityIds = (record.liability_ids || [])
                    .map(this.toId)
                    .filter(Boolean);
                if (!Array.isArray(record.liability_ids)) {
                    for (const person of people) {
                        const responsibilityRows = await this.listFor(
                            "LiabilityResponsibility",
                            `application_party = ${this.quote(this.toId(person.application_party_id))}`,
                        );
                        liabilityIds.push(
                            ...responsibilityRows
                                .map((row) => this.toId(row.liability))
                                .filter(Boolean),
                        );
                    }
                }
                liabilityIds = [...new Set(liabilityIds)];

                form.liabilities = (
                    await this.byIds("Liability", liabilityIds)
                ).map((item) => ({
                    client_key: generateRowKey("liability"),
                    id: this.toId(item),
                    document_ids: (item.documents || [])
                        .map(this.toId)
                        .filter(Boolean),
                    creditor_name: item.creditor_name || "",
                    // Maps legacy string values to LiabilityType IDs.
                    liability_type: this.resolveTypeId(
                        item.liability_type,
                        this.liabilityTypes,
                    ),
                    outstanding_balance: item.outstanding_balance ?? null,
                    payment_amount: item.payment_amount ?? null,
                    payment_frequency: item.payment_frequency || "",
                    credit_limit: item.credit_limit ?? null,
                    assessed_payment: item.assessed_payment ?? null,
                    description: item.description || "",
                    is_to_be_paid_off: item.is_to_be_paid_off === true,
                    is_secured: item.is_secured === true,
                    // Points at the asset's client key, like the dropdown.
                    secured_asset_ref:
                        this.assetForRef(item.secured_asset)?.client_key || "",
                    responsibilities: [],
                }));

                for (const liability of form.liabilities) {
                    const responsibilityRows = await this.listFor(
                        "LiabilityResponsibility",
                        `liability = ${this.quote(liability.id)}`,
                    );
                    liability.responsibilities = responsibilityRows.map(
                        (row) => ({
                            client_key: generateRowKey("resp"),
                            id: this.toId(row),
                            application_party_id: this.toId(
                                row.application_party,
                            ),
                            percentage:
                                Number(row.responsibility_percentage) || 0,
                            responsibility_type:
                                row.responsibility_type || "Borrower",
                        }),
                    );

                    if (!liability.responsibilities.length) {
                        liability.responsibilities = [
                            {
                                client_key: generateRowKey("resp"),
                                id: null,
                                application_party_id: "",
                                percentage: 100,
                                responsibility_type: "Borrower",
                            },
                        ];
                    }
                }

                // --- Rebuild expenses ---
                // Same rule as liabilities: trust expense_ids when it exists.
                let expenseRecords = [];
                if (Array.isArray(record.expense_ids)) {
                    expenseRecords = await this.byIds(
                        "Expense",
                        record.expense_ids,
                    );
                } else {
                    for (const person of people) {
                        expenseRecords = expenseRecords.concat(
                            await this.listFor(
                                "Expense",
                                `applicationpartiesid = ${this.quote(this.toId(person.application_party_id))}`,
                            ),
                        );
                    }
                }

                // Projected insurance expenses are rebuilt from their
                // collateral below, not shown as declared expenses.
                const projectedRecords = expenseRecords.filter(
                    (item) => item.is_projected === true,
                );
                form.expenses = expenseRecords
                    .filter((item) => item.is_projected !== true)
                    .map((item) => ({
                    client_key: generateRowKey("expense"),
                    id: this.toId(item),
                    document_ids: (item.documents || [])
                        .map(this.toId)
                        .filter(Boolean),
                    application_party_id: this.toId(item.applicationpartiesid),
                    expense_name: item.expense_name || "",
                    // Maps legacy string values to ExpenseType IDs.
                    expense_type: this.resolveTypeId(
                        item.expense_type,
                        this.expenseTypes,
                    ),
                    amount: item.amount ?? null,
                    frequency: item.frequency || "",
                    is_household: item.is_household === true,
                }));

                // --- Rebuild collateral onto its assets ---
                // Each Collateral record points at one of the declared assets.
                // Records whose asset is no longer on the application are
                // ignored.
                const collateralRows = await this.listFor(
                    "Collateral",
                    `application = ${this.quote(applicationId)}`,
                );
                collateralRows.forEach((item) => {
                    const assetId = this.toId(item.asset_id);
                    const asset = form.assets.find(
                        (entry) => assetId && this.toId(entry.id) === assetId,
                    );
                    if (!asset || asset.collateral.enabled) return;

                    asset.collateral = Object.assign(createEmptyCollateral(), {
                        enabled: true,
                        id: this.toId(item),
                        description: item.description || "",
                        document_ids: (item.documents || [])
                            .map(this.toId)
                            .filter(Boolean),
                        insurance: {
                            type: item.insurance_type || "",
                            status: item.insurance_status || "Quote",
                            provider: item.insurance_provider || "",
                            reference: item.insurance_reference || "",
                            coverage_amount: item.insurance_coverage_amount ?? null,
                            premium: item.insurance_premium ?? null,
                            premium_frequency: item.insurance_premium_frequency || "",
                            expiry_date: item.insurance_expiry_date || "",
                        },
                    });

                    const projected = projectedRecords.find(
                        (entry) => this.toId(entry.collateral) === this.toId(item),
                    );
                    if (projected) {
                        asset.collateral.projected_expense_id = this.toId(projected);
                    }
                });

                // Everything just loaded is what's on the server right now.
                this.rememberPersistedIds();

                // Defer the saved step until requirements load in mounted().
                const savedStep = Number(
                    localStorage.getItem(this.draftKey("draft_step")),
                );
                if (Number.isFinite(savedStep))
                    this.pendingDraftStep = savedStep;
                if (this.retiredLoanName) this.pendingDraftStep = 0;

                // Work out the Yes/No answers from what was saved: "yes" if
                // there's anything in the section, "no" if the applicant
                // already went past it, otherwise not answered yet.
                const passed = (stepId) => {
                    const index = this.path.findIndex((item) => item.id === stepId);
                    return Number.isFinite(savedStep) && index >= 0 && savedStep > index;
                };
                const answerFor = (hasItems, stepId) =>
                    hasItems ? true : passed(stepId) ? false : null;
                this.answers = {
                    // The draft is first saved when leaving the people step.
                    others: form.parties.length > 0,
                    assets: answerFor(
                        form.assets.some((asset) => !asset.is_purchase),
                        "assets",
                    ),
                    debts: answerFor(form.liabilities.length > 0, "liabilities"),
                    bills: answerFor(form.expenses.length > 0, "expenses"),
                };

                this.warn("Welcome back. We've loaded your saved application.", "success");
            } catch (error) {
                this.warn(
                    "The saved draft could not be restored safely.",
                    "warning",
                );
            }
        },

        /**
         * Final submission: validates all applicants and documents, saves the
         * draft, then runs the submit workflow, which sets the status,
         * submitted_at, and application_number together (and gives a number
         * only when there's none, so a retry never gets a second one). The
         * record is re-read to confirm before the receipt shows. If anything
         * fails, the application stays a draft and Send can be pressed again.
         */
        async submit() {
            const allApplicantsValid = this.allApplicants.every(
                (person) => person.role && this.validApplicant(person),
            );
            if (!allApplicantsValid) {
                this.setStep(
                    this.path.findIndex((item) => item.id === "parties"),
                );
                return this.warn(
                    "Complete every applicant and consent declaration.",
                );
            }
            if (this.guarantorIssue()) {
                this.setStep(this.path.findIndex((item) => item.id === "parties"));
                return this.warn(this.guarantorIssue());
            }
            const requestIssue = this.requestMainIssue() || this.requestDetailsIssue();
            if (requestIssue) {
                this.setStep(this.path.findIndex((item) => item.id === "request"));
                return this.warn(requestIssue);
            }

            const missing = this.firstIncompleteScope();
            if (missing) {
                const stepByKind = {
                    application: "documents",
                    applicant: "parties",
                    identification: "parties",
                    income: "parties",
                    asset: "assets",
                    liability: "liabilities",
                    expense: "expenses",
                    collateral: "assets",
                };
                const index = this.path.findIndex(
                    (item) => item.id === stepByKind[missing.kind],
                );
                if (index >= 0) this.setStep(index);
                return this.warn(
                    `Upload the required documents for ${missing.label} before submitting.`,
                );
            }

            this.saving = true;

            try {
                await this.saveDraft(true);

                const resource = new Resource(this, "Application");

                // Submit workflow: sets the status, date, and number.
                const workflowReply = await new Resource(this).request(
                    "post",
                    `/workflows/execute/MXHGYH/${this.formData.id}`,
                );

                // Confirm the server persisted the submitted status, waiting
                // briefly for the workflow to assign the reference number.
                const saved = await this.waitForSubmission(
                    resource,
                    this.formData.id,
                );
                if (
                    !saved ||
                    String(saved.status || "").toLowerCase() !== "submitted"
                ) {
                    console.error("[Loan form] Submission not verified", {
                        workflowReply,
                        status: saved?.status,
                        application_number: saved?.application_number,
                    });
                    throw Error("Submission not verified");
                }

                this.formData.application_number =
                    saved.application_number || "";
                this.receipt = { number: saved.application_number || "" };

                // A submitted application is no longer a draft.
                localStorage.removeItem(this.draftKey("draft_app_id"));
                localStorage.removeItem(this.draftKey("draft_step"));
            } catch (error) {
                console.error("[Loan form] Sending failed", error);
                this.warn(
                    "We couldn't send your application. Please try again.",
                    "error",
                );
            } finally {
                this.saving = false;
            }
        },

        /**
         * Re-reads the application until the submit workflow has finished:
         * the status is "submitted" and application_number is set. The
         * workflow writes the number before the status (and on a retry the
         * number is already there), so waiting for the number alone can
         * stop too early. Returns the latest record either way.
         */
        async waitForSubmission(resource, id, attempts = 10, delayMs = 1000) {
            let saved = null;
            for (let attempt = 0; attempt < attempts; attempt++) {
                saved = this.recordOf(await resource.get(id));
                if (
                    saved?.application_number &&
                    String(saved.status || "").toLowerCase() === "submitted"
                ) {
                    return saved;
                }
                await new Promise((resolve) => setTimeout(resolve, delayMs));
            }
            return saved;
        },

        /** Clears the draft and returns the wizard to a blank first step. */
        reset() {
            localStorage.removeItem(this.draftKey("draft_app_id"));
            localStorage.removeItem(this.draftKey("draft_step"));
            this.formData = createEmptyApplication();
            this.activePartyTab = this.formData.primary.client_key;
            this.setStep(0);
            this.answers = { others: null, assets: null, debts: null, bills: null };
            this.pendingDraftStep = null;
            this.persistedIds = {};
            this.receipt = null;
            this.alert = { text: "", type: "warning" };
            this.documentState = {};
        },
    },
};
</script>

<style scoped>
/*
 * No styles here on purpose: Saturn doesn't apply <style> blocks reliably.
 * Everything is styled in the template with Tailwind classes, plus inline
 * styles for the brand colours (CSS variables set on the form) and exact sizes.
 */
</style>
