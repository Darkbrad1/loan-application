<template>
  <section>
    <p v-if="!draft.length" class="helper block w-full mt-1 empty-note">Nothing added yet.</p>

    <!-- One card per declared asset -->
    <article
      v-for="(asset, index) in draft"
      :key="asset.client_key"
      class="item-card"
    >
      <div class="item-title">
        <strong>{{ asset.name || `Item ${index + 1}` }}</strong>
        <div>
          <el-tag v-if="asset.is_purchase" type="success" size="small">Buying with this loan</el-tag>
          <el-tag v-if="isCollateral(asset)" type="warning" size="small">Secures the loan</el-tag>
          <el-button v-if="!asset.is_purchase" text type="danger" @click="remove(index)">Remove</el-button>
        </div>
      </div>

      <!-- The vehicle or property being bought comes from "Your request" -->
      <div v-if="asset.is_purchase" class="purchase-summary mb-3">
        <p>
          You're buying this for {{ money(asset.declared_value) }}.
        </p>
        <p class="helper block w-full mt-1">
          To change it, go back to "The loan you need". Below, tell us who will
          own it and how it will be insured.
        </p>
      </div>

      <div v-else class="field-grid">
        <el-form-item label="What is it?" required :error="need(asset.name)">
          <FormField
            :model-value="asset.name"
            :property="field('Asset', 'name', 'Asset name', 'input')"
            :form="asset"
            @update:model-value="set(index, 'name', $event)"
          />
          <small class="helper block w-full mt-1">For example "My house in Grand Anse" or "2018 Honda Fit".</small>
        </el-form-item>

        <el-form-item label="Type" required :error="need(asset.asset_type)">
          <FormField
            :model-value="asset.asset_type"
            :property="field('Asset', 'asset_type', 'Asset type', 'select')"
            :form="asset"
            @update:model-value="set(index, 'asset_type', $event)"
          />
        </el-form-item>

        <el-form-item label="What is it worth? (EC$)" required :error="need(asset.declared_value)">
          <FormField
            :model-value="asset.declared_value"
            :property="field('Asset', 'declared_value', 'Declared value (EC$)', 'number')"
            :form="asset"
            @update:model-value="set(index, 'declared_value', $event)"
          />
        </el-form-item>

        <el-form-item v-if="asset.description || moreFor[asset.client_key]" label="Description">
          <FormField
            :model-value="asset.description"
            :property="field('Asset', 'description', 'Description', 'input')"
            :form="asset"
            @update:model-value="set(index, 'description', $event)"
          />
        </el-form-item>

        <!-- Identifiers for vehicles and for land or property -->
        <template v-if="isVehicle(asset)">
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
        </template>
        <template v-if="isProperty(asset)">
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
        </template>
      </div>

      <!-- Existing loans secured on this asset (from the Liabilities step) -->
      <el-alert
        v-if="(liens[asset.client_key] || []).length"
        type="warning"
        :closable="false"
        show-icon
        class="lien-note mt-2 mb-3"
        :title="`You still owe money on this: ${liens[asset.client_key].join('; ')}`"
      />

      <el-button
        v-if="!asset.is_purchase && !asset.description && !moreFor[asset.client_key]"
        text
        type="primary"
        @click="moreFor[asset.client_key] = true"
      >
        + Add a description
      </el-button>

      <!-- Ownership percentages must total 100% across saved parties -->
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
      <small v-if="ownedOnlyByThirdParty(asset)" class="helper block w-full mt-1">
        Someone who isn't borrowing owns all of this, so it can only be listed
        if it secures the loan.
      </small>

      <!-- Required documents for this asset; uploads unlock once it's complete -->
      <AdaptiveLoanDocumentRequirements
        :scope="documentScopes[`asset:${asset.client_key}`]"
        :uploading-key="uploadingKey"
        :disabled="documentsDisabled"
        @stage-file="$emit('stage-file', $event)"
        @remove-file="$emit('remove-file', $event)"
        @request-file-upload="$emit('request-file-upload', $event)"
        @file-rejected="$emit('file-rejected', $event)"
      />

      <!-- Collateral: only for loans that need it -->
      <section v-if="requiresCollateral" class="context">
        <!-- The asset being bought is always the collateral -->
        <p v-if="asset.is_purchase" class="helper block w-full mt-1">
          This secures the loan. Tell us how it's insured below; a quote is fine for now.
        </p>
        <el-form-item v-else label="Use this to secure the loan">
          <FormField
            :model-value="asset.collateral.enabled"
            :property="field(null, 'enabled', 'Use this to secure the loan', 'checkbox')"
            :form="asset.collateral"
            @update:model-value="setCollateral(index, 'enabled', $event)"
          />
        </el-form-item>

        <template v-if="asset.collateral.enabled">
          <h4 class="subheading mt-4 mb-2 font-semibold">How is it insured?</h4>
          <p class="helper block w-full mt-1">If you don't have insurance yet, get a quote from an insurer and enter it here.</p>
          <div class="field-grid">
            <el-form-item label="Type of insurance" required :error="need(asset.collateral.insurance.type)">
              <FormField
                :model-value="asset.collateral.insurance.type"
                :property="field('Collateral', 'insurance_type', 'Insurance type', 'select')"
                :form="asset.collateral.insurance"
                @update:model-value="setInsurance(index, 'type', $event)"
              />
            </el-form-item>

            <el-form-item label="Do you have a policy, or a quote?" required :error="need(asset.collateral.insurance.status)">
              <FormField
                :model-value="asset.collateral.insurance.status"
                :property="field('Collateral', 'insurance_status', 'Policy or quote?', 'select')"
                :form="asset.collateral.insurance"
                @update:model-value="setInsurance(index, 'status', $event)"
              />
            </el-form-item>

            <el-form-item label="Insurance company" required :error="need(asset.collateral.insurance.provider)">
              <FormField
                :model-value="asset.collateral.insurance.provider"
                :property="field('Collateral', 'insurance_provider', 'Insurer', 'input')"
                :form="asset.collateral.insurance"
                @update:model-value="setInsurance(index, 'provider', $event)"
              />
            </el-form-item>

            <el-form-item
              :label="isPolicy(asset) ? 'Policy number' : 'Quote number'"
              :required="isPolicy(asset)"
            >
              <FormField
                :model-value="asset.collateral.insurance.reference"
                :property="field('Collateral', 'insurance_reference', isPolicy(asset) ? 'Policy number' : 'Quote number', 'input')"
                :form="asset.collateral.insurance"
                @update:model-value="setInsurance(index, 'reference', $event)"
              />
            </el-form-item>

            <el-form-item v-if="insuranceMore[asset.client_key] || asset.collateral.insurance.coverage_amount" label="Amount covered (EC$)">
              <FormField
                :model-value="asset.collateral.insurance.coverage_amount"
                :property="field('Collateral', 'insurance_coverage_amount', 'Amount covered (EC$)', 'number')"
                :form="asset.collateral.insurance"
                @update:model-value="setInsurance(index, 'coverage_amount', $event)"
              />
            </el-form-item>

            <el-form-item label="Cost of the insurance (EC$)" required :error="need(asset.collateral.insurance.premium)">
              <FormField
                :model-value="asset.collateral.insurance.premium"
                :property="field('Collateral', 'insurance_premium', 'Premium (EC$)', 'number')"
                :form="asset.collateral.insurance"
                @update:model-value="setInsurance(index, 'premium', $event)"
              />
            </el-form-item>

            <el-form-item label="How often do you pay it?" required :error="need(asset.collateral.insurance.premium_frequency)">
              <FormField
                :model-value="asset.collateral.insurance.premium_frequency"
                :property="field('Collateral', 'insurance_premium_frequency', 'Premium paid', 'select')"
                :form="asset.collateral.insurance"
                @update:model-value="setInsurance(index, 'premium_frequency', $event)"
              />
            </el-form-item>

            <el-form-item v-if="isPolicy(asset)" label="Policy expiry date">
              <FormField
                :model-value="asset.collateral.insurance.expiry_date"
                :property="field('Collateral', 'insurance_expiry_date', 'Policy expiry date', 'date')"
                :form="asset.collateral.insurance"
                @update:model-value="setInsurance(index, 'expiry_date', $event, 'date')"
              />
            </el-form-item>
          </div>

          <el-button
            v-if="!insuranceMore[asset.client_key] && !asset.collateral.insurance.coverage_amount"
            text
            type="primary"
            @click="insuranceMore[asset.client_key] = true"
          >
            + Add the amount covered and notes
          </el-button>
          <el-form-item v-if="insuranceMore[asset.client_key] || asset.collateral.description" label="Notes">
            <FormField
              :model-value="asset.collateral.description"
              :property="field('Collateral', 'description', 'Collateral notes', 'input')"
              :form="asset.collateral"
              @update:model-value="setCollateral(index, 'description', $event)"
            />
          </el-form-item>

          <!-- Collateral documents; uploads unlock once the asset is complete -->
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
      </section>
    </article>

    <el-button type="primary" plain size="large" @click="add">
      <v-icon start>mdi-plus</v-icon>
      Add {{ draft.length ? 'something else' : 'something you own' }}
    </el-button>
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
    };
  },

  mounted() {
    // They said they own something, so start with one to fill in.
    if (!this.draft.length) this.add();
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
      if (!this.showErrors) return '';
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
.subheading {
  margin: 16px 0 8px;
  font-size: 0.95rem;
  font-weight: 600;
}

.purchase-summary {
  margin-bottom: 12px;
}

.lien-note {
  margin: 8px 0 12px;
}
</style>
