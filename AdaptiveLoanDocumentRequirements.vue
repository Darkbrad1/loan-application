<template>
  <!--
    Shares the .document-section styles defined in the parent form, so the
    requirement cards look the same everywhere they're used.
  -->
  <section
    v-if="requirements.length"
    class="document-section item-documents"
    :class="{ standalone }"
  >
    <div class="documents-heading">
      <div>
        <h4 v-text="title"></h4>
        <p v-if="description" v-text="description"></p>
      </div>
      <span
        class="scope-badge"
        :class="{ complete: complete }"
        v-text="complete ? 'Complete' : `${uploadedCount}/${requirements.length} uploaded`"
      ></span>
    </div>

    <div v-if="lockedMessage" class="lock-notice">
      <v-icon size="small">mdi-lock-outline</v-icon>
      <span v-text="lockedMessage"></span>
    </div>

    <div class="requirement-list">
      <section
        v-for="(requirement, index) in requirements"
        :key="requirementKey(requirement, index)"
        class="requirement-card"
        :class="{
          uploaded: isUploaded(requirement),
          uploading: isUploading(requirement),
          error: hasError(requirement),
        }"
      >
        <div class="requirement-top">
          <div class="document-mark" aria-hidden="true">
            <v-icon v-if="isUploaded(requirement)">mdi-check</v-icon>
            <v-icon v-else>mdi-file-document-outline</v-icon>
          </div>
          <div class="requirement-title">
            <div class="title-line">
              <h5 v-text="requirement.name || `Document ${index + 1}`"></h5>
              <span
                class="status-pill"
                :class="statusClass(requirement)"
                v-text="statusLabel(requirement)"
              ></span>
            </div>
            <p v-if="requirement.description" v-text="requirement.description"></p>
            <small v-text="allowedLabel(requirement)"></small>
          </div>
        </div>

        <div v-if="isUploaded(requirement)" class="uploaded-file">
          <v-icon color="success">mdi-check-circle</v-icon>
          <div>
            <strong v-text="fileName(requirement) || 'Document attached'"></strong>
            <span>Uploaded and linked.</span>
          </div>
        </div>

        <template v-else>
          <label
            v-if="!hasStagedFile(requirement)"
            class="drop-zone"
            :class="{ disabled: inputDisabled(requirement) }"
            @dragover.prevent
            @drop.prevent="onDrop(requirement, $event)"
          >
            <input
              type="file"
              :accept="acceptFor(requirement)"
              :disabled="inputDisabled(requirement)"
              @change="onBrowse(requirement, $event)"
            />
            <span class="upload-symbol" aria-hidden="true">
              <v-icon>mdi-cloud-upload-outline</v-icon>
            </span>
            <strong>Drag a file here or <u>browse</u></strong>
            <small>Choosing a file doesn't upload it yet.</small>
          </label>

          <div v-else class="staged-file">
            <div class="staged-icon" aria-hidden="true">
              <v-icon>mdi-file-check-outline</v-icon>
            </div>
            <div class="staged-details">
              <strong v-text="fileName(requirement)"></strong>
              <span v-text="fileSizeLabel(requirement)"></span>
            </div>
            <el-button
              text
              :disabled="disabled || isUploading(requirement)"
              @click="removeStaged(requirement)"
            >
              Remove
            </el-button>
            <el-button
              type="primary"
              :loading="isUploading(requirement)"
              :disabled="disabled || Boolean(lockedMessage)"
              @click="requestUpload(requirement)"
            >
              <v-icon v-if="!isUploading(requirement)" start>mdi-upload</v-icon>
              <span v-text="hasError(requirement) ? 'Retry upload' : 'Upload'"></span>
            </el-button>
          </div>

          <div v-if="errorFor(requirement)" class="error-message" role="alert">
            <v-icon size="small">mdi-alert-circle-outline</v-icon>
            <span v-text="errorFor(requirement)"></span>
          </div>
        </template>
      </section>
    </div>
  </section>
</template>

<script>
/**
 * The required documents for one owner: the application, one applicant,
 * or one asset, liability, expense, or collateral item. Fully controlled:
 * upload state lives in the parent form, and this component only emits
 * what the applicant does.
 *
 * Every emitted payload carries scopeKey and requirementKey, which the
 * parent uses to find the owner record and the attachment type.
 */
