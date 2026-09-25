<template>
  <section>
    <!-- Section header with add button -->
    <div class="collection-header">
      <div>
        <h3>Collateral</h3>
        <p>Choose what secures this loan, and tell us how it's insured.</p>
      </div>
      <el-button type="primary" plain @click="add">
        <v-icon start>mdi-plus</v-icon>
        Add collateral
      </el-button>
    </div>

    <el-empty v-if="!draft.length" description="No collateral selected" />

    <!-- One card per collateral entry -->
    <article
      v-for="(item, index) in draft"
      :key="item.client_key"
      class="item-card"
    >
      <div class="item-title">
        <strong>Collateral {{ index + 1 }}</strong>
        <el-button text type="danger" @click="remove(index)">Remove</el-button>
      </div>

      <!-- Who owns the asset: one of the applicants, or someone else -->
      <div class="field-grid">
        <el-form-item label="Who owns this asset?" required>
          <el-select
            :model-value="item.ownership"
            @update:model-value="set(index, 'ownership', $event)"
          >
            <el-option label="One of the applicants" value="applicant" />
            <el-option label="Someone else (third party)" value="third_party" />
          </el-select>
        </el-form-item>
      </div>

      <!-- Applicant-owned: pick one of the declared assets -->
      <div v-if="item.ownership !== 'third_party'" class="field-grid">
        <!--
          References assets by server ID when saved, or client key when
          the asset hasn't been persisted yet.
        -->
        <el-form-item label="Declared asset" required>
          <el-select
            :model-value="item.asset_ref"
            placeholder="Select an asset"
            @update:model-value="set(index, 'asset_ref', $event)"
          >
            <el-option
              v-for="asset in assets"
              :key="asset.id || asset.client_key"
              :label="asset.name || 'Unnamed asset'"
              :value="asset.id || asset.client_key"
            />
          </el-select>
        </el-form-item>
      </div>

      <!-- Third-party: describe the asset and its owner on the card -->
      <template v-else>
        <h4 class="subheading">The asset</h4>
        <div class="field-grid">
          <el-form-item label="Asset name" required>
            <el-input
              :model-value="item.third_party.name"
              @input="setIn(index, 'third_party', 'name', $event)"
            />
          </el-form-item>

          <el-form-item label="Asset type" required>
            <el-select
              :model-value="item.third_party.asset_type"
              placeholder="Select a type"
              @update:model-value="setIn(index, 'third_party', 'asset_type', $event)"
            >
              <el-option
                v-for="option in lookup('asset_type')"
                :key="option.value"
                :label="option.label"
                :value="option.value"
              />
            </el-select>
          </el-form-item>

          <el-form-item label="Value (EC$)" required>
            <el-input-number
              :model-value="item.third_party.declared_value"
              :min="0"
              @update:model-value="setIn(index, 'third_party', 'declared_value', $event)"
            />
          </el-form-item>

          <el-form-item label="Asset description">
            <el-input
              :model-value="item.third_party.description"
              @input="setIn(index, 'third_party', 'description', $event)"
            />
          </el-form-item>
        </div>

        <h4 class="subheading">The owner</h4>
        <div class="field-grid">
          <el-form-item label="Owner is a" required>
            <el-select
              :model-value="item.third_party.owner_kind"
              @update:model-value="setIn(index, 'third_party', 'owner_kind', $event)"
            >
              <el-option label="Person" value="PERSON" />
              <el-option label="Business" value="ORGANIZATION" />
            </el-select>
          </el-form-item>

          <el-form-item
            v-if="item.third_party.owner_kind === 'ORGANIZATION'"
            label="Business name"
            required
          >
            <el-input
              :model-value="item.third_party.business_name"
              @input="setIn(index, 'third_party', 'business_name', $event)"
            />
          </el-form-item>

          <template v-else>
            <el-form-item label="First name" required>
              <el-input
                :model-value="item.third_party.first_name"
                @input="setIn(index, 'third_party', 'first_name', $event)"
              />
            </el-form-item>
            <el-form-item label="Last name" required>
              <el-input
                :model-value="item.third_party.last_name"
                @input="setIn(index, 'third_party', 'last_name', $event)"
              />
            </el-form-item>
          </template>

          <el-form-item label="Relationship to you" required>
            <el-select
              v-if="lookup('third_party_relationship').length"
              :model-value="item.third_party.relationship"
              placeholder="Select a relationship"
              @update:model-value="setIn(index, 'third_party', 'relationship', $event)"
            >
              <el-option
                v-for="option in lookup('third_party_relationship')"
                :key="option.value"
                :label="option.label"
                :value="option.value"
              />
            </el-select>
            <el-input
              v-else
              :model-value="item.third_party.relationship"
              placeholder="e.g. Parent"
              @input="setIn(index, 'third_party', 'relationship', $event)"
            />
          </el-form-item>

          <el-form-item label="Phone" required>
            <el-input
              :model-value="item.third_party.phone"
              @input="setIn(index, 'third_party', 'phone', $event)"
            />
          </el-form-item>

          <el-form-item label="Email">
            <el-input
              :model-value="item.third_party.email"
              @input="setIn(index, 'third_party', 'email', $event)"
            />
          </el-form-item>
        </div>
      </template>

      <!-- Fields for every collateral item -->
      <div class="field-grid">
        <el-form-item label="Estimated value (EC$)">
          <el-input-number
            :model-value="item.estimated_value"
            :min="0"
            @update:model-value="set(index, 'estimated_value', $event)"
          />
        </el-form-item>

        <el-form-item label="Description">
          <el-input
            :model-value="item.description"
            @input="set(index, 'description', $event)"
          />
        </el-form-item>
      </div>

      <!-- Insurance: a current policy, or a quote when there's no policy yet -->
      <h4 class="subheading">Insurance</h4>
      <div class="field-grid">
        <el-form-item label="Insurance type" required>
          <el-select
            v-if="lookup('insurance_type').length"
            :model-value="item.insurance.type"
            placeholder="Select a type"
            @update:model-value="setIn(index, 'insurance', 'type', $event)"
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
            :model-value="item.insurance.type"
            placeholder="e.g. Comprehensive"
            @input="setIn(index, 'insurance', 'type', $event)"
          />
        </el-form-item>

        <el-form-item label="Policy or quote?" required>
          <el-select
            :model-value="item.insurance.status"
            @update:model-value="setIn(index, 'insurance', 'status', $event)"
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
            :model-value="item.insurance.provider"
            @input="setIn(index, 'insurance', 'provider', $event)"
          />
        </el-form-item>

        <el-form-item
          :label="isPolicy(item) ? 'Policy number' : 'Quote number'"
          :required="isPolicy(item)"
        >
          <el-input
            :model-value="item.insurance.reference"
            @input="setIn(index, 'insurance', 'reference', $event)"
          />
        </el-form-item>

        <el-form-item label="Amount covered (EC$)">
          <el-input-number
            :model-value="item.insurance.coverage_amount"
            :min="0"
            @update:model-value="setIn(index, 'insurance', 'coverage_amount', $event)"
          />
        </el-form-item>

        <el-form-item label="Premium (EC$)" required>
          <el-input-number
            :model-value="item.insurance.premium"
            :min="0"
            @update:model-value="setIn(index, 'insurance', 'premium', $event)"
          />
        </el-form-item>

        <el-form-item label="Premium paid" required>
          <el-select
            :model-value="item.insurance.premium_frequency"
            placeholder="How often?"
            @update:model-value="setIn(index, 'insurance', 'premium_frequency', $event)"
          >
            <el-option
              v-for="option in lookup('insurance_premium_frequency')"
              :key="option.value"
              :label="option.label"
              :value="option.value"
            />
          </el-select>
        </el-form-item>

        <el-form-item v-if="isPolicy(item)" label="Policy expiry date">
          <el-date-picker
            :model-value="item.insurance.expiry_date"
            type="date"
            value-format="YYYY-MM-DD"
            placeholder="Select a date"
            @update:model-value="setIn(index, 'insurance', 'expiry_date', $event || '')"
          />
        </el-form-item>
      </div>

      <!-- Required documents for this collateral; uploads unlock once the item is complete -->
      <AdaptiveLoanDocumentRequirements
        :scope="documentScopes[`collateral:${item.client_key}`]"
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
 * Collateral step of the loan wizard (only shown for secured loans).
 * Each entry is either one of the applicants' declared assets, or an asset
 * owned by someone else (a third party) described on the card. Every entry
 * also has insurance: a current policy, or a quote when there's no policy
 * yet. Uses the same local draft pattern as the other collection sections:
 * edits happen on a deep-cloned copy and are committed upward via
 * update:modelValue. Required documents for each entry are shown inside
 * its card.
 *
 * The row shape must match createEmptyCollateral() in the main form.
 * The category isn't entered here; the main form derives it from the
 * asset's type when saving.
 */
