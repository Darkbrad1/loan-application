<template>
  <section>
    <!-- Section header with add button -->
    <div class="collection-header">
      <div>
        <h3>Assets</h3>
        <p>
          Only the people on the Applicants step can own assets here. If someone
          else owns all or part of an asset, go back and add them as a Third
          Party Owner.
        </p>
        <p v-if="requiresCollateral">
          This loan needs collateral. Mark the asset or assets securing it as
          collateral.
        </p>
      </div>
      <el-button type="primary" plain @click="add">
        <v-icon start>mdi-plus</v-icon>
        Add asset
      </el-button>
    </div>

    <el-empty v-if="!draft.length" description="No assets declared" />

    <!-- One card per declared asset -->
    <article
      v-for="(asset, index) in draft"
      :key="asset.client_key"
      class="item-card"
    >
      <div class="item-title">
        <strong>{{ asset.name || `Asset ${index + 1}` }}</strong>
        <div>
          <el-tag v-if="isCollateral(asset)" type="warning" size="small">Collateral</el-tag>
          <el-button text type="danger" @click="remove(index)">Remove</el-button>
        </div>
      </div>

      <div class="field-grid">
        <el-form-item label="Asset name" required>
          <el-input
            :model-value="asset.name"
            @input="set(index, 'name', $event)"
          />
        </el-form-item>

        <el-form-item label="Asset type" required>
          <el-select
            :model-value="asset.asset_type"
            placeholder="Select asset type"
            @update:model-value="set(index, 'asset_type', $event)"
          >
            <el-option
              v-for="option in lookup('asset_type')"
              :key="option.value"
              :label="option.label"
              :value="option.value"
            />
          </el-select>
        </el-form-item>

        <el-form-item label="Declared value (EC$)" required>
          <el-input-number
            :model-value="asset.declared_value"
            :min="0"
            @update:model-value="set(index, 'declared_value', $event)"
          />
        </el-form-item>

        <el-form-item label="Description">
          <el-input
            :model-value="asset.description"
            @input="set(index, 'description', $event)"
          />
        </el-form-item>
      </div>

      <!-- Ownership percentages must total 100% across saved parties -->
      <AdaptiveLoanAllocationEditor
        :model-value="asset.owners"
        :options="partyOptions"
        reference-key="party_id"
        title="Ownership"
        select-label="Party owner"
        total-label="Ownership total"
        @update:model-value="set(index, 'owners', $event)"
      />
      <small v-if="ownedOnlyByThirdParty(asset)" class="helper">
        This asset is owned only by a Third Party Owner, so it can only be on
        this application as collateral.
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
        <el-checkbox
          :model-value="asset.collateral.enabled"
          @update:model-value="setCollateral(index, 'enabled', $event)"
        >
          Use this asset as collateral for this loan
        </el-checkbox>

        <template v-if="asset.collateral.enabled">
          <h4 class="subheading">Insurance</h4>
          <div class="field-grid">
            <el-form-item label="Insurance type" required>
              <el-select
                v-if="lookup('insurance_type').length"
                :model-value="asset.collateral.insurance.type"
                placeholder="Select a type"
                @update:model-value="setInsurance(index, 'type', $event)"
              >
                <el-option
                  v-for="option in lookup('insurance_type')"
                  :key="option.value"
                  :label="option.label"
                  :value="option.value"
                />
              </el-select>
              <el-input
                v-else
                :model-value="asset.collateral.insurance.type"
                placeholder="e.g. Comprehensive"
                @input="setInsurance(index, 'type', $event)"
              />
            </el-form-item>

            <el-form-item label="Policy or quote?" required>
              <el-select
                :model-value="asset.collateral.insurance.status"
                @update:model-value="setInsurance(index, 'status', $event)"
              >
                <el-option
                  v-for="option in insuranceStatusOptions"
                  :key="option.value"
                  :label="option.label"
                  :value="option.value"
                />
              </el-select>
            </el-form-item>

            <el-form-item label="Insurer" required>
              <el-input
                :model-value="asset.collateral.insurance.provider"
                @input="setInsurance(index, 'provider', $event)"
              />
            </el-form-item>

            <el-form-item
              :label="isPolicy(asset) ? 'Policy number' : 'Quote number'"
              :required="isPolicy(asset)"
            >
              <el-input
                :model-value="asset.collateral.insurance.reference"
                @input="setInsurance(index, 'reference', $event)"
              />
            </el-form-item>

            <el-form-item label="Amount covered (EC$)">
              <el-input-number
                :model-value="asset.collateral.insurance.coverage_amount"
                :min="0"
                @update:model-value="setInsurance(index, 'coverage_amount', $event)"
              />
            </el-form-item>

            <el-form-item label="Premium (EC$)" required>
              <el-input-number
                :model-value="asset.collateral.insurance.premium"
                :min="0"
                @update:model-value="setInsurance(index, 'premium', $event)"
              />
            </el-form-item>

            <el-form-item label="Premium paid" required>
              <el-select
                :model-value="asset.collateral.insurance.premium_frequency"
                placeholder="How often?"
                @update:model-value="setInsurance(index, 'premium_frequency', $event)"
              >
                <el-option
                  v-for="option in lookup('insurance_premium_frequency')"
                  :key="option.value"
                  :label="option.label"
                  :value="option.value"
                />
              </el-select>
            </el-form-item>

            <el-form-item v-if="isPolicy(asset)" label="Policy expiry date">
              <el-date-picker
                :model-value="asset.collateral.insurance.expiry_date"
                type="date"
                value-format="YYYY-MM-DD"
                placeholder="Select a date"
                @update:model-value="setInsurance(index, 'expiry_date', $event || '')"
              />
            </el-form-item>
          </div>

          <el-form-item label="Collateral notes">
            <el-input
              :model-value="asset.collateral.description"
              @input="setCollateral(index, 'description', $event)"
            />
          </el-form-item>

          <!-- Collateral documents; uploads unlock once the asset is complete -->
          <AdaptiveLoanDocumentRequirements
            title="Collateral documents"
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
 */
export default {
  props: {
    /** Assets array owned by the parent form. */
    modelValue: {
      type: Array,
      default: () => [],
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
    };
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

    isCollateral(asset) {
      return this.requiresCollateral && asset.collateral.enabled;
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

    /** Pushes the current draft up to the parent. */
    notify() {
      this.$emit('update:modelValue', this.copy(this.draft));
    },

    /** Updates one field on one asset and notifies the parent. */
    set(index, fieldName, value) {
      this.draft[index][fieldName] = value;
      this.notify();
    },

    /** Updates one collateral field on one asset. */
    setCollateral(index, fieldName, value) {
      this.draft[index].collateral[fieldName] = value;
      this.notify();
    },

    /** Updates one insurance field on one asset's collateral. */
    setInsurance(index, fieldName, value) {
      this.draft[index].collateral.insurance[fieldName] = value;
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
        document_ids: [],
        owners: [
          {
            client_key: this.generateRowKey('owner'),
            id: null,
            party_id: '',
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
</style>