export default {
  props: {
    /**
     * { key, label, requirements, lockedMessage } from the parent's
     * documentScopes. Nothing renders when it's missing or has no
     * requirements.
     */
    scope: {
      type: Object,
      default: null,
    },
    /** "<scopeKey>::<requirementKey>" of the upload in progress, if any. */
    uploadingKey: {
      type: String,
      default: '',
    },
    disabled: {
      type: Boolean,
      default: false,
    },
    title: {
      type: String,
      default: 'Supporting documents',
    },
    description: {
      type: String,
      default: '',
    },
    /** Used on its own step rather than inside an item card. */
    standalone: {
      type: Boolean,
      default: false,
    },
  },

  emits: ['stage-file', 'remove-file', 'request-file-upload', 'file-rejected'],

  data() {
    return {
      // Format errors from rejected files, keyed by requirement.
      transientErrors: {},
    };
  },

  computed: {
    requirements() {
      return Array.isArray(this.scope?.requirements) ? this.scope.requirements : [];
    },

    scopeKey() {
      return String(this.scope?.key || '');
    },

    lockedMessage() {
      return this.scope?.lockedMessage || '';
    },

    uploadedCount() {
      return this.requirements.filter(this.isUploaded).length;
    },

    complete() {
      return this.requirements.length > 0 && this.uploadedCount === this.requirements.length;
    },
  },

  methods: {
    requirementKey(requirement, index = 0) {
      return String(
        (requirement &&
          (requirement.key || requirement.id || requirement.attachmentTypeId)) ||
          `requirement-${index}`
      );
    },

    slotKey(requirement) {
      return `${this.scopeKey}::${this.requirementKey(requirement)}`;
    },

    isUploaded(requirement) {
      return String(requirement?.status || '').toLowerCase() === 'uploaded';
    },

    isUploading(requirement) {
      return (
        String(requirement?.status || '').toLowerCase() === 'uploading' ||
        this.uploadingKey === this.slotKey(requirement)
      );
    },

    inputDisabled(requirement) {
      return this.disabled || Boolean(this.lockedMessage) || this.isUploading(requirement);
    },

    statusLabel(requirement) {
      if (this.isUploaded(requirement)) return 'Uploaded';
      if (this.isUploading(requirement)) return 'Uploading';
      if (this.hasError(requirement)) return 'Needs attention';
      if (this.hasStagedFile(requirement)) return 'Ready';
      return 'Required';
    },

    statusClass(requirement) {
      if (this.isUploaded(requirement)) return 'success';
      if (this.isUploading(requirement)) return 'working';
      if (this.hasError(requirement)) return 'danger';
      if (this.hasStagedFile(requirement)) return 'ready';
      return 'required';
    },

    hasStagedFile(requirement) {
      return Boolean(requirement?.file || requirement?.fileName);
    },

    fileName(requirement) {
      return (
        requirement?.fileName ||
        requirement?.uploadedFileName ||
        requirement?.file?.name ||
        ''
      );
    },

    fileSizeLabel(requirement) {
      const size = Number(requirement?.file?.size || requirement?.fileSize || 0);
      if (!size) return 'Ready to upload';
      if (size < 1024) return `${size} B · Ready to upload`;
      if (size < 1024 * 1024) return `${Math.round(size / 1024)} KB · Ready to upload`;
      return `${(size / (1024 * 1024)).toFixed(1)} MB · Ready to upload`;
    },

    /** Allowed types as ".pdf" extensions or "image/*" MIME patterns. */
    allowedValues(requirement) {
      const raw = requirement?.allowed_file_types || requirement?.allowedFileTypes || [];
      const values = Array.isArray(raw) ? raw : String(raw).split(',');
      return values
        .map((value) => String(value || '').trim().toLowerCase())
        .filter(Boolean)
        .map((value) => (value.includes('/') || value.startsWith('.') ? value : `.${value}`));
    },

    acceptFor(requirement) {
      return this.allowedValues(requirement).join(',');
    },

    allowedLabel(requirement) {
      const allowed = this.allowedValues(requirement);
      if (!allowed.length) return 'All standard document formats accepted';
      return `Accepted: ${allowed.map((value) => value.toUpperCase()).join(', ')}`;
    },

    acceptsFile(requirement, file) {
      const allowed = this.allowedValues(requirement);
      if (!allowed.length || !file) return true;

      const name = String(file.name || '').toLowerCase();
      const type = String(file.type || '').toLowerCase();
      return allowed.some((value) => {
        if (value.startsWith('.')) return name.endsWith(value);
        if (value.endsWith('/*')) return type.startsWith(value.slice(0, -1));
        return type === value;
      });
    },

    errorFor(requirement) {
      return this.transientErrors[this.slotKey(requirement)] || requirement?.error || '';
    },

    hasError(requirement) {
      return (
        Boolean(this.errorFor(requirement)) ||
        String(requirement?.status || '').toLowerCase() === 'error'
      );
    },

    payloadFor(requirement) {
      return {
        scopeKey: this.scopeKey,
        requirementKey: this.requirementKey(requirement),
      };
    },

    stage(requirement, file) {
      if (!file || this.inputDisabled(requirement)) return;

      if (!this.acceptsFile(requirement, file)) {
        const message = `Choose one of the allowed formats: ${this.allowedLabel(requirement).replace(
          'Accepted: ',
          ''
        )}.`;
        this.transientErrors[this.slotKey(requirement)] = message;
        this.$emit('file-rejected', { ...this.payloadFor(requirement), file, message });
        return;
      }

      delete this.transientErrors[this.slotKey(requirement)];
      this.$emit('stage-file', { ...this.payloadFor(requirement), file });
    },

    onBrowse(requirement, event) {
      const file = event.target.files && event.target.files[0];
      this.stage(requirement, file);
      event.target.value = '';
    },

    onDrop(requirement, event) {
      const file = event.dataTransfer.files && event.dataTransfer.files[0];
      this.stage(requirement, file);
    },

    removeStaged(requirement) {
      delete this.transientErrors[this.slotKey(requirement)];
      this.$emit('remove-file', this.payloadFor(requirement));
    },

    requestUpload(requirement) {
      if (!this.hasStagedFile(requirement) || this.disabled || this.lockedMessage) return;
      delete this.transientErrors[this.slotKey(requirement)];
      this.$emit('request-file-upload', this.payloadFor(requirement));
    },
  },
};
</script>