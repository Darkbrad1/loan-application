<template>
  <section>
    <!-- ===== Who else is on the loan ===== -->
    <template v-if="screen === 'people'">
      <article
        v-for="(person, index) in parties"
        :key="person.client_key"
        class="item-card open mb-4 p-5 bg-white rounded-lg" style="border:2px solid var(--brand);box-shadow:0 0 0 3px var(--brand-tint)"
      >
        <div class="item-head head-open flex flex-wrap items-center gap-3" style="margin-bottom:20px;padding-bottom:16px;border-bottom:1px solid #e5e7eb">
          <span class="item-icon flex items-center justify-center rounded-lg" style="width:44px;height:44px;flex:0 0 auto;background:var(--brand-tint);color:var(--brand-text)"><v-icon>mdi-account-outline</v-icon></span>
          <div class="item-text" style="flex:1 1 160px;min-width:0">
            <strong class="item-name block text-lg font-bold" style="line-height:1.3;overflow-wrap:anywhere">{{ personName(person) || `Person ${index + 1}` }}</strong>
            <span class="item-sub block text-sm text-gray-600" v-if="roleLabel(person.role)">{{ roleLabel(person.role) }}</span>
          </div>
          <div class="item-actions flex gap-1" style="margin-left:auto">
            <button type="button" class="link-btn danger px-3 py-2 rounded text-base font-semibold text-red-700" style="background:transparent;border:0;cursor:pointer" @click="$emit('request-remove', index)">Remove</button>
          </div>
        </div>

        <el-form-item label="How are they involved?" required :error="need(person.role)">
          <div class="choice-list flex flex-col gap-3 w-full" role="radiogroup">
            <button
              v-for="option in roleOptions"
              :key="option.value"
              type="button"
              role="radio"
              class="choice flex items-center gap-3 w-full px-4 py-3 rounded-lg text-base text-left" style="min-height:56px;border-width:2px;border-style:solid;cursor:pointer;color:#111827;line-height:1.35"
              :style="(person.role === option.value) ? 'border-color:var(--brand);background:var(--brand-tint);box-shadow:inset 0 0 0 1px var(--brand)' : 'border-color:#d1d5db;background:#ffffff'"
              :aria-checked="person.role === option.value"
              @click="setPerson(index, 'role', option.value)"
            >
              <span class="choice-mark flex items-center justify-center rounded-full" style="width:26px;height:26px;flex:0 0 auto;border-width:2px;border-style:solid" :style="(person.role === option.value) ? 'background:var(--brand);border-color:var(--brand);color:var(--brand-ink)' : 'background:#ffffff;border-color:#d1d5db;color:transparent'"><v-icon size="16">mdi-check</v-icon></span>
              <span>
                {{ option.label }}
                <small v-if="roleHelp(option.value)" class="choice-note block text-sm text-gray-600 font-normal">{{ roleHelp(option.value) }}</small>
              </span>
            </button>
          </div>
        </el-form-item>

        <div class="field-grid flex flex-wrap" style="column-gap:16px">
          <el-form-item class="grid-cell" style="flex:1 1 240px;min-width:0" label="First name">
            <FormField
              :model-value="person.first_name"
              :property="field('Party', 'first_name', 'First name', 'input')"
              :form="person"
              @update:model-value="setPerson(index, 'first_name', $event)"
            />
          </el-form-item>
          <el-form-item class="grid-cell" style="flex:1 1 240px;min-width:0" label="Last name">
            <FormField
              :model-value="person.last_name"
              :property="field('Party', 'last_name', 'Last name', 'input')"
              :form="person"
              @update:model-value="setPerson(index, 'last_name', $event)"
            />
          </el-form-item>
        </div>
      </article>

      <button type="button" class="add-button flex items-center justify-center gap-2 w-full bg-white rounded-lg text-lg font-semibold" style="min-height:56px;border:2px dashed #d1d5db;color:var(--brand-text);cursor:pointer" @click="$emit('request-add')">
        <v-icon>mdi-account-plus-outline</v-icon>
        Add {{ parties.length ? 'another person' : 'a person' }}
      </button>
      <p class="hint mt-2 text-base text-gray-600">
        If someone else co-owns something that secures the loan, add them here too.
      </p>
    </template>

    <!-- ===== References: a personal reference and next of kin ===== -->
    <template v-if="screen === 'references'">
      <article
        v-for="(row, index) in references"
        :key="row.client_key"
        class="item-card open mb-4 p-5 bg-white rounded-lg" style="border:2px solid var(--brand);box-shadow:0 0 0 3px var(--brand-tint)"
      >
        <div class="item-head head-open flex flex-wrap items-center gap-3" style="margin-bottom:20px;padding-bottom:16px;border-bottom:1px solid #e5e7eb">
          <span class="item-icon flex items-center justify-center rounded-lg" style="width:44px;height:44px;flex:0 0 auto;background:var(--brand-tint);color:var(--brand-text)">
            <v-icon>{{ row.reference_type === 'Next of kin' ? 'mdi-home-heart' : 'mdi-account-voice' }}</v-icon>
          </span>
          <div class="item-text" style="flex:1 1 160px;min-width:0">
            <strong class="item-name block text-lg font-bold" style="line-height:1.3;overflow-wrap:anywhere">{{ row.reference_type === 'Next of kin' ? 'Your next of kin' : 'Someone who knows you' }}</strong>
            <span class="item-sub block text-sm text-gray-600">
              {{
                row.reference_type === 'Next of kin'
                  ? 'Your closest family member, like a spouse, parent, or adult child.'
                  : 'A friend, co-worker, or employer who has known you for a while.'
              }}
            </span>
          </div>
        </div>
        <el-form-item label="Full name" required :error="need(row.name)">
          <FormField
            :model-value="row.name"
            :property="field('Reference', 'name', 'Full name', 'input')"
            :form="row"
            @update:model-value="setReference(index, 'name', $event)"
          />
        </el-form-item>
        <el-form-item label="How do they know you?" required :error="need(row.relationship)">
          <FormField
            :model-value="row.relationship"
            :property="field('Reference', 'relationship', 'How do they know you?', 'select')"
            :form="row"
            @update:model-value="setReference(index, 'relationship', $event)"
          />
        </el-form-item>
        <div class="field-grid flex flex-wrap" style="column-gap:16px">
          <el-form-item class="grid-cell" style="flex:1 1 240px;min-width:0" label="Phone number" required :error="need(row.phone)">
            <FormField
              :model-value="row.phone"
              :property="field('Reference', 'phone', 'Phone number', 'input')"
              :form="row"
              @update:model-value="setReference(index, 'phone', $event)"
            />
          </el-form-item>
          <el-form-item class="grid-cell" style="flex:1 1 240px;min-width:0" label="Email (if they have one)">
            <FormField
              :model-value="row.email"
              :property="field('Reference', 'email', 'Email (if they have one)', 'input')"
              :form="row"
              @update:model-value="setReference(index, 'email', $event)"
            />
          </el-form-item>
        </div>
      </article>
    </template>
  </section>
