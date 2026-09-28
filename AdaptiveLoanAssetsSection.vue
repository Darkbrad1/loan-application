<template>
  <section>
    <!-- One card per thing they own: a short summary, or the details while editing -->
    <article
      v-for="(asset, index) in draft"
      :key="asset.client_key"
      class="item-card"
      :class="{ open: isOpen(asset), 'needs-work': !isOpen(asset) && showErrors && missing(asset) }"
    >
      <div class="item-head">
        <span class="item-icon"><v-icon>{{ iconFor(asset) }}</v-icon></span>
        <div class="item-text">
          <strong>{{ asset.name || `Item ${index + 1}` }}</strong>
          <span v-if="!isOpen(asset) && missing(asset)" class="needs">Some details are missing</span>
          <span v-else>{{ summaryOf(asset) }}</span>
        </div>
        <div class="item-actions">
          <button v-if="!isOpen(asset)" type="button" class="link-btn" @click="openItem(asset)">Change</button>
          <button v-if="!asset.is_purchase" type="button" class="link-btn danger" @click="remove(index)">Remove</button>
        </div>
      </div>

      <template v-if="isOpen(asset)">
        <div v-if="asset.is_purchase || isCollateral(asset)" class="item-tags">
          <span v-if="asset.is_purchase" class="tag"><v-icon size="14">mdi-cart-outline</v-icon> Buying with this loan</span>
          <span v-if="isCollateral(asset)" class="tag"><v-icon size="14">mdi-shield-check-outline</v-icon> Secures the loan</span>
        </div>

        <!-- The vehicle or property being bought comes from "The loan you need" -->
        <p v-if="asset.is_purchase" class="soft-box">
          You're buying this for <strong>{{ money(asset.declared_value) }}</strong>.
          To change the price, go back to "The loan you need".
        </p>

        <template v-else>
          <el-form-item label="What is it?" required :error="need(asset.name)">
            <FormField
              :model-value="asset.name"
              :property="field('Asset', 'name', 'What is it?', 'input')"
              :form="asset"
              @update:model-value="set(index, 'name', $event)"
            />
            <small class="helper">For example &quot;My house in Grand Anse&quot; or &quot;2018 Honda Fit&quot;.</small>
          </el-form-item>
          <el-form-item label="What kind of thing is it?" required :error="need(asset.asset_type)">
            <template v-if="choices('asset_type')">
              <div class="choice-list inline" role="radiogroup">
                <button
                  v-for="option in choices('asset_type')"
                  :key="String(option.value)"
                  type="button"
                  role="radio"
                  class="choice"
                  :class="{ selected: asset.asset_type === option.value }"
                  :aria-checked="asset.asset_type === option.value"
                  @click="set(index, 'asset_type', option.value)"
                >
                  <span class="choice-mark"><v-icon size="16">mdi-check</v-icon></span>
                  <span>{{ option.label }}</span>
                </button>
              </div>
            </template>
            <template v-else>
              <FormField
                :model-value="asset.asset_type"
                :property="field('Asset', 'asset_type', 'What kind of thing is it?', 'select')"
                :form="asset"
                @update:model-value="set(index, 'asset_type', $event)"
              />
            </template>
          </el-form-item>
          <el-form-item label="What is it worth? (EC$)" required :error="need(asset.declared_value)">
            <FormField
              :model-value="asset.declared_value"
              :property="field('Asset', 'declared_value', 'What is it worth? (EC$)', 'number')"
              :form="asset"
              @update:model-value="set(index, 'declared_value', $event)"
            />
            <small class="helper">Your best guess is fine.</small>
          </el-form-item>
          <div v-if="isVehicle(asset)" class="field-grid">
            <el-form-item label="Registration number">
              <FormField
                :model-value="asset.registration_number"
                :property="field('Asset', 'registration_number', 'Registration number', 'input')"
                :form="asset"
                @update:model-value="set(index, 'registration_number', $event)"
              />
            </el-form-item>
            <el-form-item label="Chassis number (VIN)">
              <FormField
                :model-value="asset.chassis_number"
                :property="field('Asset', 'chassis_number', 'Chassis number (VIN)', 'input')"
                :form="asset"
                @update:model-value="set(index, 'chassis_number', $event)"
              />
            </el-form-item>
          </div>
          <div v-if="isProperty(asset)" class="field-grid">
            <el-form-item label="Block and parcel">
              <FormField
                :model-value="asset.block_and_parcel"
                :property="field('Asset', 'block_and_parcel', 'Block and parcel', 'input')"
                :form="asset"
                @update:model-value="set(index, 'block_and_parcel', $event)"
              />
            </el-form-item>
            <el-form-item label="Deed number">
              <FormField
                :model-value="asset.deed_number"
                :property="field('Asset', 'deed_number', 'Deed number', 'input')"
                :form="asset"
                @update:model-value="set(index, 'deed_number', $event)"
              />
            </el-form-item>
          </div>
          <el-form-item label="Description" v-if="asset.description || moreFor[asset.client_key]">
            <FormField
              :model-value="asset.description"
              :property="field('Asset', 'description', 'Description', 'input')"
              :form="asset"
              @update:model-value="set(index, 'description', $event)"
            />
          </el-form-item>
          <button v-else type="button" class="more-link" @click="moreFor[asset.client_key] = true">
            <v-icon size="20">mdi-plus</v-icon>
            Add a description
          </button>
        </template>

        <!-- Existing loans secured on this (from "Money you owe") -->
        <p v-if="(liens[asset.client_key] || []).length" class="soft-box warn">
          You still owe money on this: {{ liens[asset.client_key].join('; ') }}
        </p>

        <!-- Ownership percentages must total 100% across the people on the loan -->
        <AdaptiveLoanAllocationEditor
          :model-value="asset.owners"
          :options="partyOptions"
          reference-key="party_id"
          simple
          :default-value="primaryPartyId"
          title="Who owns it?"
          select-label="Owner"
          total-label="Shares add up to"
          @update:model-value="set(index, 'owners', $event)"
        />
        <small v-if="ownedOnlyByThirdParty(asset)" class="helper">
          Someone who isn't borrowing owns all of this, so it can only be listed
          if it secures the loan.
        </small>

        <!-- Collateral: only for loans that need it -->
        <template v-if="requiresCollateral">
          <p v-if="asset.is_purchase" class="soft-box info">
            This secures the loan. Tell us how it's insured below; a quote is fine for now.
          </p>
          <el-form-item label="Use this to secure the loan?" v-else>
            <div class="choice-list inline" role="radiogroup">
              <button
                v-for="option in [{ value: true, label: 'Yes' }, { value: false, label: 'No' }]"
                :key="String(option.value)"
                type="button"
                role="radio"
                class="choice"
                :class="{ selected: Boolean(asset.collateral.enabled) === option.value }"
                :aria-checked="Boolean(asset.collateral.enabled) === option.value"
                @click="setCollateral(index, 'enabled', option.value)"
              >
                <span class="choice-mark"><v-icon size="16">mdi-check</v-icon></span>
                <span>{{ option.label }}</span>
              </button>
            </div>
          </el-form-item>

          <template v-if="asset.collateral.enabled">
            <p class="subheading">How is it insured?</p>
            <p class="hint">If you don't have insurance yet, get a quote from an insurer and enter it here.</p>
            <el-form-item label="Do you have a policy, or a quote?" required :error="need(asset.collateral.insurance.status)">
              <template v-if="choices('insurance_status')">
                <div class="choice-list inline" role="radiogroup">
                  <button
                    v-for="option in choices('insurance_status')"
                    :key="String(option.value)"
                    type="button"
                    role="radio"
                    class="choice"
                    :class="{ selected: asset.collateral.insurance.status === option.value }"
                    :aria-checked="asset.collateral.insurance.status === option.value"
                    @click="setInsurance(index, 'status', option.value)"
                  >
                    <span class="choice-mark"><v-icon size="16">mdi-check</v-icon></span>
                    <span>{{ option.label }}</span>
                  </button>
                </div>
              </template>
              <template v-else>
                <FormField
                  :model-value="asset.collateral.insurance.status"
                  :property="field('Collateral', 'insurance_status', 'Do you have a policy, or a quote?', 'select')"
                  :form="asset.collateral.insurance"
                  @update:model-value="setInsurance(index, 'status', $event)"
                />
              </template>
            </el-form-item>
            <el-form-item label="Type of insurance" required :error="need(asset.collateral.insurance.type)">
              <template v-if="choices('insurance_type')">
                <div class="choice-list" role="radiogroup">
                  <button
                    v-for="option in choices('insurance_type')"
                    :key="String(option.value)"
                    type="button"
                    role="radio"
                    class="choice"
                    :class="{ selected: asset.collateral.insurance.type === option.value }"
                    :aria-checked="asset.collateral.insurance.type === option.value"
                    @click="setInsurance(index, 'type', option.value)"
                  >
                    <span class="choice-mark"><v-icon size="16">mdi-check</v-icon></span>
                    <span>{{ option.label }}</span>
                  </button>
                </div>
              </template>
              <template v-else>
                <FormField
                  :model-value="asset.collateral.insurance.type"
                  :property="field('Collateral', 'insurance_type', 'Type of insurance', 'select')"
                  :form="asset.collateral.insurance"
                  @update:model-value="setInsurance(index, 'type', $event)"
                />
              </template>
            </el-form-item>
            <div class="field-grid">
              <el-form-item label="Insurance company" required :error="need(asset.collateral.insurance.provider)">
                <FormField
                  :model-value="asset.collateral.insurance.provider"
                  :property="field('Collateral', 'insurance_provider', 'Insurance company', 'input')"
                  :form="asset.collateral.insurance"
                  @update:model-value="setInsurance(index, 'provider', $event)"
                />
              </el-form-item>
              <el-form-item
                :label="isPolicy(asset) ? 'Policy number' : 'Quote number'"
                :required="isPolicy(asset)"
                :error="isPolicy(asset) ? need(asset.collateral.insurance.reference) : ''"
              >
                <FormField
                  :model-value="asset.collateral.insurance.reference"
                  :property="field('Collateral', 'insurance_reference', isPolicy(asset) ? 'Policy number' : 'Quote number', 'input')"
                  :form="asset.collateral.insurance"
                  @update:model-value="setInsurance(index, 'reference', $event)"
                />
              </el-form-item>
            </div>
            <el-form-item label="What does it cost? (EC$)" required :error="need(asset.collateral.insurance.premium)">
              <FormField
                :model-value="asset.collateral.insurance.premium"
                :property="field('Collateral', 'insurance_premium', 'What does it cost? (EC$)', 'number')"
                :form="asset.collateral.insurance"
                @update:model-value="setInsurance(index, 'premium', $event)"
              />
            </el-form-item>
            <el-form-item label="How often do you pay it?" required :error="need(asset.collateral.insurance.premium_frequency)">
              <template v-if="choices('insurance_premium_frequency')">
                <div class="choice-list inline" role="radiogroup">
                  <button
                    v-for="option in choices('insurance_premium_frequency')"
                    :key="String(option.value)"
                    type="button"
                    role="radio"
                    class="choice"
                    :class="{ selected: asset.collateral.insurance.premium_frequency === option.value }"
                    :aria-checked="asset.collateral.insurance.premium_frequency === option.value"
                    @click="setInsurance(index, 'premium_frequency', option.value)"
                  >
                    <span class="choice-mark"><v-icon size="16">mdi-check</v-icon></span>
                    <span>{{ option.label }}</span>
                  </button>
                </div>
              </template>
              <template v-else>
                <FormField
                  :model-value="asset.collateral.insurance.premium_frequency"
                  :property="field('Collateral', 'insurance_premium_frequency', 'How often do you pay it?', 'select')"
                  :form="asset.collateral.insurance"
                  @update:model-value="setInsurance(index, 'premium_frequency', $event)"
                />
              </template>
            </el-form-item>
            <el-form-item v-if="isPolicy(asset)" label="When does the policy end?">
              <FormField
                :model-value="asset.collateral.insurance.expiry_date"
                :property="field('Collateral', 'insurance_expiry_date', 'Policy expiry date', 'date')"
                :form="asset.collateral.insurance"
                @update:model-value="setInsurance(index, 'expiry_date', $event, 'date')"
              />
            </el-form-item>

            <template v-if="insuranceMore[asset.client_key] || asset.collateral.insurance.coverage_amount || asset.collateral.description">
              <el-form-item label="Amount covered (EC$)">
                <FormField
                  :model-value="asset.collateral.insurance.coverage_amount"
                  :property="field('Collateral', 'insurance_coverage_amount', 'Amount covered (EC$)', 'number')"
                  :form="asset.collateral.insurance"
                  @update:model-value="setInsurance(index, 'coverage_amount', $event)"
                />
              </el-form-item>
              <el-form-item label="Notes">
                <FormField
                  :model-value="asset.collateral.description"
                  :property="field('Collateral', 'description', 'Notes', 'input')"
                  :form="asset.collateral"
                  @update:model-value="setCollateral(index, 'description', $event)"
                />
              </el-form-item>
            </template>
            <button v-else type="button" class="more-link" @click="insuranceMore[asset.client_key] = true">
              <v-icon size="20">mdi-plus</v-icon>
              Add the amount covered and notes
            </button>

            <!-- Collateral documents; uploads unlock once the details are complete -->
            <AdaptiveLoanDocumentRequirements
              title="Documents for this"
              :scope="documentScopes[`collateral:${asset.client_key}`]"
              :uploading-key="uploadingKey"
              :disabled="documentsDisabled"
              @stage-file="$emit('stage-file', $event)"
              @remove-file="$emit('remove-file', $event)"
              @request-file-upload="$emit('request-file-upload', $event)"
              @file-rejected="$emit('file-rejected', $event)"
            />
          </template>
        </template>

        <!-- Required documents for this; uploads unlock once it is complete -->
        <AdaptiveLoanDocumentRequirements
          :scope="documentScopes[`asset:${asset.client_key}`]"
          :uploading-key="uploadingKey"
          :disabled="documentsDisabled"
          @stage-file="$emit('stage-file', $event)"
          @remove-file="$emit('remove-file', $event)"
          @request-file-upload="$emit('request-file-upload', $event)"
          @file-rejected="$emit('file-rejected', $event)"
        />

        <div class="item-done">
          <button type="button" class="small-btn" @click="finish(asset)">
            <v-icon size="20">mdi-check</v-icon>
            Done
          </button>
        </div>
      </template>
    </article>

    <button type="button" class="add-button" @click="add">
      <v-icon>mdi-plus</v-icon>
      Add {{ draft.length ? 'something else' : 'something you own' }}
    </button>
  </section>
