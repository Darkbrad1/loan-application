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
          <el-select
            :model-value="item.application_party_id"
            placeholder="Select applicant"
            @update:model-value="set(index, 'application_party_id', $event)"
          >
            <el-option
              v-for="option in applicationPartyOptions"
              :key="option.value"
              :label="option.label"
              :value="option.value"
            />
          </el-select>
        </el-form-item>

        <!--
          Options are ExpenseType records filtered by the parent to the
          selected loan category. Grouped when the types define a group.
        -->
        <el-form-item label="Expense type" required>
          <el-select
            :model-value="item.expense_type"
            placeholder="Select expense type"
            @update:model-value="set(index, 'expense_type', $event)"
          >
            <!-- Keeps a no-longer-applicable value readable instead of showing its ID -->
            <el-option
              v-if="isInapplicable(item)"
              :key="`current-${item.expense_type}`"
              :label="typeName(item.expense_type) || 'Unavailable type'"
              :value="item.expense_type"
              disabled
            />

            <template v-if="hasGroups">
              <el-option-group
                v-for="group in groupedOptions"
                :key="group.label"
                :label="group.label"
              >
                <el-option
                  v-for="option in group.options"
                  :key="option.value"
                  :label="option.label"
                  :value="option.value"
                />
              </el-option-group>
            </template>
            <template v-else>
              <el-option
                v-for="option in expenseTypeOptions"
                :key="option.value"
                :label="option.label"
                :value="option.value"
              />
            </template>
          </el-select>
          <small v-if="isInapplicable(item)" class="helper invalid">
            This expense doesn't apply to a {{ loanCategoryLabel }}. Choose
            another type or remove it.
          </small>
        </el-form-item>

        <el-form-item label="Expense name">
          <el-input
            :model-value="item.expense_name"
            :placeholder="typeName(item.expense_type) || 'Optional description'"
            @input="set(index, 'expense_name', $event)"
          />
        </el-form-item>

        <el-form-item label="Amount (EC$)" required>
          <el-input-number
            :model-value="item.amount"
            :min="0"
            @update:model-value="set(index, 'amount', $event)"
          />
        </el-form-item>

        <el-form-item label="Frequency">
          <el-select
            :model-value="item.frequency"
            placeholder="Select frequency"
            @update:model-value="set(index, 'frequency', $event)"
          >
            <el-option
              v-for="option in lookup('expense_frequency')"
              :key="option.value"
              :label="option.label"
              :value="option.value"
            />
          </el-select>
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

    hasGroups() {
      return this.expenseTypeOptions.some((option) => option.group);
    },

    /** Options grouped by ExpenseType.group, in first-seen order. */
    groupedOptions() {
      const groups = [];
      this.expenseTypeOptions.forEach((option) => {
        const label = option.group || 'Other';
        let group = groups.find((entry) => entry.label === label);
        if (!group) {
          group = { label, options: [] };
          groups.push(group);
        }
        group.options.push(option);
      });
      return groups;
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

    title(item, index) {
      return item.expense_name || this.typeName(item.expense_type) || `Expense ${index + 1}`;
    },

    /** Pushes the current draft up to the parent. */
    notify() {
      this.$emit('update:modelValue', this.copy(this.draft));
    },

    /** Updates one field on one expense and notifies the parent. */
    set(index, fieldName, value) {
      this.draft[index][fieldName] = value;
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