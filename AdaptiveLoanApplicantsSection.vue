<template>
  <section>
    <el-alert
      type="info"
      :closable="false"
      show-icon
      class="owner-note"
      title="Only the people listed here can own assets on this application."
      description="If an asset you're offering as collateral is fully or partly owned by someone who isn't applying, add them here with the role Third Party Owner. We only need a few details about them."
    />
    <el-tabs
      :model-value="activeTab"
      type="border-card"
      @update:model-value="$emit('update:activeTab', $event)"
    >
      <el-tab-pane
        v-for="(person, index) in applicants"
        :key="person.client_key"
        :label="label(person, index)"
        :name="person.client_key"
      >
        <div v-if="index > 0" class="pane-actions">
          <el-button text type="danger" @click="remove(index - 1)">
            <v-icon start>mdi-delete</v-icon>
            Remove
          </el-button>
        </div>

        <AdaptiveLoanApplicantEditor
          :model-value="person"
          :lookups="lookups"
          :resource-props="resourceProps"
          :role-options="additionalRoleOptions"
          :show-role="index > 0"
          :minimum-identifications="minimumIdentifications"
          :deductions="deductions[person.client_key] || null"
          :document-scopes="documentScopes"
          :uploading-key="uploadingKey"
          :documents-disabled="documentsDisabled"
          @update:model-value="updatePerson(index, $event)"
          @stage-file="$emit('stage-file', $event)"
          @remove-file="$emit('remove-file', $event)"
          @request-file-upload="$emit('request-file-upload', $event)"
          @file-rejected="$emit('file-rejected', $event)"
        />

        <!--
          This applicant's required documents. Uploads unlock once their
          details and consent are complete (and, for co-applicants, once
          the primary applicant is complete).
        -->
        <AdaptiveLoanDocumentRequirements
          title="Applicant documents"
          description="Stored with this applicant's part of the application."
          :scope="documentScopes[`applicant:${person.client_key}`]"
          :uploading-key="uploadingKey"
          :disabled="documentsDisabled"
          @stage-file="$emit('stage-file', $event)"
          @remove-file="$emit('remove-file', $event)"
          @request-file-upload="$emit('request-file-upload', $event)"
          @file-rejected="$emit('file-rejected', $event)"
        />
      </el-tab-pane>
    </el-tabs>

    <el-button
      class="add-button"
      type="primary"
      plain
      @click="$emit('request-add')"
    >
      <v-icon start>mdi-account-plus</v-icon>
      Add another applicant or guarantor
    </el-button>

    <!-- References for the primary applicant: one personal reference, one next of kin -->
    <section class="context references">
      <h3>References</h3>
      <p>Someone who knows you, and your next of kin. They shouldn't be applying with you.</p>
      <article
        v-for="(row, index) in references"
        :key="row.client_key"
        class="item-card"
      >
        <div class="item-title">
          <strong>{{ row.reference_type }}</strong>
        </div>
        <div class="field-grid">
          <el-form-item label="Full name" required>
            <FormField
              :model-value="row.name"
              :property="field('Reference', 'name', 'Full name', 'input')"
              :form="row"
              @update:model-value="setReference(index, 'name', $event)"
            />
          </el-form-item>
          <el-form-item label="Relationship to you" required>
            <FormField
              :model-value="row.relationship"
              :property="field('Reference', 'relationship', 'Relationship to you', 'select')"
              :form="row"
              @update:model-value="setReference(index, 'relationship', $event)"
            />
          </el-form-item>
          <el-form-item label="Phone" required>
            <FormField
              :model-value="row.phone"
              :property="field('Reference', 'phone', 'Phone', 'input')"
              :form="row"
              @update:model-value="setReference(index, 'phone', $event)"
            />
          </el-form-item>
          <el-form-item label="Email">
            <FormField
              :model-value="row.email"
              :property="field('Reference', 'email', 'Email', 'input')"
              :form="row"
              @update:model-value="setReference(index, 'email', $event)"
            />
          </el-form-item>
        </div>
        <el-form-item label="Address">
          <FormField
            :model-value="row.address"
            :property="field('Reference', 'address', 'Address', 'textarea')"
            :form="row"
            @update:model-value="setReference(index, 'address', $event)"
          />
        </el-form-item>
      </article>
    </section>
  </section>
