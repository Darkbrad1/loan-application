<template>
  <section>
    <!-- Section header with add button -->
    <div class="collection-header">
      <div>
        <h3>Expenses</h3>
        <p>Each expense is assigned to an ApplicationParty.</p>
      </div>
      <el-button type="primary" plain @click="add">
        <v-icon start>mdi-plus</v-icon>
        Add expense
      </el-button>
    </div>

    <el-empty v-if="!draft.length" description="No expenses declared" />

    <!-- One card per declared expense -->
    <article
      v-for="(item, index) in draft"
      :key="item.client_key"
      class="item-card"
    >
      <div class="item-title">
        <strong>{{ title(item, index) }}</strong>
        <el-button text type="danger" @click="remove(index)">Remove</el-button>
      </div>

      <div class="field-grid">
        <el-form-item label="Responsible applicant" required>
          <FormField
            :model-value="item.application_party_id"
            :property="field('Expense', 'applicationpartiesid', 'Responsible applicant', 'select', { options: applicationPartyOptions, force: true })"
            :form="item"
            @update:model-value="set(index, 'application_party_id', $event)"
          />
        </el-form-item>

        <!--
          Options are ExpenseType records filtered by the parent to the
          selected loan category, sorted by group.
        -->
        <el-form-item label="Expense type" required>
          <FormField
            :model-value="item.expense_type"
            :property="field('Expense', 'expense_type', 'Expense type', 'select', { options: typeOptionsFor(item), force: true })"
            :form="item"
            @update:model-value="set(index, 'expense_type', $event)"
          />
          <small v-if="isInapplicable(item)" class="helper invalid">
            This expense doesn't apply to a {{ loanCategoryLabel }}. Choose
            another type or remove it.
          </small>
        </el-form-item>

        <el-form-item label="Expense name">
          <FormField
            :model-value="item.expense_name"
            :property="field('Expense', 'expense_name', 'Expense name', 'input')"
            :form="item"
            @update:model-value="set(index, 'expense_name', $event)"
          />
        </el-form-item>

        <el-form-item label="Amount (EC$)" required>
          <FormField
            :model-value="item.amount"
            :property="field('Expense', 'amount', 'Amount (EC$)', 'number')"
            :form="item"
            @update:model-value="set(index, 'amount', $event)"
          />
        </el-form-item>

        <el-form-item label="Frequency">
          <FormField
            :model-value="item.frequency"
            :property="field('Expense', 'frequency', 'Frequency', 'select', { options: lookup('expense_frequency') })"
            :form="item"
            @update:model-value="set(index, 'frequency', $event)"
          />
        </el-form-item>
      </div>

      <!-- Required documents for this expense; uploads unlock once it's complete -->
      <AdaptiveLoanDocumentRequirements
        :scope="documentScopes[`expense:${item.client_key}`]"
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
 * Expenses step of the loan wizard. Expense types are ExpenseType records.
 * The parent passes the options that apply to the selected loan category
 * (expenseTypeOptions) plus every active type (expenseTypes) so existing
 * expenses whose type no longer applies can still be labelled and flagged.
 *
 * Uses the local draft pattern: edits happen on a deep-cloned copy and are
 * committed upward via update:modelValue.
 *
 * Every input is Saturn's built-in FormField, using Saturn's own
 * definitions of the Expense properties when they exist. The responsible
 * applicant and expense type always use the options from the parent.
 */
export default {
  props: {
    /** Expenses array owned by the parent form. */
    modelValue: {
      type: Array,
      default: () => [],
    },
    /** Select options for the responsible applicant, keyed by ApplicationParty ID. */
    applicationPartyOptions: {
      type: Array,
      default: () => [],
    },
    /** Selectable types for this loan category: { value, label, group }. */
    expenseTypeOptions: {
      type: Array,
      default: () => [],
    },
    /** All active ExpenseType records, used for labels. */
    expenseTypes: {
      type: Array,
      default: () => [],
    },
    /** e.g. "personal loan", used in messages. */
    loanCategoryLabel: {
      type: String,
      default: 'this loan',
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
    /** Dropdown option lists keyed by field name (expense_frequency). */
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

  computed: {
    /** IDs of types the applicant may currently choose. */
    allowedTypeIds() {
      return new Set(this.expenseTypeOptions.map((option) => option.value));
    },

    /**
     * Selectable types as dropdown options, with types in the same
     * ExpenseType.group kept together (in first-seen order).
     */
    sortedTypeOptions() {
      const groups = [];
      this.expenseTypeOptions.forEach((option) => {
        if (!groups.includes(option.group || '')) groups.push(option.group || '');
      });
      const rows = [];
      groups.forEach((group) => {
        this.expenseTypeOptions
          .filter((option) => (option.group || '') === group)
          .forEach((option) => rows.push({ label: option.label, value: option.value }));
      });
      return rows;
    },
  },

  created() {
    // FormField configs, reused while unchanged (see field()).
    this.fieldCache = {};
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

    /** Display name of an ExpenseType by ID. */
    typeName(value) {
      const id = this.typeId(value);
      if (!id) return '';
      const type = this.expenseTypes.find((entry) => this.typeId(entry) === id);
      return type ? type.name : '';
    },

    /**
     * True when an expense has a type that isn't selectable for this loan
     * category (e.g. a business expense after switching to a personal loan).
     * System-generated types that aren't user-selectable but still apply
     * are validated by the parent, not flagged here.
     */
    isInapplicable(item) {
      const id = this.typeId(item.expense_type);
      if (!id || this.allowedTypeIds.has(id)) return false;

      const type = this.expenseTypes.find((entry) => this.typeId(entry) === id);
      if (type && type.user_selectable === false) return false;
      return true;
    },

    /**
     * Type options for one expense. A type that no longer applies stays in
     * the list (marked unavailable) so it shows its name, not its ID.
     */
    typeOptionsFor(item) {
      if (!this.isInapplicable(item)) return this.sortedTypeOptions;
      return [
        {
          label: `${this.typeName(item.expense_type) || 'Unavailable type'} (not available)`,
          value: item.expense_type,
        },
      ].concat(this.sortedTypeOptions);
    },

    title(item, index) {
      return item.expense_name || this.typeName(item.expense_type) || `Expense ${index + 1}`;
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
     * of the property (with our label) when there is one, unless
     * extra.force is set (for dropdowns whose options the form decides).
     * Otherwise builds a basic one from the Saturn guide; a dropdown with
     * no options becomes a text box. Configs are reused while unchanged, so
     * FormField isn't handed a new object on every keystroke.
     */
    field(resourceName, name, label, kind, extra) {
      const options = (extra && extra.options) || [];
      const saved = extra && extra.force ? null : this.savedProperty(resourceName, name);
      let config;
      if (saved) {
        config = Object.assign({}, saved, { property: name, label });
      } else if (kind === 'number' || kind === 'date') {
        config = { property: name, label, type: kind };
      } else if (kind === 'checkbox') {
        config = { property: name, label, type: 'boolean', input_properties: { type: 'check-box' } };
      } else if (kind === 'select' && options.length) {
        config = { property: name, label, type: 'string', lookup_type: 'values', map: { values: options } };
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

    /** Updates one field on one expense and notifies the parent. */
    set(index, fieldName, event) {
      this.draft[index][fieldName] = this.valueOf(event);
      this.notify();
    },

    /** Appends a blank, unassigned expense. */
    add() {
      this.draft.push({
        client_key: this.key('expense'),
        id: null,
        application_party_id: '',
        expense_name: '',
        expense_type: '',
        amount: null,
        frequency: '',
        document_ids: [],
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