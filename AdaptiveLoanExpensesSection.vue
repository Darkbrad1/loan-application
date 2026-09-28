<template>
  <section>
    <p v-if="!draft.length" class="helper">Nothing added yet.</p>

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
        <!--
          Options are ExpenseType records filtered by the parent to the
          selected loan category. Grouped when the types define a group.
        -->
        <el-form-item label="What is it for?" required :error="need(item.expense_type)">
          <el-select
            :model-value="item.expense_type"
            placeholder="Choose one"
            filterable
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
            This doesn't apply to a {{ loanCategoryLabel }}. Choose something
            else or remove it.
          </small>
        </el-form-item>

        <el-form-item label="How much? (EC$)" required :error="need(item.amount)">
          <FormField
            :model-value="item.amount"
            :property="field('Expense', 'amount', 'How much? (EC$)', 'number')"
            :form="item"
            @update:model-value="set(index, 'amount', $event)"
          />
        </el-form-item>

        <el-form-item label="How often?">
          <FormField
            :model-value="item.frequency"
            :property="field('Expense', 'frequency', 'How often?', 'select')"
            :form="item"
            @update:model-value="set(index, 'frequency', $event)"
          />
        </el-form-item>

        <!--
          Only asked when someone else is borrowing too. The form decides the
          choices (the household or this application's borrowers), so it's
          an el-select.
        -->
        <el-form-item v-if="hasOthers" label="Who pays it?" required :error="item.is_household ? '' : need(item.application_party_id)">
          <el-select
            :model-value="item.is_household ? householdValue : item.application_party_id"
            placeholder="Choose one"
            @update:model-value="setPayer(index, $event)"
          >
            <el-option label="Shared by the household" :value="householdValue" />
            <el-option
              v-for="option in applicationPartyOptions"
              :key="option.value"
              :label="option.label"
              :value="option.value"
            />
          </el-select>
        </el-form-item>
      </div>

      <el-button
        v-if="!item.expense_name && !nameFor[item.client_key]"
        text
        type="primary"
        @click="nameFor[item.client_key] = true"
      >
        + Give it a name
      </el-button>
      <el-form-item v-else label="Name">
        <FormField
          :model-value="item.expense_name"
          :property="field('Expense', 'expense_name', 'Name', 'input')"
          :form="item"
          @update:model-value="set(index, 'expense_name', $event)"
        />
      </el-form-item>

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

    <el-button type="primary" plain size="large" @click="add">
      <v-icon start>mdi-plus</v-icon>
      Add {{ draft.length ? 'another bill' : 'a bill' }}
    </el-button>

    <!-- Worked out from the insurance on collateral; can't be edited here -->
    <section v-if="projectedExpenses.length" class="context">
      <h3>Insurance we've added for you</h3>
      <p class="helper">
        This is the insurance you told us about on the thing securing the loan.
        To change it, go back to "Things you own".
      </p>
      <div
        v-for="item in projectedExpenses"
        :key="item.asset_key"
        class="projected-row"
      >
        <span>{{ item.name }}</span>
        <strong>{{ money(item.monthly) }} a month</strong>
      </div>
    </section>
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
 * A shared household expense (is_household) isn't assigned to one
 * applicant. Projected collateral insurance is shown read-only below the
 * list; the parent works it out and saves it.
 *
 * Every input is Saturn's built-in FormField, using Saturn's own
 * definitions of the Expense properties when they exist. The responsible
 * applicant and expense type are el-selects of the options from the parent.
 */
export default {
  props: {
    /** Expenses array owned by the parent form. */
    modelValue: {
      type: Array,
      default: () => [],
    },
    /** True when someone else is borrowing too, so "Who pays it?" is asked. */
    hasOthers: {
      type: Boolean,
      default: false,
    },
    /** After a failed Next, shows "this is needed" under empty required boxes. */
    showErrors: {
      type: Boolean,
      default: false,
    },
    /** Projected insurance: { asset_key, name, monthly }, read-only. */
    projectedExpenses: {
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
      // Shows the optional name, by the expense's client key.
      nameFor: {},
      // The "Shared by the household" choice in "Who pays it?".
      householdValue: '__household__',
    };
  },

  mounted() {
    // They said they have bills, so start with one to fill in.
    if (!this.draft.length) this.add();
    // With only one borrower, every bill is theirs.
    if (!this.hasOthers && this.soleLink) {
      let changed = false;
      this.draft.forEach((item) => {
        if (!item.is_household && !item.application_party_id) {
          item.application_party_id = this.soleLink;
          changed = true;
        }
      });
      if (changed) this.notify();
    }
  },

  computed: {
    /** The only borrower's ApplicationParty ID, when there's just one. */
    soleLink() {
      return this.applicationPartyOptions.length === 1
        ? this.applicationPartyOptions[0].value
        : '';
    },

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

    /** "This is needed" under an empty required box, after a failed Next. */
    need(value) {
      if (!this.showErrors) return '';
      return value === null || value === undefined || value === '' ? 'This is needed' : '';
    },

    /** Sets who pays a bill: one borrower, or shared by the household. */
    setPayer(index, value) {
      const item = this.draft[index];
      if (value === this.householdValue) {
        item.is_household = true;
        item.application_party_id = '';
      } else {
        item.is_household = false;
        item.application_party_id = value;
      }
      this.notify();
    },

    title(item, index) {
      return item.expense_name || this.typeName(item.expense_type) || `Expense ${index + 1}`;
    },

    money(value) {
      return `EC$ ${Number(value || 0).toLocaleString(undefined, {
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
        application_party_id: this.hasOthers ? '' : this.soleLink,
        expense_name: '',
        expense_type: '',
        amount: null,
        frequency: '',
        is_household: false,
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

<style scoped>
.projected-row {
  display: flex;
  justify-content: space-between;
  gap: 12px;
  padding: 6px 0;
}
</style>