</template>

<script>
/**
 * Applicants step of the loan wizard: one tab per applicant, with the
 * primary applicant first. Fully controlled by the parent. Each tab also
 * shows that applicant's required documents; upload state lives in the
 * parent form.
 *
 * Below the tabs are the primary applicant's references (a personal
 * reference and next of kin), saved as Reference records. Their inputs are
 * Saturn's FormField, using Saturn's own Reference property definitions.
 */
export default {
  props: {
    primary: {
      type: Object,
      required: true,
    },
    parties: {
      type: Array,
      default: () => [],
    },
    lookups: {
      type: Object,
      default: () => ({}),
    },
    /** Saturn's property definitions, keyed by resource name. */
    resourceProps: {
      type: Object,
      default: () => ({}),
    },
    activeTab: {
      type: String,
      default: '',
    },
    /** Estimated NIS, income tax, and net income, keyed by client_key. */
    deductions: {
      type: Object,
      default: () => ({}),
    },
    /** Institution-wide minimum number of identifications per applicant. */
    minimumIdentifications: {
      type: Number,
      default: 1,
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
    /** The personal reference and next of kin (Reference records). */
    references: {
      type: Array,
      default: () => [],
    },
  },

  emits: [
    'update:primary',
    'update:parties',
    'update:references',
    'update:activeTab',
    'request-add',
    'request-remove',
    // Document events are passed straight through to the parent form.
    'stage-file',
    'remove-file',
    'request-file-upload',
    'file-rejected',
  ],

  computed: {
    applicants() {
      return [this.primary].concat(this.parties);
    },

    /** Roles for additional applicants; "Primary Applicant" is reserved. */
    additionalRoleOptions() {
      return (this.lookups.role || []).filter(
        (option) => String(option.value).trim().toLowerCase() !== 'primary applicant'
      );
    },
  },

  methods: {
    label(person, index) {
      const name =
        person.kind === 'ORGANIZATION'
          ? String(person.business_name || '').trim()
          : `${person.first_name || ''} ${person.last_name || ''}`.trim();
      const fallback = index === 0 ? 'Primary Applicant' : 'Additional Party';
      return name ? `${name} · ${person.role || fallback}` : person.role || fallback;
    },

    updatePerson(index, value) {
      if (index === 0) {
        this.$emit('update:primary', value);
      } else {
        const rows = this.parties.slice();
        rows[index - 1] = value;
        this.$emit('update:parties', rows);
      }
    },

    remove(index) {
      this.$emit('request-remove', index);
    },

    /** Updates one field on one reference and sends a fresh copy up. */
    setReference(index, key, event) {
      const rows = JSON.parse(JSON.stringify(this.references));
      rows[index][key] = this.valueOf(event);
      this.$emit('update:references', rows);
    },

    // ---- Saturn FormField helpers (the same in every section) ----

    /**
     * FormField's update event may send the value itself or
     * { property, data } (the Saturn guide isn't clear), so accept both.
     */
    valueOf(event) {
      if (event && typeof event === 'object' && 'property' in event && 'data' in event) {
        return event.data;
      }
      return event;
    },

    /** Saturn's definition of one property of a resource, or null. */
    savedProperty(resourceName, name) {
      const rows = this.resourceProps[resourceName] || [];
      return rows.find((row) => String(row.property || row.key || row.name || '') === name) || null;
    },

    /**
     * FormField property config for one field: Saturn's own definition of
     * the property (with our label) when there is one, otherwise a basic
     * text box. Configs are reused while unchanged, so FormField isn't
     * handed a new object on every keystroke.
     */
    field(resourceName, name, label, kind) {
      const saved = this.savedProperty(resourceName, name);
      const config = saved
        ? Object.assign({}, saved, { property: name, label })
        : {
            property: name,
            label,
            type: 'string',
            input_properties: { type: kind === 'textarea' ? 'textarea' : 'input' },
          };
      if (!this.fieldCache) this.fieldCache = {};
      const cacheKey = JSON.stringify(config);
      if (!this.fieldCache[cacheKey]) this.fieldCache[cacheKey] = config;
      return this.fieldCache[cacheKey];
    },
  },
};
</script>

<style scoped>
.owner-note {
  margin-bottom: 16px;
}

.references {
  margin-top: 24px;
}
</style>
