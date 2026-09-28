<template>
  <section>
    <!-- One card per bill: a short summary, or the details while editing -->
    <article
      v-for="(item, index) in draft"
      :key="item.client_key"
      class="item-card mb-4 p-5 bg-white rounded-lg"
      :style="['border-width:2px;border-style:solid', isOpen(item) ? 'border-color:var(--brand);box-shadow:0 0 0 3px var(--brand-tint)' : (!isOpen(item) && showErrors && missing(item)) ? 'border-color:#b91c1c' : 'border-color:#e5e7eb']"
    >
      <div class="item-head flex flex-wrap items-center gap-3" :style="isOpen(item) ? 'margin-bottom:20px;padding-bottom:16px;border-bottom:1px solid #e5e7eb' : ''">
        <span class="item-icon flex items-center justify-center rounded-lg" style="width:44px;height:44px;flex:0 0 auto;background:var(--brand-tint);color:var(--brand-text)"><v-icon>mdi-receipt-text-outline</v-icon></span>
        <div class="item-text" style="flex:1 1 160px;min-width:0">
          <strong class="item-name block text-lg font-bold" style="line-height:1.3;overflow-wrap:anywhere">{{ title(item, index) }}</strong>
          <span v-if="!isOpen(item) && missing(item)" class="needs block text-sm font-semibold text-red-700">Some details are missing</span>
          <span v-else class="item-sub block text-sm text-gray-600">{{ summaryOf(item) }}</span>
          <span
            v-if="!isOpen(item) && docsNeeded(`expense:${item.client_key}`)"
            class="item-sub block text-sm font-semibold text-orange-700"
          >
            {{ docsNeeded(`expense:${item.client_key}`) }} {{ docsNeeded(`expense:${item.client_key}`) === 1 ? 'document' : 'documents' }} to upload
          </span>
        </div>
        <div class="item-actions flex gap-1" style="margin-left:auto">
          <button v-if="!isOpen(item)" type="button" class="link-btn px-3 py-2 rounded text-base font-semibold" style="color:var(--brand-text);background:transparent;border:0;cursor:pointer" @click="openItem(item)">{{ docsNeeded(`expense:${item.client_key}`) ? 'Upload documents' : 'Change' }}</button>
          <button type="button" class="link-btn danger px-3 py-2 rounded text-base font-semibold text-red-700" style="background:transparent;border:0;cursor:pointer" @click="remove(index)">Remove</button>
        </div>
      </div>

      <template v-if="isOpen(item)">
        <!--
          Options are ExpenseType records filtered by the parent to the
          selected loan category. Grouped when the types define a group.
        -->
        <el-form-item label="What is it for?" required :error="need(item.expense_type)">
          <el-select
            :model-value="item.expense_type"
            placeholder="Choose or type to search"
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
              <el-option-group v-for="group in groupedOptions" :key="group.label" :label="group.label">
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
          <small v-if="isInapplicable(item)" class="helper invalid block w-full mt-1 text-sm font-semibold text-red-700" style="flex:1 1 100%;line-height:1.45">
            This doesn't apply to a {{ loanCategoryLabel }}. Choose something else or remove it.
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
          <template v-if="choices('expense_frequency')">
            <div class="choice-list inline flex flex-wrap gap-3 w-full" role="radiogroup">
              <button
                v-for="option in choices('expense_frequency')"
                :key="String(option.value)"
                type="button"
                role="radio"
                class="choice flex items-center gap-3 w-full px-4 py-3 rounded-lg text-base text-left"
                :style="['min-height:56px;border-width:2px;border-style:solid;cursor:pointer;color:#111827;line-height:1.35', (item.frequency === option.value) ? 'border-color:var(--brand);background:var(--brand-tint);box-shadow:inset 0 0 0 1px var(--brand)' : 'border-color:#d1d5db;background:#ffffff']"
                :aria-checked="item.frequency === option.value"
                @click="set(index, 'frequency', option.value)"
              >
                <span class="choice-mark flex items-center justify-center rounded-full" :style="['width:26px;height:26px;flex:0 0 auto;border-width:2px;border-style:solid', (item.frequency === option.value) ? 'background:var(--brand);border-color:var(--brand);color:var(--brand-ink)' : 'background:#ffffff;border-color:#d1d5db;color:transparent']"><v-icon size="16">mdi-check</v-icon></span>
                <span>{{ option.label }}</span>
              </button>
            </div>
          </template>
          <template v-else>
            <FormField
              :model-value="item.frequency"
              :property="field('Expense', 'frequency', 'How often?', 'select')"
              :form="item"
              @update:model-value="set(index, 'frequency', $event)"
            />
          </template>
        </el-form-item>

        <!-- Only asked when someone else is borrowing too -->
        <el-form-item v-if="hasOthers" label="Who pays it?" required :error="item.is_household ? '' : need(item.application_party_id)">
          <div class="choice-list flex flex-col gap-3 w-full" role="radiogroup">
            <button
              v-for="option in [{ value: householdValue, label: 'Shared by the household' }].concat(applicationPartyOptions)"
              :key="String(option.value)"
              type="button"
              role="radio"
              class="choice flex items-center gap-3 w-full px-4 py-3 rounded-lg text-base text-left"
              :style="['min-height:56px;border-width:2px;border-style:solid;cursor:pointer;color:#111827;line-height:1.35', ((item.is_household ? householdValue : item.application_party_id) === option.value) ? 'border-color:var(--brand);background:var(--brand-tint);box-shadow:inset 0 0 0 1px var(--brand)' : 'border-color:#d1d5db;background:#ffffff']"
              :aria-checked="(item.is_household ? householdValue : item.application_party_id) === option.value"
              @click="setPayer(index, option.value)"
            >
              <span class="choice-mark flex items-center justify-center rounded-full" :style="['width:26px;height:26px;flex:0 0 auto;border-width:2px;border-style:solid', ((item.is_household ? householdValue : item.application_party_id) === option.value) ? 'background:var(--brand);border-color:var(--brand);color:var(--brand-ink)' : 'background:#ffffff;border-color:#d1d5db;color:transparent']"><v-icon size="16">mdi-check</v-icon></span>
              <span>{{ option.label }}</span>
            </button>
          </div>
        </el-form-item>

        <el-form-item label="Name" v-if="item.expense_name || nameFor[item.client_key]">
          <FormField
            :model-value="item.expense_name"
            :property="field('Expense', 'expense_name', 'Name', 'input')"
            :form="item"
            @update:model-value="set(index, 'expense_name', $event)"
          />
        </el-form-item>
        <button v-else type="button" class="more-link inline-flex items-center gap-1 mb-5 py-1 text-base font-semibold" style="color:var(--brand-text);background:transparent;border:0;cursor:pointer;text-align:left" @click="nameFor[item.client_key] = true">
          <v-icon size="20">mdi-plus</v-icon>
          Give it a name
        </button>

        <!-- Required documents for this bill; uploads unlock once it's complete -->
        <AdaptiveLoanDocumentRequirements
          :scope="documentScopes[`expense:${item.client_key}`]"
          :uploading-key="uploadingKey"
          :disabled="documentsDisabled"
          @stage-file="$emit('stage-file', $event)"
          @remove-file="$emit('remove-file', $event)"
          @request-file-upload="$emit('request-file-upload', $event)"
          @file-rejected="$emit('file-rejected', $event)"
        />

        <div class="item-done flex justify-end mt-2">
          <button type="button" class="small-btn inline-flex items-center gap-2 px-5 rounded-lg text-base font-bold" style="min-height:44px;background:var(--brand);color:var(--brand-ink);border:0;cursor:pointer" @click="finish(item)">
            <v-icon size="20">mdi-check</v-icon>
            Done
          </button>
        </div>
      </template>
    </article>

    <button type="button" class="add-button flex items-center justify-center gap-2 w-full bg-white rounded-lg text-lg font-semibold" style="min-height:56px;border:2px dashed #d1d5db;color:var(--brand-text);cursor:pointer" @click="add">
      <v-icon>mdi-plus</v-icon>
      Add {{ draft.length ? 'another bill' : 'a bill' }}
    </button>

    <!-- Worked out from the insurance on collateral; can't be changed here -->
    <div v-if="projectedExpenses.length" class="soft-box projected mt-5 mb-5 p-4 rounded-lg bg-gray-100">
      <p class="question mb-3 text-base font-semibold" style="color:#111827">Insurance we've added for you</p>
      <p class="hint mt-2 text-base text-gray-600">
        This is the insurance on the thing securing the loan. To change it, go back to "Things you own".
      </p>
      <div v-for="item in projectedExpenses" :key="item.asset_key" class="sum-row flex justify-between gap-4 py-1">
        <span>{{ item.name }}</span>
        <strong>{{ money(item.monthly) }} a month</strong>
      </div>
    </div>
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
      // The one item being edited (the rest fold up into a summary).
      openKey: '',
      // True after "Done" is pressed on an item with missing details.
      checking: false,
    };
  },

  mounted() {
    // They said they have bills, so start with one to fill in.
    if (!this.draft.length) {
      this.add();
      return;
    }
    // Open the first item that still needs details, if any.
    const unfinished = this.draft.find((item) => this.missing(item));
    this.openKey = unfinished ? unfinished.client_key : '';
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
    // After a failed Continue, open the first item that's missing details.
    showErrors(value) {
      if (!value) return;
      const open = this.draft.find((item) => this.isOpen(item));
      if (open && this.missing(open)) return;
      const unfinished = this.draft.find((item) => this.missing(item));
      if (unfinished) this.openKey = unfinished.client_key;
    },
  },

  methods: {
    /**
     * The answers for a question as tappable cards, when Saturn's list is
     * short (2 to 7 answers). Otherwise null, and Saturn's dropdown is used.
     */
    choices(key) {
      const options = this.lookups[key] || [];
      return options.length >= 2 && options.length <= 7 ? options : null;
    },

    /** How many documents for these scopes still need uploading. */
    docsNeeded() {
      return Array.from(arguments).reduce((count, key) => {
        const scope = this.documentScopes[key];
        if (!scope) return count;
        return count + scope.requirements.filter((row) => row.status !== 'uploaded').length;
      }, 0);
    },

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
      if (!this.showErrors && !this.checking) return '';
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
      this.openKey = this.draft[this.draft.length - 1].client_key;
      this.checking = false;
      this.notify();
    },

    // ---- Fold-up cards ----

    /** True when an item's details are showing. */
    isOpen(item) {
      return this.openKey === item.client_key;
    },

    openItem(item) {
      this.openKey = item.client_key;
      this.checking = false;
    },

    /** "Done": folds the item up, or points out what's missing. */
    finish(item) {
      if (this.missing(item)) {
        this.checking = true;
        return;
      }
      this.checking = false;
      this.openKey = '';
    },

    /** True when a required detail is still empty. */
    missing(item) {
      const empty = (value) => value === null || value === undefined || value === '';
      if (empty(item.expense_type) || empty(item.amount)) return true;
      if (this.hasOthers && !item.is_household && empty(item.application_party_id)) return true;
      return false;
    },

    /** One line under the item's name. */
    summaryOf(item) {
      const empty = (value) => value === null || value === undefined || value === '';
      const parts = [];
      if (!empty(item.amount)) parts.push(this.money(item.amount));
      const frequency = this.lookup('expense_frequency').find((option) => option.value === item.frequency);
      if (frequency) parts.push(frequency.label.toLowerCase());
      else if (item.frequency) parts.push(String(item.frequency).toLowerCase());
      return parts.join(' ');
    },

    remove(index) {
      this.draft.splice(index, 1);
      this.notify();
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
