<template>
  <section>
    <!-- Section header with add button -->
    <div class="collection-header">
      <div>
        <h3>Assets</h3>
        <p>Ownership is allocated to saved Party records.</p>
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
        <el-button text type="danger" @click="remove(index)">Remove</el-button>
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
    </article>
  </section>
</template>

<script>
/**
 * Assets step of the loan wizard. Maintains a local editable copy
 * (draft) of the assets array so the parent only receives clean,
 * committed updates via update:modelValue. Required documents for each
 * asset are shown inside its card; upload state lives in the parent.
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
    generateRowKey(prefix) {
      return `${prefix}_${Date.now()}_${Math.random().toString(36).slice(2, 8)}`;
    },

    /** Dropdown options for a field, or [] when none were loaded. */
    lookup(fieldName) {
      return this.lookups[fieldName] || [];
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