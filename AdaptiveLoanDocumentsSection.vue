<template>
  <section class="document-section">
    <template v-if="hasRequirements">
      <div
        class="progress-track"
        role="progressbar"
        aria-label="Overall document upload progress"
        :aria-valuemin="0"
        :aria-valuemax="totalRequirements"
        :aria-valuenow="uploadedRequirements"
      >
        <i :style="{ width: progressPercent + '%' }"></i>
      </div>

      <nav class="scope-tabs" role="tablist" aria-label="Document owners">
        <button
          v-for="(scope, index) in displayScopes"
          :key="scopeKey(scope, index)"
          type="button"
          role="tab"
          :aria-selected="isActive(scope, index)"
          :disabled="!scopeUnlocked(index)"
          :class="{
            active: isActive(scope, index),
            complete: scopeComplete(scope),
            locked: !scopeUnlocked(index)
          }"
          @click="selectScope(scope, index)"
        >
          <span class="tab-icon" aria-hidden="true">
            <v-icon v-if="scopeComplete(scope)" size="small">mdi-check</v-icon>
            <v-icon v-else-if="!scopeUnlocked(index)" size="small">mdi-lock-outline</v-icon>
            <span v-else v-text="index + 1"></span>
          </span>
          <span class="tab-text">
            <b v-text="scope.label || `Documents ${index + 1}`"></b>
            <small v-text="scopeProgress(scope)"></small>
          </span>
        </button>
      </nav>

      <article v-if="activeScope" class="scope-panel" role="tabpanel">
        <div class="scope-heading">
          <div>
            <p class="scope-kicker" v-text="activeScope.roleLabel || 'Required documents'"></p>
            <h4 v-text="activeScope.label"></h4>
            <p v-if="activeScope.description" v-text="activeScope.description"></p>
          </div>
          <span
            class="scope-badge"
            :class="{ complete: scopeComplete(activeScope) }"
            v-text="scopeComplete(activeScope) ? 'Complete' : scopeProgress(activeScope)"
          ></span>
        </div>

        <div class="requirement-list">
          <section
            v-for="(requirement, requirementIndex) in requirementsOf(activeScope)"
            :key="requirementKey(requirement, requirementIndex)"
            class="requirement-card"
            :class="{
              uploaded: isUploaded(requirement),
              uploading: isUploading(activeScope, requirement),
              error: hasError(requirement)
            }"
          >
            <div class="requirement-top">
              <div class="document-mark" aria-hidden="true">
                <v-icon v-if="isUploaded(requirement)">mdi-check</v-icon>
                <v-icon v-else>mdi-file-document-outline</v-icon>
              </div>
              <div class="requirement-title">
                <div class="title-line">
                  <h5 v-text="requirement.name || `Document ${requirementIndex + 1}`"></h5>
                  <span
                    class="status-pill"
                    :class="statusClass(activeScope, requirement)"
                    v-text="statusLabel(activeScope, requirement)"
                  ></span>
                </div>
                <p
                  v-if="requirement.description"
                  v-text="requirement.description"
                ></p>
                <small v-text="allowedLabel(requirement)"></small>
              </div>
            </div>

            <div v-if="isUploaded(requirement)" class="uploaded-file">
              <v-icon color="success">mdi-check-circle</v-icon>
              <div>
                <strong v-text="fileName(requirement) || 'Document attached'"></strong>
                <span>Successfully uploaded and linked.</span>
              </div>
            </div>

            <template v-else>
              <label
                v-if="!hasStagedFile(requirement)"
                class="drop-zone"
                :class="{ disabled: disabled || isUploading(activeScope, requirement) }"
                @dragover.prevent
                @drop.prevent="onDrop(activeScope, requirement, $event)"
              >
                <input
                  type="file"
                  :accept="acceptFor(requirement)"
                  :disabled="disabled || isUploading(activeScope, requirement)"
                  @change="onBrowse(activeScope, requirement, $event)"
                />
                <span class="upload-symbol" aria-hidden="true">
                  <v-icon>mdi-cloud-upload-outline</v-icon>
                </span>
                <strong>Drag a file here or <u>browse</u></strong>
                <small>Selection does not start the upload.</small>
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
                  :disabled="disabled || isUploading(activeScope, requirement)"
                  @click="removeStaged(activeScope, requirement)"
                >
                  Remove
                </el-button>
                <el-button
                  type="primary"
                  :loading="isUploading(activeScope, requirement)"
                  :disabled="disabled"
                  @click="requestUpload(activeScope, requirement)"
                >
                  <v-icon v-if="!isUploading(activeScope, requirement)" start>
                    mdi-upload
                  </v-icon>
                  <span v-text="hasError(requirement) ? 'Retry upload' : 'Upload'"></span>
                </el-button>
              </div>

              <div
                v-if="errorFor(requirement)"
                class="error-message"
                role="alert"
              >
                <v-icon size="small">mdi-alert-circle-outline</v-icon>
                <span v-text="errorFor(requirement)"></span>
              </div>
            </template>
          </section>
        </div>

        <div v-if="scopeComplete(activeScope)" class="scope-complete" aria-live="polite">
          <v-icon>mdi-check-circle</v-icon>
          <div>
            <strong v-text="`${activeScope.label} documents complete`"></strong>
            <span v-text="nextScopeMessage"></span>
          </div>
        </div>
      </article>
    </template>

    <!-- Backward-compatible queue used until the parent form supplies scopes. -->
    <section v-else class="legacy-queue">
      <el-upload
        action="#"
        multiple
        drag
        :auto-upload="false"
        :file-list="queuedDocuments"
        :on-change="queueLegacy"
        :on-remove="removeLegacy"
      >
        <v-icon size="large">mdi-cloud-upload-outline</v-icon>
        <div class="legacy-copy">Drop documents here or click to browse</div>
        <small>Files upload only after the button below is clicked.</small>
      </el-upload>

      <el-button
        v-if="queuedDocuments.length"
        type="primary"
        :loading="uploading"
        class="legacy-upload-button"
        @click="$emit('request-upload')"
      >
        Upload <span v-text="queuedDocuments.length"></span> document(s)
      </el-button>

      <el-alert
        v-if="documentIds.length"
        type="success"
        :closable="false"
        :title="`${documentIds.length} document(s) attached.`"
        show-icon
      />
    </section>
  </section>