</template>

<script>
/**
 * Assets step of the loan wizard. Maintains a local editable copy
 * (draft) of the assets array so the parent only receives clean,
 * committed updates via update:modelValue. Required documents for each
 * asset are shown inside its card; upload state lives in the parent.
 *
 * On loans that need collateral, each asset can be marked as collateral.
 * Its collateral details (insurance and notes) live on asset.collateral,
 * matching createEmptyCollateral() in the main form. The asset's own name,
 * type, and value are used for the collateral.
 *
 * The vehicle or property a purchase loan is buying (is_purchase) is added
 * by the main form from the "Your request" step: its details are shown,
 * not edited, and it's always collateral. Vehicles and land or property
 * also ask for their identifiers (registration and chassis numbers, or
 * block and parcel and deed numbers).
 *
 * Every input is Saturn's built-in FormField, using Saturn's own
 * definitions of the Asset and Collateral properties when they exist.
 */
export default {
  props: {
    /** Assets array owned by the parent form. */
    modelValue: {
      type: Array,
      default: () => [],
    },
    /** The primary applicant's Party ID: the owner for "Just me". */
    primaryPartyId: {
      type: String,
      default: '',
    },
    /** After a failed Next, shows "this is needed" under empty required boxes. */
    showErrors: {
      type: Boolean,
      default: false,
    },
    /** Existing loans secured on each asset, keyed by client key (labels). */
    liens: {
      type: Object,
      default: () => ({}),
    },
    /** Select options for ownership, keyed by saved Party ID. */
    partyOptions: {
      type: Array,
      default: () => [],
    },
    /** Party IDs of the Third Party Owners (people who aren't borrowing). */
    thirdPartyOwnerIds: {
      type: Array,
      default: () => [],
    },
    /** True when this loan needs collateral (shows the collateral options). */
    requiresCollateral: {
      type: Boolean,
      default: false,
    },
    /** Dropdown option lists keyed by field name. */
    lookups: {
      type: Object,
      default: () => ({}),
    },
    /** Saturn's property definitions, keyed by resource name. */
    resourceProps: {
      type: Object,
      default: () => ({}),
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
      // Shows the optional description, by the asset's client key.
      moreFor: {},
      // Shows the optional insurance extras, by the asset's client key.
      insuranceMore: {},
      // The one item being edited (the rest fold up into a summary).
      openKey: '',
      // True after "Done" is pressed on an item with missing details.
      checking: false,
    };
  },

  mounted() {
    // They said they own something, so start with one to fill in.
    if (!this.draft.length) {
      this.add();
      return;
    }
    // Open the first item that still needs details, if any.
    const unfinished = this.draft.find((asset) => this.missing(asset));
    this.openKey = unfinished ? unfinished.client_key : '';
  },

  created() {
    // FormField configs, reused while unchanged (see field()).
    this.fieldCache = {};
  },

  computed: {
    /** Policy or quote, from the lookup when loaded, else the two defaults. */
    insuranceStatusOptions() {
      const loaded = this.lookup('insurance_status');
      if (loaded.length) return loaded;
      return [
        { label: 'Policy', value: 'Policy' },
        { label: 'Quote', value: 'Quote' },
      ];
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
      const open = this.draft.find((asset) => this.isOpen(asset));
      if (open && this.missing(open)) return;
      const unfinished = this.draft.find((asset) => this.missing(asset));
      if (unfinished) this.openKey = unfinished.client_key;
    },
  },

  methods: {
    /**
     * Deep-clones an array so edits never mutate the parent's state, and
     * gives older rows the collateral details they're missing.
     */
    copy(value) {
      return JSON.parse(JSON.stringify(value || [])).map((row) => {
        row.collateral = Object.assign(this.emptyCollateral(), row.collateral || {});
        row.collateral.insurance = Object.assign(
          this.emptyCollateral().insurance,
          row.collateral.insurance || {}
        );
        return row;
      });
    },

    /** Empty collateral details: not collateral, with a blank insurance quote. */
    emptyCollateral() {
      return {
        enabled: false,
        id: null,
        description: '',
        document_ids: [],
        insurance: {
          type: '',
          status: 'Quote',
          provider: '',
          reference: '',
          coverage_amount: null,
          premium: null,
          premium_frequency: '',
          expiry_date: '',
        },
      };
    },

    /** Unique client-side key for rows that have no server ID yet. */
    generateRowKey(prefix) {
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

    isCollateral(asset) {
      return this.requiresCollateral && asset.collateral.enabled;
    },

    /** Label of a dropdown value, falling back to the value itself. */
    lookupLabel(fieldName, value) {
      const option = this.lookup(fieldName).find((entry) => entry.value === value);
      return option ? option.label : value || '';
    },

    /** Matches isVehicleType in the main form. */
    isVehicle(asset) {
      return /vehicle|car|auto|truck|motor/i.test(this.lookupLabel('asset_type', asset.asset_type));
    },

    /** Matches isPropertyType in the main form. */
    isProperty(asset) {
      return /property|land|house|home|real estate|building|lot/i.test(
        this.lookupLabel('asset_type', asset.asset_type)
      );
    },

    money(value) {
      if (value === null || value === undefined || value === '') return 'not set';
      return `EC$ ${Number(value).toLocaleString(undefined, {
        minimumFractionDigits: 2,
        maximumFractionDigits: 2,
      })}`;
    },

    /** True when the asset's insurance is a current policy (not a quote). */
    isPolicy(asset) {
      return String(asset.collateral.insurance.status || '').trim().toLowerCase() === 'policy';
    },

    /** True when every assigned owner is a Third Party Owner. */
    ownedOnlyByThirdParty(asset) {
      const rows = (asset.owners || []).filter((row) => row.party_id);
      return (
        rows.length > 0 &&
        rows.every((row) => this.thirdPartyOwnerIds.includes(row.party_id))
      );
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

    /** Updates one field on one asset and notifies the parent. */
    set(index, fieldName, event) {
      this.draft[index][fieldName] = this.valueOf(event);
      this.notify();
    },

    /** Updates one collateral field on one asset. */
    setCollateral(index, fieldName, event) {
      this.draft[index].collateral[fieldName] = this.valueOf(event);
      this.notify();
    },

    /** Updates one insurance field on one asset's collateral. */
    setInsurance(index, fieldName, event, kind) {
      this.draft[index].collateral.insurance[fieldName] = this.valueOf(event, kind);
      this.notify();
    },

    /**
     * Appends a blank asset with a single 100% owner row, ready to be
     * assigned to a party.
     */
    add() {
      this.draft.push({
        client_key: this.generateRowKey('asset'),
        id: null,
        name: '',
        asset_type: '',
        description: '',
        declared_value: null,
        registration_number: '',
        chassis_number: '',
        block_and_parcel: '',
        deed_number: '',
        is_purchase: false,
        document_ids: [],
        owners: [
          {
            client_key: this.generateRowKey('owner'),
            id: null,
            party_id: this.primaryPartyId,
            percentage: 100,
          },
        ],
        collateral: this.emptyCollateral(),
      });
      this.openKey = this.draft[this.draft.length - 1].client_key;
      this.checking = false;
      this.notify();
    },

    // ---- Fold-up cards ----

    /** True when an item's details are showing. */
    isOpen(asset) {
      return this.openKey === asset.client_key;
    },

    openItem(asset) {
      this.openKey = asset.client_key;
      this.checking = false;
    },

    /** "Done": folds the item up, or points out what's missing. */
    finish(asset) {
      if (this.missing(asset)) {
        this.checking = true;
        return;
      }
      this.checking = false;
      this.openKey = '';
    },

    /** True when a required detail is still empty. */
    missing(asset) {
      const empty = (value) => value === null || value === undefined || value === '';
      if (!asset.is_purchase && (empty(asset.name) || empty(asset.asset_type) || empty(asset.declared_value))) {
        return true;
      }
      if (this.requiresCollateral && asset.collateral && asset.collateral.enabled) {
        const insurance = asset.collateral.insurance || {};
        if (['type', 'status', 'provider', 'premium', 'premium_frequency'].some((key) => empty(insurance[key]))) {
          return true;
        }
        if (this.isPolicy(asset) && empty(insurance.reference)) return true;
      }
      return false;
    },

    /** One line under the item's name: its type and value. */
    summaryOf(asset) {
      const parts = [];
      const type = this.lookupLabel('asset_type', asset.asset_type);
      if (type) parts.push(type);
      if (asset.declared_value !== null && asset.declared_value !== undefined && asset.declared_value !== '') {
        parts.push(this.money(asset.declared_value));
      }
      if (this.isCollateral(asset)) parts.push('Secures the loan');
      return parts.join(' · ');
    },

    iconFor(asset) {
      if (this.isVehicle(asset)) return 'mdi-car-outline';
      if (this.isProperty(asset)) return 'mdi-home-outline';
      return 'mdi-diamond-stone';
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
