<template>
    <section>
        <template v-if="show('main')">
        <div class="field-grid">
            <el-form-item label="How much would you like to borrow? (EC$)" required :error="need(draft.requested_loan_amount)">
                <FormField
                    :model-value="draft.requested_loan_amount"
                    :property="fields.requested_loan_amount"
                    :form="draft"
                    @update:model-value="set('requested_loan_amount', $event)"
                />
                <small v-if="selectedProduct" class="helper">
                    Between {{ money(amountMinimum) }} and {{ money(amountMaximum) }}.
                </small>
            </el-form-item>

            <el-form-item label="How many months to pay it back?" required :error="need(draft.requested_loan_term)">
                <FormField
                    :model-value="draft.requested_loan_term"
                    :property="fields.requested_loan_term"
                    :form="draft"
                    @update:model-value="set('requested_loan_term', $event)"
                />
                <small v-if="selectedProduct" class="helper">
                    Between {{ termMinimum }} and {{ termMaximum }} months ({{ yearsLabel(termMinimum) }} to {{ yearsLabel(termMaximum) }}).
                </small>
            </el-form-item>

            <el-form-item label="How often would you like to pay?">
                <FormField
                    :model-value="draft.repayment_frequency"
                    :property="fields.repayment_frequency"
                    :form="draft"
                    @update:model-value="set('repayment_frequency', $event)"
                />
            </el-form-item>
        </div>

        <el-form-item label="What is the loan for?" required :error="need(draft.loan_purpose)">
            <FormField
                :model-value="draft.loan_purpose"
                :property="fields.loan_purpose"
                :form="draft"
                @update:model-value="set('loan_purpose', $event)"
            />
            <small class="helper">A sentence is enough, for example "To buy a used car for work".</small>
        </el-form-item>
        </template>

        <template v-if="show('details')">

        <section v-if="loanCategory === 'auto'" class="context">
            <div class="field-grid">
                <el-form-item label="Make">
                    <FormField
                        :model-value="draft.vehicle_make"
                        :property="fields.vehicle_make"
                        :form="draft"
                        @update:model-value="set('vehicle_make', $event)"
                    />
                </el-form-item>
                <el-form-item label="Model">
                    <FormField
                        :model-value="draft.vehicle_model"
                        :property="fields.vehicle_model"
                        :form="draft"
                        @update:model-value="set('vehicle_model', $event)"
                    />
                </el-form-item>
                <el-form-item label="Year">
                    <FormField
                        :model-value="draft.vehicle_year"
                        :property="fields.vehicle_year"
                        :form="draft"
                        @update:model-value="set('vehicle_year', $event)"
                    />
                </el-form-item>
                <el-form-item label="Condition">
                    <FormField
                        :model-value="draft.vehicle_condition"
                        :property="fields.vehicle_condition"
                        :form="draft"
                        @update:model-value="set('vehicle_condition', $event)"
                    />
                </el-form-item>
                <el-form-item label="Registration number (if it has one)">
                    <FormField
                        :model-value="draft.vehicle_registration_number"
                        :property="fields.vehicle_registration_number"
                        :form="draft"
                        @update:model-value="set('vehicle_registration_number', $event)"
                    />
                </el-form-item>
                <el-form-item label="Chassis number (VIN)">
                    <FormField
                        :model-value="draft.vehicle_chassis_number"
                        :property="fields.vehicle_chassis_number"
                        :form="draft"
                        @update:model-value="set('vehicle_chassis_number', $event)"
                    />
                </el-form-item>
            </div>
        </section>

        <section v-if="loanCategory === 'home'" class="context">
            <el-form-item label="Property address">
                <FormField
                    :model-value="draft.property_address"
                    :property="fields.property_address"
                    :form="draft"
                    @update:model-value="set('property_address', $event)"
                />
            </el-form-item>
            <div class="field-grid">
                <el-form-item label="Property type">
                    <FormField
                        :model-value="draft.property_type"
                        :property="fields.property_type"
                        :form="draft"
                        @update:model-value="set('property_type', $event)"
                    />
                </el-form-item>
                <el-form-item label="Estimated value (EC$)">
                    <FormField
                        :model-value="draft.property_value"
                        :property="fields.property_value"
                        :form="draft"
                        @update:model-value="set('property_value', $event)"
                    />
                </el-form-item>
                <el-form-item label="Block and parcel">
                    <FormField
                        :model-value="draft.property_block_and_parcel"
                        :property="fields.property_block_and_parcel"
                        :form="draft"
                        @update:model-value="set('property_block_and_parcel', $event)"
                    />
                </el-form-item>
                <el-form-item label="Deed number">
                    <FormField
                        :model-value="draft.property_deed_number"
                        :property="fields.property_deed_number"
                        :form="draft"
                        @update:model-value="set('property_deed_number', $event)"
                    />
                </el-form-item>
            </div>
        </section>

        <!--
          Purchase details (auto and home). With a purchase price, the vehicle
          or property being bought is added on the Assets step as collateral.
        -->
        <section v-if="isPurchaseCategory" class="context">
            <h3>Are you buying it?</h3>
            <p class="helper">
                Leave the purchase price blank if you're not buying (for example, a
                refinance). With a price, we'll add the
                {{ loanCategory === 'auto' ? 'vehicle' : 'property' }} to your assets
                as collateral for this loan.
            </p>
            <div class="field-grid">
                <el-form-item label="Purchase price (EC$)">
                    <FormField
                        :model-value="draft.purchase_price"
                        :property="fields.purchase_price"
                        :form="draft"
                        @update:model-value="set('purchase_price', $event)"
                    />
                </el-form-item>
                <el-form-item label="Down payment (EC$)">
                    <FormField
                        :model-value="draft.down_payment_amount"
                        :property="fields.down_payment_amount"
                        :form="draft"
                        @update:model-value="set('down_payment_amount', $event)"
                    />
                </el-form-item>
                <el-form-item
                    v-if="Number(draft.down_payment_amount) > 0"
                    label="Where is the down payment coming from?"
                    required
                    :error="need(draft.source_of_funds)"
                >
                    <FormField
                        :model-value="draft.source_of_funds"
                        :property="fields.source_of_funds"
                        :form="draft"
                        @update:model-value="set('source_of_funds', $event)"
                    />
                </el-form-item>
                <el-form-item label="Seller type">
                    <FormField
                        :model-value="draft.seller_type"
                        :property="fields.seller_type"
                        :form="draft"
                        @update:model-value="set('seller_type', $event)"
                    />
                </el-form-item>
                <el-form-item label="Seller name">
                    <FormField
                        :model-value="draft.seller_name"
                        :property="fields.seller_name"
                        :form="draft"
                        @update:model-value="set('seller_name', $event)"
                    />
                </el-form-item>
            </div>
            <el-form-item
                v-if="Number(draft.down_payment_amount) > 0"
                label="Down payment details"
                :required="String(draft.source_of_funds).toLowerCase() === 'other'"
            >
                <FormField
                    :model-value="draft.source_of_funds_details"
                    :property="fields.source_of_funds_details"
                    :form="draft"
                    @update:model-value="set('source_of_funds_details', $event)"
                />
            </el-form-item>
            <p v-if="loanToValue !== null" class="helper">
                The loan is {{ loanToValue }}% of the purchase price.
            </p>
        </section>

        <!-- The business is saved as its own Party record -->
        <section v-if="loanCategory === 'business'" class="context">
            <div class="field-grid">
                <el-form-item label="Business name" required :error="need(draft.business_name)">
                    <FormField
                        :model-value="draft.business_name"
                        :property="fields.business_name"
                        :form="draft"
                        @update:model-value="set('business_name', $event)"
                    />
                </el-form-item>
                <el-form-item label="Registration number">
                    <FormField
                        :model-value="draft.business_registration_number"
                        :property="fields.business_registration_number"
                        :form="draft"
                        @update:model-value="set('business_registration_number', $event)"
                    />
                </el-form-item>
                <el-form-item label="Business type">
                    <FormField
                        :model-value="draft.business_type"
                        :property="fields.business_type"
                        :form="draft"
                        @update:model-value="set('business_type', $event)"
                    />
                </el-form-item>
                <el-form-item label="Incorporation date">
                    <FormField
                        :model-value="draft.business_incorporation_date"
                        :property="fields.business_incorporation_date"
                        :form="draft"
                        @update:model-value="set('business_incorporation_date', $event, 'date')"
                    />
                </el-form-item>
                <el-form-item label="Number of employees">
                    <FormField
                        :model-value="draft.business_employee_count"
                        :property="fields.business_employee_count"
                        :form="draft"
                        @update:model-value="set('business_employee_count', $event)"
                    />
                </el-form-item>
            </div>
        </section>
        </template>
    </section>
