<template>
  <section>
    <!-- Section header with add button -->
    <div class="collection-header">
      <div>
        <h3>Collateral</h3>
        <p>Collateral links to a stable declared Asset.</p>
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

      <div class="field-grid">
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

        <el-form-item label="Category">
          <el-select
            :model-value="item.category"
            @update:model-value="set(index, 'category', $event)"
          >
            <el-option
              v-for="option in lookup('collateral_category')"
              :key="option.value"
              :label="option.label"
              :value="option.value"
            />
          </el-select>
        </el-form-item>

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

      <!-- Required documents for this collateral; uploads unlock once an asset is selected -->
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
 * Each collateral entry links to a declared asset. Uses the same local
 * draft pattern as the other collection sections: edits happen on a
 * deep-cloned copy and are committed upward via update:modelValue.
 * Required documents for each entry are shown inside its card.
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

    /** Updates one field on one collateral entry and notifies the parent. */
    set(index, fieldName, value) {
      this.draft[index][fieldName] = value;
      this.notify();
    },

    /** Appends a blank collateral entry with no asset linked yet. */
    add() {
      this.draft.push({
        client_key: this.generateRowKey('collateral'),
        id: null,
        asset_ref: '',
        category: '',
        estimated_value: null,
        description: '',
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