</template>

<script>
export default {
  props: {
    /**
     * Controlled document scopes. Each scope contains label, key/type,
     * ownerId and a requirements array. Requirements carry staged file,
     * status, upload ID, error and Attachment_type metadata.
     */
    scopes: {
      type: Array,
      default: () => [],
    },
    activeScopeKey: {
      type: String,
      default: '',
    },
    uploadingKey: {
      type: String,
      default: '',
    },
    disabled: {
      type: Boolean,
      default: false,
    },

    // Legacy props retained while the parent form is migrated.
    queuedDocuments: {
      type: Array,
      default: () => [],
    },
    documentIds: {
      type: Array,
      default: () => [],
    },
    uploading: {
      type: Boolean,
      default: false,
    },
  },

  emits: [
    'update:active-scope',
    'stage-file',
    'remove-file',
    'request-file-upload',
    'file-rejected',
    'scope-complete',
    // Legacy events
    'queue-file',
    'request-upload',
  ],

  data() {
    return {
      transientErrors: {},
      advancedScopes: {},
      advanceTimer: null,
    };
  },

  computed: {
    displayScopes() {
      return (this.scopes || []).filter(
        (scope) => this.requirementsOf(scope).length > 0
      );
    },

    hasRequirements() {
      return this.displayScopes.length > 0;
    },

    activeIndex() {
      const requested = this.displayScopes.findIndex(
        (scope, index) => this.scopeKey(scope, index) === this.activeScopeKey
      );
      if (requested >= 0 && this.scopeUnlocked(requested)) return requested;

      const firstIncomplete = this.displayScopes.findIndex(
        (scope, index) => this.scopeUnlocked(index) && !this.scopeComplete(scope)
      );
      return firstIncomplete >= 0 ? firstIncomplete : 0;
    },

    activeScope() {
      return this.displayScopes[this.activeIndex] || null;
    },

    totalRequirements() {
      return this.displayScopes.reduce(
        (total, scope) => total + this.requirementsOf(scope).length,
        0
      );
    },

    uploadedRequirements() {
      return this.displayScopes.reduce(
        (total, scope) =>
          total + this.requirementsOf(scope).filter(this.isUploaded).length,
        0
      );
    },

    progressPercent() {
      if (!this.totalRequirements) return 0;
      return Math.round(
        (this.uploadedRequirements / this.totalRequirements) * 100
      );
    },

    progressLabel() {
      return `${this.uploadedRequirements} of ${this.totalRequirements}`;
    },

    completionSignature() {
      return this.displayScopes
        .map((scope, index) =>
          `${this.scopeKey(scope, index)}:${this.scopeComplete(scope) ? 1 : 0}`
        )
        .join('|');
    },

    nextScopeMessage() {
      const next = this.displayScopes[this.activeIndex + 1];
      return next
        ? `${next.label} is now available.`
        : 'All requested document sets are complete.';
    },
  },

  watch: {
    completionSignature() {
      this.advanceIfReady();
    },
  },

  mounted() {
    this.advanceIfReady();
  },

  beforeUnmount() {
    if (this.advanceTimer) clearTimeout(this.advanceTimer);
  },

  methods: {
    requirementsOf(scope) {
      return Array.isArray(scope && scope.requirements)
        ? scope.requirements
        : [];
    },

    scopeKey(scope, index = 0) {
      return String(
        (scope && (scope.key || scope.id || scope.scopeKey)) || `scope-${index}`
      );
    },

    requirementKey(requirement, index = 0) {
      return String(
        (requirement &&
          (requirement.key ||
            requirement.id ||
            requirement.attachmentTypeId ||
            requirement.attachment_type_id)) ||
          `requirement-${index}`
      );
    },

    slotKey(scope, requirement) {
      return `${this.scopeKey(scope)}::${this.requirementKey(requirement)}`;
    },

    isActive(scope, index) {
      return this.activeIndex === index;
    },

    scopeUnlocked(index) {
      if (index <= 0) return true;
      return this.displayScopes
        .slice(0, index)
        .every((scope) => this.scopeComplete(scope));
    },

    selectScope(scope, index) {
      if (!this.scopeUnlocked(index)) return;
      this.$emit('update:active-scope', this.scopeKey(scope, index));
    },

    isUploaded(requirement) {
      const status = String((requirement && requirement.status) || '').toLowerCase();
      return Boolean(
        status === 'uploaded' ||
          (requirement &&
            (requirement.uploadId ||
              requirement.upload_id ||
              requirement.documentId ||
              requirement.document_id))
      );
    },

    isUploading(scope, requirement) {
      return Boolean(
        String((requirement && requirement.status) || '').toLowerCase() ===
          'uploading' || this.uploadingKey === this.slotKey(scope, requirement)
      );
    },

    scopeComplete(scope) {
      const requirements = this.requirementsOf(scope);
      return requirements.length > 0 && requirements.every(this.isUploaded);
    },

    scopeProgress(scope) {
      const requirements = this.requirementsOf(scope);
      const uploaded = requirements.filter(this.isUploaded).length;
      return `${uploaded}/${requirements.length} uploaded`;
    },

    statusLabel(scope, requirement) {
      if (this.isUploaded(requirement)) return 'Uploaded';
      if (this.isUploading(scope, requirement)) return 'Uploading';
      if (this.hasError(requirement)) return 'Needs attention';
      if (this.hasStagedFile(requirement)) return 'Ready';
      return 'Required';
    },

    statusClass(scope, requirement) {
      if (this.isUploaded(requirement)) return 'success';
      if (this.isUploading(scope, requirement)) return 'working';
      if (this.hasError(requirement)) return 'danger';
      if (this.hasStagedFile(requirement)) return 'ready';
      return 'required';
    },

    fileObject(requirement) {
      if (!requirement) return null;
      return requirement.file || requirement.rawFile || requirement.raw_file || null;
    },

    hasStagedFile(requirement) {
      return Boolean(this.fileObject(requirement) || requirement.fileName);
    },

    fileName(requirement) {
      const file = this.fileObject(requirement);
      return (
        (requirement && (requirement.fileName || requirement.uploadedFileName)) ||
        (file && file.name) ||
        ''
      );
    },

    fileSizeLabel(requirement) {
      const file = this.fileObject(requirement);
      const size = Number((file && file.size) || requirement.fileSize || 0);
      if (!size) return 'Ready to upload';
      if (size < 1024) return `${size} B · Ready to upload`;
      if (size < 1024 * 1024) {
        return `${Math.round(size / 1024)} KB · Ready to upload`;
      }
      return `${(size / (1024 * 1024)).toFixed(1)} MB · Ready to upload`;
    },

    allowedValues(requirement) {
      const raw =
        (requirement &&
          (requirement.allowedFileTypes || requirement.allowed_file_types)) ||
        [];
      const values = Array.isArray(raw) ? raw : String(raw).split(',');
      return values
        .map((value) => String(value || '').trim().toLowerCase())
        .filter(Boolean)
        .map((value) =>
          value.includes('/') || value.startsWith('.') ? value : `.${value}`
        );
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

    setTransientError(scope, requirement, message) {
      this.transientErrors[this.slotKey(scope, requirement)] = message;
    },

    clearTransientError(scope, requirement) {
      delete this.transientErrors[this.slotKey(scope, requirement)];
    },

    errorFor(requirement) {
      if (!requirement) return '';
      const scope = this.activeScope;
      const transient = scope
        ? this.transientErrors[this.slotKey(scope, requirement)]
        : '';
      return transient || requirement.error || requirement.errorMessage || '';
    },

    hasError(requirement) {
      return Boolean(
        this.errorFor(requirement) ||
          String((requirement && requirement.status) || '').toLowerCase() ===
            'error'
      );
    },

    stage(scope, requirement, file) {
      if (!file || this.disabled) return;

      if (!this.acceptsFile(requirement, file)) {
        const message = `Choose one of the allowed formats: ${this.allowedLabel(
          requirement
        ).replace('Accepted: ', '')}.`;
        this.setTransientError(scope, requirement, message);
        this.$emit('file-rejected', {
          scopeKey: this.scopeKey(scope),
          requirementKey: this.requirementKey(requirement),
          file,
          message,
        });
        return;
      }

      this.clearTransientError(scope, requirement);
      this.$emit('stage-file', {
        scopeKey: this.scopeKey(scope),
        scopeType: scope.type || scope.scopeType || '',
        ownerId: scope.ownerId || scope.owner_id || null,
        requirementKey: this.requirementKey(requirement),
        attachmentTypeId:
          requirement.attachmentTypeId ||
          requirement.attachment_type_id ||
          requirement.id ||
          null,
        file,
      });
    },

    onBrowse(scope, requirement, event) {
      const file = event.target.files && event.target.files[0];
      this.stage(scope, requirement, file);
      event.target.value = '';
    },

    onDrop(scope, requirement, event) {
      if (this.disabled || this.isUploading(scope, requirement)) return;
      const file = event.dataTransfer.files && event.dataTransfer.files[0];
      this.stage(scope, requirement, file);
    },

    removeStaged(scope, requirement) {
      this.clearTransientError(scope, requirement);
      this.$emit('remove-file', {
        scopeKey: this.scopeKey(scope),
        requirementKey: this.requirementKey(requirement),
      });
    },

    requestUpload(scope, requirement) {
      if (!this.hasStagedFile(requirement) || this.disabled) return;
      this.clearTransientError(scope, requirement);
      this.$emit('request-file-upload', {
        scopeKey: this.scopeKey(scope),
        scopeType: scope.type || scope.scopeType || '',
        ownerId: scope.ownerId || scope.owner_id || null,
        requirementKey: this.requirementKey(requirement),
        attachmentTypeId:
          requirement.attachmentTypeId ||
          requirement.attachment_type_id ||
          requirement.id ||
          null,
      });
    },

    advanceIfReady() {
      const scope = this.activeScope;
      if (!scope) return;

      const key = this.scopeKey(scope, this.activeIndex);
      if (!this.scopeComplete(scope)) {
        delete this.advancedScopes[key];
        return;
      }
      if (this.advancedScopes[key]) return;

      this.advancedScopes[key] = true;
      this.$emit('scope-complete', {
        scopeKey: key,
        scopeType: scope.type || scope.scopeType || '',
        ownerId: scope.ownerId || scope.owner_id || null,
      });

      const next = this.displayScopes[this.activeIndex + 1];
      if (!next) return;

      if (this.advanceTimer) clearTimeout(this.advanceTimer);
      this.advanceTimer = setTimeout(() => {
        this.$emit(
          'update:active-scope',
          this.scopeKey(next, this.activeIndex + 1)
        );
      }, 650);
    },

    queueLegacy(file) {
      this.$emit('queue-file', file);
    },

    removeLegacy(file) {
      this.$emit('remove-file', file);
    },
  },
};
</script>
