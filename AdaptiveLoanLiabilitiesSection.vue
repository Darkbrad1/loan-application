<template>
  <section>
    <!-- Section header with add button -->
    <div class="collection-header">
      <div>
        <h3>Liabilities</h3>
        <p>Responsibility is allocated to saved ApplicationParty records.</p>
      </div>
      <el-button type="primary" plain @click="add">
        <v-icon start>mdi-plus</v-icon>
        Add liability
      </el-button>
    </div>

    <el-empty v-if="!draft.length" description="No liabilities declared" />

    <!-- One card per declared liability -->
    <article
      v-for="(item, index) in draft"
      :key="item.client_key"
      class="item-card"
    >
      <div class="item-title">
        <strong>{{ item.creditor_name || `Liability ${index + 1}` }}</strong>
        <el-button text type="danger" @click="remove(index)">Remove</el-button>
      </div>

      <div class="field-grid">
        <el-form-item label="Creditor" required>
          <FormField
            :model-value="item.creditor_name"
            :property="field('Liability', 'creditor_name', 'Creditor', 'input')"
            :form="item"
            @update:model-value="set(index, 'creditor_name', $event)"
          />
        </el-form-item>

        <!-- Options come from LiabilityType records; value is the record ID -->
        <el-form-item label="Liability type" required>
          <el-select
            :model-value="item.liability_type"
            placeholder="Select liability type"
            @update:model-value="set(index, 'liability_type', $event)"
          >
            <el-option
              v-for="option in liabilityTypeOptions"
              :key="option.value"
              :label="option.label"
              :value="option.value"
            />
          </el-select>
          <small
            v-if="item.liability_type && !typeOf(item)"
            class="helper invalid"
          >
            This type is no longer available. Choose another.
          </small>
        </el-form-item>

        <el-form-item label="Outstanding balance (EC$)" required>
          <FormField
            :model-value="item.outstanding_balance"
            :property="field('Liability', 'outstanding_balance', 'Outstanding balance (EC$)', 'number')"
            :form="item"
            @update:model-value="set(index, 'outstanding_balance', $event)"
          />
        </el-form-item>

        <!-- Revolving credit (credit cards, overdrafts) needs its limit -->
        <el-form-item
          v-if="isRevolving(item)"
          label="Credit limit (EC$)"
          required
        >
          <FormField
            :model-value="item.credit_limit"
            :property="field('Liability', 'credit_limit', 'Credit limit (EC$)', 'number')"
            :form="item"
            @update:model-value="set(index, 'credit_limit', $event)"
          />
        </el-form-item>

        <el-form-item label="Payment amount (EC$)" required>
          <FormField
            :model-value="item.payment_amount"
            :property="field('Liability', 'payment_amount', 'Payment amount (EC$)', 'number')"
            :form="item"
            @update:model-value="set(index, 'payment_amount', $event)"
          />
        </el-form-item>

        <el-form-item label="Payment frequency">
          <FormField
            :model-value="item.payment_frequency"
            :property="field('Liability', 'payment_frequency', 'Payment frequency', 'select')"
            :form="item"
            @update:model-value="set(index, 'payment_frequency', $event)"
          />
        </el-form-item>

        <!--
          Assessed repayment preview: a fixed % of the credit limit rather
          than the outstanding balance. The server recalculates this figure.
        -->
        <p v-if="isRevolving(item)" class="assessed-note">
          <template v-if="assessed(item) !== null">
            Assessed monthly repayment: {{ money(assessed(item)) }}
            ({{ percentLabel(rateFor(item)) }} of the credit limit)
          </template>
          <template v-else>
            Enter the credit limit. Repayment will be assessed at
            {{ percentLabel(rateFor(item)) }} of the limit.
          </template>
        </p>
      </div>

      <!-- Responsibility percentages must total 100% across applicants -->
      <AdaptiveLoanAllocationEditor
        :model-value="item.responsibilities"
        :options="applicationPartyOptions"
        reference-key="application_party_id"
        title="Responsibility"
        select-label="Responsible applicant"
        total-label="Responsibility total"
        responsibility-type="Borrower"
        @update:model-value="set(index, 'responsibilities', $event)"
      />

      <!-- Required documents for this liability; uploads unlock once it's complete -->
      <AdaptiveLoanDocumentRequirements
        :scope="documentScopes[`liability:${item.client_key}`]"
        :uploading-key="uploadingKey"
        :disabled="documentsDisabled"
        @stage-file="$emit('stage-file', $event)"
        @remove-file="$emit('remove-file', $event)"
        @request-file-upload="$emit('request-file-upload', $event)"
        @file-rejected="$emit('file-rejected', $event)"
      />
    </article>
  </section>
