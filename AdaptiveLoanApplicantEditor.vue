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
            <el-option
              v-for="option in ownerKindOptions"
              :key="option.value"
              :label="option.label"
              :value="option.value"
            />
          </el-select>
        </el-form-item>

        <el-form-item v-if="draft.kind === 'ORGANIZATION'" label="Business name" required>
          <FormField
            :model-value="draft.business_name"
            :property="field('Party', 'business_name', 'Business name', 'input')"
            :form="draft"
            @update:model-value="set('business_name', $event)"
          />
        </el-form-item>

        <template v-else>
          <el-form-item label="First name" required>
            <FormField
              :model-value="draft.first_name"
              :property="field('Party', 'first_name', 'First name', 'input')"
              :form="draft"
              @update:model-value="set('first_name', $event)"
            />
          </el-form-item>

          <el-form-item label="Last name" required>
            <FormField
              :model-value="draft.last_name"
              :property="field('Party', 'last_name', 'Last name', 'input')"
              :form="draft"
              @update:model-value="set('last_name', $event)"
            />
          </el-form-item>
        </template>

        <el-form-item label="Relationship to the primary applicant" required>
          <FormField
            :model-value="draft.relationship_to_applicant"
            :property="field('ApplicationParty', 'relationship_to_applicant', 'Relationship to the primary applicant', 'select')"
            :form="draft"
            @update:model-value="set('relationship_to_applicant', $event)"
          />
        </el-form-item>

        <el-form-item label="Phone" required>
          <FormField
            :model-value="draft.phone"
            :property="field('Party', 'phone', 'Phone', 'input')"
            :form="draft"
            @update:model-value="set('phone', $event)"
          />
        </el-form-item>

        <el-form-item label="Email">
          <FormField
            :model-value="draft.email"
            :property="field('Party', 'email', 'Email', 'input')"
            :form="draft"
            @update:model-value="set('email', $event)"
          />
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
          <FormField
            :model-value="draft.first_name"
            :property="field('Party', 'first_name', 'First name', 'input')"
            :form="draft"
            @update:model-value="set('first_name', $event)"
          />
        </el-form-item>

        <el-form-item label="Last name" required>
          <FormField
            :model-value="draft.last_name"
            :property="field('Party', 'last_name', 'Last name', 'input')"
            :form="draft"
            @update:model-value="set('last_name', $event)"
          />
        </el-form-item>

        <el-form-item label="Email" required>
          <FormField
            :model-value="draft.email"
            :property="field('Party', 'email', 'Email', 'input')"
            :form="draft"
            @update:model-value="set('email', $event)"
          />
        </el-form-item>

        <el-form-item label="Phone" required>
          <FormField
            :model-value="draft.phone"
            :property="field('Party', 'phone', 'Phone', 'input')"
            :form="draft"
            @update:model-value="set('phone', $event)"
          />
        </el-form-item>

        <el-form-item label="Date of birth" required>
          <FormField
            :model-value="draft.date_of_birth"
            :property="field('Party', 'date_of_birth', 'Date of birth', 'date')"
            :form="draft"
            @update:model-value="set('date_of_birth', $event, 'date')"
          />
        </el-form-item>

        <el-form-item label="Marital status">
          <FormField
            :model-value="draft.marital_status"
            :property="field('Party', 'marital_status', 'Marital status', 'select')"
            :form="draft"
            @update:model-value="set('marital_status', $event)"
          />
        </el-form-item>

        <el-form-item label="NIS number" required>
          <FormField
            :model-value="draft.nis_number"
            :property="field('Party', 'nis_number', 'NIS number', 'input')"
            :form="draft"
            @update:model-value="set('nis_number', $event)"
          />
        </el-form-item>
      </div>

      <!-- Home address, split into street, parish, and country -->
      <section class="context">
        <h3>Home address</h3>
        <el-form-item label="Street address" required>
          <FormField
            :model-value="draft.address"
            :property="field('Party', 'address', 'Street address', 'textarea')"
            :form="draft"
            @update:model-value="set('address', $event)"
          />
        </el-form-item>
        <div class="field-grid">
          <el-form-item label="Country" required>
            <FormField
              :model-value="draft.country"
              :property="field('Party', 'country', 'Country', 'select')"
              :form="draft"
              @update:model-value="set('country', $event)"
            />
          </el-form-item>

          <!-- Parish only applies to addresses in Grenada -->
          <el-form-item v-if="inGrenada" label="Parish" required>
            <FormField
              :model-value="draft.parish"
              :property="field('Party', 'parish', 'Parish', 'select')"
              :form="draft"
              @update:model-value="set('parish', $event)"
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
            <!--
              Uses Saturn's own PartyIdentification.identification_type
              setup. Using a type twice is caught when the applicant continues.
            -->
            <el-form-item label="Identification type" required>
              <FormField
                :model-value="row.identification_type"
                :property="field('PartyIdentification', 'identification_type', 'Identification type', 'select')"
                :form="row"
                @update:model-value="setIdentification(index, 'identification_type', $event)"
              />
            </el-form-item>

            <el-form-item label="Identification number" required>
              <FormField
                :model-value="row.identification_number"
                :property="field('PartyIdentification', 'identification_number', 'Identification number', 'input')"
                :form="row"
                @update:model-value="setIdentification(index, 'identification_number', $event)"
              />
            </el-form-item>

            <el-form-item label="Issuing country">
              <FormField
                :model-value="row.issuing_country"
                :property="field('PartyIdentification', 'issuing_country', 'Issuing country', 'select')"
                :form="row"
                @update:model-value="setIdentification(index, 'issuing_country', $event)"
              />
            </el-form-item>

            <el-form-item label="Issue date">
              <FormField
                :model-value="row.issue_date"
                :property="field('PartyIdentification', 'issue_date', 'Issue date', 'date')"
                :form="row"
                @update:model-value="setIdentification(index, 'issue_date', $event, 'date')"
              />
            </el-form-item>

            <el-form-item label="Expiry date">
              <FormField
                :model-value="row.expiry_date"
                :property="field('PartyIdentification', 'expiry_date', 'Expiry date', 'date')"
                :form="row"
                @update:model-value="setIdentification(index, 'expiry_date', $event, 'date')"
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
            <FormField
              :model-value="draft.employment_status"
              :property="field('ApplicationParty', 'employment_status', 'Employment status', 'select')"
              :form="draft"
              @update:model-value="set('employment_status', $event)"
            />
          </el-form-item>

          <el-form-item label="Gross monthly income (EC$)" required>
            <FormField
              :model-value="draft.gross_monthly_income"
              :property="field('ApplicationParty', 'gross_monthly_income', 'Gross monthly income (EC$)', 'number')"
              :form="draft"
              @update:model-value="set('gross_monthly_income', $event)"
            />
          </el-form-item>

          <template v-if="showEmploymentDetails">
            <el-form-item label="Employer / business">
              <FormField
                :model-value="draft.employer_name"
                :property="field('ApplicationParty', 'employer_name', 'Employer / business', 'input')"
                :form="draft"
                @update:model-value="set('employer_name', $event)"
              />
            </el-form-item>

            <el-form-item label="Job title">
              <FormField
                :model-value="draft.job_title"
                :property="field('ApplicationParty', 'job_title', 'Job title', 'input')"
                :form="draft"
                @update:model-value="set('job_title', $event)"
              />
            </el-form-item>

            <el-form-item label="Years employed">
              <FormField
                :model-value="draft.years_employed"
                :property="field('ApplicationParty', 'years_employed', 'Years employed', 'number')"
                :form="draft"
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
        <el-form-item label="I confirm the information is true and complete.">
          <FormField
            :model-value="draft.consent_accuracy_confirmation"
            :property="field('ApplicationParty', 'consent_accuracy_confirmation', 'I confirm the information is true and complete.', 'checkbox')"
            :form="draft"
            @update:model-value="set('consent_accuracy_confirmation', $event)"
          />
        </el-form-item>
        <el-form-item label="I authorize a credit check.">
          <FormField
            :model-value="draft.consent_credit_check"
            :property="field('ApplicationParty', 'consent_credit_check', 'I authorize a credit check.', 'checkbox')"
            :form="draft"
            @update:model-value="set('consent_credit_check', $event)"
          />
        </el-form-item>
        <el-form-item label="I agree to privacy and data-processing terms.">
          <FormField
            :model-value="draft.consent_data_processing"
            :property="field('ApplicationParty', 'consent_data_processing', 'I agree to privacy and data-processing terms.', 'checkbox')"
            :form="draft"
            @update:model-value="set('consent_data_processing', $event)"
          />
        </el-form-item>
      </div>
    </template>
  </div>