</template>

<script>
/**
 * Two short screens of the "About you" step, picked by `screen`:
 *
 * - "people": the other people on the loan (co-borrowers, guarantors, and
 *   Third Party Owners who co-own collateral). Each gets a role and a
 *   name here; their details are filled in on the screens that follow.
 * - "references": the primary applicant's personal reference and next of
 *   kin, saved as Reference records.
 *
 * Adding and removing people is done by the parent (request-add and
 * request-remove), which also clears anything assigned to someone removed.
 * Inputs are Saturn's FormField, using Saturn's own property definitions;
 * the role is an el-select because the form decides its choices.
 */
export default {
  props: {
    /** Which screen to show: "people" or "references". */
    screen: {
      type: String,
      default: 'people',
    },
    primary: {
      type: Object,
      default: () => ({}),
    },
    parties: {
      type: Array,
      default: () => [],
    },
    /** The personal reference and next of kin (Reference records). */
    references: {
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
    /** After a failed Next, shows "this is needed" under empty required boxes. */
    showErrors: {
      type: Boolean,
      default: false,
    },
  },

  emits: ['update:parties', 'update:references', 'request-add', 'request-remove'],

  created() {
    // FormField configs, reused while unchanged (see field()).
    this.fieldCache = {};
  },

  computed: {
    /** Roles for the other people; "Primary Applicant" is only for the applicant. */
    roleOptions() {
      return (this.lookups.role || []).filter(
        (option) => String(option.value).trim().toLowerCase() !== 'primary applicant'
      );
    },
  },

  methods: {
    personName(person) {
      return person.kind === 'ORGANIZATION'
        ? String(person.business_name || '').trim()
        : `${person.first_name || ''} ${person.last_name || ''}`.trim();
    },

    /** The label of a role value, for the card heading. */
    roleLabel(role) {
      const option = this.roleOptions.find((entry) => entry.value === role);
      return option ? option.label : '';
    },

    /** A plain explanation of a role. */
    roleHelp(role) {
      const value = String(role || '').trim().toLowerCase();
      if (value === 'guarantor') return 'Promises to pay the loan if you can\'t.';
      if (value === 'third party owner') {
        return 'Co-owns something that secures the loan, but isn\'t borrowing. We only need a few details.';
      }
      if (value.includes('co')) return 'Borrows the money with you and pays it back with you.';
      return '';
    },

    /** "This is needed" under an empty required box, after a failed Next. */
    need(value) {
      if (!this.showErrors) return '';
      return value === null || value === undefined || value === '' ? 'This is needed' : '';
    },

    /** Updates one field on one person and sends a fresh copy up. */
    setPerson(index, key, event) {
      const rows = JSON.parse(JSON.stringify(this.parties));
      rows[index][key] = this.valueOf(event);
      this.$emit('update:parties', rows);
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
      const cacheKey = JSON.stringify(config);
      if (!this.fieldCache[cacheKey]) this.fieldCache[cacheKey] = config;
      return this.fieldCache[cacheKey];
    },
  },
};
</script>

<style scoped>
/*
 * No styles here on purpose: Saturn doesn't apply <style> blocks reliably.
 * Everything is styled in the template with Tailwind classes, plus inline
 * styles for the brand colours (CSS variables set on the form) and exact sizes.
 */
</style>