</template>

<script>
/**
 * Every field on this step: [form key, label, input kind, resource, Saturn
 * property]. Most are Application properties. The vehicle and property
 * identifiers are saved on the Asset being bought, and the business details
 * on the business's own Party record, so their FormField uses those
 * resources' definitions.
 */
const FIELDS = [
    ["requested_loan_amount", "Requested amount (EC$)", "number", "Application", "requested_loan_amount"],
    ["requested_loan_term", "Term (months)", "number", "Application", "requested_loan_term"],
    ["repayment_frequency", "Repayment frequency", "select", "Application", "repayment_frequency"],
    ["loan_purpose", "Purpose of the loan", "textarea", "Application", "loan_purpose"],
    ["vehicle_make", "Make", "input", "Application", "vehicle_make"],
    ["vehicle_model", "Model", "input", "Application", "vehicle_model"],
    ["vehicle_year", "Year", "input", "Application", "vehicle_year"],
    ["vehicle_condition", "Condition", "select", "Application", "vehicle_condition"],
    ["vehicle_registration_number", "Registration number", "input", "Asset", "registration_number"],
    ["vehicle_chassis_number", "Chassis number (VIN)", "input", "Asset", "chassis_number"],
    ["property_address", "Property address", "input", "Application", "property_address"],
    ["property_type", "Property type", "select", "Application", "property_type"],
    ["property_value", "Estimated value (EC$)", "number", "Application", "property_value"],
    ["property_block_and_parcel", "Block and parcel", "input", "Asset", "block_and_parcel"],
    ["property_deed_number", "Deed number", "input", "Asset", "deed_number"],
    ["purchase_price", "Purchase price (EC$)", "number", "Application", "purchase_price"],
    ["down_payment_amount", "Down payment (EC$)", "number", "Application", "down_payment_amount"],
    ["source_of_funds", "Where is the down payment coming from?", "select", "Application", "source_of_funds"],
    ["source_of_funds_details", "Down payment details", "textarea", "Application", "source_of_funds_details"],
    ["seller_type", "Seller type", "select", "Application", "seller_type"],
    ["seller_name", "Seller name", "input", "Application", "seller_name"],
    ["business_name", "Business name", "input", "Party", "business_name"],
    ["business_registration_number", "Registration number", "input", "Party", "registration_number"],
    ["business_type", "Business type", "select", "Party", "business_type"],
    ["business_incorporation_date", "Incorporation date", "date", "Party", "incorporation_date"],
    ["business_employee_count", "Number of employees", "number", "Party", "number_of_employees"],
];

