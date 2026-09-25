<template>
  <div class="applicant-editor">
    <!-- Third Party Owner: owns collateral but isn't borrowing, so only a few details -->
    <template v-if="isThirdPartyOwner">
      <p class="helper">
        A Third Party Owner owns all or part of an asset offered as collateral,
        but isn't borrowing. We only need their name, relationship to you, and
        contact details.
      </p>
      <div class="field-grid">
        <el-form-item v-if="showRole" label="Role" required>
          <el-select
            :model-value="draft.role"
            placeholder="Select role"
            @update:model-value="set('role', $event)"
          >
            <el-option
              v-for="option in roleOptions"
              :key="option.value"
              :label="option.label"
              :value="option.value"
            />
          </el-select>
        </el-form-item>

        <el-form-item label="Owner is a" required>
          <el-select
            :model-value="draft.kind || 'PERSON'"
            @update:model-value="set('kind', $event)"
          >
            <el-option label="Person" value="PERSON" />
            <el-option label="Business" value="ORGANIZATION" />
          </el-select>
        </el-form-item>

        <el-form-item v-if="draft.kind === 'ORGANIZATION'" label="Business name" required>
          <el-input :model-value="draft.business_name" @input="set('business_name', $event)" />
        </el-form-item>

        <template v-else>
          <el-form-item label="First name" required>
            <el-input :model-value="draft.first_name" @input="set('first_name', $event)" />
          </el-form-item>

          <el-form-item label="Last name" required>
            <el-input :model-value="draft.last_name" @input="set('last_name', $event)" />
          </el-form-item>
        </template>

        <el-form-item label="Relationship to the primary applicant" required>
          <el-select
            v-if="lookup('relationship_to_applicant').length"
            :model-value="draft.relationship_to_applicant"
            placeholder="Select relationship"
            @update:model-value="set('relationship_to_applicant', $event)"
          >
            <el-option
              v-for="option in lookup('relationship_to_applicant')"
              :key="option.value"
              :label="option.label"
              :value="option.value"
            />
          </el-select>
          <el-input
            v-else
            :model-value="draft.relationship_to_applicant"
            placeholder="e.g. Parent"
            @input="set('relationship_to_applicant', $event)"
          />
        </el-form-item>

        <el-form-item label="Phone" required>
          <el-input :model-value="draft.phone" @input="set('phone', $event)" />
        </el-form-item>

        <el-form-item label="Email">
          <el-input :model-value="draft.email" type="email" @input="set('email', $event)" />
        </el-form-item>
      </div>
    </template>

    <template v-else>
    <!-- Personal details -->
    <div class="field-grid">
      <el-form-item v-if="showRole" label="Role" required>
        <el-select
          :model-value="draft.role"
          placeholder="Select role"
          @update:model-value="set('role', $event)"
        >
          <el-option
            v-for="option in roleOptions"
            :key="option.value"
            :label="option.label"
            :value="option.value"
          />
        </el-select>
      </el-form-item>

      <el-form-item label="First name" required>
        <el-input :model-value="draft.first_name" @input="set('first_name', $event)" />
      </el-form-item>

      <el-form-item label="Last name" required>
        <el-input :model-value="draft.last_name" @input="set('last_name', $event)" />
      </el-form-item>

      <el-form-item label="Email" required>
        <el-input :model-value="draft.email" type="email" @input="set('email', $event)" />
      </el-form-item>

      <el-form-item label="Phone" required>
        <el-input :model-value="draft.phone" @input="set('phone', $event)" />
      </el-form-item>

      <el-form-item label="Date of birth" required>
        <el-date-picker
          :model-value="draft.date_of_birth"
          type="date"
          value-format="YYYY-MM-DD"
          @update:model-value="set('date_of_birth', $event)"
        />
      </el-form-item>

      <el-form-item label="Marital status">
        <el-select
          :model-value="draft.marital_status"
          placeholder="Select marital status"
          @update:model-value="set('marital_status', $event)"
        >
          <el-option
            v-for="option in lookup('marital_status')"
            :key="option.value"
            :label="option.label"
            :value="option.value"
          />
        </el-select>
      </el-form-item>

      <el-form-item label="NIS number" required>
        <el-input :model-value="draft.nis_number" @input="set('nis_number', $event)" />
      </el-form-item>
    </div>

    <!-- Home address, split into street, parish, and country -->
    <section class="context">
      <h3>Home address</h3>
      <el-form-item label="Street address" required>
        <el-input
          :model-value="draft.address"
          type="textarea"
          :rows="2"
          placeholder="House number, street, and village or town"
          @input="set('address', $event)"
        />
      </el-form-item>
      <div class="field-grid">
        <el-form-item label="Country" required>
          <el-select
            v-if="lookup('country').length"
            :model-value="draft.country"
            filterable
            placeholder="Select country"
            @update:model-value="set('country', $event)"
          >
            <el-option
              v-for="option in lookup('country')"
              :key="option.value"
              :label="option.label"
              :value="option.value"
            />
          </el-select>
          <el-input
            v-else
            :model-value="draft.country"
            @input="set('country', $event)"
          />
        </el-form-item>

        <!-- Parish only applies to addresses in Grenada -->
        <el-form-item v-if="inGrenada" label="Parish" required>
          <el-select
            v-if="lookup('parish').length"
            :model-value="draft.parish"
            placeholder="Select parish"
            @update:model-value="set('parish', $event)"
          >
            <el-option
              v-for="option in lookup('parish')"
              :key="option.value"
              :label="option.label"
              :value="option.value"
            />
          </el-select>
          <el-input
            v-else
            :model-value="draft.parish"
            @input="set('parish', $event)"
          />
        </el-form-item>
      </div>
    </section>

    <!-- Identification: one or more PartyIdentification records -->
    <section class="context">
      <div class="collection-header">
        <div>
          <h3>Identification</h3>
          <p>{{ identificationHint }}</p>
        </div>
        <el-button type="primary" plain @click="addIdentification">
          <v-icon start>mdi-plus</v-icon>
          Add identification
        </el-button>
      </div>

      <article
        v-for="(row, index) in draft.identifications"
        :key="row.client_key"
        class="item-card"
      >
        <div class="item-title">
          <strong>{{ identificationTitle(row, index) }}</strong>
          <div>
            <el-tag v-if="row.is_primary" type="success" size="small">Primary</el-tag>
            <el-button v-else text @click="setPrimary(index)">Make primary</el-button>
            <el-button
              v-if="draft.identifications.length > 1"
              text
              type="danger"
              @click="removeIdentification(index)"
            >
              Remove
            </el-button>
          </div>
        </div>

        <div class="field-grid">
          <el-form-item label="Identification type" required>
            <el-select
              :model-value="row.identification_type"
              placeholder="Select type"
              @update:model-value="setIdentification(index, 'identification_type', $event)"
            >
              <el-option
                v-for="option in lookup('identification_type')"
                :key="option.value"
                :label="option.label"
                :value="option.value"
                :disabled="typeTakenElsewhere(option.value, index)"
              />
            </el-select>
          </el-form-item>

          <el-form-item label="Identification number" required>
            <el-input
              :model-value="row.identification_number"
              @input="setIdentification(index, 'identification_number', $event)"
            />
          </el-form-item>

          <el-form-item label="Issuing country">
            <el-select
              v-if="lookup('country').length"
              :model-value="row.issuing_country"
              filterable
              placeholder="Select country"
              @update:model-value="setIdentification(index, 'issuing_country', $event)"
            >
              <el-option
                v-for="option in lookup('country')"
                :key="option.value"
                :label="option.label"
                :value="option.value"
              />
            </el-select>
            <el-input
              v-else
              :model-value="row.issuing_country"
              @input="setIdentification(index, 'issuing_country', $event)"
            />
          </el-form-item>

          <el-form-item label="Issue date">
            <el-date-picker
              :model-value="row.issue_date"
              type="date"
              value-format="YYYY-MM-DD"
              @update:model-value="setIdentification(index, 'issue_date', $event)"
            />
          </el-form-item>

          <el-form-item label="Expiry date">
            <el-date-picker
              :model-value="row.expiry_date"
              type="date"
              value-format="YYYY-MM-DD"
              @update:model-value="setIdentification(index, 'expiry_date', $event)"
            />
            <small v-if="isExpired(row)" class="helper invalid">
              This identification has expired. Replace it with a current one.
            </small>
          </el-form-item>
        </div>

        <!-- Scan of this identification, if the credit union requires one -->
        <AdaptiveLoanDocumentRequirements
          title="Identification scan"
          :scope="documentScopes[`identification:${row.client_key}`]"
          :uploading-key="uploadingKey"
          :disabled="documentsDisabled"
          @stage-file="$emit('stage-file', $event)"
          @remove-file="$emit('remove-file', $event)"
          @request-file-upload="$emit('request-file-upload', $event)"
          @file-rejected="$emit('file-rejected', $event)"
        />
      </article>
    </section>

    <!-- Employment and income -->
    <section class="context">
      <h3>Employment and income</h3>
      <div class="field-grid">
        <el-form-item label="Employment status">
          <el-select
            :model-value="draft.employment_status"
            placeholder="Select employment status"
            @update:model-value="set('employment_status', $event)"
          >
            <el-option
              v-for="option in lookup('employment_status')"
              :key="option.value"
              :label="option.label"
              :value="option.value"
            />
          </el-select>
        </el-form-item>

        <el-form-item label="Gross monthly income (EC$)" required>
          <el-input-number
            :model-value="draft.gross_monthly_income"
            :min="0"
            controls-position="right"
            @update:model-value="set('gross_monthly_income', $event)"
          />
        </el-form-item>

        <template v-if="showEmploymentDetails">
          <el-form-item label="Employer / business">
            <el-input :model-value="draft.employer_name" @input="set('employer_name', $event)" />
          </el-form-item>

          <el-form-item label="Job title">
            <el-input :model-value="draft.job_title" @input="set('job_title', $event)" />
          </el-form-item>

          <el-form-item label="Years employed">
            <el-input-number
              :model-value="draft.years_employed"
              :min="0"
              controls-position="right"
              @update:model-value="set('years_employed', $event)"
            />
          </el-form-item>
        </template>

      </div>

      <!-- NIS and income tax are calculated by the parent form, not entered -->
      <div v-if="showDeductions" class="deduction-summary">
        <div class="deduction-row">
          <span>
            Estimated NIS
            <small>{{ deductions.nisBasis }}</small>
          </span>
          <strong>{{ money(deductions.nis) }}</strong>
        </div>
        <div class="deduction-row">
          <span>Estimated income tax (PAYE)</span>
          <strong>{{ money(deductions.incomeTax) }}</strong>
        </div>
        <div class="deduction-row net">
          <span>Estimated net monthly income</span>
          <strong>{{ money(deductions.net) }}</strong>
        </div>
        <small>
          Calculated from gross monthly income using current NIS and income tax
          rates. Your payslip may differ.
        </small>
      </div>
    </section>

    <div class="consents">
      <strong>Declarations and consent</strong>
      <el-checkbox
        :model-value="draft.consent_accuracy_confirmation"
        @update:model-value="set('consent_accuracy_confirmation', $event)"
      >
        I confirm the information is true and complete.
      </el-checkbox>
      <el-checkbox
        :model-value="draft.consent_credit_check"
        @update:model-value="set('consent_credit_check', $event)"
      >
        I authorize a credit check.
      </el-checkbox>
      <el-checkbox
        :model-value="draft.consent_data_processing"
        @update:model-value="set('consent_data_processing', $event)"
      >
        I agree to privacy and data-processing terms.
      </el-checkbox>
    </div>
    </template>
  </div>
