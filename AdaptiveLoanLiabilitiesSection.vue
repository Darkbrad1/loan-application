<template>
  <section>
    <!-- One card per debt: a short summary, or the details while editing -->
    <article
      v-for="(item, index) in draft"
      :key="item.client_key"
      class="item-card"
      :class="{ open: isOpen(item), 'needs-work': !isOpen(item) && showErrors && missing(item) }"
    >
      <div class="item-head">
        <span class="item-icon"><v-icon>{{ isRevolving(item) ? 'mdi-credit-card-outline' : 'mdi-bank-outline' }}</v-icon></span>
        <div class="item-text">
          <strong>{{ item.creditor_name || `Debt ${index + 1}` }}</strong>
          <span v-if="!isOpen(item) && missing(item)" class="needs">Some details are missing</span>
          <span v-else>{{ summaryOf(item) }}</span>
        </div>
        <div class="item-actions">
          <button v-if="!isOpen(item)" type="button" class="link-btn" @click="openItem(item)">Change</button>
          <button type="button" class="link-btn danger" @click="remove(index)">Remove</button>
        </div>
      </div>

      <template v-if="isOpen(item)">
        <el-form-item label="Who do you owe?" required :error="need(item.creditor_name)">
          <FormField
            :model-value="item.creditor_name"
            :property="field('Liability', 'creditor_name', 'Who do you owe?', 'input')"
            :form="item"
            @update:model-value="set(index, 'creditor_name', $event)"
          />
          <small class="helper">The bank, credit union, shop, or person.</small>
        </el-form-item>
        <!-- Options come from LiabilityType records; the value is the record ID -->
        <el-form-item label="What kind of debt is it?" required :error="need(item.liability_type)">
          <template v-if="liabilityTypeOptions.length <= 7">
            <div class="choice-list inline" role="radiogroup">
              <button
                v-for="option in liabilityTypeOptions"
                :key="String(option.value)"
                type="button"
                role="radio"
                class="choice"
                :class="{ selected: item.liability_type === option.value }"
                :aria-checked="item.liability_type === option.value"
                @click="set(index, 'liability_type', option.value)"
              >
                <span class="choice-mark"><v-icon size="16">mdi-check</v-icon></span>
                <span>{{ option.label }}</span>
              </button>
            </div>
          </template>
          <el-select
            v-else
            :model-value="item.liability_type"
            placeholder="Choose one"
            @update:model-value="set(index, 'liability_type', $event)"
          >
            <el-option
              v-for="option in liabilityTypeOptions"
              :key="option.value"
              :label="option.label"
              :value="option.value"
            />
          </el-select>
          <small v-if="item.liability_type && !typeOf(item)" class="helper invalid">
            This type is no longer available. Choose another.
          </small>
        </el-form-item>
        <el-form-item label="How much do you still owe? (EC$)" required :error="need(item.outstanding_balance)">
          <FormField
            :model-value="item.outstanding_balance"
            :property="field('Liability', 'outstanding_balance', 'How much do you still owe? (EC$)', 'number')"
            :form="item"
            @update:model-value="set(index, 'outstanding_balance', $event)"
          />
        </el-form-item>
        <el-form-item label="Credit limit (EC$)" required :error="need(item.credit_limit)" v-if="isRevolving(item)">
          <FormField
            :model-value="item.credit_limit"
            :property="field('Liability', 'credit_limit', 'Credit limit (EC$)', 'number')"
            :form="item"
            @update:model-value="set(index, 'credit_limit', $event)"
          />
          <small class="helper">The most you're allowed to owe on it.</small>
        </el-form-item>
        <div class="field-grid">
          <el-form-item label="How much do you pay? (EC$)" required :error="need(item.payment_amount)">
            <FormField
              :model-value="item.payment_amount"
              :property="field('Liability', 'payment_amount', 'How much do you pay? (EC$)', 'number')"
              :form="item"
              @update:model-value="set(index, 'payment_amount', $event)"
            />
          </el-form-item>
          <el-form-item label="How often?">
            <FormField
              :model-value="item.payment_frequency"
              :property="field('Liability', 'payment_frequency', 'How often?', 'select')"
              :form="item"
              @update:model-value="set(index, 'payment_frequency', $event)"
            />
          </el-form-item>
        </div>

        <!--
          Assessed repayment: a fixed % of the credit limit rather than the
          balance. The server recalculates this figure.
        -->
        <p v-if="isRevolving(item)" class="soft-box info">
          <template v-if="assessed(item) !== null">
            We'll count {{ money(assessed(item)) }} a month for this
            ({{ percentLabel(rateFor(item)) }} of the credit limit).
          </template>
          <template v-else>
            We'll count {{ percentLabel(rateFor(item)) }} of the credit limit as your monthly payment.
          </template>
        </p>
        <el-form-item label="Will the new loan pay this off?">
          <div class="choice-list inline" role="radiogroup">
            <button
              v-for="option in [{ value: true, label: 'Yes' }, { value: false, label: 'No' }]"
              :key="String(option.value)"
              type="button"
              role="radio"
              class="choice"
              :class="{ selected: item.is_to_be_paid_off === true === option.value }"
              :aria-checked="item.is_to_be_paid_off === true === option.value"
              @click="set(index, 'is_to_be_paid_off', option.value)"
            >
              <span class="choice-mark"><v-icon size="16">mdi-check</v-icon></span>
              <span>{{ option.label }}</span>
            </button>
          </div>
        </el-form-item>
        <el-form-item label="Is something you own held against it?">
          <div class="choice-list inline" role="radiogroup">
            <button
              v-for="option in [{ value: true, label: 'Yes' }, { value: false, label: 'No' }]"
              :key="String(option.value)"
              type="button"
              role="radio"
              class="choice"
              :class="{ selected: item.is_secured === true === option.value }"
              :aria-checked="item.is_secured === true === option.value"
              @click="set(index, 'is_secured', option.value)"
            >
              <span class="choice-mark"><v-icon size="16">mdi-check</v-icon></span>
              <span>{{ option.label }}</span>
            </button>
          </div>
          <small class="helper">For example, a car loan on your car.</small>
        </el-form-item>
        <el-form-item v-if="item.is_secured" label="Which thing?" required :error="need(item.secured_asset_ref)">
          <div class="choice-list" role="radiogroup">
            <button
              v-for="option in assetOptions"
              :key="String(option.value)"
              type="button"
              role="radio"
              class="choice"
              :class="{ selected: item.secured_asset_ref === option.value }"
              :aria-checked="item.secured_asset_ref === option.value"
              @click="set(index, 'secured_asset_ref', option.value)"
            >
              <span class="choice-mark"><v-icon size="16">mdi-check</v-icon></span>
              <span>{{ option.label }}</span>
            </button>
          </div>
          <small v-if="!assetOptions.length" class="helper invalid">
            Add it under "Things you own" first.
          </small>
        </el-form-item>
        <el-form-item label="Note" v-if="item.description || notesFor[item.client_key]">
          <FormField
            :model-value="item.description"
            :property="field('Liability', 'description', 'Note', 'textarea')"
            :form="item"
            @update:model-value="set(index, 'description', $event)"
          />
        </el-form-item>
        <button v-else type="button" class="more-link" @click="notesFor[item.client_key] = true">
          <v-icon size="20">mdi-plus</v-icon>
          Add a note (for example, unusual payment terms)
        </button>

        <!-- Responsibility percentages must total 100% across the people on the loan -->
        <AdaptiveLoanAllocationEditor
          :model-value="item.responsibilities"
          :options="applicationPartyOptions"
          reference-key="application_party_id"
          simple
          :default-value="primaryLinkId"
          title="Who pays it?"
          select-label="Person"
          total-label="Shares add up to"
          me-label="You pay this."
          responsibility-type="Borrower"
          @update:model-value="set(index, 'responsibilities', $event)"
        />

        <!-- Required documents for this debt; uploads unlock once it's complete -->
        <AdaptiveLoanDocumentRequirements
          :scope="documentScopes[`liability:${item.client_key}`]"
          :uploading-key="uploadingKey"
          :disabled="documentsDisabled"
          @stage-file="$emit('stage-file', $event)"
          @remove-file="$emit('remove-file', $event)"
          @request-file-upload="$emit('request-file-upload', $event)"
          @file-rejected="$emit('file-rejected', $event)"
        />

        <div class="item-done">
          <button type="button" class="small-btn" @click="finish(item)">
            <v-icon size="20">mdi-check</v-icon>
            Done
          </button>
        </div>
      </template>
    </article>

    <button type="button" class="add-button" @click="add">
      <v-icon>mdi-plus</v-icon>
      Add {{ draft.length ? 'another debt' : 'a debt' }}
    </button>
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
 * Each liability also records whether this loan will pay it off, whether
 * it's secured on one of the applicant's assets (a lien), and free-text
 * notes. The balance date is set by the parent when it saves.
 *
 * Every input is Saturn's built-in FormField, using Saturn's own
 * definitions of the Liability properties when they exist. Liability type
 * and "Secured on" are el-selects of choices passed in by the parent.
 */
