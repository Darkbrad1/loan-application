<template>
    <section>
        <div class="field-grid">
            <el-form-item label="Requested amount (EC$)" required>
                <FormField
                    :model-value="draft.requested_loan_amount"
                    :property="fields.requested_loan_amount"
                    :form="draft"
                    @update:model-value="set('requested_loan_amount', $event)"
                />
                <small v-if="selectedProduct" class="helper">
                    Allowed: {{ money(amountMinimum) }} to {{ money(amountMaximum) }}
                </small>
            </el-form-item>

            <el-form-item label="Term (months)" required>
                <FormField
                    :model-value="draft.requested_loan_term"
                    :property="fields.requested_loan_term"
                    :form="draft"
                    @update:model-value="set('requested_loan_term', $event)"
                />
                <small v-if="selectedProduct" class="helper">
                    Allowed: {{ termMinimum }} to {{ termMaximum }} months
                </small>
            </el-form-item>

            <el-form-item label="Repayment frequency">
                <FormField
                    :model-value="draft.repayment_frequency"
                    :property="fields.repayment_frequency"
                    :form="draft"
                    @update:model-value="set('repayment_frequency', $event)"
                />
            </el-form-item>
        </div>

        <el-form-item label="Purpose of the loan" required>
            <FormField
                :model-value="draft.loan_purpose"
                :property="fields.loan_purpose"
                :form="draft"
                @update:model-value="set('loan_purpose', $event)"
            />
        </el-form-item>

        <section v-if="loanCategory === 'auto'" class="context">
            <h3>Vehicle details</h3>
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
            </div>
        </section>

        <section v-if="loanCategory === 'home'" class="context">
            <h3>Property details</h3>
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
            </div>
        </section>

        <section v-if="loanCategory === 'business'" class="context">
            <h3>Business details</h3>
            <div class="field-grid">
                <el-form-item label="Business name">
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
                        @update:model-value="set('business_incorporation_date', $event)"
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
    </section>
</template>

<script>
/**
 * "Your request" step: amount, term, purpose, and the auto, home, or
 * business details. Every input is Saturn's built-in FormField.
 *
 * Each field uses Saturn's own definition of that Application property
 * (from loadResourceProps) when it exists, so its input type and dropdown
 * options follow the resource settings in Saturn. If a property isn't
 * found, a basic definition is built here instead, using the dropdown
 * options the main form already loaded.
 *
 * The amount and term limits aren't enforced by the inputs; the main form
 * checks them when the applicant presses Continue.
 */
/** Every field on this step: [property name, label, input kind]. */
const FIELDS = [
    ["requested_loan_amount", "Requested amount (EC$)", "number"],
    ["requested_loan_term", "Term (months)", "number"],
    ["repayment_frequency", "Repayment frequency", "select"],
    ["loan_purpose", "Purpose of the loan", "textarea"],
    ["vehicle_make", "Make", "input"],
    ["vehicle_model", "Model", "input"],
    ["vehicle_year", "Year", "input"],
    ["vehicle_condition", "Condition", "select"],
    ["property_address", "Property address", "input"],
    ["property_type", "Property type", "select"],
    ["property_value", "Estimated value (EC$)", "number"],
    ["business_name", "Business name", "input"],
    ["business_registration_number", "Registration number", "input"],
    ["business_type", "Business type", "select"],
    ["business_incorporation_date", "Incorporation date", "date"],
    ["business_employee_count", "Number of employees", "number"],
];

export default {
    props: {
        modelValue: { type: Object, default: () => ({}) },
        loanCategory: { type: String, default: "" },
        selectedProduct: { type: Object, default: null },
        lookups: { type: Object, default: () => ({}) },
        /** Saturn's property definitions for the Application resource. */
        applicationProps: { type: Array, default: () => [] },
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
         * FormField property configs keyed by field name. Built once (and
         * again only if the props or lookups change), so FormField isn't
         * handed a new object on every keystroke.
         */
        fields() {
            const result = {};
            FIELDS.forEach((row) => {
                result[row[0]] = this.field(row[0], row[1], row[2]);
            });
            return result;
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
         */
        valueOf(event) {
            if (
                event &&
                typeof event === "object" &&
                "property" in event &&
                "data" in event
            ) {
                return event.data;
            }
            return event;
        },

        set(key, event) {
            this.draft[key] = this.valueOf(event);
            this.$emit("update:modelValue", this.copy(this.draft));
        },

        lookup(key) {
            return this.lookups[key] || [];
        },

        /** Saturn's definition of an Application property, or null. */
        savedProperty(name) {
            return (
                this.applicationProps.find(
                    (row) =>
                        String(row.property || row.key || row.name || "") === name,
                ) || null
            );
        },

        /**
         * The FormField property config for one field. Uses Saturn's own
         * definition (with our label) when there is one; otherwise builds
         * one of the kinds shown in the Saturn guide.
         */
        field(name, label, kind) {
            const saved = this.savedProperty(name);
            if (saved) {
                return Object.assign({}, saved, { property: name, label });
            }

            if (kind === "number") {
                return { property: name, label, type: "number" };
            }
            if (kind === "date") {
                return { property: name, label, type: "date" };
            }
            if (kind === "select") {
                return {
                    property: name,
                    label,
                    type: "string",
                    lookup_type: "values",
                    map: { values: this.lookup(name) },
                };
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

        money(value) {
            return `EC$ ${Number(value).toLocaleString(undefined, { minimumFractionDigits: 2, maximumFractionDigits: 2 })}`;
        },
    },
};
</script>