</template>

<script>
/**
 * Editor for one applicant: personal details, split home address, NIS
 * number, identifications, employment and income (with estimated
 * statutory deductions calculated by the parent), and consent. Edits a deep-cloned draft and emits a fresh
 * copy on every change.
 *
 * A Third Party Owner (owns collateral but isn't borrowing) gets a short
 * form instead: person or business, name, relationship, phone, and email.
 */
export default {
  props: {
    modelValue: {
      type: Object,
      default: () => ({}),
    },
    lookups: {
      type: Object,
      default: () => ({}),
    },
    roleOptions: {
      type: Array,
      default: () => [],
    },
    showRole: {
      type: Boolean,
      default: false,
    },
    /** Estimated { nis, nisBasis, incomeTax, net } from the parent form. */
    deductions: {
      type: Object,
      default: null,
    },
    /** Institution-wide minimum number of identifications. */
    minimumIdentifications: {
      type: Number,
      default: 1,
    },
    /** Document scopes from the parent (for identification scans). */
    documentScopes: {
      type: Object,
      default: () => ({}),
    },
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
      draft: this.copy(this.modelValue),
    };
  },

  computed: {
    /** Matches THIRD_PARTY_OWNER_ROLE in the main form. */
    isThirdPartyOwner() {
      return String(this.draft.role || '').trim().toLowerCase() === 'third party owner';
    },

    showEmploymentDetails() {
      const value = String(this.draft.employment_status || '').trim().toLowerCase();
      return value !== 'unemployed' && value !== 'retired';
    },

    inGrenada() {
      const country = String(this.draft.country || '').trim().toLowerCase();
      return !country || country === 'grenada';
    },

    identificationHint() {
      const minimum = this.minimumIdentifications;
      const base =
        minimum > 1
          ? `At least ${minimum} forms of identification are required.`
          : 'At least one form of identification is required.';
      return `${base} Each must be a different type and not expired.`;
    },

    /** Show the estimate once there's a gross income to base it on. */
    showDeductions() {
      const gross = this.draft.gross_monthly_income;
      return Boolean(this.deductions) && gross !== null && gross !== undefined;
    },
  },

  watch: {
    modelValue: {
      deep: true,
      handler(value) {
        this.draft = this.copy(value);
      },
    },
  },

  methods: {
    copy(value) {
      const copy = JSON.parse(JSON.stringify(value || {}));
      if (!Array.isArray(copy.identifications)) copy.identifications = [];
      return copy;
    },

    key(prefix) {
      return `${prefix}_${Date.now()}_${Math.random().toString(36).slice(2, 8)}`;
    },

    lookup(key) {
      return this.lookups[key] || [];
    },

    lookupLabel(key, value) {
      const option = this.lookup(key).find((entry) => entry.value === value);
      return option ? option.label : value || '';
    },

    money(value) {
      return `EC$ ${Number(value).toLocaleString(undefined, {
        minimumFractionDigits: 2,
        maximumFractionDigits: 2,
      })}`;
    },

    notify() {
      this.$emit('update:modelValue', this.copy(this.draft));
    },

    set(key, value) {
      this.draft[key] = value;
      // Parish only applies in Grenada; clear it when the country changes.
      if (key === 'country' && !this.inGrenada) this.draft.parish = '';
      this.notify();
    },

    // ---- Identifications ----

    identificationTitle(row, index) {
      return (
        this.lookupLabel('identification_type', row.identification_type) ||
        `Identification ${index + 1}`
      );
    },

    isExpired(row) {
      return Boolean(row.expiry_date && row.expiry_date < new Date().toISOString().slice(0, 10));
    },

    /** Each type can only be used once per applicant. */
    typeTakenElsewhere(value, index) {
      return this.draft.identifications.some(
        (row, rowIndex) => rowIndex !== index && row.identification_type === value
      );
    },

    setIdentification(index, key, value) {
      this.draft.identifications[index][key] = value;
      this.notify();
    },

    setPrimary(index) {
      this.draft.identifications.forEach((row, rowIndex) => {
        row.is_primary = rowIndex === index;
      });
      this.notify();
    },

    addIdentification() {
      this.draft.identifications.push({
        client_key: this.key('ident'),
        id: null,
        identification_type: '',
        identification_number: '',
        issuing_country: 'Grenada',
        issue_date: '',
        expiry_date: '',
        is_primary: this.draft.identifications.length === 0,
        document_ids: [],
      });
      this.notify();
    },

    /** Removing the primary ID makes the first remaining one primary. */
    removeIdentification(index) {
      const removed = this.draft.identifications.splice(index, 1)[0];
      if (removed?.is_primary && this.draft.identifications.length) {
        this.draft.identifications[0].is_primary = true;
      }
      this.notify();
    },
  },
};
</script>