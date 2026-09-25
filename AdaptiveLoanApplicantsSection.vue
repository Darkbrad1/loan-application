<template>
  <section>
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
  </section>
</template>

<script>
/**
 * Applicants step of the loan wizard: one tab per applicant, with the
 * primary applicant first. Fully controlled by the parent. Each tab also
 * shows that applicant's required documents; upload state lives in the
 * parent form.
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
  },

  emits: [
    'update:primary',
    'update:parties',
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
      return [this.primary, ...this.parties];
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
      const name = `${person.first_name || ''} ${person.last_name || ''}`.trim();
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
  },
};
</script>