</template>

<script>
/**
 * Liabilities step of the loan wizard. Liability types are LiabilityType
 * records (normalized by the parent). Revolving types require a credit
 * limit and show an assessed repayment of limit × rate, where the rate is
 * the type's revolving_rate or the institution default.
 *
 * Uses the local draft pattern: edits happen on a deep-cloned copy and are
 * committed upward via update:modelValue.
 *
 * Every input is Saturn's built-in FormField, using Saturn's own
 * definitions of the Liability properties when they exist. Liability type
 * is an el-select of the LiabilityType records passed in by the parent.
 */
export default {
  props: {
    /** Liabilities array owned by the parent form. */
    modelValue: {
      type: Array,
      default: () => [],
    },
    /** Select options for responsibility, keyed by ApplicationParty ID. */
    applicationPartyOptions: {
      type: Array,
      default: () => [],
    },
    /**
     * Active LiabilityType records. The parent normalizes is_revolving to a
     * boolean and revolving_rate to a fraction (e.g. 0.03) or null.
     */
    liabilityTypes: {
      type: Array,
      default: () => [],
    },
    /** Fallback % of limit (as a fraction) when a type has no rate. */
    defaultRevolvingRate: {
      type: Number,
      default: 0.03,
    },
    /** Document scopes from the parent, keyed by scope key. */
    documentScopes: {
      type: Object,
      default: () => ({}),
    },
    /** Key of the upload in progress, passed through to the requirements. */
    uploadingKey: {
      type: String,
      default: '',
    },
    documentsDisabled: {
      type: Boolean,
      default: false,
    },
    /** Dropdown option lists keyed by field name (payment_frequency). */
    lookups: {
      type: Object,
      default: () => ({}),
    },
    /** Saturn's property definitions, keyed by resource name. */
    resourceProps: {
      type: Object,
      default: () => ({}),
    },
  },

  emits: [
    'update:modelValue',
    // Document events are passed straight through to the parent form.
    'stage-file',
    'remove-file',
    'request-file-upload',
    'file-rejected',
  ],

  data() {
    return {
      // Local working copy; synced back to the parent on every change.
      draft: this.copy(this.modelValue),
    };
  },

  created() {
    // FormField configs, reused while unchanged (see field()).
    this.fieldCache = {};
  },

  computed: {
    /** LiabilityType records as dropdown options (value is the record ID). */
    liabilityTypeOptions() {
      return this.liabilityTypes.map((type) => ({
        label: type.name,
        value: this.typeId(type),
      }));
    },
  },

  watch: {
    // Keep the draft in sync if the parent replaces the array
    // (e.g. when a saved draft is restored from the server).
    modelValue: {
      deep: true,
      handler(value) {
        this.draft = this.copy(value);
      },
    },
  },

  methods: {
    /** Deep-clones an array so edits never mutate the parent's state. */
    copy(value) {
      return JSON.parse(JSON.stringify(value || []));
    },

    /** Unique client-side key for rows that have no server ID yet. */
    key(prefix) {
      return `${prefix}_${Date.now()}_${Math.random().toString(36).slice(2, 8)}`;
    },

    /** Dropdown options for a field, or [] when none were loaded. */
    lookup(fieldName) {
      return this.lookups[fieldName] || [];
    },

    /** Normalizes an ID reference (raw ID or record object). */
    typeId(value) {
      if (!value) return null;
      if (typeof value !== 'object') return value;
      return value.id || value.data?.id || null;
    },

    /** The LiabilityType record selected on a liability, or null. */
    typeOf(item) {
      const id = this.typeId(item.liability_type);
      return this.liabilityTypes.find((type) => this.typeId(type) === id) || null;
    },

    isRevolving(item) {
      return Boolean(this.typeOf(item)?.is_revolving);
    },

    /** Rate for a revolving liability: the type's own rate or the default. */
    rateFor(item) {
      return this.typeOf(item)?.revolving_rate || this.defaultRevolvingRate;
    },

    /** Assessed repayment preview (limit × rate), or null without a limit. */
    assessed(item) {
      const limit = Number(item.credit_limit);
      if (!this.isRevolving(item) || !(limit > 0)) return null;
      return Math.round(limit * this.rateFor(item) * 100) / 100;
    },

    percentLabel(rate) {
      return `${Number((rate * 100).toFixed(2))}%`;
    },

    money(value) {
      return `EC$ ${Number(value).toLocaleString(undefined, {
        minimumFractionDigits: 2,
        maximumFractionDigits: 2,
      })}`;
    },

    // ---- Saturn FormField helpers (the same in every section) ----

    /**
     * FormField's update event may send the value itself or
     * { property, data } (the Saturn guide isn't clear), so accept both.
     * Dates are kept as YYYY-MM-DD strings.
     */
    valueOf(event, kind) {
      let value = event;
      if (value && typeof value === 'object' && 'property' in value && 'data' in value) {
        value = value.data;
      }
      return kind === 'date' ? this.toDateString(value) : value;
    },

    /** A date as YYYY-MM-DD (the local date for Date objects), or ''. */
    toDateString(value) {
      if (!value) return '';
      if (value instanceof Date) {
        const pad = (number) => String(number).padStart(2, '0');
        return `${value.getFullYear()}-${pad(value.getMonth() + 1)}-${pad(value.getDate())}`;
      }
      return String(value).slice(0, 10);
    },

    /** Saturn's definition of one property of a resource, or null. */
    savedProperty(resourceName, name) {
      const rows = this.resourceProps[resourceName] || [];
      return rows.find((row) => String(row.property || row.key || row.name || '') === name) || null;
    },

    /**
     * FormField property config for one field. Uses Saturn's own definition
     * of the property (with our label) when there is one. Otherwise builds a
     * basic one from the Saturn guide; a dropdown becomes a text box, since
     * FormField's own option lists (lookup_type "values") don't work in
     * Saturn. Dropdowns whose choices the form decides use el-select
     * instead. Configs are reused while unchanged, so FormField isn't handed
     * a new object on every keystroke.
     */
    field(resourceName, name, label, kind) {
      const saved = this.savedProperty(resourceName, name);
      let config;
      if (saved) {
        config = Object.assign({}, saved, { property: name, label });
      } else if (kind === 'number' || kind === 'date') {
        config = { property: name, label, type: kind };
      } else if (kind === 'checkbox') {
        config = { property: name, label, type: 'boolean', input_properties: { type: 'check-box' } };
      } else {
        config = {
          property: name,
          label,
          type: 'string',
          input_properties: { type: kind === 'textarea' ? 'textarea' : 'input' },
        };
      }
      const cacheKey = JSON.stringify(config);
      if (!this.fieldCache[cacheKey]) this.fieldCache[cacheKey] = config;
      return this.fieldCache[cacheKey];
    },

    /** Pushes the current draft up to the parent. */
    notify() {
      this.$emit('update:modelValue', this.copy(this.draft));
    },

    /**
     * Updates one field on one liability and notifies the parent. Switching
     * to a non-revolving type clears the limit so stale values aren't saved.
     */
    set(index, fieldName, event) {
      const item = this.draft[index];
      item[fieldName] = this.valueOf(event);

      if (fieldName === 'liability_type' && !this.isRevolving(item)) {
        item.credit_limit = null;
        item.assessed_payment = null;
      }

      this.notify();
    },

    /** Appends a blank liability with a single 100% responsibility row. */
    add() {
      this.draft.push({
        client_key: this.key('liability'),
        id: null,
        creditor_name: '',
        liability_type: '',
        outstanding_balance: null,
        payment_amount: null,
        payment_frequency: '',
        credit_limit: null,
        assessed_payment: null,
        document_ids: [],
        responsibilities: [
          {
            client_key: this.key('resp'),
            id: null,
            application_party_id: '',
            percentage: 100,
            responsibility_type: 'Borrower',
          },
        ],
      });
      this.notify();
    },

    remove(index) {
      this.draft.splice(index, 1);
      this.notify();
    },
  },
};
</script>