/**
 * "Your request" step: amount, term, purpose, and the auto, home, or
 * business details, plus the purchase details for auto and home loans.
 * Every input is Saturn's built-in FormField, using Saturn's own definition
 * of each property when it exists (see FIELDS for which resource).
 *
 * The amount and term limits aren't enforced by the inputs; the main form
 * checks them when the applicant presses Continue.
 */
export default {
    props: {
        modelValue: { type: Object, default: () => ({}) },
        loanCategory: { type: String, default: "" },
        selectedProduct: { type: Object, default: null },
        lookups: { type: Object, default: () => ({}) },
        /** Saturn's property definitions for the Application resource. */
        applicationProps: { type: Array, default: () => [] },
        /** Saturn's property definitions, keyed by resource name. */
        resourceProps: { type: Object, default: () => ({}) },
        /** Which short screen to show: "main" (amount) or "details"; "" shows both. */
        screen: { type: String, default: "" },
        /** After a failed Next, shows "this is needed" under empty required boxes. */
        showErrors: { type: Boolean, default: false },
        amountMinimum: { type: Number, default: 0 },
        amountMaximum: { type: Number, default: 999999999 },
        termMinimum: { type: Number, default: 1 },
        termMaximum: { type: Number, default: 600 },
    },
    emits: ["update:modelValue"],
    data() {
        return { draft: this.copy(this.modelValue) };
    },
    computed: {
        /**
         * FormField property configs keyed by form key. Built once (and
         * again only if the props change), so FormField isn't handed a new
         * object on every keystroke.
         */
        fields() {
            const result = {};
            FIELDS.forEach((row) => {
                result[row[0]] = this.field(row[3], row[4], row[1], row[2]);
            });
            return result;
        },

        isPurchaseCategory() {
            return this.loanCategory === "auto" || this.loanCategory === "home";
        },

        /** The requested amount as a % of the purchase price (loan-to-value). */
        loanToValue() {
            const price = Number(this.draft.purchase_price);
            const amount = Number(this.draft.requested_loan_amount);
            if (!(price > 0) || !(amount > 0)) return null;
            return Math.round((amount / price) * 1000) / 10;
        },
    },
    watch: {
        modelValue: {
            deep: true,
            handler(value) {
                this.draft = this.copy(value);
            },
        },
    },
    methods: {
        copy(value) {
            return JSON.parse(JSON.stringify(value || {}));
        },

        /**
         * FormField's update event may send the value itself or
         * { property, data } (the Saturn guide isn't clear), so accept both.
         * Dates are kept as YYYY-MM-DD strings.
         */
        valueOf(event, kind) {
            let value = event;
            if (
                value &&
                typeof value === "object" &&
                "property" in value &&
                "data" in value
            ) {
                value = value.data;
            }
            if (kind !== "date") return value;
            if (!value) return "";
            if (value instanceof Date) {
                const pad = (number) => String(number).padStart(2, "0");
                return `${value.getFullYear()}-${pad(value.getMonth() + 1)}-${pad(value.getDate())}`;
            }
            return String(value).slice(0, 10);
        },

        set(key, event, kind) {
            this.draft[key] = this.valueOf(event, kind);
            this.$emit("update:modelValue", this.copy(this.draft));
        },

        /** Saturn's definition of one property of a resource, or null. */
        savedProperty(resourceName, name) {
            const rows =
                resourceName === "Application" && this.applicationProps.length
                    ? this.applicationProps
                    : this.resourceProps[resourceName] || [];
            return (
                rows.find(
                    (row) =>
                        String(row.property || row.key || row.name || "") === name,
                ) || null
            );
        },

        /**
         * The FormField property config for one field. Uses Saturn's own
         * definition (with our label) when there is one; otherwise builds
         * one of the kinds shown in the Saturn guide. FormField's own option
         * lists (lookup_type "values") don't work in Saturn, so a dropdown
         * with no Saturn definition is a text box.
         */
        field(resourceName, name, label, kind) {
            const saved = this.savedProperty(resourceName, name);
            if (saved) {
                return Object.assign({}, saved, { property: name, label });
            }
            if (kind === "number" || kind === "date") {
                return { property: name, label, type: kind };
            }
            return {
                property: name,
                label,
                type: "string",
                input_properties: {
                    type: kind === "textarea" ? "textarea" : "input",
                },
            };
        },

        /** True when this part is on screen. */
        show(part) {
            return !this.screen || this.screen === part;
        },

        /** "This is needed" under an empty required box, after a failed Next. */
        need(value) {
            if (!this.showErrors) return "";
            return value === null || value === undefined || value === ""
                ? "This is needed"
                : "";
        },

        /** Months as years, e.g. 60 -> "5 years". */
        yearsLabel(months) {
            const years = Math.round((Number(months) / 12) * 10) / 10;
            return `${years} ${years === 1 ? "year" : "years"}`;
        },

        money(value) {
            return `EC$ ${Number(value).toLocaleString(undefined, { minimumFractionDigits: 2, maximumFractionDigits: 2 })}`;
        },
    },
};
</script>
