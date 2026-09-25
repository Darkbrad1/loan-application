<template>
    <main class="adaptive-form" :style="brandStyle">
        <!-- Success receipt shown after a verified submission -->
        <section v-if="receipt" class="receipt">
            <div class="receipt-icon"><v-icon>mdi-check</v-icon></div>
            <p class="eyebrow">Application submitted</p>
            <h1>We have your request</h1>
            <template v-if="receipt.number">
                <p>
                    Keep this reference number. You'll need it if you contact
                    us about your application.
                </p>
                <strong class="reference-card">{{ receipt.number }}</strong>
            </template>
            <p v-else class="reference-pending">
                Your reference number is still being assigned. Contact us if
                you need it before we get in touch.
            </p>
            <el-button type="primary" @click="reset"
                >Start another application</el-button
            >
        </section>

        <template v-else>
            <!-- Company header driven by system branding configuration -->
            <header class="app-header">
                <div class="brand">
                    <div v-if="companyLogo" class="brand-logo">
                        <img :src="companyLogo" alt="Company logo" />
                    </div>
                    <strong>{{ companyName }}</strong>
                </div>
                <div class="header-actions">
                    <!-- Testing only: fills each step with sample data -->
                    <label v-if="testModeAvailable" class="test-switch">
                        <el-switch
                            :model-value="testMode"
                            @update:model-value="toggleTestMode"
                        />
                        <span>Test mode</span>
                    </label>
                    <label v-if="testMode" class="test-switch">
                        <el-switch
                            :model-value="testKeepDraft"
                            @update:model-value="toggleTestKeepDraft"
                        />
                        <span>Remember draft on reload</span>
                    </label>
                    <span>Secure loan application</span>
                </div>
            </header>

            <div class="layout">
                <!-- Left rail: step tracker -->
                <aside class="path">
                    <p class="eyebrow">Your application</p>
                    <ol>
                        <li
                            v-for="(stepItem, stepIndex) in path"
                            :key="stepItem.id"
                            :class="{
                                active: stepIndex === step,
                                done: stepIndex < step,
                            }"
                        >
                            <span>
                                <v-icon v-if="stepIndex < step" size="small"
                                    >mdi-check</v-icon
                                >
                                <template v-else>{{ stepIndex + 1 }}</template>
                            </span>
                            <div>
                                <b>{{ stepItem.title }}</b>
                                <small>{{ stepItem.note }}</small>
                            </div>
                        </li>
                    </ol>
                </aside>

                <!-- Center: the current step's form -->
                <section class="workspace">
                    <div class="progress">
                        <i :style="{ width: progressWidth }"></i>
                    </div>

                    <div class="heading">
                        <p class="eyebrow">
                            Step {{ step + 1 }} of {{ path.length }}
                        </p>
                        <h1>{{ current.title }}</h1>
                        <p>{{ current.description }}</p>
                    </div>

                    <el-alert
                        v-if="alert.text"
                        :title="alert.text"
                        :type="alert.type"
                        :closable="false"
                        show-icon
                        class="alert"
                    />

                    <el-form label-position="top" class="card" @submit.prevent>
                        <AdaptiveLoanProductSection
                            v-if="current.id === 'loan'"
                            :loans="loans"
                            :products="productsForLoan"
                            :loan-category="formData.loan_category"
                            :loan-type-id="formData.loan_type_id"
                            @select-category="chooseLoan"
                            @select-product="selectProduct"
                        />

                        <AdaptiveLoanApplicantsSection
                            v-else-if="current.id === 'parties'"
                            :primary="formData.primary"
                            :parties="formData.parties"
                            :lookups="lookups"
                            :resource-props="resourceProps"
                            :active-tab="activePartyTab"
                            :minimum-identifications="minimumIdentifications"
                            :deductions="deductionsByApplicant"
                            @update:primary="formData.primary = $event"
                            @update:parties="formData.parties = $event"
                            @update:active-tab="activePartyTab = $event"
                            @request-add="addApplicant"
                            @request-remove="removeApplicant"
                            :document-scopes="documentScopes"
                            :uploading-key="uploadingDocumentKey"
                            :documents-disabled="documentsDisabled"
                            @stage-file="stageDocument"
                            @remove-file="removeStagedDocument"
                            @request-file-upload="uploadScopedDocument"
                            @file-rejected="handleRejectedDocument"
                        />

                        <AdaptiveLoanRequestSection
                            v-else-if="current.id === 'request'"
                            :model-value="requestData"
                            :loan-category="formData.loan_category"
                            :selected-product="selectedProduct"
                            :lookups="lookups"
                            :application-props="applicationProps"
                            :amount-minimum="amountMinimum"
                            :amount-maximum="amountMaximum"
                            :term-minimum="termMinimum"
                            :term-maximum="termMaximum"
                            @update:model-value="patchRequest"
                        />

                        <AdaptiveLoanAssetsSection
                            v-else-if="current.id === 'assets'"
                            :model-value="formData.assets"
                            :party-options="partyOptions"
                            :third-party-owner-ids="thirdPartyOwnerPartyIds"
                            :requires-collateral="requiresCollateral"
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
                        Liability types are now LiabilityType records. Revolving
                        types (credit cards, overdrafts) require a credit limit
                        and show an assessed repayment based on a % of the limit.
                        -->
                        <AdaptiveLoanLiabilitiesSection
                            v-else-if="current.id === 'liabilities'"
                            :model-value="formData.liabilities"
                            :application-party-options="applicationPartyOptions"
                            :liability-types="liabilityTypes"
                            :default-revolving-rate="defaultRevolvingRate"
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
                        Expense types are now ExpenseType records, filtered by
                        the selected loan category and user_selectable.
                        -->
                        <AdaptiveLoanExpensesSection
                            v-else-if="current.id === 'expenses'"
                            :model-value="formData.expenses"
                            :application-party-options="applicationPartyOptions"
                            :expense-type-options="expenseTypeOptions"
                            :expense-types="expenseTypes"
                            :loan-category-label="loanCategoryLabel"
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
                        uploaded inside their own sections (collateral
                        documents inside the asset's card).
                        -->
                        <AdaptiveLoanDocumentRequirements
                            v-else-if="current.id === 'documents'"
                            standalone
                            title="Application documents"
                            :description="`Documents required for the ${formData.loan_name} application.`"
                            :scope="documentScopes.application"
                            :uploading-key="uploadingDocumentKey"
                            :disabled="documentsDisabled"
                            @stage-file="stageDocument"
                            @remove-file="removeStagedDocument"
                            @request-file-upload="uploadScopedDocument"
                            @file-rejected="handleRejectedDocument"
                        />

                        <!-- Final step: review everything before submitting -->
                        <AdaptiveLoanReviewSection
                            v-else
                            :application="formData"
                            :applicants="allApplicants"
                            :requires-collateral="requiresCollateral"
                        />

                        <footer class="form-footer">
                            <el-button
                                :disabled="step === 0 || saving"
                                @click="back"
                            >
                                Back
                            </el-button>
                            <span>{{
                                formData.id ? "Draft saved" : "Not yet saved"
                            }}</span>
                            <el-button
                                v-if="step > 1 && step < path.length - 1"
                                plain
                                :loading="savingDraft"
                                @click="saveDraft(false)"
                            >
                                Save draft
                            </el-button>
                            <el-button
                                v-if="step < path.length - 1"
                                type="primary"
                                :loading="saving || documentsLoading"
                                @click="next"
                            >
                                Continue
                            </el-button>
                            <el-button
                                v-else
                                type="success"
                                :loading="saving"
                                @click="submit"
                            >
                                Submit application
                            </el-button>
                        </footer>
                    </el-form>
                </section>

                <!-- Right rail: live summary of the in-progress application -->
                <aside class="summary">
                    <p class="eyebrow">Live summary</p>
                    <div>
                        <span>Product</span>
                        <strong>{{
                            formData.loan_name || "Not selected"
                        }}</strong>
                    </div>
                    <div>
                        <span>Security</span>
                        <strong>{{ securityLabel }}</strong>
                    </div>
                    <div>
                        <span>Amount</span>
                        <strong>{{
                            money(formData.requested_loan_amount)
                        }}</strong>
                    </div>
                    <div>
                        <span>Term</span>
                        <strong>
                            {{
                                formData.requested_loan_term
                                    ? `${formData.requested_loan_term} months`
                                    : "—"
                            }}
                        </strong>
                    </div>
                    <div v-if="requiredDocumentCount">
                        <span>Documents</span>
                        <strong>
                            {{ uploadedDocumentCount }}/{{
                                requiredDocumentCount
                            }}
                            uploaded
                        </strong>
                    </div>
                </aside>
            </div>
        </template>
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
    "Collateral",
    "AssetOwnership",
    "LiabilityResponsibility",
    "Expense",
    "Liability",
    "Asset",
    "ApplicationParty",
];