export default {
  props: {
    /** Collateral array owned by the parent form. */
    modelValue: {
      type: Array,
      default: () => [],
    },
    /** Declared assets available to link as collateral. */
    assets: {
      type: Array,
      default: () => [],
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
     * fills in any missing parts so older rows have every field.
     */
    copy(value) {
      return JSON.parse(JSON.stringify(value || [])).map((row) =>
        this.withDefaults(row),
      );
    },

    /** An empty third-party asset and owner. */
    emptyThirdParty() {
      return {
        asset_id: null,
        ownership_id: null,
        name: '',
        asset_type: '',
        declared_value: null,
        description: '',
        owner_party_id: null,
        owner_kind: 'PERSON',
        first_name: '',
        last_name: '',
        business_name: '',
        relationship: '',
        phone: '',
        email: '',
      };
    },

    /** Empty insurance details (a quote until the applicant says otherwise). */
    emptyInsurance() {
      return {
        type: '',
        status: 'Quote',
        provider: '',
        reference: '',
        coverage_amount: null,
        premium: null,
        premium_frequency: '',
        expiry_date: '',
      };
    },

    /** Adds any missing fields to a row, keeping the values it already has. */
    withDefaults(row) {
      const result = Object.assign(
        {
          ownership: 'applicant',
          asset_ref: '',
          estimated_value: null,
          description: '',
          document_ids: [],
        },
        row,
      );
      result.third_party = Object.assign(this.emptyThirdParty(), row.third_party || {});
      result.insurance = Object.assign(this.emptyInsurance(), row.insurance || {});
      return result;
    },

    /** Unique client-side key for rows that have no server ID yet. */
    generateRowKey(prefix) {
      return `${prefix}_${Date.now()}_${Math.random().toString(36).slice(2, 8)}`;
    },

    /** Dropdown options for a field, or [] when none were loaded. */
    lookup(fieldName) {
      return this.lookups[fieldName] || [];
    },

    /** True when the item's insurance is a current policy (not a quote). */
    isPolicy(item) {
      return String(item.insurance.status || '').trim().toLowerCase() === 'policy';
    },

    /** Pushes the current draft up to the parent. */
    notify() {
      this.$emit('update:modelValue', this.copy(this.draft));
    },

    /** Updates one field on one collateral entry and notifies the parent. */
    set(index, fieldName, value) {
      this.draft[index][fieldName] = value;
      this.notify();
    },

    /** Updates one field inside a group (third_party or insurance). */
    setIn(index, group, fieldName, value) {
      this.draft[index][group][fieldName] = value;
      this.notify();
    },

    /** Appends a blank collateral entry, owned by an applicant by default. */
    add() {
      this.draft.push(
        this.withDefaults({
          client_key: this.generateRowKey('collateral'),
          id: null,
        }),
      );
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