export default {
  props: {
    /** The primary applicant's ApplicationParty ID: "Just me" for who pays. */
    primaryLinkId: {
      type: String,
      default: '',
    },
    /** After a failed Next, shows "this is needed" under empty required boxes. */
    showErrors: {
      type: Boolean,
      default: false,
    },
    /** The application's assets as { value: client key, label } choices. */
    assetOptions: {
      type: Array,
      default: () => [],
    },
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
      // Shows the optional note, by the liability's client key.
      notesFor: {},
      // The one item being edited (the rest fold up into a summary).
      openKey: '',
      // True after "Done" is pressed on an item with missing details.
      checking: false,
    };
  },

  mounted() {
    // They said they owe money, so start with one to fill in.
    if (!this.draft.length) {
      this.add();
      return;
    }
    // Open the first item that still needs details, if any.
    const unfinished = this.draft.find((item) => this.missing(item));
    this.openKey = unfinished ? unfinished.client_key : '';
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

    /** "This is needed" under an empty required box, after a failed Next. */
    need(value) {
      if (!this.showErrors && !this.checking) return '';
      return value === null || value === undefined || value === '' ? 'This is needed' : '';
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
      // Not secured any more: forget which asset it was secured on.
      if (fieldName === 'is_secured' && !item.is_secured) {
        item.secured_asset_ref = '';
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
        description: '',
        is_to_be_paid_off: false,
        is_secured: false,
        secured_asset_ref: '',
        document_ids: [],
        responsibilities: [
          {
            client_key: this.key('resp'),
            id: null,
            application_party_id: this.primaryLinkId,
            percentage: 100,
            responsibility_type: 'Borrower',
          },
        ],
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
      if (['creditor_name', 'liability_type', 'outstanding_balance', 'payment_amount'].some((key) => empty(item[key]))) {
        return true;
      }
      if (this.isRevolving(item) && empty(item.credit_limit)) return true;
      if (item.is_secured && empty(item.secured_asset_ref)) return true;
      return false;
    },

    /** One line under the item's name. */
    summaryOf(item) {
      const empty = (value) => value === null || value === undefined || value === '';
      const parts = [];
      const type = this.typeOf(item);
      if (type && type.name) parts.push(type.name);
      if (!empty(item.outstanding_balance)) parts.push(`${this.money(item.outstanding_balance)} left`);
      if (item.is_to_be_paid_off) parts.push('Paid off by this loan');
      return parts.join(' · ');
    },

    remove(index) {
      this.draft.splice(index, 1);
      this.notify();
    },
  },
};
</script>

<style scoped>
/* Styled in the main form's stylesheet (.item-card, .choice, .soft-box, ...). */
</style>