/**
 * Generates a unique client-side key for list rows that have no server ID yet.
 */
const generateRowKey = (prefix = "row") =>
    `${prefix}_${Date.now()}_${Math.random().toString(36).slice(2, 9)}`;

/**
 * Minimum forms of identification every applicant must provide. This is
 * an institution-wide rule rather than a per-product one; change it here
 * if the credit union requires two.
 */
const MINIMUM_IDENTIFICATIONS = 1;

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
    // Linked to Party.ids. saved_identification_ids is what was on the
    // server at the last save or restore, so removed IDs can be deleted.
    identifications: [createEmptyIdentification(true)],
    saved_identification_ids: [],
    employment_status: "",
    employer_name: "",
    job_title: "",
    years_employed: null,
    gross_monthly_income: null,
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
    // Home-loan-specific fields
    property_address: "",
    property_type: "",
    property_value: null,
    // Business-loan-specific fields
    business_name: "",
    business_registration_number: "",
    business_type: "",
    business_incorporation_date: "",
    business_employee_count: null,
    // Nested collections
    primary: createEmptyApplicant("Primary Applicant"),
    parties: [],
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

            // Institution-wide minimum, passed to the applicant editor.
            minimumIdentifications: MINIMUM_IDENTIFICATIONS,

            // Server data
            products: [],

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
            },

            // Top-level loan categories shown on step 1
            loans: [
                {
                    id: "personal",
                    title: "Personal loan",
                    note: "Flexible financing",
                    icon: "mdi-account-cash",
                },
                {
                    id: "auto",
                    title: "Auto loan",
                    note: "Vehicle financing",
                    icon: "mdi-car",
                },
                {
                    id: "home",
                    title: "Home loan",
                    note: "Purchase or refinance",
                    icon: "mdi-home",
                },
                {
                    id: "business",
                    title: "Business loan",
                    note: "Growth capital",
                    icon: "mdi-storefront",
                },
            ],
        };
    },

    computed: {
        /** CSS custom properties derived from the tenant's branding. */
        brandStyle() {
            return {
                "--brand": this.primaryColor || "#1178bd",
                "--accent": this.secondaryColor || "#f7a31f",
                "--logo-bg": this.logoBackground || "#fff",
            };
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

        /** Products filtered down to the selected loan category. */
        productsForLoan() {
            return this.products.filter(
                (product) =>
                    String(product.category || "").toLowerCase() ===
                    this.formData.loan_category,
            );
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
            const loan = this.loans.find(
                (item) => item.id === this.formData.loan_category,
            );
            return loan ? loan.title.toLowerCase() : "this loan";
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

        /** Auto and home loans (or explicitly secured products) need collateral. */
        requiresCollateral() {
            return (
                ["auto", "home"].includes(this.formData.loan_category) ||
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
                requested_loan_amount: form.requested_loan_amount,
                requested_loan_term: form.requested_loan_term,
                repayment_frequency: form.repayment_frequency,
                loan_purpose: form.loan_purpose,
                vehicle_make: form.vehicle_make,
                vehicle_model: form.vehicle_model,
                vehicle_year: form.vehicle_year,
                vehicle_condition: form.vehicle_condition,
                property_address: form.property_address,
                property_type: form.property_type,
                property_value: form.property_value,
                business_name: form.business_name,
                business_registration_number: form.business_registration_number,
                business_type: form.business_type,
                business_incorporation_date: form.business_incorporation_date,
                business_employee_count: form.business_employee_count,
            };
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
            const add = (key, kind, label, types, lockedMessage = "") => {
                if (!types.length) return;
                scopes.push({
                    key,
                    kind,
                    label,
                    types,
                    lockedMessage,
                    requirements: this.requirementsFor(key, types),
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
            const primaryReady = this.validApplicant(form.primary);
            this.allApplicants.forEach((person, index) => {
                // Third Party Owners give no documents or IDs for now.
                if (this.isThirdPartyOwner(person)) return;

                let locked = "";
                if (!this.validApplicant(person)) {
                    locked =
                        "Complete this applicant's details and consent to upload documents.";
                } else if (index > 0 && !primaryReady) {
                    locked = "Complete the primary applicant's details first.";
                }
                add(
                    `applicant:${person.client_key}`,
                    "applicant",
                    this.applicantName(person, index),
                    applicantTypes,
                    locked,
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
                {
                    id: "loan",
                    title: "Choose loan",
                    note: "Select financing",
                    description: "Choose the loan category and product.",
                },
                {
                    id: "parties",
                    title: "Applicants",
                    note: "Identity and consent",
                    description:
                        "Provide identity, employment, and consent information.",
                },
                {
                    id: "request",
                    title: "Your request",
                    note: "Amount and term",
                    description: "Enter values within the product limits.",
                },
                {
                    id: "assets",
                    title: "Assets",
                    note: this.requiresCollateral
                        ? "Ownership and collateral"
                        : "Party ownership",
                    description: this.requiresCollateral
                        ? "Declare assets and ownership, and mark the assets securing this loan as collateral."
                        : "Declare assets and ownership.",
                },
                {
                    id: "liabilities",
                    title: "Liabilities",
                    note: "Responsibility",
                    description: "Declare liabilities and responsibility.",
                },
                {
                    id: "expenses",
                    title: "Expenses",
                    note: "Recurring costs",
                    description: "Assign expenses to applicants.",
                },
            ];

            // Only application-level documents have their own step.
            if (this.documentScopes.application) {
                steps.push({
                    id: "documents",
                    title: "Documents",
                    note: "Application uploads",
                    description:
                        "Upload the documents required for this loan product.",
                });
            }

            steps.push({
                id: "review",
                title: "Review",
                note: "Confirm and submit",
                description: "Review before submission.",
            });

            return steps;
        },

        /** The step descriptor for the current position in the wizard. */
        current() {
            return this.path[Math.min(this.step, this.path.length - 1)];
        },

        progressWidth() {
            return `${((this.step + 1) / this.path.length) * 100}%`;
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
            this.step = Math.min(this.pendingDraftStep, this.path.length - 1);
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

        /** Unemployed and retired applicants don't need employer details. */
        showsEmploymentDetails(status) {
            const value = String(status || "")
                .trim()
                .toLowerCase();
            return value !== "unemployed" && value !== "retired";
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
                type.applies_to.includes(this.formData.loan_category)
            );
        },

        /** Merges partial updates from the request step into the form. */
        patchRequest(patch) {
            Object.assign(this.formData, patch);
        },

        /** Switching categories clears the product and dependent values. */
        chooseLoan(category) {
            Object.assign(this.formData, {
                loan_category: category,
                loan_type_id: "",
                loan_name: "",
                requested_loan_amount: null,
                requested_loan_term: null,
            });
        },

        /**
         * Requirements follow the product name automatically. For a saved
         * draft, documents already linked may satisfy the new product's
         * requirements, so re-check them.
         */
        selectProduct(id) {
            this.formData.loan_type_id = id;
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
            localStorage.setItem("gccu_draft_app_id", this.formData.id);
            localStorage.setItem("gccu_draft_step", String(this.step));
        },

        /** Removes the saved draft location, so a reload starts fresh. */
        forgetDraftLocation() {
            localStorage.removeItem("gccu_draft_app_id");
            localStorage.removeItem("gccu_draft_step");
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
                    // An auto loan exercises collateral too, when one exists.
                    const categories = this.products.map((product) =>
                        String(product.category || "").toLowerCase(),
                    );
                    const category = categories.includes("auto")
                        ? "auto"
                        : categories[0] || "personal";
                    this.chooseLoan(category);
                }
                if (!this.toId(form.loan_type_id) && this.productsForLoan.length) {
                    this.selectProduct(this.toId(this.productsForLoan[0]));
                }
            }

            if (id === "parties") {
                this.fillBlanks(primary, {
                    first_name: "Test",
                    last_name: "Applicant",
                    email: "test.applicant@example.com",
                    phone: "473-555-0100",
                    date_of_birth: "1990-05-15",
                    marital_status: this.testOption("marital_status", /single/i),
                    address: "12 Test Street, Grand Anse",
                    country: DEFAULT_COUNTRY,
                    parish: this.testOption("parish", /george/i, "St. George"),
                    nis_number: "TEST123456",
                    employment_status: this.testOption("employment_status", /^employed/i),
                    employer_name: "Test Employer Ltd",
                    job_title: "Clerk",
                    years_employed: 5,
                    gross_monthly_income: 6000,
                });
                primary.consent_accuracy_confirmation = true;
                primary.consent_credit_check = true;
                primary.consent_data_processing = true;

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
                if (form.loan_category === "auto") {
                    this.fillBlanks(form, {
                        vehicle_make: "Toyota",
                        vehicle_model: "Corolla",
                        vehicle_year: "2020",
                        vehicle_condition: this.testOption("vehicle_condition", /used/i),
                    });
                }
                if (form.loan_category === "home") {
                    this.fillBlanks(form, {
                        property_address: "5 Test Road, St. George's",
                        property_type: this.testOption("property_type", /house|single/i),
                        property_value: 350000,
                    });
                }
                if (form.loan_category === "business") {
                    this.fillBlanks(form, {
                        business_name: "Test Business Ltd",
                        business_registration_number: "TEST-REG-001",
                        business_type: this.testOption("business_type", null),
                        business_incorporation_date: "2015-06-01",
                        business_employee_count: 5,
                    });
                }
            }

            if (id === "assets" && !form.assets.length) {
                const asset = {
                    client_key: generateRowKey("asset"),
                    id: null,
                    name: "Test vehicle",
                    asset_type: this.testOption("asset_type", /vehicle|car|auto/i),
                    description: "Sample asset added in test mode.",
                    declared_value: 45000,
                    document_ids: [],
                    owners: [
                        {
                            client_key: generateRowKey("owner"),
                            id: null,
                            party_id: this.toId(primary.party_id) || "",
                            percentage: 100,
                        },
                    ],
                    collateral: createEmptyCollateral(),
                };
                if (this.requiresCollateral) {
                    asset.collateral.enabled = true;
                    asset.collateral.description = "Sample collateral.";
                    asset.collateral.insurance = {
                        type: this.testOption("insurance_type", /comprehensive/i, "Comprehensive"),
                        status: this.testOption("insurance_status", /policy/i, "Policy"),
                        provider: "Test Insurance Co.",
                        reference: "POL-TEST-001",
                        coverage_amount: 45000,
                        premium: 150,
                        premium_frequency: this.testOption(
                            "insurance_premium_frequency",
                            /month/i,
                        ),
                        expiry_date: "2030-12-31",
                    };
                }
                form.assets.push(asset);
            }

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

            if (id === "expenses" && !form.expenses.length) {
                const type = this.expenseTypeOptions[0] || null;
                form.expenses.push({
                    client_key: generateRowKey("expense"),
                    id: null,
                    application_party_id: this.toId(primary.application_party_id) || "",
                    expense_name: "",
                    expense_type: type ? type.value : "",
                    amount: 800,
                    frequency: this.testOption("expense_frequency", /month/i),
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

        /** True when every requirement in the scope has been uploaded. */
        scopeComplete(scope) {
            return (
                !scope ||
                scope.requirements.every((item) => item.status === "uploaded")
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
        requirementsFor(scopeKey, types) {
            return (types || []).map((type) => {
                const typeId = this.toId(type);
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
                        name: type.name || "Supporting document",
                        description: type.description || "",
                        allowed_file_types: type.allowed_file_types || [],
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
                .filter(Boolean);
            const links = { parties: partyLinks, application_parties: partyLinks };
            const resource = new Resource(this, "Application");

            if (this.formData.id) {
                await resource.update(this.formData.id, links);
                return;
            }

            const createdId = this.toId(
                await resource.create({ ...this.appPayload(), ...links }),
            );
            if (!createdId) {
                throw Error("Application did not return a record ID.");
            }
            this.formData.id = createdId;
            this.rememberDraftLocation();
        },

        /** Updates the application's asset/liability/expense ID lists. */
        async linkFinancialsToApplication() {
            const ids = this.collectPersistedIds();
            await new Resource(this, "Application").update(this.formData.id, {
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

            if (owner.kind === "applicant" || owner.kind === "identification") {
                // An identification is saved as part of its applicant.
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
            }

            ["owners", "responsibilities", "identifications"].forEach((listKey) =>
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
                    await new Resource(this, "Upload").update(uploadedId, {
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
                await new Resource(this, resourceName).update(ownerId, {
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
            ]);

            this.products = this.toList(products);
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
            };

            this.liabilityTypes = this.activeSortedTypes(liabilityTypeRows).map(
                (type) => ({
                    ...type,
                    is_revolving: this.toBool(type.is_revolving),
                    revolving_rate: this.normalizeRate(type.revolving_rate),
                }),
            );

            this.expenseTypes = this.activeSortedTypes(expenseTypeRows).map(
                (type) => ({
                    ...type,
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
                role: this.options(applicationPartyProps, "role"),
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
                business_type: this.options(applicationProps, "business_type"),
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
            });
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
         * retired, or anyone under 16 or at pensionable age.
         * Returns { amount, basis } where basis explains the figure.
         */
        estimateNis(person) {
            const settings = STATUTORY_DEDUCTIONS.nis;
            const gross = Number(person.gross_monthly_income) || 0;
            const status = String(person.employment_status || "")
                .trim()
                .toLowerCase();
            const age = this.ageOn(person.date_of_birth);

            if (gross <= 0) return { amount: 0, basis: "No income entered" };
            if (status === "unemployed" || status === "retired") {
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
            const gross = Number(person.gross_monthly_income) || 0;
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
            const gross = Number(person.gross_monthly_income) || 0;
            const nis = this.estimateNis(person);
            const incomeTax = this.estimateIncomeTax(person);
            return {
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

        /** Today's date as YYYY-MM-DD, for comparing expiry dates. */
        today() {
            return new Date().toISOString().slice(0, 10);
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
            if (rows.length < MINIMUM_IDENTIFICATIONS) {
                return MINIMUM_IDENTIFICATIONS === 1
                    ? "Add a form of identification."
                    : `Add at least ${MINIMUM_IDENTIFICATIONS} forms of identification.`;
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
            if (
                !person.first_name ||
                !person.last_name ||
                !person.email ||
                !person.phone ||
                !person.date_of_birth
            ) {
                return "Complete the name, contact details, and date of birth.";
            }
            if (
                !person.address ||
                !person.country ||
                (this.isGrenada(person.country) && !person.parish)
            ) {
                return this.isGrenada(person.country)
                    ? "Complete the street address, parish, and country."
                    : "Complete the street address and country.";
            }
            if (!person.nis_number) return "Enter the NIS number.";

            const identificationIssue = this.identificationsIssue(person);
            if (identificationIssue) return identificationIssue;

            if (
                person.gross_monthly_income === null ||
                person.gross_monthly_income === undefined
            ) {
                return "Enter the gross monthly income.";
            }
            if (
                !person.consent_accuracy_confirmation ||
                !person.consent_credit_check ||
                !person.consent_data_processing
            ) {
                return "Accept all three declarations.";
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
                    ? "Enter the business name."
                    : "Enter the first and last name.";
            }
            if (!person.relationship_to_applicant) {
                return "Enter their relationship to the primary applicant.";
            }
            if (!person.phone) return "Enter a phone number.";
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
            }

            if (this.current.id === "request") {
                const amount = Number(form.requested_loan_amount);
                const term = Number(form.requested_loan_term);

                if (!amount || !term || !form.loan_purpose) {
                    return "Complete amount, term, and purpose.";
                }
                if (
                    amount < this.amountMinimum ||
                    amount > this.amountMaximum
                ) {
                    return `Amount must be between ${this.money(
                        this.amountMinimum,
                    )} and ${this.money(this.amountMaximum)}.`;
                }
                if (term < this.termMinimum || term > this.termMaximum) {
                    return `Term must be between ${this.termMinimum} and ${this.termMaximum} months.`;
                }
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

                const hasInvalidExpense = form.expenses.some(
                    (item) =>
                        !this.isBorrowerLink(item.application_party_id) ||
                        !item.expense_type ||
                        item.amount === null,
                );
                if (hasInvalidExpense) {
                    return "Complete and assign every expense.";
                }
            }

            // Required documents for the items on this step. Field checks
            // above run first, since items can't receive uploads until
            // they're complete.
            const kindsByStep = {
                parties: ["applicant", "identification"],
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
         * Advances the wizard, auto-saving the draft when leaving steps that
         * persist server data.
         */
        async next() {
            const issue = this.validate();
            if (issue) return this.warn(issue);

            this.alert = { text: "", type: "warning" };

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

            this.step++;
            if (this.formData.id) this.rememberDraftLocation();
        },

        back() {
            if (this.step > 0) this.step--;
        },

        // ---- Persistence payloads ----

        /**
         * Builds the Application resource payload. Only fields relevant to the
         * chosen loan category (auto/home/business) are included.
         */
        appPayload() {
            const form = this.formData;

            // application_number is deliberately not sent: the submit
            // workflow assigns it, and sending it would overwrite it.
            const payload = {
                status: form.status || "draft",
                loan_category: form.loan_category,
                loan_type_id: this.toId(form.loan_type_id),
                loan_name: form.loan_name,
                requested_loan_amount: form.requested_loan_amount,
                requested_loan_term: form.requested_loan_term,
                repayment_frequency: form.repayment_frequency,
                loan_purpose: form.loan_purpose,
                documents: form.document_ids.map(this.toId).filter(Boolean),
            };

            if (form.loan_category === "auto") {
                Object.assign(payload, {
                    vehicle_make: form.vehicle_make,
                    vehicle_model: form.vehicle_model,
                    vehicle_year: form.vehicle_year,
                    vehicle_condition: form.vehicle_condition,
                });
            }
            if (form.loan_category === "home") {
                Object.assign(payload, {
                    property_address: form.property_address,
                    property_type: form.property_type,
                    property_value: form.property_value,
                });
            }
            if (form.loan_category === "business") {
                Object.assign(payload, {
                    business_name: form.business_name,
                    business_registration_number:
                        form.business_registration_number,
                    business_type: form.business_type,
                    business_incorporation_date:
                        form.business_incorporation_date,
                    business_employee_count: String(
                        form.business_employee_count || "",
                    ),
                });
            }

            return payload;
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
                    business_name: isBusiness ? person.business_name : "",
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
                // ids are set after the identifications are saved, since a
                // PartyIdentification can't exist before its Party.
            };
        },

        /** One PartyIdentification record, linked to its owning Party. */
        identificationPayload(row, partyId) {
            return {
                party: this.toId(partyId),
                party_id: this.toId(partyId),
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
        applicationPartyPayload(person, isPrimary) {
            // Third Party Owners aren't borrowing: no employment, income,
            // deductions, or consents are asked for or kept.
            if (!isPrimary && this.isThirdPartyOwner(person)) {
                return {
                    party: this.toId(person.party_id),
                    party_id: this.toId(person.party_id),
                    role: THIRD_PARTY_OWNER_ROLE,
                    relationship_status: "Active",
                    is_primary_contact: false,
                    relationship_to_applicant: person.relationship_to_applicant,
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

            return {
                party: this.toId(person.party_id),
                party_id: this.toId(person.party_id),
                role: isPrimary ? "Primary Applicant" : person.role,
                relationship_status: "Active",
                is_primary_contact: isPrimary,
                relationship_to_applicant: "",
                employment_status: person.employment_status,
                employer_name: showEmployment ? person.employer_name : "",
                job_title: showEmployment ? person.job_title : "",
                years_employed: showEmployment ? person.years_employed : null,
                gross_monthly_income: person.gross_monthly_income,
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

        /** Creates or updates a resource record and returns its ID. */
        async upsert(resourceName, id, payload) {
            const resource = new Resource(this, resourceName);

            if (id) {
                await resource.update(this.toId(id), payload);
                return this.toId(id);
            }

            const createdId = this.toId(await resource.create(payload));
            if (!createdId) {
                throw Error(`${resourceName} did not return a record ID.`);
            }
            return createdId;
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
            await new Resource(this, "Party").update(person.party_id, {
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

            return person.application_party_id;
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
            asset.id = await this.upsert("Asset", asset.id, {
                name: asset.name,
                asset_type: asset.asset_type,
                description: asset.description,
                declared_value: asset.declared_value,
                status: "Declared",
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
            expense.id = await this.upsert("Expense", expense.id, {
                applicationpartiesid: this.toId(expense.application_party_id),
                expense_name:
                    expense.expense_name ||
                    this.expenseTypeName(expense.expense_type),
                expense_type: this.toId(expense.expense_type),
                amount: expense.amount,
                frequency: expense.frequency,
                status: "Declared",
                documents: this.documentIdsOf(expense),
            });
            return expense.id;
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
            for (const asset of this.formData.assets) {
                assetIds.push(await this.saveAsset(asset));
                if (this.isCollateral(asset)) await this.saveCollateral(asset);
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
                Expense: idsOf(form.expenses),
                // Collateral counts only while its asset is marked as
                // collateral and the loan requires it.
                Collateral: idsOf(
                    form.assets
                        .filter((asset) => this.isCollateral(asset))
                        .map((asset) => asset.collateral),
                ),
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

                // 2. Bundle the newly minted party IDs into the core payload
                const resource = new Resource(this, "Application");
                const payload = Object.assign(this.appPayload(), {
                    parties: partyLinks,
                    application_parties: partyLinks,
                    documents: this.formData.document_ids,
                });

                // 3. Create or update the Application in a single shot
                if (!this.formData.id) {
                    this.formData.id = this.toId(
                        await resource.create(payload),
                    );
                } else {
                    await resource.update(this.formData.id, payload);
                }

                // 4. Save financials (these still run after since Collateral relies on this.formData.id)
                const financials = await this.saveFinancials();

                // Link the financials back to the application
                await resource.update(this.formData.id, {
                    asset_ids: financials.assetIds,
                    liability_ids: financials.liabilityIds,
                    expense_ids: financials.expenseIds,
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
                this.warn("The draft could not be saved.", "error");
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
            const applicationId = localStorage.getItem("gccu_draft_app_id");
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
                form.document_ids = (record.documents || [])
                    .map(this.toId)
                    .filter(Boolean);

                // --- Rebuild applicants ---
                // Trust the array of IDs stored on the application record
                const partyIds =
                    record.parties || record.application_parties || [];

                const links = await this.byIds("ApplicationParty", partyIds);

                // Join each link record with its Party record.
                const people = [];
                for (const link of links) {
                    const partyId = this.toId(link.party || link.party_id);
                    if (!partyId) continue;

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
                            business_name: party.business_name || "",
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
                            identifications: identificationRows.length
                                ? identificationRows
                                : [createEmptyIdentification(true)],
                            saved_identification_ids: identificationRows.map(
                                (row) => row.id,
                            ),
                            employment_status: link.employment_status || "",
                            employer_name: link.employer_name || "",
                            job_title: link.job_title || "",
                            years_employed: link.years_employed ?? null,
                            gross_monthly_income:
                                link.gross_monthly_income ?? null,
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
                        owners: [],
                        collateral: createEmptyCollateral(),
                    }),
                );

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
                        expenseRecords.push(
                            ...(await this.listFor(
                                "Expense",
                                `applicationpartiesid = ${this.quote(this.toId(person.application_party_id))}`,
                            )),
                        );
                    }
                }

                form.expenses = expenseRecords.map((item) => ({
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
                });

                // Everything just loaded is what's on the server right now.
                this.rememberPersistedIds();

                // Defer the saved step until requirements load in mounted().
                const savedStep = Number(
                    localStorage.getItem("gccu_draft_step"),
                );
                if (Number.isFinite(savedStep))
                    this.pendingDraftStep = savedStep;

                this.warn("Your application draft was restored.", "success");
            } catch (error) {
                this.warn(
                    "The saved draft could not be restored safely.",
                    "warning",
                );
            }
        },

        /**
         * Final submission: validates all applicants and documents, saves,
         * marks the application submitted, triggers the post-submit workflow,
         * then re-reads the record to verify before showing a receipt.
         */
        async submit() {
            const allApplicantsValid = this.allApplicants.every(
                (person) => person.role && this.validApplicant(person),
            );
            if (!allApplicantsValid) {
                this.step = this.path.findIndex(
                    (item) => item.id === "parties",
                );
                return this.warn(
                    "Complete every applicant and consent declaration.",
                );
            }

            const missing = this.firstIncompleteScope();
            if (missing) {
                const stepByKind = {
                    application: "documents",
                    applicant: "parties",
                    identification: "parties",
                    asset: "assets",
                    liability: "liabilities",
                    expense: "expenses",
                    collateral: "assets",
                };
                const index = this.path.findIndex(
                    (item) => item.id === stepByKind[missing.kind],
                );
                if (index >= 0) this.step = index;
                return this.warn(
                    `Upload the required documents for ${missing.label} before submitting.`,
                );
            }

            this.saving = true;

            try {
                await this.saveDraft(true);

                const resource = new Resource(this, "Application");
                await resource.update(this.formData.id, {
                    status: "submitted",
                    submitted_at: new Date().toISOString(),
                });

                // Submit workflow: also assigns the application number.
                await new Resource(this).request(
                    "post",
                    `/workflows/execute/MXHGYH/${this.formData.id}`,
                );

                // Confirm the server persisted the submitted status, waiting
                // briefly for the workflow to assign the reference number.
                const saved = await this.waitForApplicationNumber(
                    resource,
                    this.formData.id,
                );
                if (
                    !saved ||
                    String(saved.status || "").toLowerCase() !== "submitted"
                ) {
                    throw Error("Submission not verified");
                }

                this.formData.application_number =
                    saved.application_number || "";
                this.receipt = { number: saved.application_number || "" };

                // A submitted application is no longer a draft.
                localStorage.removeItem("gccu_draft_app_id");
                localStorage.removeItem("gccu_draft_step");
            } catch (error) {
                this.warn("Submission could not be verified.", "error");
            } finally {
                this.saving = false;
            }
        },

        /**
         * Re-reads the application until the submit workflow has assigned
         * application_number, in case the workflow finishes after its request
         * returns. Returns the latest record either way.
         */
        async waitForApplicationNumber(resource, id, attempts = 5, delayMs = 1000) {
            let saved = null;
            for (let attempt = 0; attempt < attempts; attempt++) {
                saved = this.recordOf(await resource.get(id));
                if (saved?.application_number) return saved;
                await new Promise((resolve) => setTimeout(resolve, delayMs));
            }
            return saved;
        },

        /** Clears the draft and returns the wizard to a blank first step. */
        reset() {
            localStorage.removeItem("gccu_draft_app_id");
            localStorage.removeItem("gccu_draft_step");
            this.formData = createEmptyApplication();
            this.activePartyTab = this.formData.primary.client_key;
            this.step = 0;
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
:global(body) {
    margin: 0;
    background: #f4f7fb;
    font-family: Inter, system-ui, sans-serif;
}

.adaptive-form {
    /* ---- Design tokens ---- */
    --brand: #1178bd;
    --brand-soft: #eff8ff;
    --accent: #f7a31f;

    --ink: #172033;
    --ink-2: #334155;
    --muted: #64748b;
    --muted-2: #94a3b8;

    --success: #15803d;
    --success-dark: #166534;
    --success-soft: #dcfce7;
    --success-bg: #f0fdf4;
    --success-border: #bbf7d0;
    --danger: #b91c1c;
    --danger-soft: #fee2e2;
    --danger-border: #fecaca;
    --warn: #92400e;
    --warn-soft: #fef3c7;
    --info: #1d4ed8;
    --info-soft: #dbeafe;

    --border: #dfe7ec;
    --surface: #fff;
    --surface-soft: #fbfdfe;
    --surface-muted: #f8fafc;

    /* Type scale */
    --fs-xs: 11px; /* helpers, badges, kickers */
    --fs-sm: 12px; /* labels, secondary text */
    --fs-md: 13px; /* body */
    --fs-lg: 18px; /* sub-section titles */
    --fs-xl: 21px; /* section titles */
    --fs-2xl: 28px; /* page title */

    /* Spacing scale */
    --sp-1: 4px;
    --sp-2: 8px;
    --sp-3: 12px;
    --sp-4: 16px;
    --sp-5: 20px;
    --sp-6: 28px;

    /* Radii */
    --r-sm: 10px;
    --r-md: 12px;
    --r-lg: 16px;
    --r-xl: 20px;

    min-height: 100vh;
    color: var(--ink);
    background: #f4f7fb;
}

.header-actions {
    display: flex;
    align-items: center;
    gap: var(--sp-4);
}

.test-switch {
    display: flex;
    align-items: center;
    gap: 8px;
    padding: 2px 10px;
    border: 1px dashed rgba(255, 255, 255, 0.7);
    border-radius: 6px;
    font-size: 0.85rem;
    cursor: pointer;
}

.app-header {
    display: flex;
    align-items: center;
    justify-content: space-between;
    padding: var(--sp-4) max(24px, calc((100% - 1280px) / 2));
    color: #fff;
    background: var(--brand);
}

.brand {
    display: flex;
    align-items: center;
    gap: var(--sp-3);
}

.brand-logo {
    display: grid;
    place-items: center;
    width: 112px;
    height: 52px;
    padding: var(--sp-1);
    overflow: hidden;
    border-radius: var(--r-sm);
    background: var(--logo-bg);
}

.brand-logo img {
    display: block;
    max-width: 100%;
    max-height: 100%;
    object-fit: contain;
}

.layout {
    display: grid;
    grid-template-columns: 220px minmax(0, 760px) 230px;
    gap: 24px;
    max-width: 1280px;
    margin: auto;
    padding: 32px 24px;
}

.path,
.summary {
    position: sticky;
    top: var(--sp-5);
    align-self: start;
}

.path {
    max-height: calc(100vh - 40px);
    overflow: auto;
}

/* Shared kicker style */
.eyebrow,
:deep(.document-section .eyebrow),
:deep(.document-section .scope-kicker) {
    margin: 0 0 var(--sp-2);
    color: var(--brand);
    font-size: var(--fs-xs);
    font-weight: 800;
    letter-spacing: 0.12em;
    text-transform: uppercase;
}

.path ol {
    padding: 0;
    margin: var(--sp-4) 0;
    list-style: none;
}

.path li {
    display: flex;
    gap: var(--sp-3);
    padding-bottom: var(--sp-5);
    color: var(--muted-2);
}

.path li > span {
    display: grid;
    place-items: center;
    width: 28px;
    height: 28px;
    flex: 0 0 auto;
    border: 1px solid #cbd5e1;
    border-radius: 50%;
    background: var(--surface);
    font-size: var(--fs-sm);
}

.path li b,
.path li small {
    display: block;
}

.path li b {
    font-size: var(--fs-md);
}

.path li small {
    font-size: var(--fs-xs);
}

.path li.active {
    color: var(--brand);
}

.path li.active > span {
    color: #fff;
    border-color: var(--brand);
    background: var(--brand);
}

.path li.done > span {
    color: var(--brand);
    border-color: var(--brand);
    background: var(--brand-soft);
}

.progress,
:deep(.document-section .progress-track) {
    height: 6px;
    overflow: hidden;
    border-radius: 999px;
    background: #e5edf2;
}

.progress i,
:deep(.document-section .progress-track i) {
    display: block;
    height: 100%;
    border-radius: inherit;
    background: var(--accent);
    transition: width 0.3s ease;
}

:deep(.document-section .progress-track) {
    margin-bottom: var(--sp-4);
}

.heading {
    padding: var(--sp-6) 0 var(--sp-4);
}

.heading h1 {
    margin: 0 0 var(--sp-2);
    font-size: var(--fs-2xl);
}

.heading p:not(.eyebrow) {
    margin: 0;
    color: var(--muted);
}

.alert {
    margin-bottom: var(--sp-4);
}

.card {
    padding: var(--sp-6);
    border: 1px solid var(--border);
    border-radius: var(--r-xl);
    background: var(--surface);
    box-shadow: 0 18px 40px #10243d0a;
}

.form-footer {
    display: flex;
    align-items: center;
    justify-content: space-between;
    gap: var(--sp-3);
    margin-top: var(--sp-6);
    padding-top: var(--sp-5);
    border-top: 1px solid var(--border);
}

.form-footer span {
    margin-right: auto;
    color: var(--muted);
    font-size: var(--fs-xs);
}

.summary {
    padding: var(--sp-5);
    color: #fff;
    border-radius: var(--r-lg);
    background: var(--brand);
}

.summary .eyebrow {
    color: #fff;
    opacity: 0.78;
}

.summary > div {
    padding: var(--sp-3) 0;
    border-bottom: 1px solid #ffffff33;
}

.summary > div:last-child {
    border: 0;
}

.summary span,
.summary strong {
    display: block;
}

.summary span {
    font-size: var(--fs-xs);
    opacity: 0.75;
}

.summary strong {
    margin-top: var(--sp-1);
    font-size: var(--fs-md);
}

.mono {
    font-family: ui-monospace, monospace;
}

.receipt {
    max-width: 560px;
    margin: 80px auto;
    padding: 44px;
    text-align: center;
    border: 1px solid var(--border);
    border-radius: var(--r-xl);
    background: var(--surface);
}

.receipt-icon {
    display: grid;
    place-items: center;
    width: 64px;
    height: 64px;
    margin: 0 auto var(--sp-4);
    color: #fff;
    border-radius: 50%;
    background: var(--success);
}

.reference-pending {
    margin: var(--sp-5) 0;
    padding: var(--sp-4);
    color: var(--ink-2);
    border-radius: var(--r-sm);
    background: var(--surface-muted);
}

.reference-card {
    display: block;
    margin: var(--sp-5) 0;
    padding: var(--sp-4);
    color: var(--brand);
    border-radius: var(--r-sm);
    background: var(--brand-soft);
    font-family: ui-monospace, monospace;
}

/* ---- Shared child-component layout ---- */

:deep(.loan-grid) {
    display: grid;
    grid-template-columns: repeat(2, minmax(0, 1fr));
    gap: var(--sp-3);
}

:deep(.loan-grid button) {
    display: flex;
    min-height: 132px;
    flex-direction: column;
    align-items: flex-start;
    padding: var(--sp-4);
    cursor: pointer;
    text-align: left;
    border: 1px solid var(--border);
    border-radius: var(--r-md);
    background: var(--surface-soft);
    transition:
        border-color 0.2s,
        background 0.2s;
}

:deep(.loan-grid button:hover),
:deep(.loan-grid button.selected) {
    border: 2px solid var(--brand);
    background: var(--brand-soft);
}

:deep(.loan-grid .v-icon) {
    color: var(--brand);
}

:deep(.loan-grid b) {
    margin: var(--sp-3) 0 var(--sp-1);
}

:deep(.loan-grid small),
:deep(.helper),
:deep(.collection-header p),
:deep(.context p) {
    color: var(--muted);
}

:deep(.field-grid) {
    display: grid;
    grid-template-columns: repeat(2, minmax(0, 1fr));
    gap: 0 var(--sp-4);
}

:deep(.context),
:deep(.item-card) {
    margin-top: var(--sp-5);
    padding: var(--sp-5);
    border: 1px solid var(--border);
    border-radius: var(--r-md);
    background: var(--surface-soft);
}

:deep(.context h3) {
    margin: 0 0 var(--sp-3);
}

:deep(.helper) {
    display: block;
    margin-top: var(--sp-1);
    font-size: var(--fs-xs);
}

/* Assessed repayment note under a revolving liability's credit limit */
:deep(.assessed-note) {
    grid-column: 1 / -1;
    margin: calc(var(--sp-2) * -1) 0 var(--sp-4);
    padding: var(--sp-2) var(--sp-3);
    color: var(--info);
    border-radius: var(--r-sm);
    background: var(--info-soft);
    font-size: var(--fs-sm);
}

:deep(.helper.invalid) {
    color: var(--danger);
}

:deep(.pane-actions) {
    display: flex;
    justify-content: flex-end;
}

:deep(.consents) {
    display: grid;
    gap: var(--sp-2);
    margin-top: var(--sp-4);
    padding: var(--sp-4);
    border: 1px solid var(--border);
    border-radius: var(--r-md);
    background: var(--surface-muted);
}

:deep(.add-button) {
    margin-top: var(--sp-4);
}

:deep(.collection-header),
:deep(.item-title),
:deep(.allocation-heading) {
    display: flex;
    align-items: center;
    justify-content: space-between;
    gap: var(--sp-3);
}

:deep(.collection-header h3),
:deep(.collection-header p) {
    margin: 0;
}

:deep(.collection-header p) {
    font-size: var(--fs-sm);
}

:deep(.item-title) {
    margin-bottom: var(--sp-4);
}

:deep(.allocation) {
    margin-top: var(--sp-3);
    padding: var(--sp-4);
    border-radius: var(--r-md);
    background: #f1f5f9;
}

:deep(.allocation-row) {
    display: grid;
    grid-template-columns: minmax(0, 1.6fr) minmax(120px, 0.7fr) 36px;
    gap: var(--sp-3);
    align-items: end;
}

:deep(.total) {
    margin: var(--sp-2) 0 0;
    font-size: var(--fs-sm);
    font-weight: 700;
}

:deep(.valid) {
    color: var(--success);
}

:deep(.invalid) {
    color: var(--danger);
}

:deep(.review-block) {
    display: grid;
    grid-template-columns: repeat(2, minmax(0, 1fr));
    gap: 1px;
    overflow: hidden;
    border-radius: var(--r-lg);
}

:deep(.review-row) {
    padding: var(--sp-4);
    color: #fff;
    background: var(--brand);
}

:deep(.review-row span),
:deep(.review-row strong) {
    display: block;
}

:deep(.review-row span) {
    font-size: var(--fs-xs);
    opacity: 0.72;
}

:deep(.review-row strong) {
    margin-top: var(--sp-1);
    font-size: var(--fs-md);
}

/* ---- Document upload section ---- */

:deep(.document-section) {
    color: var(--ink);
}

:deep(.document-section .section-header) {
    display: flex;
    align-items: flex-start;
    justify-content: space-between;
    gap: var(--sp-5);
    margin-bottom: var(--sp-4);
}

:deep(.document-section .section-header h3),
:deep(.document-section .scope-heading h4),
:deep(.document-section .requirement-title h5) {
    margin: 0;
}

:deep(.document-section .section-header h3) {
    font-size: var(--fs-xl);
}

:deep(.document-section .section-copy),
:deep(.document-section .scope-heading p:not(.scope-kicker)),
:deep(.document-section .requirement-title p) {
    color: var(--muted);
}

:deep(.document-section .section-copy) {
    max-width: 570px;
    margin: var(--sp-2) 0 0;
    font-size: var(--fs-md);
    line-height: 1.55;
}

:deep(.document-section .overall-progress) {
    min-width: 126px;
    padding: var(--sp-3) var(--sp-4);
    text-align: right;
    border: 1px solid var(--border);
    border-radius: var(--r-md);
    background: var(--surface-muted);
}

:deep(.document-section .overall-progress strong),
:deep(.document-section .overall-progress span) {
    display: block;
}

:deep(.document-section .overall-progress strong) {
    color: var(--brand);
    font-size: var(--fs-lg);
}

:deep(.document-section .overall-progress span) {
    color: var(--muted);
    font-size: var(--fs-xs);
}

:deep(.document-section .scope-tabs) {
    display: flex;
    gap: var(--sp-2);
    overflow-x: auto;
    padding: 2px 2px var(--sp-2);
}

:deep(.document-section .scope-tabs button) {
    display: flex;
    align-items: center;
    gap: var(--sp-2);
    min-width: 148px;
    padding: var(--sp-3);
    cursor: pointer;
    text-align: left;
    color: var(--muted);
    border: 1px solid var(--border);
    border-radius: var(--r-md);
    background: var(--surface);
    transition:
        border-color 0.2s,
        background 0.2s,
        color 0.2s;
}

:deep(.document-section .scope-tabs button:hover:not(:disabled)),
:deep(.document-section .scope-tabs button.active) {
    color: var(--brand);
    border-color: var(--brand);
    background: var(--brand-soft);
}

:deep(.document-section .scope-tabs button.complete:not(.active)) {
    color: var(--success);
    border-color: var(--success-border);
    background: var(--success-bg);
}

:deep(.document-section .scope-tabs button.locked) {
    cursor: not-allowed;
    opacity: 0.52;
}

:deep(.document-section .tab-icon) {
    display: grid;
    place-items: center;
    width: 28px;
    height: 28px;
    flex: 0 0 auto;
    color: inherit;
    border-radius: 50%;
    background: #eef3f6;
    font-size: var(--fs-xs);
    font-weight: 800;
}

:deep(.document-section .scope-tabs button.active .tab-icon) {
    color: #fff;
    background: var(--brand);
}

:deep(.document-section .scope-tabs button.complete .tab-icon) {
    color: var(--success);
    background: var(--success-soft);
}

:deep(.document-section .tab-text b),
:deep(.document-section .tab-text small) {
    display: block;
}

:deep(.document-section .tab-text b) {
    max-width: 150px;
    overflow: hidden;
    font-size: var(--fs-sm);
    text-overflow: ellipsis;
    white-space: nowrap;
}

:deep(.document-section .tab-text small) {
    margin-top: 2px;
    font-size: var(--fs-xs);
    font-weight: 500;
}

:deep(.document-section .scope-panel) {
    margin-top: var(--sp-2);
    padding: var(--sp-5);
    border: 1px solid var(--border);
    border-radius: var(--r-lg);
    background: var(--surface-soft);
}

:deep(.document-section .scope-heading) {
    display: flex;
    align-items: flex-start;
    justify-content: space-between;
    gap: var(--sp-4);
    margin-bottom: var(--sp-4);
}

:deep(.document-section .scope-heading h4) {
    font-size: var(--fs-lg);
}

:deep(.document-section .scope-heading p:not(.scope-kicker)) {
    margin: var(--sp-1) 0 0;
    font-size: var(--fs-sm);
}

:deep(.document-section .scope-badge),
:deep(.document-section .status-pill) {
    flex: 0 0 auto;
    padding: var(--sp-1) var(--sp-2);
    border-radius: 999px;
    font-size: var(--fs-xs);
    font-weight: 800;
}

:deep(.document-section .scope-badge),
:deep(.document-section .status-pill.required) {
    color: var(--warn);
    background: var(--warn-soft);
}

:deep(.document-section .scope-badge.complete),
:deep(.document-section .status-pill.success) {
    color: var(--success-dark);
    background: var(--success-soft);
}

:deep(.document-section .status-pill.ready) {
    color: var(--info);
    background: var(--info-soft);
}

:deep(.document-section .status-pill.working) {
    color: #0369a1;
    background: #e0f2fe;
}

:deep(.document-section .status-pill.danger) {
    color: var(--danger);
    background: var(--danger-soft);
}

:deep(.document-section .requirement-list) {
    display: grid;
    gap: var(--sp-3);
}

:deep(.document-section .requirement-card) {
    padding: var(--sp-4);
    border: 1px solid var(--border);
    border-radius: var(--r-md);
    background: var(--surface);
    transition:
        border-color 0.2s,
        box-shadow 0.2s;
}

:deep(.document-section .requirement-card.uploaded) {
    border-color: var(--success-border);
    background: var(--success-bg);
}

:deep(.document-section .requirement-card.uploading) {
    border-color: #93c5fd;
    box-shadow: 0 0 0 3px #dbeafe80;
}

:deep(.document-section .requirement-card.error) {
    border-color: var(--danger-border);
}

:deep(.document-section .requirement-top) {
    display: flex;
    gap: var(--sp-3);
}

:deep(.document-section .document-mark) {
    display: grid;
    place-items: center;
    width: 38px;
    height: 38px;
    flex: 0 0 auto;
    color: var(--brand);
    border-radius: var(--r-sm);
    background: var(--brand-soft);
}

:deep(.document-section .requirement-card.uploaded .document-mark) {
    color: var(--success);
    background: var(--success-soft);
}

:deep(.document-section .requirement-title) {
    min-width: 0;
    flex: 1;
}

:deep(.document-section .title-line) {
    display: flex;
    align-items: center;
    justify-content: space-between;
    gap: var(--sp-2);
}

:deep(.document-section .requirement-title h5) {
    font-size: var(--fs-md);
}

:deep(.document-section .requirement-title p) {
    margin: var(--sp-1) 0;
    font-size: var(--fs-sm);
    line-height: 1.45;
}

:deep(.document-section .requirement-title small) {
    color: var(--muted-2);
    font-size: var(--fs-xs);
}

:deep(.document-section .drop-zone) {
    display: flex;
    min-height: 112px;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    margin-top: var(--sp-4);
    padding: var(--sp-4);
    cursor: pointer;
    text-align: center;
    border: 2px dashed #cbd9e2;
    border-radius: var(--r-md);
    background: var(--surface-muted);
}

:deep(.document-section .drop-zone:hover:not(.disabled)) {
    border-color: var(--brand);
    background: var(--brand-soft);
}

:deep(.document-section .drop-zone.disabled) {
    cursor: not-allowed;
    opacity: 0.6;
}

:deep(.document-section .drop-zone input) {
    position: absolute;
    width: 1px;
    height: 1px;
    overflow: hidden;
    opacity: 0;
}

:deep(.document-section .upload-symbol) {
    color: var(--brand);
}

:deep(.document-section .drop-zone strong) {
    margin-top: var(--sp-1);
    color: var(--ink-2);
    font-size: var(--fs-sm);
}

:deep(.document-section .drop-zone u) {
    color: var(--brand);
    text-underline-offset: 2px;
}

:deep(.document-section .drop-zone small) {
    margin-top: var(--sp-1);
    color: var(--muted-2);
    font-size: var(--fs-xs);
}

:deep(.document-section .staged-file),
:deep(.document-section .uploaded-file) {
    display: flex;
    align-items: center;
    gap: var(--sp-3);
    margin-top: var(--sp-4);
    padding: var(--sp-3);
    border-radius: var(--r-sm);
}

:deep(.document-section .staged-file) {
    background: #f1f5f9;
}

:deep(.document-section .uploaded-file) {
    color: var(--success-dark);
    background: var(--success-bg);
}

:deep(.document-section .staged-icon) {
    display: grid;
    place-items: center;
    width: 34px;
    height: 34px;
    flex: 0 0 auto;
    color: var(--brand);
    border-radius: var(--r-sm);
    background: var(--surface);
}

:deep(.document-section .staged-details),
:deep(.document-section .uploaded-file div) {
    min-width: 0;
    flex: 1;
}

:deep(.document-section .staged-details strong),
:deep(.document-section .staged-details span),
:deep(.document-section .uploaded-file strong),
:deep(.document-section .uploaded-file span) {
    display: block;
}

:deep(.document-section .staged-details strong),
:deep(.document-section .uploaded-file strong) {
    overflow: hidden;
    font-size: var(--fs-sm);
    text-overflow: ellipsis;
    white-space: nowrap;
}

:deep(.document-section .staged-details span),
:deep(.document-section .uploaded-file span) {
    margin-top: 2px;
    color: var(--muted);
    font-size: var(--fs-xs);
}

:deep(.document-section .error-message) {
    display: flex;
    align-items: center;
    gap: var(--sp-1);
    margin-top: var(--sp-2);
    color: var(--danger);
    font-size: var(--fs-xs);
}

:deep(.document-section .scope-complete) {
    display: flex;
    align-items: center;
    gap: var(--sp-3);
    margin-top: var(--sp-4);
    padding: var(--sp-3) var(--sp-4);
    color: var(--success-dark);
    border: 1px solid var(--success-border);
    border-radius: var(--r-md);
    background: var(--success-bg);
}

:deep(.document-section .scope-complete strong),
:deep(.document-section .scope-complete span) {
    display: block;
}

:deep(.document-section .scope-complete strong) {
    font-size: var(--fs-sm);
}

:deep(.document-section .scope-complete span) {
    margin-top: 2px;
    font-size: var(--fs-xs);
}

:deep(.document-section .legacy-queue) {
    padding: var(--sp-4);
    border: 1px solid var(--border);
    border-radius: var(--r-md);
    background: var(--surface-soft);
}

:deep(.document-section .legacy-copy) {
    margin-top: var(--sp-2);
    font-size: var(--fs-md);
    font-weight: 700;
}

:deep(.document-section .legacy-queue small) {
    color: var(--muted);
}

:deep(.document-section .legacy-upload-button) {
    margin: var(--sp-4) 0;
}

/* ---- Estimated statutory deductions (applicant editor) ---- */

:deep(.deduction-summary) {
    margin-top: var(--sp-2);
    padding: var(--sp-3) var(--sp-4);
    border: 1px solid var(--info-soft);
    border-radius: var(--r-md);
    background: #f5f9ff;
}

:deep(.deduction-summary .deduction-row) {
    display: flex;
    align-items: baseline;
    justify-content: space-between;
    gap: var(--sp-3);
    padding: var(--sp-1) 0;
    font-size: var(--fs-sm);
}

:deep(.deduction-summary .deduction-row small) {
    display: block;
    color: var(--muted);
    font-size: var(--fs-xs);
}

:deep(.deduction-summary .deduction-row.net) {
    margin-top: var(--sp-1);
    padding-top: var(--sp-2);
    border-top: 1px solid var(--info-soft);
    font-weight: 700;
}

:deep(.deduction-summary > small) {
    display: block;
    margin-top: var(--sp-2);
    color: var(--muted);
    font-size: var(--fs-xs);
}

/* ---- Documents inside a section (applicant tab or item card) ---- */

:deep(.item-documents) {
    margin-top: var(--sp-4);
    padding-top: var(--sp-4);
    border-top: 1px dashed var(--border);
}

:deep(.item-documents.standalone) {
    margin-top: 0;
    padding-top: 0;
    border-top: 0;
}

:deep(.item-documents .documents-heading) {
    display: flex;
    align-items: flex-start;
    justify-content: space-between;
    gap: var(--sp-3);
    margin-bottom: var(--sp-3);
}

:deep(.item-documents .documents-heading h4) {
    margin: 0;
    font-size: var(--fs-md);
}

:deep(.item-documents.standalone .documents-heading h4) {
    font-size: var(--fs-lg);
}

:deep(.item-documents .documents-heading p) {
    margin: 2px 0 0;
    color: var(--muted);
    font-size: var(--fs-xs);
}

:deep(.item-documents .lock-notice) {
    display: flex;
    align-items: center;
    gap: var(--sp-2);
    margin-bottom: var(--sp-3);
    padding: var(--sp-2) var(--sp-3);
    color: var(--warn);
    border-radius: var(--r-sm);
    background: var(--warn-soft);
    font-size: var(--fs-sm);
}

/* ---- Saturn/Element Plus controls ---- */

:deep(.el-select),
:deep(.el-date-editor),
:deep(.el-input-number) {
    width: 100%;
}

:deep(.el-form-item__label) {
    color: var(--ink-2);
    font-size: var(--fs-sm) !important;
    font-weight: 700;
}

:deep(.el-button--primary) {
    --el-button-bg-color: var(--accent);
    --el-button-border-color: var(--accent);
    --el-button-hover-bg-color: var(--accent);
    --el-button-hover-border-color: var(--accent);
}

@media (max-width: 1100px) {
    .layout {
        grid-template-columns: 200px minmax(0, 1fr);
    }

    .summary {
        display: none;
    }
}

@media (max-width: 720px) {
    .layout {
        display: block;
        padding: var(--sp-5) var(--sp-4);
    }

    .path {
        display: none;
    }

    .card {
        padding: var(--sp-5);
    }

    :deep(.field-grid),
    :deep(.loan-grid),
    :deep(.review-block),
    :deep(.allocation-row) {
        grid-template-columns: 1fr;
    }

    .app-header {
        padding: var(--sp-3) var(--sp-4);
    }

    .brand-logo {
        width: 86px;
        height: 42px;
    }

    .heading h1 {
        font-size: 24px;
    }

    .form-footer {
        flex-wrap: wrap;
    }

    .form-footer span {
        order: 4;
        width: 100%;
    }

    .receipt {
        margin: 40px var(--sp-4);
        padding: var(--sp-6) var(--sp-5);
    }
}

@media (max-width: 640px) {
    :deep(.document-section .section-header),
    :deep(.document-section .scope-heading),
    :deep(.document-section .title-line) {
        align-items: stretch;
        flex-direction: column;
    }

    :deep(.document-section .overall-progress) {
        min-width: 0;
        text-align: left;
    }

    :deep(.document-section .scope-panel) {
        padding: var(--sp-4);
    }

    :deep(.document-section .requirement-card) {
        padding: var(--sp-4);
    }

    :deep(.document-section .staged-file) {
        align-items: stretch;
        flex-wrap: wrap;
    }

    :deep(.document-section .staged-details) {
        min-width: calc(100% - 48px);
    }
}
</style>