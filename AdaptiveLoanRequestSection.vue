<template>
    <section>
        <div class="field-grid">
            <el-form-item label="Requested amount (EC$)" required
                ><el-input-number
                    :model-value="draft.requested_loan_amount"
                    :min="amountMinimum"
                    :max="amountMaximum"
                    :step="100"
                    controls-position="right"
                    @update:model-value="set('requested_loan_amount', $event)"
                /><small v-if="selectedProduct" class="helper"
                    >Allowed: {{ money(amountMinimum) }} to
                    {{ money(amountMaximum) }}</small
                ></el-form-item
            ><el-form-item label="Term (months)" required
                ><el-input-number
                    :model-value="draft.requested_loan_term"
                    :min="termMinimum"
                    :max="termMaximum"
                    controls-position="right"
                    @update:model-value="set('requested_loan_term', $event)"
                /><small v-if="selectedProduct" class="helper"
                    >Allowed: {{ termMinimum }} to
                    {{ termMaximum }} months</small
                ></el-form-item
            ><el-form-item label="Repayment frequency"
                ><el-select
                    :model-value="draft.repayment_frequency"
                    placeholder="Select frequency"
                    @update:model-value="set('repayment_frequency', $event)"
                    ><el-option
                        v-for="option in lookup('repayment_frequency')"
                        :key="option.value"
                        :label="option.label"
                        :value="option.value" /></el-select
            ></el-form-item>
        </div>
        <el-form-item label="Purpose of the loan" required
            ><el-input
                :model-value="draft.loan_purpose"
                type="textarea"
                :rows="4"
                @input="set('loan_purpose', $event)"
        /></el-form-item>
        <section v-if="loanCategory === 'auto'" class="context">
            <h3>Vehicle details</h3>
            <div class="field-grid">
                <el-form-item label="Make"
                    ><el-input
                        :model-value="draft.vehicle_make"
                        @input="set('vehicle_make', $event)" /></el-form-item
                ><el-form-item label="Model"
                    ><el-input
                        :model-value="draft.vehicle_model"
                        @input="set('vehicle_model', $event)" /></el-form-item
                ><el-form-item label="Year"
                    ><el-input
                        :model-value="draft.vehicle_year"
                        @input="set('vehicle_year', $event)" /></el-form-item
                ><el-form-item label="Condition"
                    ><el-select
                        :model-value="draft.vehicle_condition"
                        @update:model-value="set('vehicle_condition', $event)"
                        ><el-option
                            v-for="option in lookup('vehicle_condition')"
                            :key="option.value"
                            :label="option.label"
                            :value="option.value" /></el-select
                ></el-form-item>
            </div>
        </section>
        <section v-if="loanCategory === 'home'" class="context">
            <h3>Property details</h3>
            <el-form-item label="Property address"
                ><el-input
                    :model-value="draft.property_address"
                    @input="set('property_address', $event)"
            /></el-form-item>
            <div class="field-grid">
                <el-form-item label="Property type"
                    ><el-select
                        :model-value="draft.property_type"
                        @update:model-value="set('property_type', $event)"
                        ><el-option
                            v-for="option in lookup('property_type')"
                            :key="option.value"
                            :label="option.label"
                            :value="option.value" /></el-select></el-form-item
                ><el-form-item label="Estimated value (EC$)"
                    ><el-input-number
                        :model-value="draft.property_value"
                        :min="0"
                        @update:model-value="set('property_value', $event)"
                /></el-form-item>
            </div>
        </section>
        <section v-if="loanCategory === 'business'" class="context">
            <h3>Business details</h3>
            <div class="field-grid">
                <el-form-item label="Business name"
                    ><el-input
                        :model-value="draft.business_name"
                        @input="set('business_name', $event)" /></el-form-item
                ><el-form-item label="Registration number"
                    ><el-input
                        :model-value="draft.business_registration_number"
                        @input="
                            set('business_registration_number', $event)
                        " /></el-form-item
                ><el-form-item label="Business type"
                    ><el-select
                        :model-value="draft.business_type"
                        @update:model-value="set('business_type', $event)"
                        ><el-option
                            v-for="option in lookup('business_type')"
                            :key="option.value"
                            :label="option.label"
                            :value="option.value" /></el-select></el-form-item
                ><el-form-item label="Incorporation date"
                    ><el-date-picker
                        :model-value="draft.business_incorporation_date"
                        type="date"
                        value-format="YYYY-MM-DD"
                        @update:model-value="
                            set('business_incorporation_date', $event)
                        " /></el-form-item
                ><el-form-item label="Number of employees"
                    ><el-input-number
                        :model-value="draft.business_employee_count"
                        :min="0"
                        @update:model-value="
                            set('business_employee_count', $event)
                        "
                /></el-form-item>
            </div>
        </section>
    </section>
</template>
<script>
export default {
    props: {
        modelValue: { type: Object, default: () => ({}) },
        loanCategory: { type: String, default: "" },
        selectedProduct: { type: Object, default: null },
        lookups: { type: Object, default: () => ({}) },
        amountMinimum: { type: Number, default: 0 },
        amountMaximum: { type: Number, default: 999999999 },
        termMinimum: { type: Number, default: 1 },
        termMaximum: { type: Number, default: 600 },
    },
    emits: ["update:modelValue"],
    data() {
        return { draft: this.copy(this.modelValue) };
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
        set(key, value) {
            this.draft[key] = value;
            this.$emit("update:modelValue", this.copy(this.draft));
        },
        lookup(key) {
            return this.lookups[key] || [];
        },
        money(value) {
            return `EC$ ${Number(value).toLocaleString(undefined, { minimumFractionDigits: 2, maximumFractionDigits: 2 })}`;
        },
    },
};
</script>