</template>

<script>
/**
 * Editor for one applicant: personal details, split home address, NIS
 * number, identifications, employment and income (with estimated
 * statutory deductions calculated by the parent), and consent. Edits a
 * deep-cloned draft and emits a fresh copy on every change.
 *
 * A Third Party Owner (owns collateral but isn't borrowing) gets a short
 * form instead: person or business, name, relationship, phone, and email.
 *
 * Every input is Saturn's built-in FormField. Each field uses Saturn's own
 * definition of the property (Party, ApplicationParty, or
 * PartyIdentification) when there is one. Role and "Owner is a" are
 * el-selects, since the form decides their choices ("Primary Applicant" is
 * left out of Role).
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
    /** Saturn's property definitions, keyed by resource name. */
    resourceProps: {
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
      ownerKindOptions: [
        { label: 'Person', value: 'PERSON' },
        { label: 'Business', value: 'ORGANIZATION' },
      ],
    };
  },

  created() {
    // FormField configs, reused while unchanged (see field()).
    this.fieldCache = {};
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

    notify() {
      this.$emit('update:modelValue', this.copy(this.draft));
    },

    set(key, event, kind) {
      this.draft[key] = this.valueOf(event, kind);
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
      return Boolean(row.expiry_date && row.expiry_date < this.toDateString(new Date()));
    },

    setIdentification(index, key, event, kind) {
      this.draft.identifications[index][key] = this.valueOf(event, kind);
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
      if (removed && removed.is_primary && this.draft.identifications.length) {
        this.draft.identifications[0].is_primary = true;
      }
      this.notify();
    },
  },
};
</script>
